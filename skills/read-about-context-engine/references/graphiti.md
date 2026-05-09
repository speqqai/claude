# Graphiti Reference

Graphiti is a framework by Zep for building and querying temporally-aware knowledge graphs for AI agents. Unlike static RAG, Graphiti continuously integrates user interactions, structured data, and external information into a coherent, queryable graph without requiring full recomputation.

Source: https://github.com/getzep/graphiti

---

## Getting Started / Lifecycle

The minimal lifecycle for a Graphiti application:

```python
from graphiti_core import Graphiti
from graphiti_core.llm_client import OpenAIClient
from graphiti_core.embedder import OpenAIEmbedder
from graphiti_core.nodes import EpisodeType
from datetime import datetime

# 1. Initialize the client
graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    llm_client=OpenAIClient(model="gpt-4.1-mini"),
    embedder=OpenAIEmbedder(model="text-embedding-3-small"),
)

# 2. Build indices and constraints (REQUIRED on first run)
await graphiti.build_indices_and_constraints()

# 3. Add episodes
await graphiti.add_episode(
    name="conversation_turn_1",
    episode_body="User: I just started a new job at Acme Corp.",
    source=EpisodeType.message,
    source_description="user conversation",
    reference_time=datetime.now(),
    group_id="user_123",
)

# 4. Search the graph
results = await graphiti.search(
    query="Where does the user work?",
    group_ids=["user_123"],
    num_results=10,
)

for edge in results:
    print(f"{edge.fact} (valid: {edge.valid_at} - {edge.invalid_at})")

# 5. Close the client (REQUIRED — releases driver connections)
await graphiti.close()
```

Key points:
- `build_indices_and_constraints()` must be called once before the first `add_episode()`. It creates the database indices and constraints needed for entity resolution and search. Calling it again is safe (idempotent).
- `close()` must be called when the client is no longer needed. It releases the underlying graph driver connection pool. Omitting this leaks connections.

---

## Core Architecture

### Episodes — The Atomic Data Unit

Episodes are raw data ingested into the graph. Three types:

| Type | Description | Use Case |
|------|-------------|----------|
| `message` | Short text with actor attribution | Conversations, chat memory |
| `text` | Longer unstructured text | Documents, articles, notes |
| `json` | Structured key-value data | CRM records, API responses, business data |

Each episode carries:
- `name`: descriptive label
- `episode_body`: the raw content (parameter name for `add_episode()`)
- `source`: one of `message`, `text`, `json`
- `source_description`: context about the data origin
- `reference_timestamp` (`t_ref`): when the event occurred (enables relative date resolution)
- `group_id`: namespace for multi-tenant isolation

Note on naming: When using `add_episode()`, the content is passed as the `episode_body` parameter. When using `RawEpisode` for bulk ingestion, the field is named `content`. They serve the same purpose but have different parameter names.

Episodes are **immutable** — they serve as a non-lossy data store from which entities and facts are extracted. Episodic edges connect episodes to their extracted semantic entities bidirectionally, enabling both forward traversal (episode -> entities) and backward traversal (entity -> source episodes for citation).

### Entity Extraction Pipeline

1. **Initial extraction**: Process current message + last n messages (n=4, two conversation turns) for named entity recognition. Speakers are auto-extracted as entities.
2. **Reflection**: Inspired by Reflexion (Shinn et al., 2023) — a second pass to minimize hallucinations and improve extraction coverage.
3. **Summary generation**: Extract entity summary from episode context for downstream resolution and retrieval.
4. **Embedding**: Embed entity name into 1024-dimensional vector space.
5. **Entity resolution**: Hybrid search (cosine similarity on entity name embeddings + fulltext search on entity names and summaries) to find candidate duplicates. LLM-based resolution prompt determines if new entity matches an existing one. On match, generate updated name and summary.
6. **Graph integration**: Insert using predefined Cypher queries (not LLM-generated) for schema consistency.

### Fact (Edge) Extraction Pipeline

1. **Extraction**: For each entity pair, extract facts with predicate labels. The same fact can appear between different entity pairs (hyper-edge modeling).
2. **Embedding**: Generate embeddings for fact text.
3. **Edge deduplication**: Hybrid search constrained to edges between the same entity pair (reduces search space, prevents cross-entity false matches). LLM determines if new edge duplicates an existing one.
4. **Temporal extraction**: Extract temporal information using `t_ref`. Handles both absolute timestamps ("born on June 23, 1912") and relative timestamps ("two weeks ago"). Produces four timestamps:
   - `t'_created` / `t'_expired` (transaction timeline T'): when facts were recorded/removed in the system
   - `t_valid` / `t_invalid` (event timeline T): when facts were true in reality
5. **Edge invalidation**: Compare new edges against semantically related existing edges. When contradictions are found with temporal overlap, set `t_invalid` of the old edge to `t_valid` of the new edge. New information always wins on T'.

### Community Detection

- Uses **label propagation algorithm** (not Leiden) for straightforward dynamic extension.
- **Dynamic update**: When a new entity is added, survey neighbor communities, assign to plurality community, update summary.
- **Periodic refresh**: Full label propagation run needed periodically as dynamic updates diverge over time. Call `build_communities()` to trigger a full refresh.
- **Summaries**: Map-reduce style summarization of member nodes (following GraphRAG).
- **Community names**: Generated with key terms and subjects, embedded for cosine similarity search.

**Community management guidance**: After bulk ingestion or significant graph changes, call `await graphiti.build_communities()` to recompute all communities from scratch. For ongoing incremental ingestion, communities are updated dynamically per-episode. Plan a periodic full refresh (e.g., daily or weekly depending on ingestion volume) since incremental updates diverge over time.

---

## MCP Server

Graphiti exposes its capabilities through an MCP (Model Context Protocol) server.

### Transport Options
- **HTTP** (default): endpoint at `http://localhost:8000/mcp/`
- **stdio**: for Claude Desktop, direct process communication

### Available MCP Tools

| Tool | Description |
|------|-------------|
| `add_episode` | Add an episode to the knowledge graph (text, JSON, or message format) |
| `search_nodes` | Search for relevant entity node summaries |
| `search_facts` | Search for relevant facts (edges between entities) |
| `delete_entity_edge` | Delete an entity edge by UUID |
| `delete_episode` | Delete an episode by UUID |
| `get_entity_edge` | Get an entity edge by UUID |
| `get_episodes` | Get most recent episodes for a group |
| `clear_graph` | Clear all data and rebuild indices |
| `get_status` | Get server and database connection status |

Note: The MCP server uses `delete_episode` as the tool name. In the Python SDK, the equivalent method is `remove_episode()`. They perform the same operation (removing an episode and its episodic edges from the graph).

### Configuration (config.yaml)

```yaml
server:
  transport: "http"  # or "stdio"

llm:
  provider: "openai"  # openai, anthropic, gemini, groq, azure_openai
  model: "gpt-4.1"

embedder:
  provider: "openai"
  model: "text-embedding-3-small"

database:
  provider: "neo4j"  # neo4j, falkordb, kuzu, neptune
  providers:
    neo4j:
      uri: "bolt://localhost:7687"
      username: "neo4j"
      password: "your_password"
      database: "neo4j"
    falkordb:
      uri: "redis://localhost:6379"
      password: ""
      database: "default_db"

graphiti:
  entity_types:
    - name: "Preference"
      description: "User preferences, choices, opinions, or selections"
    - name: "Requirement"
      description: "Specific needs, features, or functionality"
    - name: "Procedure"
      description: "Standard operating procedures and sequential instructions"
    - name: "Location"
      description: "Physical or virtual places"
    - name: "Event"
      description: "Time-bound activities or occurrences"
    - name: "Organization"
      description: "Companies, institutions, or groups"
    - name: "Document"
      description: "Information content (books, articles, reports)"
    - name: "Topic"
      description: "Subject of conversation or knowledge domain (fallback)"
    - name: "Object"
      description: "Physical items, tools, or devices (fallback)"
```

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `NEO4J_URI` | Neo4j database URI (default: `bolt://localhost:7687`) |
| `NEO4J_USER` | Neo4j username (default: `neo4j`) |
| `NEO4J_PASSWORD` | Neo4j password |
| `OPENAI_API_KEY` | OpenAI API key (required for OpenAI LLM/embedder) |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `GOOGLE_API_KEY` | Gemini API key |
| `GROQ_API_KEY` | Groq API key |
| `SEMAPHORE_LIMIT` | Episode processing concurrency (default: 10) |
| `GRAPHITI_TELEMETRY_ENABLED` | Set to `false` to disable telemetry |

### Concurrency Tuning (SEMAPHORE_LIMIT)

Each episode triggers multiple LLM calls (entity extraction, deduplication, summarization), so actual concurrent LLM requests = SEMAPHORE_LIMIT * multiplier.

| Provider | Tier | RPM | Recommended SEMAPHORE_LIMIT |
|----------|------|-----|----------------------------|
| OpenAI | Tier 1 (free) | 3 | 1-2 |
| OpenAI | Tier 2 | 60 | 5-8 |
| OpenAI | Tier 3 | 500 | 10-15 |
| OpenAI | Tier 4 | 5,000 | 20-50 |
| Anthropic | Default | 50 | 5-8 |
| Anthropic | High | 1,000 | 15-30 |
| Ollama | Local | Hardware-dependent | 1-5 |

Symptoms of misconfiguration:
- **Too high**: 429 rate limit errors, increased API costs
- **Too low**: Slow throughput, underutilized quota

### Integration Examples

**Cursor IDE / HTTP clients:**
```json
{
  "mcpServers": {
    "graphiti-memory": {
      "url": "http://<HOST_IP>:8000/mcp/"
    }
  }
}
```

**Claude Desktop (requires mcp-remote bridge):**
```json
{
  "mcpServers": {
    "graphiti-memory": {
      "command": "npx",
      "args": ["mcp-remote", "http://<HOST_IP>:8000/mcp/"]
    }
  }
}
```

**stdio transport (Claude Desktop native):**
```json
{
  "mcpServers": {
    "graphiti-memory": {
      "transport": "stdio",
      "command": "/path/to/uv",
      "args": [
        "run", "--isolated", "--directory",
        "/path/to/graphiti/mcp_server",
        "--project", ".", "main.py",
        "--transport", "stdio"
      ],
      "env": {
        "NEO4J_URI": "bolt://localhost:7687",
        "NEO4J_USER": "neo4j",
        "NEO4J_PASSWORD": "password",
        "OPENAI_API_KEY": "sk-..."
      }
    }
  }
}
```

### Docker Deployment

**Default (FalkorDB combined container):**
```bash
cd graphiti/mcp_server && docker compose up
```

**With Neo4j:**
```bash
cd graphiti/mcp_server && docker compose -f docker/docker-compose-neo4j.yml up
```

### Working with JSON Data

```python
add_episode(
    name="Customer Profile",
    episode_body='{"company": {"name": "Acme"}, "products": [{"id": "P001", "name": "CloudSync"}]}',
    source="json",
    source_description="CRM data"
)
```

JSON episodes automatically extract entities and relationships from structured data, making them ideal for ingesting business data alongside conversational data.

---

## Graphiti Python SDK (Direct Usage)

### Client Initialization

```python
from graphiti_core import Graphiti
from graphiti_core.llm_client import OpenAIClient
from graphiti_core.embedder import OpenAIEmbedder

graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    llm_client=OpenAIClient(model="gpt-4.1-mini"),
    embedder=OpenAIEmbedder(model="text-embedding-3-small"),
    store_raw_episode_content=True,       # store full episode text in graph (default True; set False for privacy)
    max_coroutines=10,                    # controls concurrent LLM calls per episode
)
```

**Constructor parameters:**
- `uri`: Graph database connection URI
- `user`: Database username
- `password`: Database password
- `llm_client`: LLM client instance for extraction and resolution
- `embedder`: Embedding client instance for vector operations
- `graph_driver`: Optional — inject a pre-configured graph driver directly instead of using uri/user/password. Useful for custom connection pooling or non-standard backends.
- `cross_encoder`: Optional — inject a cross-encoder model for result reranking. Improves search precision by reranking candidate results after initial retrieval.
- `store_raw_episode_content`: Whether to persist full episode text in the graph (default `True`). Set to `False` when episode content contains sensitive/PII data that should not be stored persistently — entities and facts are still extracted but the raw text is discarded after processing.
- `max_coroutines`: Controls the concurrency of LLM calls during episode processing (default `10`). Equivalent to `SEMAPHORE_LIMIT` in the MCP server.

**Supported graph backends:**

| Backend | URI Example | Notes |
|---------|-------------|-------|
| Neo4j | `bolt://localhost:7687` | Full-featured, most common |
| FalkorDB | `redis://localhost:6379` | Redis-compatible graph DB |
| Kuzu | `kuzu:///path/to/db` | Embedded graph DB, no server needed |
| Amazon Neptune | `wss://your-cluster.neptune.amazonaws.com:8182/...` | AWS managed graph service |

**Direct driver injection:**

```python
from graphiti_core.driver import Neo4jDriver

driver = Neo4jDriver(uri="bolt://localhost:7687", user="neo4j", password="password")
graphiti = Graphiti(
    graph_driver=driver,
    llm_client=OpenAIClient(model="gpt-4.1-mini"),
    embedder=OpenAIEmbedder(model="text-embedding-3-small"),
)
```

### Setup and Teardown

```python
# REQUIRED: Call once before first add_episode()
await graphiti.build_indices_and_constraints()

# ... use the client ...

# REQUIRED: Call when done to release connections
await graphiti.close()
```

### Adding Episodes

```python
from graphiti_core.nodes import EpisodeType
from datetime import datetime

await graphiti.add_episode(
    name="conversation_turn_1",
    episode_body="User: I just started a new job at Acme Corp as a senior engineer.",
    source=EpisodeType.message,
    source_description="user conversation",
    reference_time=datetime.now(),
    group_id="user_123",
)
```

**Advanced `add_episode()` parameters:**

```python
from pydantic import BaseModel, Field

# Custom entity types (passed per-episode, NOT to constructor)
class Product(BaseModel):
    """A software product or service."""
    name: str = Field(description="Product name")

class Actor(BaseModel):
    """A person or role that interacts with the system."""
    name: str = Field(description="Person or role name")

# Custom edge types (Pydantic BaseModel subclasses, NOT EntityEdge subclasses)
class Manages(BaseModel):
    """An actor manages a product."""
    name: str = Field(description="Relationship label")

class DependsOn(BaseModel):
    """One entity depends on another."""
    name: str = Field(description="Relationship label")

await graphiti.add_episode(
    name="product_discussion",
    episode_body="Alice manages the CloudSync product, which depends on the AuthService.",
    source=EpisodeType.message,
    source_description="team standup",
    reference_time=datetime.now(),
    group_id="team_42",
    # Ontology control (all optional):
    entity_types={"Product": Product, "Actor": Actor},       # dict[str, type[BaseModel]]
    edge_types={"Manages": Manages, "DependsOn": DependsOn}, # dict[str, type[BaseModel]]
    excluded_entity_types=["Topic", "Object"],                # suppress fallback types
    edge_type_map={"Manages": ("Actor", "Product")},          # constrain which entity pairs an edge type can connect
    # Behavior control:
    update_communities=True,                                   # update community assignments (default True)
    custom_extraction_instructions="Focus on product ownership and dependency relationships.",
)
```

### Adding Pre-Extracted Triples

When you already have structured triples (e.g., from an external NLP pipeline), use `add_triplet()` to insert them directly without LLM extraction:

```python
from graphiti_core.nodes import EntityNode
from graphiti_core.edges import EntityEdge

source_node = EntityNode(name="Alice", labels=["Actor"], summary="Engineering manager")
target_node = EntityNode(name="CloudSync", labels=["Product"], summary="Cloud synchronization product")

await graphiti.add_triplet(
    source_node=source_node,
    target_node=target_node,
    fact="Alice manages CloudSync",
    group_id="team_42",
)
```

### Bulk Ingestion

```python
from graphiti_core.nodes import RawEpisode, EpisodeType

episodes = [
    RawEpisode(
        name=f"message_{i}",
        content=msg,                      # Note: RawEpisode uses 'content', not 'episode_body'
        source=EpisodeType.message,
        source_description="chat history",
        reference_time=timestamps[i],
    )
    for i, msg in enumerate(messages)
]

await graphiti.add_episode_bulk(episodes, group_id="workspace_456")
```

### Search

```python
from graphiti_core.search.search_config import (
    SearchConfig,
    EdgeSearchConfig,
    NodeSearchConfig,
    CommunitySearchConfig,
    SearchMethod,
)
from graphiti_core.search.search_filters import SearchFilters, DateFilter, PropertyFilter

# Basic semantic search for facts (edges)
results = await graphiti.search(
    query="What is the user's current job?",
    group_ids=["user_123"],
    num_results=10,
)

# Each result contains: fact text, source entities, valid_at, invalid_at
for edge in results:
    print(f"{edge.fact} (valid: {edge.valid_at} - {edge.invalid_at})")
```

#### SearchFilters

SearchFilters control which nodes and edges are included in search results:

```python
from datetime import datetime

# Filter by node labels
filters = SearchFilters(
    node_labels=["Product", "Actor"],     # only return results involving these entity types
)

# Filter by date range
filters = SearchFilters(
    date_filter=DateFilter(
        start=datetime(2025, 1, 1),
        end=datetime(2025, 6, 30),
    ),
)

# Filter by property values
filters = SearchFilters(
    property_filter=PropertyFilter(
        key="status",
        value="active",
    ),
)

results = await graphiti.search(
    query="active products",
    group_ids=["team_42"],
    search_filter=filters,                # pass filters via search_filter param
)
```

#### SearchConfig

SearchConfig controls which search methods are used and how results are combined:

```python
# Default config searches edges, nodes, and communities
config = SearchConfig(
    edge_config=EdgeSearchConfig(
        search_methods=[SearchMethod.cosine, SearchMethod.bm25],
        rerank=True,
    ),
    node_config=NodeSearchConfig(
        search_methods=[SearchMethod.cosine, SearchMethod.bm25],
        rerank=True,
    ),
    community_config=CommunitySearchConfig(
        search_methods=[SearchMethod.cosine],
    ),
    limit=10,
)

results = await graphiti.search(
    query="What products does Alice manage?",
    group_ids=["team_42"],
    config=config,
)
```

#### center_node_uuid — Ego-Centric Search

Use `center_node_uuid` to anchor search around a specific entity node. This biases results toward facts and relationships connected to that node, which is useful for building context about a specific user or entity:

```python
# First, find the node UUID (e.g., from a previous search or stored reference)
user_node_uuid = "abc123-..."

results = await graphiti.search(
    query="recent activities",
    group_ids=["team_42"],
    center_node_uuid=user_node_uuid,      # anchor search around this entity
    num_results=10,
)
```

#### Temporal Query Example

Graphiti's bitemporality enables querying what was true at a specific point in time:

```python
# "Where did the user work in January 2025?"
# Graphiti stores valid_at/invalid_at on edges, so a search returns
# edges whose validity window overlaps the reference period.
results = await graphiti.search(
    query="Where does the user work?",
    group_ids=["user_123"],
    search_filter=SearchFilters(
        date_filter=DateFilter(
            start=datetime(2025, 1, 1),
            end=datetime(2025, 1, 31),
        ),
    ),
)

for edge in results:
    print(f"{edge.fact} (valid: {edge.valid_at}, invalid: {edge.invalid_at})")
    # e.g., "User works at Acme Corp (valid: 2024-09-01, invalid: 2025-03-15)"
    # e.g., "User works at NewCo (valid: 2025-03-15, invalid: None)"
```

### Retrieving Episodes

```python
# Retrieve recent episodes for a group
episodes = await graphiti.retrieve_episodes(
    group_id="user_123",
    last_n=10,          # number of most recent episodes to return
)

for ep in episodes:
    print(f"{ep.name}: {ep.content}")
```

### Debugging: Nodes and Edges by Episode

```python
# Get all nodes and edges extracted from a specific episode (useful for debugging extraction)
nodes, edges = await graphiti.get_nodes_and_edges_by_episode(episode_uuid="ep-uuid-123")

for node in nodes:
    print(f"Entity: {node.name} ({node.labels})")
for edge in edges:
    print(f"Fact: {edge.fact}")
```

### Removing Episodes

```python
# Remove an episode and its episodic edges from the graph
# Note: entities and facts extracted from the episode are NOT removed,
# since they may also be supported by other episodes.
await graphiti.remove_episode(episode_uuid="ep-uuid-123")
```

### Multi-Tenant group_id Semantics

`group_id` is the primary namespace mechanism for multi-tenant isolation:

- **Entity sharing**: Entity nodes are shared across groups. If two groups both mention "Acme Corp", they resolve to the same entity node.
- **Edge scoping**: Fact edges (relationships) are scoped to a `group_id`. Group A's facts about Acme are separate from Group B's facts.
- **OR search across groups**: When `group_ids=["group_a", "group_b"]` is passed to `search()`, results from both groups are returned (union/OR semantics, not intersection).
- **Isolation guarantee**: A search with `group_ids=["group_a"]` never returns edges belonging to `group_b`.

### Community Management

```python
# Full community rebuild (run after bulk ingestion or periodically)
await graphiti.build_communities()

# Communities are also updated incrementally during add_episode() when
# update_communities=True (the default).
```

### Error Handling Patterns

```python
import asyncio
from openai import RateLimitError

# Rate limit handling — back off and retry
async def add_episode_with_retry(graphiti, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            await graphiti.add_episode(**kwargs)
            return
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt
            await asyncio.sleep(wait_time)

# Connection failure — check driver connectivity
try:
    graphiti = Graphiti(
        uri="bolt://localhost:7687",
        user="neo4j",
        password="password",
        llm_client=OpenAIClient(model="gpt-4.1-mini"),
        embedder=OpenAIEmbedder(model="text-embedding-3-small"),
    )
    await graphiti.build_indices_and_constraints()
except Exception as e:
    # Common causes: database not running, wrong URI, auth failure
    print(f"Connection failed: {e}")

# Partial ingestion failure — bulk ingestion may partially succeed
# add_episode_bulk processes episodes sequentially; if one fails,
# previously ingested episodes remain in the graph. Track progress
# externally if you need to resume from the failure point.
```
