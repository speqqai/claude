# Temporal Knowledge Graphs — Scientific Foundations

This reference distills the core scientific contributions from the Graphiti paper (Rasmussen et al., 2025) and related work on using knowledge graphs to power AI agent memory.

Source: "Zep: A Temporal Knowledge Graph Architecture for Agent Memory" (arXiv:2501.13956v1)

> **Terminology note:** Graphiti is the open-source knowledge graph library described in the paper. Zep is the commercial product built on top of Graphiti. This document focuses on the Graphiti algorithm and architecture; "Zep" appears only when referring to the commercial system's benchmark results.

---

## The Problem

Traditional RAG operates on static document corpora. AI agents need dynamic, evolving memory that integrates:
- Ongoing conversations (unstructured, temporal)
- Business data (structured, changing)
- World knowledge (external, updating)

Full conversation histories and datasets cannot fit in LLM context windows. Recursive summarization loses detail. Simple vector search misses relationships and temporal evolution.

## The Solution: Temporally-Aware Dynamic Knowledge Graphs

### Formal Definition

A knowledge graph G = (N, E, phi, T) where:
- N = nodes (entities, episodes, communities)
- E = edges (facts, episodic links, community membership), where each edge carries a temporal tuple (t_valid, t_invalid, t'_created, t'_expired) as part of its structure
- phi: E -> N x N = incidence function mapping edges to node pairs
- T = bi-temporal annotation on edges (see Bi-Temporal Model below)

**Design rationale — edges as first-class citizens:** In Graphiti, edges are the primary information carriers. Facts live on edges, not nodes. Nodes represent entities (nouns), but the meaningful information — relationships, states, preferences, actions — is encoded as labeled, temporally-bounded edges between those entities. This is a deliberate design choice: relationships are more informative than entity descriptions for agent memory, because an agent needs to know what happened between entities, not just that entities exist.

### Three-Tier Hierarchical Architecture

**Tier 1 — Episode Subgraph (G_e)**
- Episodic nodes n_i in N_e contain raw input data
- Episodes are the non-lossy data store
- Episodic edges E_e connect episodes to extracted semantic entities
- Bidirectional indices: semantic artifacts trace back to sources, episodes quickly retrieve their entities and facts

**Tier 2 — Semantic Entity Subgraph (G_s)**
- Entity nodes n_i in N_s represent extracted, resolved entities
- Semantic edges e_i in E_s represent facts/relationships between entities
- Built on top of the episode subgraph through extraction and resolution

**Tier 3 — Community Subgraph (G_c)**
- Community nodes n_i in N_c represent clusters of strongly connected entities
- Contain high-level summarizations
- Community edges E_c connect communities to member entities
- Provides global domain understanding (building on GraphRAG)

### Cognitive Model Parallel

This mirrors psychological models of human memory:
- **Episodic memory** (distinct events) -> Episode Subgraph
- **Semantic memory** (concepts, associations) -> Semantic Entity Subgraph
- **Global understanding** (domain knowledge) -> Community Subgraph

The hierarchy relates episodes, facts, entities, and communities — but this is not a strictly linear pipeline. The graph contains cross-links: episodes connect to entities, entities connect to other entities via edges, communities reference their member entities, and edges trace back to source episodes. The structure is a graph with multiple traversal paths, not a simple chain.

---

## Bi-Temporal Model

A key innovation — tracking TWO timelines simultaneously:

### Timeline T (Event Time)
When facts were true in the real world.
- `t_valid`: when a fact became true
- `t_invalid`: when a fact stopped being true

### Timeline T' (Transaction Time)
When facts were recorded in the system.
- `t'_created`: when a fact was added to the graph
- `t'_expired`: when a fact was marked invalid in the system

### Why Two Timelines?
- T enables **historical queries** ("What was Alice's job in 2023?")
- T' enables **database auditing** ("When did we learn this?")
- T models the **dynamic nature of conversational data** — facts evolve over time
- T' provides the **conflict resolution rule**: new information (later T') always wins

### Temporal Extraction
- Uses reference timestamp `t_ref` from the episode
- Resolves absolute timestamps: "born on June 23, 1912" -> `t_valid = 1912-06-23`
- Resolves relative timestamps: "two weeks ago" + `t_ref` -> calculated datetime
- All timestamps in ISO 8601 with timezone offset

### Edge Invalidation vs Edge Deduplication

These are two distinct operations in the pipeline:

**Edge deduplication** (same fact restated): When a new edge expresses the same fact as an existing edge (e.g., "Alice works at Acme" stated again in a later conversation), the edges are merged. The existing edge is preserved and updated; no new edge is created. This prevents graph bloat from repeated statements.

**Edge invalidation** (contradictory fact, temporal bounds): When a new fact contradicts an existing one (e.g., "Alice now works at Beta Corp" vs existing "Alice works at Acme"):
1. New edge is compared against semantically related existing edges via LLM
2. If contradiction found with temporal overlap: `old_edge.t_invalid = new_edge.t_valid`
3. Old edge is preserved (never deleted) — history is maintained
4. New information always takes priority (T' ordering)

---

## Episode Ingestion Pipeline

Each new episode (conversation turn, business data event, etc.) is processed through a complete ordered pipeline. This is an **incremental, online process** — a major contribution of Graphiti. Unlike batch approaches (e.g., GraphRAG), Graphiti processes each episode as it arrives without requiring full graph recomputation.

1. **Entity extraction** — LLM identifies entity nodes mentioned in the episode
2. **Entity resolution** — New entities are matched against existing graph nodes to deduplicate (see Entity Resolution below)
3. **Edge extraction** — LLM identifies facts/relationships between the resolved entities
4. **Edge resolution/deduplication** — New edges are compared against existing edges between the same entity pairs; duplicates are merged
5. **Temporal extraction** — LLM extracts temporal bounds (t_valid, t_invalid) using the episode's reference timestamp
6. **Edge invalidation** — New edges are checked for contradictions against existing edges; contradicted edges receive temporal bounds
7. **Community update** — Affected communities are updated via the dynamic extension of label propagation

---

## Entity Resolution

### The Challenge
Same real-world entity may appear with different names across episodes ("Bob", "Robert Smith", "Bob Smith"). Must deduplicate without losing information.

### The Process
1. Embed new entity name into vector space (dimensionality depends on the embedding model — the paper uses BGE-m3, which produces 1024-d vectors, but this is an implementation choice, not part of the algorithm specification)
2. **Vector search**: cosine similarity against existing entity node embeddings
3. **Fulltext search**: BM25 search on existing entity names and summaries
4. Combine candidates from both searches
5. **LLM resolution**: evaluate candidates + episode context to determine duplicates
6. On match: generate updated (most complete) name and summary
7. Merge into existing node

### Why Hybrid Search?
- Vector search catches semantic similarity ("NYC" ~ "New York City")
- Fulltext search catches exact/partial name matches ("Bob Smith" ~ "Bob")
- Together they maximize recall before the LLM precision step

---

## Community Detection

### Label Propagation Algorithm
Chosen over Leiden for its dynamic extensibility. The key reason: one step of label propagation — assigning a node to the community held by the plurality of its neighbors — is exactly the operation needed when a new node is added to the graph. This makes label propagation a natural fit for incremental graph construction. Leiden, by contrast, is designed for batch execution over the full graph and does not have an efficient single-node update operation.

### Dynamic Extension Algorithm
When adding new entity n_i:
1. Survey communities of all neighboring nodes
2. Assign n_i to the community held by plurality of neighbors
3. Update community summary and graph
4. This is equivalent to one recursive step of label propagation

### Trade-offs
- **Pro**: Efficient, low latency, reduces LLM inference costs
- **Con**: Communities gradually diverge from full-run results (community drift)
- **Mitigation**: Periodic full community refresh

### Community Summaries
- Generated via iterative map-reduce summarization of member nodes
- Community names contain key terms and subjects
- Names are embedded for cosine similarity search
- Retrieval approach parallels LightRAG's high-level key search

---

## Memory Retrieval System

The retrieval pipeline uses three stages. The paper describes these as search, rerank, and context construction. The notation below (phi, rho, chi) is introduced in this document to compactly reference each stage; the paper uses a similar functional decomposition but does not assign these exact Greek letter symbols.

Pipeline: f(query) = chi(rho(phi(query)))

### Step 1: Search (phi)
Three complementary search functions:

| Method | Symbol | Targets | Similarity Type |
|--------|--------|---------|----------------|
| Cosine similarity | phi_cos | Edges (fact), Nodes (name), Communities (name) | Semantic similarity |
| BM25 fulltext | phi_bm25 | Same targets | Word/lexical similarity |
| Breadth-first search | phi_bfs | Nodes and edges within n-hops | Contextual similarity (graph proximity) |

Key insight: BFS reveals **contextual similarity** — nodes closer in the graph appeared in similar conversational contexts. Using recent episodes as BFS seeds incorporates recently mentioned entities. **Why BFS from recent episodes works:** conversational context creates graph locality. Entities mentioned together in conversations tend to be linked by short paths in the graph, so a BFS seeded from recently referenced entities efficiently surfaces the most conversationally relevant neighborhood.

### Step 2: Rerank (rho)
Multiple reranking strategies:

| Reranker | Description | Cost |
|----------|-------------|------|
| **RRF** (Reciprocal Rank Fusion) | Combines rankings from multiple search methods | Low |
| **MMR** (Maximal Marginal Relevance) | Balances relevance with diversity | Low |
| **Episode-mentions** | Prioritizes frequently referenced entities/facts | Low |
| **Node distance** | Reorders by graph distance from centroid node | Low |
| **Cross-encoder** | LLM scores relevance via cross-attention | High |

**RRF formula:** Given a document d appearing in multiple ranked lists, the RRF score is:

```
RRF(d) = sum over all ranked lists r of: 1 / (k + rank_r(d))
```

where k is a constant (typically 60) that mitigates the impact of high rankings from any single list, and rank_r(d) is the rank of document d in list r. Documents not appearing in a given list are excluded from that list's contribution.

### Step 3: Construct (chi)
Transforms graph elements into text context:
- For edges: returns fact text + valid_at + invalid_at
- For entity nodes: returns name + summary
- For community nodes: returns summary

### Context Template
```
FACTS and ENTITIES represent relevant context to the current conversation.

These are the most relevant facts and their valid date ranges.
format: FACT (Date range: from - to)
<FACTS>
{facts}
</FACTS>

These are the most relevant entities
ENTITY_NAME: entity summary
<ENTITIES>
{entities}
</ENTITIES>
```

---

## Implementation Details (from paper)

### Model Choices and Cost
- **Graph construction LLM**: GPT-4o-mini — used for entity extraction, edge extraction, entity resolution, edge resolution, temporal extraction, and edge invalidation. Chosen as a cost-effective option for the high volume of LLM calls during ingestion.
- **Embedding and reranking model**: BGE-m3 — produces 1024-d embeddings, used for both vector search and cross-encoder reranking.
- **Underlying database**: Neo4j — the paper's implementation uses Neo4j as the graph database backend for storing nodes, edges, and running graph traversals.

### Incremental vs Batch Construction
A major contribution of Graphiti is its **online, incremental** graph construction. Each episode is processed individually as it arrives. This contrasts with batch approaches like GraphRAG, which require processing the entire corpus to build the graph. Incremental construction is essential for agent memory because conversations arrive in real-time and the graph must be queryable at all times, not only after a full batch run.

---

## Experimental Results

### Deep Memory Retrieval (DMR) Benchmark
500 multi-session conversations, 5 sessions each, up to 12 messages per session. The benchmark tests question answering about facts from earlier sessions — specifically measuring whether the memory system can retrieve and surface information that was discussed in prior conversation sessions to answer questions in the current session.

| Memory System | Model | Score |
|---------------|-------|-------|
| Recursive Summarization | gpt-4-turbo | 35.3% |
| Conversation Summaries | gpt-4-turbo | 78.6% |
| MemGPT | gpt-4-turbo | 93.4% |
| Full-conversation | gpt-4-turbo | 94.4% |
| **Zep** | gpt-4-turbo | **94.8%** |
| Full-conversation | gpt-4o-mini | 98.0% |
| **Zep** | gpt-4o-mini | **98.2%** |

### LongMemEval Benchmark
Conversations averaging ~115,000 tokens. Six question types: single-session-user, single-session-assistant, single-session-preference, multi-session, knowledge-update, temporal-reasoning.

| Memory | Model | Score | Latency | Context Tokens |
|--------|-------|-------|---------|---------------|
| Full-context | gpt-4o-mini | 55.4% | 31.3s | 115k |
| **Zep** | gpt-4o-mini | **63.8%** | **3.2s** | **1.6k** |
| Full-context | gpt-4o | 60.2% | 28.9s | 115k |
| **Zep** | gpt-4o | **71.2%** | **2.58s** | **1.6k** |

Key findings (raw scores and relative improvements):
- **gpt-4o: 71.2% vs 60.2% full-context** — an absolute improvement of +11.0 percentage points (+18.3% relative improvement)
- **gpt-4o-mini: 63.8% vs 55.4% full-context** — an absolute improvement of +8.4 percentage points (+15.2% relative improvement)
- **90% latency reduction** (31s -> 3s)
- **70x fewer context tokens** (115k -> 1.6k)
- Strongest improvements by category (relative improvements over full-context baseline): temporal-reasoning (+38-48% relative), single-session-preference (+78-184% relative), multi-session (+17-31% relative). These are relative percentage improvements, not absolute percentage point differences — the large relative numbers for single-session-preference reflect low baseline scores where small absolute gains produce large relative changes.
- Weaker on single-session-assistant questions (-9 to -18% relative) — area for improvement

### Implications
1. Knowledge graphs are especially valuable for temporal reasoning and preference tracking
2. Token reduction translates directly to cost savings at scale
3. Latency improvements make real-time agent memory practical
4. More capable models (gpt-4o vs gpt-4o-mini) benefit more from structured context

---

## Graph Construction Prompts (from paper appendix)

> **Note:** The prompts below are paraphrased summaries of the appendix content, not verbatim reproductions. The actual prompts in the paper include JSON schema output constraints that enforce structured LLM responses (specifying exact field names, types, and required fields for each extraction step).

### Entity Extraction Prompt
Given previous messages and current message, extract entity nodes that are explicitly or implicitly mentioned.
Guidelines:
1. ALWAYS extract the speaker/actor as the first node
2. Extract significant entities, concepts, or actors
3. DO NOT create nodes for relationships or actions
4. DO NOT create nodes for temporal information (dates, times, years)
5. Use full names
6. Only extract from CURRENT MESSAGE, not previous messages

### Entity Resolution Prompt
Given existing nodes, previous messages, current message, and a new node — determine if the new node is a duplicate of an existing node.
Task:
1. Return is_duplicate: true/false
2. If duplicate, return UUID of existing node
3. If duplicate, return the most complete full name
Guidelines: Use both name and summary to determine duplicates.

### Fact Extraction Prompt
Given messages and entities, extract all facts between listed entities from the current message.
Guidelines:
1. Facts only between provided entities
2. Each fact = clear relationship between two DISTINCT nodes
3. relation_type: concise ALL_CAPS (e.g., WORKS_FOR, IS_FRIENDS_WITH)
4. Include detailed fact with all relevant information
5. Consider temporal aspects

### Fact Resolution Prompt
Determine if a new edge duplicates any existing edge between the same entity pair.
Guidelines: Facts don't need to be identical — just express the same information.

### Temporal Extraction Prompt
Extract time information from facts using reference timestamp.
- valid_at: when the relationship became true
- invalid_at: when the relationship stopped being true
- ISO 8601 format
- Use reference timestamp for present-tense facts
- Calculate relative dates from reference timestamp
- Leave null if no temporal information establishes/changes the relationship

---

## Related Work Comparisons

| System | Approach | Temporal | Incremental | Key Difference from Graphiti |
|--------|----------|----------|-------------|------------------------------|
| **GraphRAG** (Microsoft) | Static KG + Leiden communities + map-reduce summarization | No | No (batch only) | No temporal model, requires full recomputation on new data, uses Leiden (not suitable for incremental updates) |
| **MemGPT** | Virtual context management with paging-based memory | No | Yes (online) | Not KG-based — uses an OS-inspired paging mechanism to swap memory segments in/out of LLM context |
| **AriGraph** | Episodic + semantic memory graph for game agents | Limited | Yes | Designed for game/simulation environments; combines episodic and semantic memory but lacks bi-temporal model and community layer |
| **LightRAG** | Dual-level retrieval (low-level entity, high-level key) | No | Partial | Graphiti's community search parallels LightRAG's high-level key search, but LightRAG lacks temporal awareness and incremental construction |
| **Full-context** | Pass entire conversation history to LLM | N/A | N/A | No retrieval — limited by context window, high latency and cost at scale |
| **Recursive Summarization** | Progressively summarize conversation history | No | Yes | Lossy — detail is discarded at each summarization step |

---

## Limitations (from paper)

1. **LLM cost during construction**: Each episode requires multiple LLM calls (entity extraction, resolution, edge extraction, resolution, temporal extraction, invalidation). At scale, graph construction costs can be significant even with cost-effective models like GPT-4o-mini.
2. **Extraction error propagation**: Errors in early pipeline stages (e.g., missed entities, incorrect entity resolution) propagate downstream. A missed entity means its edges are never extracted; an incorrect entity merge corrupts the node permanently.
3. **Community drift**: The dynamic extension algorithm (single-step label propagation) causes communities to gradually diverge from what a full community detection run would produce. Periodic full refresh is needed but adds cost.

---

## Future Research Directions (from paper)

1. **Fine-tuned extraction models**: Domain-specific models for entity/edge extraction (following Triplex, Distill-SynthKG)
2. **Domain ontologies**: Formal ontologies within Graphiti framework (foundational in pre-LLM KG work)
3. **Better benchmarks**: Enterprise-focused memory benchmarks beyond needle-in-haystack
4. **Structured data synthesis**: Evaluating conversation history combined with business data
5. **Scalability metrics**: Production cost and latency benchmarks need more attention
6. **Hybrid approaches**: Combining Graphiti with LightRAG's high-level key search methodology
