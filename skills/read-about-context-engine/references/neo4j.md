# Neo4j Reference

Neo4j is the graph database that powers Graphiti's knowledge graph storage. This reference covers the essential Neo4j concepts, Cypher query language, and operational patterns needed for context engine work.

Sources: https://neo4j.com/docs/, Neo4j Cypher Manual, Neo4j JavaScript Driver docs

---

## Version Compatibility

This reference targets **Neo4j 5.x**. Key version notes:

| Feature | Minimum Version | Notes |
|---------|----------------|-------|
| Range indexes (replacing B-tree) | 5.0 | B-tree indexes were renamed to Range indexes in Neo4j 5. Existing B-tree indexes auto-migrate. |
| CALL {} subqueries | 5.0 | Inline subqueries for complex pipelines and batching |
| Vector indexes | 5.11+ | Required for embedding similarity search |
| Quantified path patterns | 5.9+ | Advanced pattern matching |

If targeting Neo4j 4.x, replace `CREATE RANGE INDEX` with `CREATE INDEX` (B-tree is the 4.x default). Vector indexes are not available in 4.x.

---

## Core Concepts

### Property Graph Model

Neo4j uses a **property graph model**:
- **Nodes**: entities with labels (types) and properties (key-value pairs)
- **Relationships**: directed, typed connections between nodes with properties
- **Labels**: classify nodes (e.g., `:Entity`, `:Episode`, `:Community`)
- **Properties**: key-value data on both nodes and relationships

```
(:Entity {name: "Acme Corp", summary: "Technology company", embedding: [0.1, 0.2, ...]})
  -[:WORKS_FOR {fact: "Alice works for Acme Corp", valid_at: datetime("2024-01-15"), invalid_at: null}]->
(:Entity {name: "Alice", summary: "Senior engineer at Acme Corp"})
```

### Data Modeling Guidelines

**Properties vs. Nodes**
- Use **properties** for simple attributes that belong to a single entity (name, timestamp, score).
- Promote to a **node** when the value has its own relationships, is shared across entities, or needs independent querying.

**Relationship Direction Conventions**
- Choose a single canonical direction and be consistent (e.g., always `(person)-[:WORKS_FOR]->(company)`, never both directions).
- Direction is required in CREATE but optional in MATCH (`-[r]-` matches both directions).

**Multi-Label Patterns**
- A node can have multiple labels: `(:Entity:Active)`, `(:Entity:Archived)`.
- Use labels for broad categories, properties for fine-grained attributes.
- Keep label count per node small (2-3 max) to avoid index overhead.

**Supernode Anti-Pattern**
- Avoid nodes with extremely high relationship counts (100k+). They cause traversal bottlenecks.
- Mitigation strategies: fan-out nodes (intermediate relationship nodes), time-bucketed partitioning, or property-based filtering to limit traversal scope.

### Indexes

Neo4j supports several index types critical for knowledge graph performance:

| Index Type | Use Case | Cypher |
|-----------|----------|--------|
| **Range** | Exact match, range queries on properties | `CREATE RANGE INDEX FOR (n:Entity) ON (n.uuid)` |
| **Full-text** | BM25 text search on entity names/summaries | `CREATE FULLTEXT INDEX entity_name_idx FOR (n:Entity) ON EACH [n.name, n.summary]` |
| **Vector** | Cosine similarity on embeddings (5.11+) | `CREATE VECTOR INDEX entity_embedding_idx FOR (n:Entity) ON (n.embedding) OPTIONS {indexConfig: {`vector.dimensions`: 1024, `vector.similarity_function`: 'cosine'}}` |
| **Composite** | Multi-property lookups | `CREATE RANGE INDEX FOR (n:Entity) ON (n.group_id, n.name)` |
| **Relationship Range** | Property lookups on relationships | `CREATE RANGE INDEX FOR ()-[r:RELATES_TO]-() ON (r.uuid)` |

> **Compatibility note:** In Neo4j 4.x, these were called "B-tree" indexes and used `CREATE INDEX` syntax. Neo4j 5 renamed them to "Range" indexes. The old `CREATE INDEX` syntax still works but `CREATE RANGE INDEX` is preferred for clarity.

> **Relationship index note:** Queries like `()-[r {uuid: $uuid}]->()` perform a full relationship scan without a relationship index. Always create a relationship index on properties used for direct relationship lookups (see Relationship Range above).

### Constraints

Constraints enforce data integrity and **implicitly create indexes** (no need to create a separate index for constrained properties).

```cypher
// Uniqueness constraint — ensures no two Entity nodes share the same uuid
CREATE CONSTRAINT entity_uuid_unique FOR (e:Entity) REQUIRE e.uuid IS UNIQUE

// Existence constraint (Enterprise only) — ensures property is always present
CREATE CONSTRAINT entity_name_exists FOR (e:Entity) REQUIRE e.name IS NOT NULL

// Node key constraint (Enterprise only) — uniqueness + existence on property combo
CREATE CONSTRAINT entity_group_name_key FOR (e:Entity) REQUIRE (e.group_id, e.name) IS NODE KEY

// Relationship property existence (Enterprise only)
CREATE CONSTRAINT edge_fact_exists FOR ()-[r:RELATES_TO]-() REQUIRE r.fact IS NOT NULL
```

> **Aura note:** Neo4j Aura Professional supports uniqueness constraints. Existence and node key constraints require Aura Enterprise.

### Neo4j Aura (Cloud)

Neo4j Aura is the managed cloud offering. Key operational details:
- **Query API v2**: HTTPS endpoint for executing Cypher queries (used by Graphiti's HTTP client)
- **Authentication**: Basic auth with username + password
- **Connection**: URI format `neo4j+s://xxxxx.databases.neo4j.io`
- **Bolt protocol**: Binary protocol for driver connections (`bolt://` or `neo4j://`)

---

## Cypher Query Language

### Node Operations

```cypher
// Create a node
CREATE (e:Entity {uuid: $uuid, name: $name, summary: $summary, group_id: $group_id, created_at: datetime()})

// Find by property
MATCH (e:Entity {uuid: $uuid}) RETURN e

// Find by label and property
MATCH (e:Entity) WHERE e.group_id = $group_id RETURN e

// Update properties
MATCH (e:Entity {uuid: $uuid})
SET e.summary = $new_summary, e.updated_at = datetime()

// Delete (with relationships)
MATCH (e:Entity {uuid: $uuid}) DETACH DELETE e
```

### MERGE — Idempotent Create/Update

MERGE is critical for idempotent operations. It matches existing data or creates it if missing.

```cypher
// Create entity only if it doesn't exist, set properties on first create
MERGE (e:Entity {uuid: $uuid})
ON CREATE SET e.name = $name, e.summary = $summary, e.group_id = $group_id, e.created_at = datetime()
ON MATCH SET e.updated_at = datetime()

// Idempotent relationship creation
MATCH (source:Entity {uuid: $source_uuid})
MATCH (target:Entity {uuid: $target_uuid})
MERGE (source)-[r:RELATES_TO {uuid: $edge_uuid}]->(target)
ON CREATE SET r.fact = $fact, r.created_at = datetime(), r.valid_at = datetime($valid_at), r.invalid_at = null
ON MATCH SET r.fact = $fact, r.updated_at = datetime()

// MERGE with full pattern — useful for ensuring a node + relationship combo exists
MERGE (e:Entity {uuid: $uuid})
ON CREATE SET e.name = $name
MERGE (c:Community {uuid: $community_uuid})
MERGE (c)-[:HAS_MEMBER]->(e)
```

> **Warning:** Never MERGE on properties that may change — MERGE uses all specified properties for matching. MERGE on a stable identifier (uuid), then SET mutable properties in ON CREATE / ON MATCH.

### OPTIONAL MATCH

OPTIONAL MATCH works like MATCH but returns null instead of filtering out rows when no match is found (similar to SQL LEFT JOIN).

```cypher
// Find entity and optionally its community (returns entity even if no community)
MATCH (e:Entity {uuid: $uuid})
OPTIONAL MATCH (e)<-[:HAS_MEMBER]-(c:Community)
RETURN e.name, e.summary, c.name AS community_name

// Find entities with optional relationships
MATCH (e:Entity {group_id: $group_id})
OPTIONAL MATCH (e)-[r:RELATES_TO]->(other:Entity)
RETURN e.name, count(r) AS relationship_count

// Check if a relationship exists without filtering
MATCH (e:Entity {uuid: $uuid})
OPTIONAL MATCH (e)-[r:RELATES_TO {uuid: $edge_uuid}]->(target)
RETURN e, r IS NOT NULL AS relationship_exists
```

### WITH Clause — Query Pipeline Operator

WITH acts as a pipeline between query parts: it passes results forward, enables aggregation mid-query, and limits scope.

```cypher
// Aggregate then filter (HAVING equivalent)
MATCH (e:Entity {group_id: $group_id})-[r]-()
WITH e, count(r) AS degree
WHERE degree > 5
RETURN e.name, degree
ORDER BY degree DESC

// Chain multiple operations
MATCH (e:Entity {uuid: $uuid})
WITH e
MATCH (e)-[r:RELATES_TO]->(neighbor:Entity)
WITH e, neighbor, r
ORDER BY r.created_at DESC
LIMIT 10
RETURN e.name, collect(neighbor.name) AS recent_connections

// Introduce computed values mid-query
MATCH (e:Entity {group_id: $group_id})
WITH e, datetime() AS now
WHERE duration.between(e.created_at, now).days < 30
RETURN e.name, e.created_at
```

### CASE/WHEN Expressions

```cypher
// Conditional property values
MATCH (e:Entity {group_id: $group_id})-[r]-()
WITH e, count(r) AS degree
RETURN e.name, degree,
  CASE
    WHEN degree > 50 THEN 'hub'
    WHEN degree > 10 THEN 'connected'
    WHEN degree > 0 THEN 'sparse'
    ELSE 'isolated'
  END AS connectivity_tier

// Use in SET operations
MATCH (e:Entity {uuid: $uuid})
SET e.status = CASE
    WHEN e.updated_at IS NULL THEN 'stale'
    WHEN duration.between(e.updated_at, datetime()).days > 30 THEN 'stale'
    ELSE 'active'
  END
```

### CALL {} Subqueries (Neo4j 5+)

Subqueries enable complex compositions, batching, and correlated queries without Cartesian products.

```cypher
// Correlated subquery — runs per input row
MATCH (e:Entity {group_id: $group_id})
CALL (e) {
  MATCH (e)-[r:RELATES_TO]->(other:Entity)
  WHERE r.invalid_at IS NULL
  RETURN count(r) AS active_edges
}
RETURN e.name, active_edges
ORDER BY active_edges DESC

// Batch processing with CALL IN TRANSACTIONS (for large writes)
CALL {
  UNWIND $entities AS entity
  MERGE (e:Entity {uuid: entity.uuid})
  ON CREATE SET e.name = entity.name, e.group_id = entity.group_id, e.created_at = datetime()
} IN TRANSACTIONS OF 500 ROWS

// Union within subquery
MATCH (e:Entity {uuid: $uuid})
CALL (e) {
  MATCH (e)-[:RELATES_TO]->(out:Entity)
  RETURN out AS connected
  UNION
  MATCH (e)<-[:RELATES_TO]-(inc:Entity)
  RETURN inc AS connected
}
RETURN DISTINCT connected.name
```

### Relationship Operations

```cypher
// Create a relationship
MATCH (source:Entity {uuid: $source_uuid})
MATCH (target:Entity {uuid: $target_uuid})
CREATE (source)-[r:RELATES_TO {
  uuid: $edge_uuid,
  fact: $fact,
  relation_type: $relation_type,
  valid_at: datetime($valid_at),
  invalid_at: null,
  created_at: datetime(),
  embedding: $embedding
}]->(target)

// Find relationships between entity pair
MATCH (a:Entity {uuid: $entity_a})-[r]->(b:Entity {uuid: $entity_b})
RETURN r

// Invalidate an edge (temporal update)
// NOTE: Requires a relationship index on RELATES_TO(uuid) for performance.
// Without it, this scans all relationships in the database.
MATCH ()-[r:RELATES_TO {uuid: $edge_uuid}]->()
SET r.invalid_at = datetime($invalid_at), r.expired_at = datetime()
```

### UNWIND for Batch Operations

```cypher
// Batch node creation
UNWIND $entities AS entity
CREATE (e:Entity {
  uuid: entity.uuid,
  name: entity.name,
  summary: entity.summary,
  group_id: entity.group_id,
  embedding: entity.embedding,
  created_at: datetime()
})

// Batch relationship creation from a list of edges
UNWIND $edges AS edge
MATCH (source:Entity {uuid: edge.source_uuid})
MATCH (target:Entity {uuid: edge.target_uuid})
CREATE (source)-[r:RELATES_TO {
  uuid: edge.uuid,
  fact: edge.fact,
  valid_at: datetime(edge.valid_at),
  invalid_at: null,
  created_at: datetime()
}]->(target)

// Batch MERGE (idempotent upsert)
UNWIND $entities AS entity
MERGE (e:Entity {uuid: entity.uuid})
ON CREATE SET e.name = entity.name, e.group_id = entity.group_id, e.created_at = datetime()
ON MATCH SET e.name = entity.name, e.updated_at = datetime()
```

### Vector Search

```cypher
// Cosine similarity search on entity embeddings
CALL db.index.vector.queryNodes('entity_embedding_idx', $top_k, $query_embedding)
YIELD node, score
WHERE node.group_id = $group_id
RETURN node.uuid, node.name, node.summary, score
ORDER BY score DESC

// Vector search on edge embeddings
CALL db.index.vector.queryRelationships('edge_embedding_idx', $top_k, $query_embedding)
YIELD relationship, score
RETURN relationship.uuid, relationship.fact, relationship.valid_at, relationship.invalid_at, score
ORDER BY score DESC
```

### Full-Text Search (BM25)

```cypher
// Full-text search on entity names and summaries
CALL db.index.fulltext.queryNodes('entity_name_idx', $search_query)
YIELD node, score
WHERE node.group_id = $group_id
RETURN node.uuid, node.name, node.summary, score
ORDER BY score DESC
LIMIT $top_k

// Full-text search on edge facts
CALL db.index.fulltext.queryRelationships('edge_fact_idx', $search_query)
YIELD relationship, score
RETURN relationship.uuid, relationship.fact, score
ORDER BY score DESC
LIMIT $top_k
```

### Breadth-First Search (Graph Traversal)

```cypher
// Find entities within n hops of a seed entity
MATCH (seed:Entity {uuid: $seed_uuid})-[*1..3]-(neighbor:Entity)
WHERE neighbor.group_id = $group_id
RETURN DISTINCT neighbor.uuid, neighbor.name, neighbor.summary

// Find edges within n hops
MATCH (seed:Entity {uuid: $seed_uuid})-[*0..2]-(:Entity)-[r]->(:Entity)
WHERE r.invalid_at IS NULL  // only valid edges
RETURN DISTINCT r.uuid, r.fact, r.valid_at

// BFS from recent episodes to find related context
MATCH (ep:Episode {uuid: $episode_uuid})-[:MENTIONS]->(entity:Entity)-[r]->(related:Entity)
WHERE r.invalid_at IS NULL
RETURN entity.name, r.fact, related.name
```

### Community Queries

```cypher
// Get community members
MATCH (c:Community {uuid: $community_uuid})-[:HAS_MEMBER]->(e:Entity)
RETURN e.uuid, e.name, e.summary

// Find community for an entity
MATCH (e:Entity {uuid: $entity_uuid})<-[:HAS_MEMBER]-(c:Community)
RETURN c.uuid, c.name, c.summary

// Update community summary
MATCH (c:Community {uuid: $community_uuid})
SET c.summary = $new_summary, c.name = $new_name, c.embedding = $new_embedding

// Label propagation for community assignment
// MATCH the existing new entity, find its best community, then link them
MATCH (new:Entity {uuid: $new_uuid})-[]-(neighbor:Entity)<-[:HAS_MEMBER]-(c:Community)
WITH c, new, count(neighbor) AS member_count
ORDER BY member_count DESC
LIMIT 1
CREATE (c)-[:HAS_MEMBER]->(new)
```

### Temporal Queries

```cypher
// Find facts valid at a specific point in time
MATCH (a:Entity)-[r]->(b:Entity)
WHERE r.valid_at <= datetime($point_in_time)
  AND (r.invalid_at IS NULL OR r.invalid_at > datetime($point_in_time))
  AND a.group_id = $group_id
RETURN a.name, r.fact, b.name, r.valid_at, r.invalid_at

// Find all historical states of a relationship between two entities
MATCH (a:Entity {name: $entity_a})-[r]->(b:Entity {name: $entity_b})
RETURN r.fact, r.valid_at, r.invalid_at, r.created_at
ORDER BY r.valid_at

// Find recently invalidated facts (knowledge updates)
MATCH ()-[r]->()
WHERE r.invalid_at IS NOT NULL
  AND r.invalid_at > datetime($since)
RETURN r.fact, r.valid_at, r.invalid_at
ORDER BY r.invalid_at DESC
```

### Aggregation and Analytics

```cypher
// Graph health metrics — use separate count{} subqueries to avoid Cartesian products
MATCH (e:Entity) WHERE e.group_id = $group_id
WITH count(e) AS entity_count
CALL {
  MATCH ()-[r:RELATES_TO]->() WHERE r.group_id = $group_id
  RETURN count(r) AS edge_count
}
CALL {
  MATCH (c:Community) WHERE c.group_id = $group_id
  RETURN count(c) AS community_count
}
RETURN entity_count, edge_count, community_count

// Alternative: separate queries (simpler, avoids any Cartesian risk)
// Query 1: MATCH (e:Entity {group_id: $group_id}) RETURN count(e) AS entity_count
// Query 2: MATCH ()-[r:RELATES_TO {group_id: $group_id}]->() RETURN count(r) AS edge_count
// Query 3: MATCH (c:Community {group_id: $group_id}) RETURN count(c) AS community_count

// Most connected entities
MATCH (e:Entity {group_id: $group_id})-[r]-()
RETURN e.name, count(r) AS degree
ORDER BY degree DESC
LIMIT 20

// Entity mention frequency across episodes
MATCH (ep:Episode)-[:MENTIONS]->(e:Entity {group_id: $group_id})
RETURN e.name, count(ep) AS mention_count
ORDER BY mention_count DESC
```

### Administration Queries

```cypher
// List all indexes
SHOW INDEXES

// List all constraints
SHOW CONSTRAINTS

// Drop an index by name
DROP INDEX entity_embedding_idx IF EXISTS

// Drop a constraint by name
DROP CONSTRAINT entity_uuid_unique IF EXISTS

// Show databases (multi-database environments)
SHOW DATABASES

// Show current database
SHOW DATABASE $current
```

### Data Import with LOAD CSV

For bulk data import from CSV files:

```cypher
// Import entities from CSV
LOAD CSV WITH HEADERS FROM 'file:///entities.csv' AS row
CREATE (e:Entity {
  uuid: row.uuid,
  name: row.name,
  summary: row.summary,
  group_id: row.group_id,
  created_at: datetime()
})

// Import with MERGE for idempotency
LOAD CSV WITH HEADERS FROM 'file:///entities.csv' AS row
MERGE (e:Entity {uuid: row.uuid})
ON CREATE SET e.name = row.name, e.summary = row.summary
```

> **Note:** LOAD CSV reads from the `import` directory by default. For Aura, use HTTPS URLs. For large files (100k+ rows), use `CALL {} IN TRANSACTIONS` batching.

---

## Neo4j HTTP Query API (Aura)

Graphiti's TypeScript client uses the Neo4j HTTP Query API v2 for direct HTTPS calls.

### Request Format

```
POST https://<neo4j-uri>/db/<database>/query/v2
Content-Type: application/json
Authorization: Basic <base64(username:password)>

{
  "statement": "MATCH (n:Entity {group_id: $group_id}) RETURN n LIMIT $limit",
  "parameters": {
    "group_id": "workspace_123",
    "limit": 10
  }
}
```

### Response Format

```json
{
  "data": {
    "fields": ["n"],
    "values": [
      [{"labels": ["Entity"], "properties": {"name": "Acme", "uuid": "..."}}]
    ]
  }
}
```

### Connection Best Practices
- Set connection timeout to 15s default, up to 60s for heavy queries
- Use connection pooling for concurrent requests
- Implement retry logic with exponential backoff for rate limits and 5xx errors
- Use parameterized queries to prevent injection and improve plan caching

---

## JavaScript/TypeScript Driver

### Installation
```bash
npm install neo4j-driver
```

### Connection Pooling Configuration

```typescript
import neo4j from 'neo4j-driver';

const driver = neo4j.driver(
  'neo4j+s://xxxxx.databases.neo4j.io',
  neo4j.auth.basic('neo4j', 'password'),
  {
    maxConnectionPoolSize: 100,       // max concurrent connections (default: 100)
    connectionAcquisitionTimeout: 60000, // ms to wait for a connection from pool (default: 60s)
    connectionTimeout: 30000,         // ms for initial connection establishment (default: 30s)
    maxTransactionRetryTime: 30000,   // ms for transaction function retries (default: 30s)
    logging: {
      level: 'warn',                  // 'error' | 'warn' | 'info' | 'debug'
      logger: (level, message) => console.log(`[neo4j-${level}]`, message),
    },
  }
);

// Verify connectivity at startup
await driver.verifyConnectivity();
```

### Integer Handling

The Neo4j JavaScript driver uses custom integer types because JavaScript numbers lose precision above 2^53. Use `neo4j.int()` for integer parameters.

```typescript
// WRONG — JavaScript number may lose precision for large integers
session.run('MATCH (e) RETURN e LIMIT $limit', { limit: 10 });

// CORRECT — use neo4j.int() for integer parameters
session.run('MATCH (e) RETURN e LIMIT $limit', { limit: neo4j.int(10) });

// Reading integers from results
const count = record.get('count');
// count is a Neo4j Integer object, not a JS number
const jsNumber = count.toNumber(); // safe if value < 2^53
const jsString = count.toString(); // always safe
```

> **Note:** For small values (< 2^53), you can also pass plain JavaScript numbers and the driver will handle conversion. But `neo4j.int()` is always safe and recommended for clarity.

### Basic Usage
```typescript
import neo4j from 'neo4j-driver';

const driver = neo4j.driver(
  'neo4j+s://xxxxx.databases.neo4j.io',
  neo4j.auth.basic('neo4j', 'password')
);

const session = driver.session({ database: 'neo4j' });

try {
  const result = await session.run(
    'MATCH (e:Entity {group_id: $groupId}) RETURN e.name, e.summary LIMIT $limit',
    { groupId: 'workspace_123', limit: neo4j.int(10) }
  );

  for (const record of result.records) {
    console.log(record.get('e.name'), record.get('e.summary'));
  }
} finally {
  await session.close();
}

await driver.close();
```

### Transaction Functions (Recommended)
```typescript
// Read transaction with automatic retry
const entities = await session.executeRead(async tx => {
  const result = await tx.run(
    'MATCH (e:Entity {group_id: $groupId}) RETURN e',
    { groupId }
  );
  return result.records.map(r => r.get('e').properties);
});

// Write transaction with automatic retry
await session.executeWrite(async tx => {
  await tx.run(
    'CREATE (e:Entity {uuid: $uuid, name: $name, group_id: $groupId})',
    { uuid, name, groupId }
  );
});
```

### Transaction Management

```typescript
// Explicit transaction with timeout
const session = driver.session({
  database: 'neo4j',
  defaultAccessMode: neo4j.session.WRITE,
});

const tx = session.beginTransaction({
  timeout: neo4j.int(30000), // 30s transaction timeout
});

try {
  await tx.run('CREATE (e:Entity {uuid: $uuid, name: $name})', { uuid, name });
  await tx.run('MATCH (a:Entity {uuid: $a}), (b:Entity {uuid: $b}) CREATE (a)-[:RELATES_TO]->(b)', { a: uuid, b: targetUuid });
  await tx.commit();
} catch (error) {
  await tx.rollback();
  throw error;
} finally {
  await session.close();
}

// Bookmark management for causal consistency
// (ensures reads see writes from previous transactions)
let savedBookmarks: string[] = [];

const writeSession = driver.session({ database: 'neo4j' });
try {
  await writeSession.executeWrite(async tx => {
    await tx.run('CREATE (e:Entity {uuid: $uuid, name: $name})', { uuid, name });
  });
  savedBookmarks = writeSession.lastBookmarks();
} finally {
  await writeSession.close();
}

// Read session uses bookmarks to guarantee it sees the write above
const readSession = driver.session({
  database: 'neo4j',
  bookmarks: savedBookmarks,
});
try {
  const result = await readSession.executeRead(async tx => {
    return tx.run('MATCH (e:Entity {uuid: $uuid}) RETURN e', { uuid });
  });
} finally {
  await readSession.close();
}
```

### Error Handling

```typescript
import neo4j, { Neo4jError } from 'neo4j-driver';

try {
  await session.executeWrite(async tx => {
    await tx.run('CREATE (e:Entity {uuid: $uuid})', { uuid });
  });
} catch (error) {
  if (error instanceof Neo4jError) {
    // Classification-based handling
    if (error.code === 'Neo.ClientError.Schema.ConstraintValidationFailed') {
      // Duplicate uuid — entity already exists, safe to handle as upsert
      console.warn('Entity already exists, skipping:', uuid);
    } else if (error.retriable) {
      // Transient error (deadlock, leader switch, etc.) — retry is safe
      // Note: executeRead/executeWrite auto-retry transient errors
      console.warn('Transient Neo4j error, will retry:', error.message);
    } else if (error.code.startsWith('Neo.ClientError')) {
      // Permanent client error — query syntax, bad parameters, etc.
      // Do NOT retry — fix the query
      throw error;
    } else if (error.code.startsWith('Neo.DatabaseError')) {
      // Server-side error — may need admin attention
      throw error;
    } else {
      throw error;
    }
  } else {
    // Non-Neo4j error (network, driver issue)
    throw error;
  }
}
```

Common error codes:
| Code | Type | Action |
|------|------|--------|
| `Neo.ClientError.Schema.ConstraintValidationFailed` | Permanent | Handle as duplicate, use MERGE instead |
| `Neo.ClientError.Statement.SyntaxError` | Permanent | Fix query |
| `Neo.ClientError.Statement.ParameterMissing` | Permanent | Fix parameters |
| `Neo.TransientError.Transaction.Terminated` | Transient | Auto-retried by transaction functions |
| `Neo.TransientError.Transaction.DeadlockDetected` | Transient | Auto-retried by transaction functions |
| `Neo.DatabaseError.General.UnknownError` | Server | Log and alert |

---

## APOC Procedures

APOC (Awesome Procedures on Cypher) is a standard library of useful procedures. Many are available on Aura, but some are restricted.

### Available on Aura

```cypher
// Batch processing with apoc.periodic.iterate (large-scale updates)
CALL apoc.periodic.iterate(
  'MATCH (e:Entity) WHERE e.group_id = $groupId RETURN e',
  'SET e.processed = true',
  {batchSize: 1000, params: {groupId: $group_id}}
)

// Convert node/relationship to map
MATCH (e:Entity {uuid: $uuid})
RETURN apoc.map.fromNode(e) AS entity_map

// Collection utilities
WITH ['a', 'b', 'c', 'a'] AS items
RETURN apoc.coll.toSet(items) AS unique_items

// Text utilities
RETURN apoc.text.join(['hello', 'world'], ' ') AS joined

// Date/time utilities
RETURN apoc.date.format(timestamp(), 'ms', 'yyyy-MM-dd') AS formatted_date
```

### Commonly Used APOC Procedures

| Procedure | Use Case |
|-----------|----------|
| `apoc.periodic.iterate` | Batch processing large datasets |
| `apoc.merge.node` | Dynamic label/property merge |
| `apoc.create.relationship` | Dynamic relationship type creation |
| `apoc.map.fromNode` | Convert node to property map |
| `apoc.coll.toSet` | Deduplicate collections |
| `apoc.text.join` | String concatenation |
| `apoc.path.expandConfig` | Configurable graph traversal |
| `apoc.refactor.mergeNodes` | Merge duplicate nodes |

> **Aura limitations:** `apoc.load.json`, `apoc.load.jdbc`, file system procedures, and schema-altering procedures are not available on Aura. Check [APOC Aura compatibility](https://neo4j.com/docs/apoc/current/overview/) for the full list.

---

## Query Analysis with EXPLAIN / PROFILE

### EXPLAIN — Shows Plan Without Executing

```cypher
EXPLAIN
MATCH (e:Entity {group_id: $group_id})-[r:RELATES_TO]->(other:Entity)
WHERE r.invalid_at IS NULL
RETURN e.name, r.fact, other.name
```

### PROFILE — Executes and Shows Actual Metrics

```cypher
PROFILE
MATCH (e:Entity {group_id: $group_id})-[r:RELATES_TO]->(other:Entity)
WHERE r.invalid_at IS NULL
RETURN e.name, r.fact, other.name
```

### Interpreting Output

Key metrics per operator:
- **Rows**: number of rows produced by this operator
- **DbHits**: number of database operations (property lookups, index seeks, etc.) — the primary cost metric
- **Estimated Rows**: planner's estimate (compare with actual to detect bad estimates)

Key operators to look for:
| Operator | Meaning | Performance |
|----------|---------|-------------|
| `NodeIndexSeek` | Using an index to find nodes | Good |
| `NodeByLabelScan` | Scanning all nodes with a label | Bad for large sets |
| `AllNodesScan` | Scanning every node in the database | Very bad |
| `RelationshipTypeScan` | Scanning all rels of a type | Bad for large sets |
| `CartesianProduct` | Cross-join of two unconnected patterns | Usually a bug — see below |
| `Filter` | Post-scan filtering | OK but check if an index could eliminate it |
| `Expand(All)` | Traversing relationships | Normal for graph queries |

### Detecting Cartesian Products

A **CartesianProduct** in the query plan means two MATCH clauses have no connection, causing every row from one to be paired with every row from the other (N x M explosion).

```cypher
// BAD — Cartesian product between entities and communities
MATCH (e:Entity) WHERE e.group_id = $group_id
MATCH (c:Community) WHERE c.group_id = $group_id
RETURN count(e), count(c)

// GOOD — use CALL {} subqueries or separate queries (see Aggregation section)
```

---

## Multi-Database and Multi-Tenancy

### Property-Based Isolation (Recommended for Speqq)

All nodes and relationships include a `group_id` property for tenant isolation. Simpler to manage, works on all Aura tiers.

```cypher
// Always filter by group_id
MATCH (e:Entity {group_id: $group_id})-[r]->(other:Entity)
WHERE r.group_id = $group_id
RETURN e, r, other

// Create composite index for performance
CREATE RANGE INDEX FOR (n:Entity) ON (n.group_id, n.uuid)
```

**Pros:** Single database, simpler operations, cross-tenant queries possible if needed.
**Cons:** Must remember group_id filter on every query (risk of data leak if forgotten), shared index space.

### Database-Based Isolation (Enterprise)

Each tenant gets its own database. Requires Neo4j Enterprise or Aura Enterprise.

```typescript
// Switch database per tenant
const session = driver.session({ database: tenantDatabaseName });
```

**Pros:** Strong isolation, independent scaling, no risk of cross-tenant data leak.
**Cons:** Higher operational overhead, requires Enterprise tier, cross-tenant queries need federation.

---

## Performance Considerations

1. **Always use parameterized queries** — avoids query plan cache misses and injection risks.
2. **Create indexes before bulk ingestion** — vector, fulltext, and range indexes must exist first.
3. **Use UNWIND for batch operations** — process arrays of data in single queries.
4. **Limit traversal depth** — BFS queries should cap at 2-3 hops to prevent explosion.
5. **Filter early in queries** — use WHERE clauses close to MATCH patterns, especially group_id filtering.
6. **Monitor query plans** — use `EXPLAIN` and `PROFILE` to identify slow queries and Cartesian products.
7. **Create relationship indexes** — any direct relationship property lookup (e.g., `{uuid: $uuid}`) needs a relationship index.
8. **Use MERGE for idempotency** — prevents duplicate nodes/relationships on retry or concurrent writes.
9. **Avoid supernodes** — entities with 100k+ relationships degrade traversal. Use fan-out patterns.
10. **Use `neo4j.int()`** — wrap integer parameters in the JavaScript driver to prevent precision loss.
