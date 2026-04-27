---
name: context-engine
description: Master context graphs for AI agents using Graphiti, Neo4j, and temporal knowledge graph architecture. Use when designing, building, querying, or debugging knowledge graphs that power agent memory — including ontology design, graph construction pipelines, temporal modeling, hybrid retrieval, community detection, and MCP server integration.
---

# Context Engine

Build and operate temporal knowledge graphs that give AI agents persistent, structured, queryable memory.

## Workflow

1. Load references based on the task.
- For graph construction or ingestion pipelines, read `references/graphiti.md` first.
- For database queries, schema, or Cypher, read `references/neo4j.md`.
- For theoretical foundations or architecture decisions, read `references/temporal-knowledge-graphs.md`.
- For defining node/edge types and domain models, read `references/ontology-design.md`.
- For search, reranking, or retrieval optimization, read `references/retrieval-patterns.md`.

2. Understand the graph architecture before making changes.
- Identify which subgraph tier is involved: episodic, semantic entity, or community.
- Determine whether the task is ingestion (write path) or retrieval (read path).
- Check if temporal modeling (bi-temporal timestamps) is relevant.

3. Design the ontology before writing code.
- Define entity node types with clear, non-overlapping descriptions.
- Define edge types as concise ALL_CAPS predicates with bidirectional mappings.
- Validate that the ontology covers the domain without over-extraction.

4. Build ingestion pipelines following the Graphiti pattern.
- Episodes are the atomic unit of ingestion (message, text, or JSON).
- Entity extraction uses LLM + reflection to minimize hallucinations.
- Entity resolution deduplicates via hybrid search (vector + fulltext) then LLM confirmation.
- Fact extraction produces typed edges between resolved entities.
- Temporal extraction attaches valid_at/invalid_at timestamps to edges.
- Edge invalidation detects contradictions and expires old facts.

5. Implement retrieval using the three-stage pipeline.
- Search phase: combine cosine similarity, BM25 fulltext, and breadth-first graph traversal.
- Rerank phase: apply RRF, MMR, episode-mentions, node-distance, or cross-encoder reranking.
- Constructor phase: format selected nodes and edges into a context string for the LLM.

6. Validate and maintain the graph.
- Monitor graph health: node count, edge count, community coverage.
- Periodically refresh communities via full label propagation.
- Track ingestion latency and LLM call counts per episode.
- Verify retrieval quality with precision/recall on known queries.

## Key Concepts

### Three-Tier Graph Architecture
- **Episode Subgraph**: Raw data store (messages, text, JSON). Non-lossy. Source of truth.
- **Semantic Entity Subgraph**: Extracted entities and facts (edges). Resolved, deduplicated, temporally tracked.
- **Community Subgraph**: Clusters of related entities with summaries. Enables global understanding.

### Bi-Temporal Model
- **Timeline T** (event time): when facts were true in the real world (valid_at, invalid_at).
- **Timeline T'** (transaction time): when facts were recorded in the system (created, expired).
- New information always wins on T' — contradictions invalidate older edges.

### Memory Types (mirrors human cognition)
- **Episodic memory**: distinct events, raw conversations, specific interactions.
- **Semantic memory**: associations, concepts, extracted knowledge, entity relationships.
- **Community memory**: high-level domain understanding, topic clusters, global context.

## Non-Negotiable Rules

1. Episodes are immutable once ingested. Never modify raw episode data.
2. Entity resolution must use hybrid search (vector + fulltext), not LLM-only.
3. Edge invalidation must preserve history — set invalid_at, never delete edges.
4. Use predefined Cypher queries for graph mutations, not LLM-generated queries.
5. Ontology changes require explicit approval — do not add node/edge types ad hoc.
6. All timestamps use ISO 8601 format with timezone offset.
7. Embeddings must be generated before graph integration, not after.

## Output Contract

Return work in this order:

1. Graph architecture summary.
- Which subgraph tiers are affected.
- Node and edge types involved.
- Temporal modeling considerations.

2. Implementation details.
- Ingestion pipeline changes (extraction, resolution, temporal).
- Retrieval pipeline changes (search, rerank, construct).
- Cypher queries used.

3. Ontology impact.
- New or modified node/edge types.
- Entity resolution implications.
- Community detection effects.

4. Validation report.
- Graph health metrics before/after.
- Retrieval quality on test queries.
- Latency and cost considerations.
