# Retrieval Patterns for Context Graphs

This reference covers search, reranking, and context construction strategies for retrieving relevant information from temporal knowledge graphs to provide to LLM agents.

---

## The Retrieval Pipeline

Retrieval is a three-stage composition: **f(query) = construct(rerank(search(query)))**

Each stage optimizes a different objective:
- **Search**: maximize recall (find all potentially relevant results)
- **Rerank**: maximize precision (prioritize the most relevant results)
- **Construct**: maximize utility (format results for LLM consumption)

---

## Query Understanding

Before executing search, classify and decompose the query to route it effectively.

### Intent Classification

Queries fall into distinct intent categories that affect optimal retrieval strategy:

| Intent | Example | Optimal Strategy |
|--------|---------|-----------------|
| Factual lookup | "Where does Alice work?" | BM25-heavy, single-hop |
| Exploratory | "What do we know about Project Aurora?" | Cosine-heavy, community summaries |
| Temporal | "When did Alice change jobs?" | Temporal filters, change history query |
| Comparative | "How does Alice's role differ from Bob's?" | Multi-entity BFS, MMR for diversity |
| Multi-hop | "Who manages the team Alice works on?" | Query decomposition, chained retrieval |

**Implementation**: Use a lightweight classifier (few-shot prompted LLM call or fine-tuned small model) to tag intent before retrieval. Route intent to search method weights:

```python
INTENT_WEIGHTS = {
    "factual":     {"cosine": 0.3, "bm25": 0.5, "bfs": 0.2},
    "exploratory": {"cosine": 0.5, "bm25": 0.1, "bfs": 0.4},
    "temporal":    {"cosine": 0.2, "bm25": 0.3, "bfs": 0.5},
    "comparative": {"cosine": 0.3, "bm25": 0.2, "bfs": 0.5},
    "multi_hop":   {"cosine": 0.3, "bm25": 0.3, "bfs": 0.4},
}
```

### Query Decomposition

Multi-hop questions should be decomposed into sub-queries before retrieval:

```
Original: "Who manages the team that Alice works on?"
Sub-query 1: "What team does Alice work on?" -> retrieves "Alice is on Team Falcon"
Sub-query 2: "Who manages Team Falcon?" -> retrieves "Bob manages Team Falcon"
```

Decomposition can be done with a single LLM call that returns a list of atomic sub-queries. Each sub-query is retrieved independently, and results are merged before reranking. See the Multi-Hop Retrieval section below for the full pattern.

---

## Stage 1: Search Methods

### 1.1 Cosine Semantic Similarity (phi_cos)

Embed the query into the same vector space as graph elements, then find nearest neighbors.

**What it catches**: Semantically similar content even with different wording.
- "Where does Alice work?" matches edge fact "Alice is employed at Acme Corp"

**Targets**:
- Edge facts (primary — most information lives on edges)
- Entity names
- Community names

**Implementation**:
```cypher
CALL db.index.vector.queryNodes('entity_embedding_idx', $top_k, $query_embedding)
YIELD node, score
WHERE node.group_id = $group_id
RETURN node, score
ORDER BY score DESC
```

**Pre-filtering warning**: The `WHERE node.group_id = $group_id` clause runs AFTER the vector search. Neo4j's vector index returns the global top_k nearest neighbors, then the group_id filter removes non-matching results. This means you may receive significantly fewer than top_k results.

**Over-fetching pattern**: Request 3x the desired top_k from the vector index, then apply the group_id filter:
```cypher
// Fetch 3x to compensate for post-filter reduction
CALL db.index.vector.queryNodes('entity_embedding_idx', $top_k * 3, $query_embedding)
YIELD node, score
WHERE node.group_id = $group_id
RETURN node, score
ORDER BY score DESC
LIMIT $top_k
```

**Embedding model selection**:

| Model | Dimensions | Relative Cost | Latency | Quality |
|-------|-----------|---------------|---------|---------|
| text-embedding-3-small | 1536 | Low | ~80ms | Good for general retrieval |
| text-embedding-3-large | 3072 | Medium | ~120ms | Better for nuanced semantic matching |
| text-embedding-3-large (reduced) | 256-1536 | Medium | ~120ms | Use OpenAI `dimensions` parameter to reduce |

OpenAI's `dimensions` parameter allows truncating text-embedding-3-large to lower dimensions (e.g., 1024 or 512) with Matryoshka representation learning. This trades quality for reduced storage and faster search. For knowledge graph retrieval where precision matters, prefer full 3072 dimensions. For high-volume, latency-sensitive workloads, 1536 dimensions is a reasonable compromise.

**Tuning**:
- Top-k should be generous at search stage (20-50), reranking will filter
- Consider separate indexes for different element types
- When switching embedding models, all stored embeddings must be regenerated (see Embedding Lifecycle section)

### 1.2 BM25 Full-Text Search (phi_bm25)

Traditional keyword search using term frequency and inverse document frequency.

**What it catches**: Exact or near-exact word matches, especially names and specific terms.
- "Acme Corp" directly matches entity name "Acme Corp"
- Better than vector search for proper nouns, acronyms, and technical terms

**Targets**: Same as cosine — edge facts, entity names, community names

**Implementation**:
```cypher
CALL db.index.fulltext.queryNodes('entity_name_idx', $search_query)
YIELD node, score
WHERE node.group_id = $group_id
RETURN node, score
ORDER BY score DESC
LIMIT $top_k
```

**Tuning**:
- Fulltext indexes should cover both name and summary fields for entities
- For edges, index the fact field
- Query preprocessing: consider stemming, removing stop words

### 1.3 Breadth-First Search (phi_bfs)

Graph traversal from seed nodes, finding related entities and edges within n hops.

**What it catches**: Contextual similarity — entities that co-occur in similar conversational contexts are close in the graph.

**Unique capabilities**:
- Can accept **nodes as seeds** (not just text queries)
- Using recent episodes as seeds incorporates recently discussed context
- Reveals relationships not captured by text similarity alone

**Implementation**:
```cypher
// BFS from seed entities
MATCH (seed:Entity {uuid: $seed_uuid})-[*1..2]-(neighbor:Entity)
WHERE neighbor.group_id = $group_id
WITH DISTINCT neighbor
MATCH (neighbor)-[r]->()
WHERE r.invalid_at IS NULL
RETURN neighbor, r
```

**Tuning**:
- Limit hop depth to 2-3 (exponential expansion)
- Seed selection strategy: use entities from the current conversation turn
- Filter to only valid edges (invalid_at IS NULL) for current state
- Include both directions of traversal

### 1.4 HyDE (Hypothetical Document Embeddings)

Generate a hypothetical answer to the query, then embed that answer instead of the raw query.

**Why it helps**: In knowledge graphs, the language of stored facts often differs significantly from the language of user queries. A user asks "Where does Alice work?" but the graph stores "Alice is employed at Acme Corp as Senior Engineer (2024-01-15 - present)." The hypothetical answer bridges this asymmetry by generating text that looks like the stored facts.

**Implementation**:
```python
def hyde_search(query: str, group_id: str, top_k: int = 50) -> list:
    # 1. Generate hypothetical answer using a fast LLM
    hypothetical = llm_generate(
        prompt=f"Write a brief factual statement that would answer this question: {query}",
        max_tokens=100,
        temperature=0.0
    )
    # Example: query="Where does Alice work?"
    # hypothetical="Alice works at a technology company as an engineer."

    # 2. Embed the hypothetical answer (not the original query)
    hyde_embedding = embed(hypothetical)

    # 3. Run vector search with the hypothetical embedding
    return vector_search(hyde_embedding, group_id, top_k=top_k)
```

**When to use**:
- Queries where user language is far from stored fact language
- Abstract or high-level questions ("What is the team's strategy?")
- Can be used alongside standard cosine search — treat as a fourth search method in fusion

**Trade-offs**:
- Adds one LLM call (~200-500ms latency, depending on model)
- Hypothetical may introduce hallucinated details that skew retrieval
- Best with a fast, cheap model (e.g., gpt-4o-mini) for generation

### Why Multiple Search Methods?

Each captures a different type of similarity:

| Method | Similarity Type | Best For |
|--------|----------------|----------|
| Cosine | Semantic | Paraphrased questions, conceptual matches |
| BM25 | Lexical/word | Names, specific terms, exact references |
| BFS | Contextual/structural | Related context, conversation threads, nearby facts |
| HyDE | Semantic (bridged) | Asymmetric query/fact language, abstract questions |

Using all methods maximizes the chance of finding the most relevant context.

---

## Stage 2: Reranking Methods

### 2.1 Reciprocal Rank Fusion (RRF)

Combines rankings from multiple search methods without requiring score normalization.

**Formula**: RRF_score(d) = sum(1 / (k + rank_i(d))) for each search method i

Where k is a constant (typically 60) that dampens the effect of high rankings.

**When to use**: Default choice when combining results from multiple search methods. Low cost, no model calls.

**Implementation**:
```python
def reciprocal_rank_fusion(ranked_lists: list[list], k: int = 60) -> list:
    scores = {}
    for ranked_list in ranked_lists:
        for rank, item in enumerate(ranked_list):
            if item.uuid not in scores:
                scores[item.uuid] = 0
            scores[item.uuid] += 1 / (k + rank + 1)
    return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

**Limitations of RRF**:
- RRF treats all ranked lists equally by default. If one search method returns mostly irrelevant results, those results still receive rank-based scores and can pollute the fused ranking.
- RRF only uses ordinal rank, discarding magnitude information. A result with cosine similarity 0.99 and one with 0.51 receive similar RRF contributions if they share the same rank position.
- For BFS results that have no inherent score ordering, the arbitrary traversal order directly becomes the rank, which may not reflect relevance.

**Alternatives to RRF**:
- **Weighted RRF**: Multiply each list's contribution by a weight: `weight_i / (k + rank_i(d))`. Use query intent classification to set weights dynamically.
- **CombMNZ**: Multiplies the sum of normalized scores by the number of lists containing the result. Rewards results found by multiple methods more aggressively than RRF.
- **CombMNZ formula**: `CombMNZ(d) = count(methods returning d) * sum(normalized_score_i(d))`

```python
def combmnz(search_results: dict[str, list[tuple]], normalize_fn) -> list:
    """
    search_results: {"cosine": [(item, score), ...], "bm25": [...], "bfs": [...]}
    normalize_fn: function to normalize scores to [0, 1] range
    """
    combined = {}
    counts = {}
    for method, results in search_results.items():
        normalized = normalize_fn(results)
        for item, score in normalized:
            if item.uuid not in combined:
                combined[item.uuid] = 0
                counts[item.uuid] = 0
            combined[item.uuid] += score
            counts[item.uuid] += 1

    final = {
        uuid: combined[uuid] * counts[uuid]
        for uuid in combined
    }
    return sorted(final.items(), key=lambda x: x[1], reverse=True)
```

### 2.2 Score Normalization for Weighted Fusion

When combining scores from different search methods (rather than just ranks), scores must be normalized to a common scale. Different methods produce scores on incompatible ranges (cosine: 0-1, BM25: 0-25+, BFS: no score).

**Min-Max Normalization**:
```python
def min_max_normalize(results: list[tuple]) -> list[tuple]:
    """Normalize scores to [0, 1] range."""
    if not results:
        return []
    scores = [score for _, score in results]
    min_s, max_s = min(scores), max(scores)
    if max_s == min_s:
        return [(item, 1.0) for item, _ in results]
    return [(item, (score - min_s) / (max_s - min_s)) for item, score in results]
```

**Z-Score Normalization**:
```python
def z_score_normalize(results: list[tuple]) -> list[tuple]:
    """Normalize scores using z-score (mean=0, std=1), then shift to [0, 1]."""
    if not results:
        return []
    scores = [score for _, score in results]
    mean = sum(scores) / len(scores)
    std = (sum((s - mean) ** 2 for s in scores) / len(scores)) ** 0.5
    if std == 0:
        return [(item, 0.5) for item, _ in results]
    z_scores = [(item, (score - mean) / std) for item, score in results]
    # Shift to positive range using sigmoid
    import math
    return [(item, 1 / (1 + math.exp(-z))) for item, z in z_scores]
```

**Handling BFS results (no scores)**: BFS traversal produces results without relevance scores. Assign synthetic scores based on graph distance from seed:
```python
def assign_bfs_scores(bfs_results: list, max_hops: int = 2) -> list[tuple]:
    """Assign decaying scores based on hop distance from seed."""
    scored = []
    for item in bfs_results:
        # hop_distance is 1, 2, or 3
        score = 1.0 / item.hop_distance  # 1.0 for 1-hop, 0.5 for 2-hop, 0.33 for 3-hop
        scored.append((item, score))
    return scored
```

### 2.3 Maximal Marginal Relevance (MMR)

Balances relevance with diversity to avoid redundant results.

**Formula**: MMR = argmax[lambda * sim(q, d) - (1-lambda) * max(sim(d, d_selected))]

Where lambda controls the relevance-diversity trade-off (0.5-0.7 typical).

**When to use**: When search results contain many near-duplicates or highly similar facts. Ensures diverse context coverage.

**Implementation**:
```python
def maximal_marginal_relevance(
    query_embedding: list[float],
    candidate_embeddings: list[tuple],  # [(item, embedding), ...]
    top_k: int = 10,
    lambda_param: float = 0.6,
) -> list:
    """
    Select top_k results balancing relevance to query and diversity among selected.

    Args:
        query_embedding: The embedded query vector.
        candidate_embeddings: List of (item, embedding) tuples from initial retrieval.
        top_k: Number of results to select.
        lambda_param: Trade-off parameter. Higher = more relevance, lower = more diversity.
    """
    selected = []
    candidates = list(candidate_embeddings)

    while len(selected) < top_k and candidates:
        best_score = float("-inf")
        best_idx = -1

        for i, (item, emb) in enumerate(candidates):
            # Relevance to query
            relevance = cosine_similarity(query_embedding, emb)

            # Max similarity to already-selected items
            if selected:
                max_sim_to_selected = max(
                    cosine_similarity(emb, sel_emb)
                    for _, sel_emb in selected
                )
            else:
                max_sim_to_selected = 0.0

            # MMR score
            mmr_score = lambda_param * relevance - (1 - lambda_param) * max_sim_to_selected

            if mmr_score > best_score:
                best_score = mmr_score
                best_idx = i

        selected.append(candidates.pop(best_idx))

    return [item for item, _ in selected]
```

### 2.4 Episode-Mentions Reranker

Graph-specific reranker that prioritizes results based on how frequently they're mentioned across episodes in a conversation.

**Intuition**: Frequently referenced entities and facts are more important to the conversation context.

**Implementation**: Count episodic edges connecting to each entity/fact, boost scores proportionally.

**When to use**: Conversation memory scenarios where recency and frequency indicate importance.

### 2.5 Node Distance Reranker

Reorders results by graph distance from a designated centroid node.

**Intuition**: Context localized to a specific area of the knowledge graph is more coherent.

**When to use**: When the query relates to a specific entity and you want tightly related context.

### 2.6 Cross-Encoder Reranker

Uses a dedicated cross-encoder model to score each query-document pair with full cross-attention. Unlike bi-encoders (which embed query and document independently), cross-encoders process the concatenated query+document pair, enabling richer interaction between tokens.

**Model recommendations**:

| Model | Latency (per pair) | Quality | Notes |
|-------|-------------------|---------|-------|
| cross-encoder/ms-marco-MiniLM-L-6-v2 | ~5ms | Good | Fastest, good for high-throughput |
| BGE-reranker-v2-m3 | ~15ms | Very good | Multilingual, strong accuracy |
| Cohere Rerank (API) | ~50ms (network) | Excellent | Managed API, no infra needed |

Note: BGE-M3 is a bi-encoder embedding model, not a reranker. The reranker variant is BGE-reranker-v2-m3.

**Implementation**:
```python
from sentence_transformers import CrossEncoder

# Load once at startup
reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2", max_length=512)

def cross_encoder_rerank(
    query: str,
    candidates: list,
    top_k: int = 10,
    batch_size: int = 32,
) -> list:
    """
    Rerank candidates using cross-encoder scoring.

    Args:
        query: The original query string.
        candidates: Pre-filtered candidate results (keep to 20-50 max).
        top_k: Number of results to return after reranking.
        batch_size: Batch size for inference. Tune based on GPU memory.
                    CPU: 16-32, GPU: 64-128.
    """
    # Prepare query-document pairs
    pairs = [(query, candidate.text) for candidate in candidates]

    # Score in batches (cross-encoders are expensive)
    scores = reranker.predict(pairs, batch_size=batch_size, show_progress_bar=False)

    # Pair scores with candidates and sort
    scored = list(zip(candidates, scores))
    scored.sort(key=lambda x: x[1], reverse=True)

    return [candidate for candidate, score in scored[:top_k]]
```

**When to use**: When precision matters more than latency. Final-stage reranking on top-k candidates.

**Production optimization**:
- **Batch size**: CPU inference benefits from batch_size 16-32. GPU inference can use 64-128. Profile on your hardware.
- **Candidate limit**: Only rerank the top 20-50 results from initial fusion. Cross-encoders are O(n) per query.
- **Distillation**: For production, distill a large cross-encoder into a smaller model. Train the small model to match the large model's scores on your domain data.
- **Quantization**: ONNX Runtime with INT8 quantization can reduce MiniLM-L-6-v2 latency by ~40% with minimal quality loss.
- **Caching**: Cross-encoder scores are query-specific and should NOT be cached (unlike embeddings).

**Latency budget for cross-encoder stage**:
- 20 candidates with MiniLM-L-6-v2 on CPU: ~100ms
- 20 candidates with BGE-reranker-v2-m3 on CPU: ~300ms
- 20 candidates with Cohere Rerank API: ~200ms (network dependent)

### Reranking Strategy Recommendations

| Scenario | Strategy |
|----------|----------|
| General retrieval | RRF (combine search methods) -> take top 20 |
| Conversation memory | RRF -> Episode-mentions -> take top 10 |
| Focused entity query | RRF -> Node distance -> take top 10 |
| High-precision (cost OK) | RRF -> MMR (dedupe) -> Cross-encoder -> take top 5 |

---

## Stage 3: Context Construction

### Template Structure

The constructor transforms graph elements into a text string injected into the LLM prompt.

```
FACTS and ENTITIES represent relevant context to the current conversation.

These are the most relevant facts and their valid date ranges.
If the fact is about an event, the event takes place during this time.
format: FACT (Date range: from - to)
<FACTS>
Alice works for Acme Corp as Senior Engineer (Date range: 2024-01-15 - present)
Acme Corp is developing Project Aurora (Date range: 2024-06-01 - present)
Alice previously worked at StartupXYZ (Date range: 2021-03-01 - 2023-12-31)
</FACTS>

These are the most relevant entities
ENTITY_NAME: entity summary
<ENTITIES>
Alice: Senior software engineer, currently at Acme Corp, previously at StartupXYZ
Acme Corp: Technology company developing Project Aurora
Project Aurora: Internal AI initiative at Acme Corp
</ENTITIES>
```

### What Each Element Contributes

| Element | Fields Used | Purpose |
|---------|------------|---------|
| Edge (fact) | fact, valid_at, invalid_at | Specific relationships with temporal context |
| Entity node | name, summary | Entity identity and background |
| Community node | summary | High-level domain overview |

### Construction Guidelines

1. **Order by relevance** — most relevant facts first (reranker determines order)
2. **Include temporal range** — always show valid_at/invalid_at for facts
3. **Deduplicate content** — entity summaries may repeat fact content; prefer facts for specifics
4. **Token budget** — set a max token count (e.g., 1,600-4,000 tokens) and truncate least relevant items
5. **Separate sections** — facts, entities, and communities in distinct blocks help LLM parsing

### Handling Conflicting Facts

Temporal knowledge graphs may contain contradictory information across different time ranges, or even within the same time range due to ingestion from conflicting sources.

**Contradictory temporal ranges**: When two facts about the same relationship have overlapping valid_at ranges with different values, include both and signal the conflict explicitly:

```
<FACTS>
Alice works for Acme Corp as Senior Engineer (Date range: 2024-01-15 - present)
Alice works for Acme Corp as Staff Engineer (Date range: 2024-06-01 - present)
[NOTE: Conflicting role information for Alice at Acme Corp. The more recent fact may reflect a promotion, or one source may be outdated.]
</FACTS>
```

**Signaling uncertainty to the LLM**: When conflicts exist, add a preamble to the context block:

```
NOTE: Some facts below may conflict with each other due to information from different
time periods or sources. When facts conflict, prefer the most recently valid fact.
If the conflict cannot be resolved, acknowledge the uncertainty in your response.
```

**Strategies**:
- Prefer facts with more recent `valid_at` timestamps when conflicts exist
- If two facts have identical temporal ranges but different values, include both and flag the conflict
- Never silently drop a conflicting fact — the LLM should see the ambiguity
- For questions where the conflict is directly relevant ("When did Alice get promoted?"), include the full history

---

## Multi-Hop Retrieval

For questions that require chaining multiple facts together ("Who manages the team Alice works on?"), single-pass retrieval often fails because no single stored fact answers the full question.

### Pattern

```python
def multi_hop_retrieve(
    query: str,
    group_id: str,
    max_hops: int = 3,
    top_k_per_hop: int = 10,
) -> list:
    """
    Decompose a multi-hop query and retrieve iteratively.

    Each hop uses the previous hop's results as context for the next sub-query.
    """
    # Step 1: Decompose query into sub-queries
    sub_queries = decompose_query(query)
    # e.g., ["What team does Alice work on?", "Who manages that team?"]

    all_results = []
    resolved_entities = {}  # Track entities resolved in previous hops

    for sub_query in sub_queries:
        # Step 2: Substitute resolved entities into the sub-query
        resolved_query = substitute_entities(sub_query, resolved_entities)
        # e.g., "Who manages Team Falcon?" (after resolving "that team" -> "Team Falcon")

        # Step 3: Retrieve for this sub-query
        hop_results = hybrid_search(resolved_query, group_id, top_k=top_k_per_hop)
        all_results.extend(hop_results)

        # Step 4: Extract entities from results for next hop
        for result in hop_results:
            if result.entity_name:
                resolved_entities[result.entity_name] = result

    # Step 5: Deduplicate and rerank all collected results
    unique_results = deduplicate(all_results)
    return rerank(query, unique_results)


def decompose_query(query: str) -> list[str]:
    """Use LLM to decompose a multi-hop question into atomic sub-queries."""
    response = llm_generate(
        prompt=f"""Break this question into sequential sub-questions where each can be
answered with a single fact lookup. Return as a JSON list of strings.
If the question is already atomic, return it as a single-element list.

Question: {query}""",
        temperature=0.0,
    )
    return parse_json_list(response)
```

**When to use**: Questions containing relational chains ("Who does X's manager report to?"), comparative questions spanning multiple entities, or any query that requires connecting facts that are not directly linked in the graph.

**Limitations**: Each hop adds latency (one search round + possible LLM call for decomposition). Limit to 2-3 hops in practice. Errors compound — if the first hop retrieves the wrong entity, subsequent hops will diverge.

---

## Hybrid Search Fusion Patterns

### Reciprocal Rank Fusion (RRF) — Standard Pattern

```python
def hybrid_search(query: str, group_id: str, top_k: int = 20) -> list:
    # 1. Generate query embedding
    query_embedding = embed(query)

    # 2. Run all three searches in parallel
    cosine_results = vector_search(query_embedding, group_id, top_k=50)
    bm25_results = fulltext_search(query, group_id, top_k=50)
    bfs_results = graph_traversal(seed_entities, group_id, max_hops=2)

    # 3. Fuse with RRF
    fused = reciprocal_rank_fusion([cosine_results, bm25_results, bfs_results])

    # 4. Return top-k
    return fused[:top_k]
```

### Weighted Fusion

Different weights for different search methods based on query type:

| Query Type | Cosine Weight | BM25 Weight | BFS Weight |
|-----------|---------------|-------------|------------|
| Conceptual ("What does X do?") | 0.5 | 0.2 | 0.3 |
| Name lookup ("Tell me about Alice") | 0.2 | 0.6 | 0.2 |
| Contextual ("What was discussed?") | 0.2 | 0.2 | 0.6 |

**Weighted fusion implementation** (using normalized scores, not RRF):
```python
def weighted_fusion(
    search_results: dict[str, list[tuple]],
    weights: dict[str, float],
    top_k: int = 20,
) -> list:
    """
    Fuse search results using weighted normalized scores.

    Args:
        search_results: {"cosine": [(item, score), ...], "bm25": [...], "bfs": [...]}
        weights: {"cosine": 0.5, "bm25": 0.2, "bfs": 0.3}
    """
    combined = {}
    for method, results in search_results.items():
        normalized = min_max_normalize(results)
        weight = weights.get(method, 1.0)
        for item, score in normalized:
            if item.uuid not in combined:
                combined[item.uuid] = {"item": item, "score": 0.0}
            combined[item.uuid]["score"] += weight * score

    ranked = sorted(combined.values(), key=lambda x: x["score"], reverse=True)
    return [entry["item"] for entry in ranked[:top_k]]
```

---

## Temporal Retrieval Patterns

### Current State Query
Retrieve only currently valid facts:
```cypher
WHERE r.invalid_at IS NULL
```

### Point-in-Time Query
Retrieve facts valid at a specific moment:
```cypher
WHERE r.valid_at <= datetime($timestamp)
  AND (r.invalid_at IS NULL OR r.invalid_at > datetime($timestamp))
```

### Change History Query
Retrieve all versions of a relationship:
```cypher
MATCH (a:Entity {name: $name})-[r]->(b)
RETURN r.fact, r.valid_at, r.invalid_at
ORDER BY r.valid_at
```

### Knowledge Update Detection
Find what changed recently:
```cypher
MATCH ()-[r]->()
WHERE r.invalid_at > datetime($since)
   OR r.created_at > datetime($since)
RETURN r
ORDER BY coalesce(r.invalid_at, r.created_at) DESC
```

---

## Retrieval Evaluation

### Metrics

Measuring retrieval quality requires standard information retrieval metrics computed against golden datasets.

| Metric | Formula | What It Measures |
|--------|---------|-----------------|
| Precision@k | (relevant in top-k) / k | How many retrieved results are relevant |
| Recall@k | (relevant in top-k) / (total relevant) | How many relevant results were found |
| MRR (Mean Reciprocal Rank) | mean(1 / rank of first relevant result) | How quickly the first relevant result appears |
| NDCG (Normalized Discounted Cumulative Gain) | DCG@k / idealDCG@k | Quality of ranking considering graded relevance |
| MAP (Mean Average Precision) | mean of AP across queries | Overall ranking quality across the dataset |

**Implementation**:
```python
def precision_at_k(retrieved: list[str], relevant: set[str], k: int) -> float:
    """Precision of top-k retrieved results."""
    top_k = retrieved[:k]
    return len(set(top_k) & relevant) / k

def recall_at_k(retrieved: list[str], relevant: set[str], k: int) -> float:
    """Recall of top-k retrieved results."""
    top_k = retrieved[:k]
    return len(set(top_k) & relevant) / len(relevant) if relevant else 0.0

def mrr(ranked_results: list[list[str]], relevant_sets: list[set[str]]) -> float:
    """Mean Reciprocal Rank across multiple queries."""
    reciprocal_ranks = []
    for retrieved, relevant in zip(ranked_results, relevant_sets):
        for rank, item in enumerate(retrieved, 1):
            if item in relevant:
                reciprocal_ranks.append(1.0 / rank)
                break
        else:
            reciprocal_ranks.append(0.0)
    return sum(reciprocal_ranks) / len(reciprocal_ranks)

def ndcg_at_k(retrieved: list[str], relevance_scores: dict[str, int], k: int) -> float:
    """NDCG@k with graded relevance (0=irrelevant, 1=partial, 2=highly relevant)."""
    import math
    dcg = sum(
        relevance_scores.get(item, 0) / math.log2(rank + 2)
        for rank, item in enumerate(retrieved[:k])
    )
    ideal = sorted(relevance_scores.values(), reverse=True)[:k]
    idcg = sum(score / math.log2(rank + 2) for rank, score in enumerate(ideal))
    return dcg / idcg if idcg > 0 else 0.0
```

### Building Golden Evaluation Datasets

For temporal knowledge graphs, evaluation datasets must account for time-sensitivity.

**Dataset construction process**:
1. Sample diverse queries across intent types (factual, exploratory, temporal, multi-hop)
2. For each query, manually annotate the set of relevant graph elements (entities, edges, communities)
3. Use graded relevance: 2 = directly answers the query, 1 = provides useful context, 0 = irrelevant
4. Include temporal variants: "Where does Alice work?" should have different golden results for 2023 vs 2024
5. Include negative examples: queries where the graph has no relevant information (tests fallback behavior)

**Recommended dataset size**: Start with 50-100 annotated query-result pairs. Expand to 300+ for regression testing.

**Evaluation cadence**: Run the evaluation suite after any change to:
- Search method weights or parameters
- Reranking pipeline configuration
- Embedding model or index configuration
- Graph schema or ingestion pipeline

---

## Failure Modes and Fallbacks

### When Retrieval Returns Nothing Relevant

Not every query has relevant information in the graph. The retrieval pipeline must detect and handle this gracefully.

**Confidence thresholds**: Set minimum score thresholds below which results are considered irrelevant:

```python
CONFIDENCE_THRESHOLDS = {
    "cosine": 0.35,        # Cosine similarity below this is noise
    "bm25": 2.0,           # BM25 score below this is weak match
    "cross_encoder": 0.1,  # Cross-encoder logit below this is irrelevant
    "rrf": 0.01,           # RRF score below this means poor agreement
}

def filter_low_confidence(results: list[tuple], method: str) -> list[tuple]:
    """Remove results below the confidence threshold for the given method."""
    threshold = CONFIDENCE_THRESHOLDS.get(method, 0.0)
    return [(item, score) for item, score in results if score >= threshold]
```

**Fallback chain**:

1. **Primary retrieval** returns results above confidence threshold -> use them
2. **All results below threshold** -> fall back to community summaries for the group_id (high-level context)
3. **No community summaries available** -> return an explicit "insufficient context" signal

```python
def retrieve_with_fallback(query: str, group_id: str, top_k: int = 10) -> RetrievalResult:
    # Primary retrieval
    results = hybrid_search(query, group_id, top_k=top_k)
    confident_results = filter_low_confidence(results, "rrf")

    if len(confident_results) >= 3:
        return RetrievalResult(results=confident_results, confidence="high")

    # Fallback: community summaries
    communities = get_community_summaries(group_id)
    if communities:
        return RetrievalResult(
            results=confident_results + communities,
            confidence="low",
            fallback_used="community_summaries",
        )

    # Final fallback: signal insufficient context
    return RetrievalResult(
        results=[],
        confidence="none",
        fallback_used="insufficient_context",
    )
```

**Signaling "I don't know" to the LLM**: When confidence is "low" or "none," add an explicit instruction to the context block:

```
[RETRIEVAL CONFIDENCE: LOW]
The following context may not be sufficient to answer the query accurately.
If the provided facts do not contain the answer, respond that you don't have
enough information rather than speculating.
```

### Common Failure Modes

| Failure | Cause | Mitigation |
|---------|-------|------------|
| No results returned | Query topic not in graph | Community summary fallback, "I don't know" signal |
| All results low relevance | Embedding space mismatch | HyDE, query expansion, check embedding model fit |
| Results from wrong group_id | Over-fetching insufficient | Increase over-fetch multiplier to 5x |
| Stale results | Entity summaries outdated | Check embedding freshness, trigger re-embedding |
| Redundant results | Near-duplicate facts | Apply MMR before context construction |

---

## End-to-End Worked Example

Walking through a single query from raw text to final constructed context.

**Query**: "What projects is Alice currently involved in?"

### Step 1: Query Understanding
- Intent classification: **exploratory** (broad question about an entity)
- No decomposition needed (single-hop question)
- Weights selected: cosine=0.5, bm25=0.2, bfs=0.3

### Step 2: Embedding
```
Input: "What projects is Alice currently involved in?"
Model: text-embedding-3-small (1536d)
Output: [0.023, -0.041, 0.088, ...] (1536 floats)
Latency: 85ms
```

### Step 3: Parallel Search

**Cosine search** (top_k=50, over-fetched to 150 for group_id filter):
```
1. "Alice is lead developer on Project Aurora" (score: 0.89)
2. "Project Aurora is an internal AI initiative" (score: 0.84)
3. "Alice contributes to the Design System v2 effort" (score: 0.78)
4. "Bob is assigned to Project Aurora" (score: 0.72)
5. "Alice attended the Q3 planning offsite" (score: 0.68)
... (45 more results after group_id filter)
```

**BM25 search** (top_k=50):
```
1. Entity "Alice" (score: 12.4)
2. Edge "Alice is lead developer on Project Aurora" (score: 8.7)
3. Entity "Project Aurora" (score: 6.1)
4. Edge "Alice contributes to the Design System v2 effort" (score: 5.3)
... (12 more results)
```

**BFS search** (seeds: Alice entity, 2 hops):
```
1. Alice -> (works_at) -> Acme Corp
2. Alice -> (lead_developer) -> Project Aurora
3. Alice -> (contributes_to) -> Design System v2
4. Alice -> (reports_to) -> Carol
5. Project Aurora -> (owned_by) -> Acme Corp
6. Carol -> (manages) -> Team Falcon
... (18 more results, no scores — assigned by hop distance)
```

### Step 4: RRF Fusion
```
1. "Alice is lead developer on Project Aurora"   RRF: 0.048 (rank 1+2+2)
2. Entity "Alice"                                 RRF: 0.041 (rank 6+1+1)
3. "Alice contributes to Design System v2"        RRF: 0.039 (rank 3+4+3)
4. Entity "Project Aurora"                        RRF: 0.033 (rank 2+3+5)
5. "Project Aurora is an internal AI initiative"  RRF: 0.030 (rank 2+7+5)
... (top 20 selected)
```

### Step 5: MMR Deduplication (lambda=0.6)
Removes near-duplicate facts. "Project Aurora is an internal AI initiative" kept because it adds information beyond Alice's direct involvement. Entity "Alice" summary and entity "Project Aurora" summary kept as they provide background.

Top 10 after MMR:
```
1. "Alice is lead developer on Project Aurora"
2. "Alice contributes to the Design System v2 effort"
3. Entity "Alice" (summary)
4. Entity "Project Aurora" (summary)
5. "Project Aurora is an internal AI initiative"
6. "Alice attended the Q3 planning offsite"
7. Entity "Acme Corp" (summary)
... (3 more)
```

### Step 6: Context Construction

```
FACTS and ENTITIES represent relevant context to the current conversation.

These are the most relevant facts and their valid date ranges.
format: FACT (Date range: from - to)
<FACTS>
Alice is lead developer on Project Aurora (Date range: 2024-06-01 - present)
Alice contributes to the Design System v2 effort (Date range: 2024-09-15 - present)
Project Aurora is an internal AI initiative at Acme Corp (Date range: 2024-06-01 - present)
Alice attended the Q3 planning offsite (Date range: 2024-07-15 - 2024-07-17)
</FACTS>

These are the most relevant entities
ENTITY_NAME: entity summary
<ENTITIES>
Alice: Senior software engineer at Acme Corp, lead developer on Project Aurora, contributor to Design System v2
Project Aurora: Internal AI initiative at Acme Corp, in active development since June 2024
Acme Corp: Technology company, parent organization for Project Aurora and Design System v2
</ENTITIES>
```

**Final token count**: ~180 tokens
**Total pipeline latency**: ~420ms (embedding 85ms + search 180ms parallel + fusion 5ms + MMR 10ms + construction 8ms)

---

## Embedding Lifecycle

### When to Re-embed

Embeddings become stale when the underlying text changes. Track and re-embed in these scenarios:

| Trigger | Action |
|---------|--------|
| Entity summary updated (after new episode ingestion) | Re-embed the entity node |
| Edge fact text modified | Re-embed the edge |
| Community summary regenerated | Re-embed the community node |
| Embedding model version change | Re-embed ALL nodes and edges (full migration) |

### Embedding Model Migration

When switching embedding models (e.g., text-embedding-3-small to text-embedding-3-large):

1. **Do not mix embeddings from different models in the same index.** Vectors from different models occupy different vector spaces and cosine similarity between them is meaningless.

2. **Migration strategy**:
```
Phase 1: Create new vector indexes with the new model's dimensions
Phase 2: Batch re-embed all nodes/edges using the new model
Phase 3: Swap query pipeline to use new indexes
Phase 4: Drop old indexes
```

3. **Batch re-embedding**: Process in batches of 100-500 texts per API call (OpenAI supports batch embedding). For a graph with 100K edges, expect:
   - text-embedding-3-small: ~200 API calls, ~$0.40, ~30 minutes
   - text-embedding-3-large: ~200 API calls, ~$2.60, ~45 minutes

4. **Zero-downtime migration**: Run both old and new indexes simultaneously during Phase 2. Query the old index until Phase 3. This doubles storage temporarily but avoids downtime.

### Staleness Detection

Track `embedded_at` timestamp on each node/edge. During retrieval, log warnings if any returned result has `embedded_at` older than `updated_at`:
```python
stale_results = [r for r in results if r.embedded_at < r.updated_at]
if stale_results:
    log.warning(f"Retrieval returned {len(stale_results)} stale embeddings")
    # Optionally trigger async re-embedding job
```

---

## Production Monitoring

### What to Log

Every retrieval call should log structured data for observability:

```python
retrieval_log = {
    "query_id": str,               # Unique ID for this retrieval call
    "query_text": str,             # The raw query (redact if PII concern)
    "group_id": str,               # Which graph group was searched
    "intent": str,                 # Classified intent type
    "search_methods": {
        "cosine": {"result_count": int, "top_score": float, "latency_ms": int},
        "bm25": {"result_count": int, "top_score": float, "latency_ms": int},
        "bfs": {"result_count": int, "latency_ms": int},
    },
    "fusion_result_count": int,    # Results after RRF/weighted fusion
    "reranker": {
        "method": str,             # "cross_encoder", "mmr", etc.
        "top_score": float,
        "bottom_score": float,
        "latency_ms": int,
    },
    "final_result_count": int,     # Results in constructed context
    "context_token_count": int,    # Token count of final context string
    "total_latency_ms": int,       # End-to-end retrieval latency
    "fallback_used": str | None,   # "community_summaries", "insufficient_context", or null
    "confidence": str,             # "high", "low", "none"
}
```

### Alerts to Set

| Alert | Condition | Indicates |
|-------|-----------|-----------|
| High latency | total_latency_ms > 3000 | Index degradation, network issues |
| Low result count | final_result_count < 3 consistently | Sparse graph, poor embedding coverage |
| Low confidence spike | confidence="none" rate > 20% over 1hr | Query distribution shift, stale embeddings |
| Score collapse | reranker top_score < threshold for >50% of queries | Embedding model drift, index corruption |
| Stale embedding rate | stale_results / total_results > 10% | Re-embedding pipeline falling behind |

### Detecting Retrieval Quality Degradation

Run the evaluation suite (see Retrieval Evaluation section) on a scheduled basis (daily or weekly) against the golden dataset. Track metrics over time:

- **Precision@10 drops >5%**: Investigate recent index or pipeline changes
- **MRR drops >10%**: Ranking quality degraded, check reranker configuration
- **Recall@20 drops >5%**: Search coverage reduced, check embedding freshness and index health

Store evaluation results in a time series to detect gradual drift vs sudden drops.

---

## Performance Optimization

### Latency Budget (target: < 3 seconds total)
- Embedding generation: ~100ms
- Vector search: ~50-200ms
- Fulltext search: ~20-50ms
- BFS traversal: ~50-100ms (2-hop limit)
- RRF fusion: ~5ms
- Cross-encoder rerank (20 candidates): ~100-300ms
- Context construction: ~10ms
- **Total retrieval**: ~250-800ms

The remaining budget is for LLM response generation.

### Token Budget
- Zep achieves strong results with ~1,600 context tokens (vs 115,000 for full-context)
- Recommended range: 1,600-4,000 tokens depending on complexity
- More tokens != better results — targeted context outperforms kitchen-sink approach

### Index Strategy
- Create all indexes (vector, fulltext, b-tree) before ingestion
- Vector indexes: one per element type (entity embeddings, edge embeddings, community embeddings)
- Fulltext indexes: entity names+summaries, edge facts, community names
- B-tree indexes: uuid, group_id (for filtering)

### Caching
- Cache query embeddings for repeated/similar queries (15-minute TTL)
- Cache community summaries (refresh on community update)
- Do NOT cache search results (graph state changes with each ingestion)
