# Ontology Design for Context Graphs

This reference covers how to design effective ontologies (node types, edge types, and their relationships) for temporal knowledge graphs powering AI agent memory.

---

## What Is a Graph Ontology?

An ontology defines the **types of entities** (node types) and **types of relationships** (edge types) that the knowledge graph can contain. It acts as a schema that guides LLM-based extraction — telling the extraction model what kinds of things to look for and how to categorize them.

A well-designed ontology:
- Improves extraction precision (fewer hallucinated entities)
- Reduces entity resolution errors (clear type boundaries)
- Makes retrieval more predictable (known structure to query)
- Enables domain-specific reasoning (types carry semantic meaning)

A poorly designed ontology:
- Over-extracts (too many types = noise)
- Under-extracts (too few types = missed information)
- Creates ambiguity (overlapping types = inconsistent categorization)

---

## Node Type Design

### Principles

1. **Non-overlapping**: Each real-world entity should clearly belong to exactly one type. Avoid types that could both claim the same entity.
2. **Descriptive**: Type descriptions guide the LLM extractor. Be specific about what qualifies.
3. **Prioritized**: Mark certain types as high-priority for extraction. These get preference when ambiguous.
4. **Fallback types**: Include 1-2 broad fallback types (e.g., Topic, Object) for entities that don't fit specific types.
5. **Domain-scoped**: Types should reflect your domain, not be generic. A product planning tool needs different types than a medical system.

### Graphiti Built-in Entity Types

| Type | Description | Priority |
|------|-------------|----------|
| Preference | User preferences, choices, opinions, selections | High (user-specific) |
| Requirement | Specific needs, features, functionality to be fulfilled | High |
| Procedure | Standard operating procedures, sequential instructions | Normal |
| Location | Physical or virtual places | Normal |
| Event | Time-bound activities, occurrences, experiences | Normal |
| Organization | Companies, institutions, groups, formal entities | Normal |
| Document | Information content (books, articles, reports, videos) | Normal |
| Topic | Subject of conversation, interest, or knowledge domain | Fallback |
| Object | Physical items, tools, devices, possessions | Fallback |

### Custom Ontology Example: Product Planning Domain

For a product like Speqq, a domain-specific ontology might include:

| Type | Description | Examples |
|------|-------------|----------|
| **Product** | A software product, platform, or named system | "Speqq", "Notion", "Linear" |
| **Actor** | A person, role, or user persona that interacts with systems | "Product Manager", "Alice Chen" |
| **Concept** | An abstract idea, methodology, or domain term | "Agile", "PRD", "Knowledge Graph" |
| **Surface** | A UI surface, page, or view within a product | "Workspace Editor", "Sidebar", "Settings" |
| **Feature** | A specific capability or behavior within a product | "Real-time collaboration", "AI chat" |
| **Flow** | A user workflow, journey, or sequence of actions | "Onboarding flow", "PRD creation" |
| **Rule** | A business rule, constraint, or policy | "Max 5 workspaces per user" |
| **Integration** | An external service, API, or connected system | "GitHub sync", "Stripe billing" |
| **Strategy** | A business or product strategy, objective, or goal | "Beta launch to 5k users" |

### Property Constraints per Node Type

Every node type should define **required** and **optional** properties, value types, and allowed enums. Properties serve two purposes: they store structured data on extracted entities, and they guide the extraction prompt so the LLM knows what fields to populate.

#### Required vs Optional Properties

- **Required properties** must be present on every instance. Extraction that fails to produce them should be flagged for review.
- **Optional properties** are populated when the source text contains the information. They default to `null` when absent.

#### Value Types and Enums

Constrain property values to specific types (`str`, `int`, `float`, `bool`, `datetime`, `list[str]`) and use enums for fields with a known set of valid values. This prevents drift during extraction and makes downstream queries predictable.

#### Property-to-Prompt Relationship

Each property's `Field(description=...)` text is injected into the extraction prompt. Write descriptions as if instructing the LLM: "Extract the priority level from the text. If not mentioned, leave null."

#### Realistic Multi-Field Models (Product Planning)

```python
from datetime import datetime
from enum import Enum
from typing import Optional
from pydantic import BaseModel, Field, field_validator


class ProductStage(str, Enum):
    IDEA = "idea"
    DEVELOPMENT = "development"
    BETA = "beta"
    GA = "ga"
    DEPRECATED = "deprecated"


class Product(BaseModel):
    """A software product, platform, or named system."""
    name: str = Field(
        description="Canonical product name. Use official casing (e.g., 'Speqq' not 'speqq')."
    )
    stage: Optional[ProductStage] = Field(
        default=None,
        description="Current lifecycle stage. Extract from context clues like 'launching beta' or 'sunsetted'."
    )
    domain: Optional[str] = Field(
        default=None,
        description="Primary domain or category (e.g., 'product planning', 'project management')."
    )
    url: Optional[str] = Field(
        default=None,
        description="Primary URL or website if mentioned."
    )

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Product name cannot be empty")
        return v.strip()


class ActorType(str, Enum):
    PERSON = "person"
    ROLE = "role"
    PERSONA = "persona"
    TEAM = "team"


class Actor(BaseModel):
    """A person, role, or user persona that interacts with systems."""
    name: str = Field(
        description="Full name for people ('Alice Chen'), title for roles ('Product Manager'), "
        "label for personas ('Power User')."
    )
    actor_type: ActorType = Field(
        description="Whether this is a specific person, a role title, a user persona, or a team."
    )
    organization: Optional[str] = Field(
        default=None,
        description="Organization this actor belongs to, if mentioned."
    )
    email: Optional[str] = Field(
        default=None,
        description="Email address if explicitly mentioned in text."
    )

    @field_validator("name")
    @classmethod
    def normalize_name(cls, v: str) -> str:
        return v.strip().title() if v else v


class FeatureStatus(str, Enum):
    PLANNED = "planned"
    IN_PROGRESS = "in_progress"
    SHIPPED = "shipped"
    DEPRECATED = "deprecated"


class FeaturePriority(str, Enum):
    CRITICAL = "critical"
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"


class Feature(BaseModel):
    """A specific capability or behavior within a product."""
    name: str = Field(
        description="Short descriptive name for the feature (e.g., 'Real-time collaboration')."
    )
    status: Optional[FeatureStatus] = Field(
        default=None,
        description="Current implementation status. Infer from phrases like 'we shipped', 'planned for Q3'."
    )
    priority: Optional[FeaturePriority] = Field(
        default=None,
        description="Priority level if stated or clearly implied."
    )
    product: Optional[str] = Field(
        default=None,
        description="Name of the product this feature belongs to."
    )
    description: Optional[str] = Field(
        default=None,
        description="Brief description of what the feature does, extracted from context."
    )


class Surface(BaseModel):
    """A UI surface, page, or view within a product."""
    name: str = Field(
        description="Surface name as referred to in the product (e.g., 'Workspace Editor')."
    )
    product: Optional[str] = Field(
        default=None,
        description="Product this surface belongs to."
    )
    surface_type: Optional[str] = Field(
        default=None,
        description="Kind of surface: 'page', 'panel', 'modal', 'drawer', 'tab'."
    )


class Rule(BaseModel):
    """A business rule, constraint, or policy."""
    name: str = Field(
        description="Short label for the rule (e.g., 'Workspace limit')."
    )
    constraint_text: Optional[str] = Field(
        default=None,
        description="The full rule statement (e.g., 'Max 5 workspaces per free-tier user')."
    )
    scope: Optional[str] = Field(
        default=None,
        description="What the rule applies to: a product, a plan tier, a user role."
    )


class Strategy(BaseModel):
    """A business or product strategy, objective, or goal."""
    name: str = Field(
        description="Strategy label (e.g., 'Beta launch to 5k users')."
    )
    target_date: Optional[str] = Field(
        default=None,
        description="Target date or quarter if mentioned (e.g., 'Q2 2026')."
    )
    metric: Optional[str] = Field(
        default=None,
        description="Key metric or KPI associated with this strategy."
    )


ENTITY_TYPES = [Product, Actor, Feature, Surface, Rule, Strategy]
```

### Integrating Models with Graphiti's Extraction Pipeline

Graphiti uses your Pydantic models in two ways:

1. **Schema injection**: The model's JSON schema (derived from `model_json_schema()`) is included in the extraction prompt. Field names, types, descriptions, and enum values all become part of the LLM instruction.
2. **Response validation**: The LLM's structured output is parsed through the Pydantic model. Validation errors trigger re-extraction or fallback handling.

```python
from graphiti_core import Graphiti
from graphiti_core.utils.ontology import OntologyConfig

ontology = OntologyConfig(
    entity_types=ENTITY_TYPES,
    # Edge types discussed in the next section
    edge_types=EDGE_TYPE_MAP,
)

graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    ontology=ontology,
)
```

### Design Checklist

- [ ] Can every expected entity in your domain be classified into exactly one type?
- [ ] Are descriptions specific enough for an LLM to make consistent classification decisions?
- [ ] Do you have fallback types for unexpected entities?
- [ ] Are high-priority types marked (e.g., user preferences for personalization)?
- [ ] Is the type count manageable (5-15 types)? Too many types increase extraction latency and confusion.
- [ ] Does every node type define required vs optional properties with value types?
- [ ] Are enum fields used where a property has a known set of valid values?
- [ ] Do field descriptions read as extraction instructions for the LLM?

---

## Type Hierarchy

### Subtypes and Abstract vs Concrete Types

In many domains, entity types naturally form hierarchies. For example, `Actor` might have subtypes `Person`, `Role`, and `Team`. There are two approaches to modeling hierarchy in Graphiti:

#### Approach 1: Flat Types with Discriminator Properties (Recommended)

Keep the node type flat and use an enum property to distinguish subtypes.

```python
class ActorType(str, Enum):
    PERSON = "person"
    ROLE = "role"
    PERSONA = "persona"
    TEAM = "team"

class Actor(BaseModel):
    """A person, role, or user persona that interacts with systems."""
    name: str
    actor_type: ActorType
```

**Advantages**: Simpler ontology, fewer types for the LLM to choose between, entity resolution operates on one type so "Alice Chen" (person) and "Alice" (person) can merge naturally.

#### Approach 2: Separate Concrete Types

Define `Person`, `Role`, and `Team` as independent node types.

```python
class Person(BaseModel):
    """A specific named individual."""
    name: str
    email: Optional[str] = None

class Role(BaseModel):
    """A job title or functional role (not a specific person)."""
    name: str
    department: Optional[str] = None
```

**Advantages**: Distinct property schemas per subtype, stricter domain/range constraints on edges. **Disadvantages**: The LLM must choose between more types, and entity resolution cannot merge across types — "Alice Chen" (Person) and "the PM" (Role) stay separate even when they refer to the same individual.

#### How Hierarchy Affects Entity Resolution

Entity resolution in Graphiti matches by name similarity within the same node type. This means:
- With flat types + discriminator: all actors are compared against each other, improving merge quality.
- With separate types: a `Person` named "Alice" and a `Role` named "Product Manager" are never compared, even if they refer to the same individual. Cross-type relationships must be captured via edges instead.

**Guideline**: Use flat types with discriminator properties (Approach 1) unless your domain requires fundamentally different property schemas per subtype and you accept the entity resolution trade-off.

---

## Edge Type Design

### Principles

1. **ALL_CAPS naming**: Edge types use concise uppercase labels (e.g., WORKS_FOR, DEPENDS_ON).
2. **Verb-based**: Edge types represent actions or relationships, not nouns.
3. **Directional clarity**: The direction matters — (A)-[MANAGES]->(B) means A manages B.
4. **Bidirectional mapping**: Define the reverse label for each edge type to handle extraction order independence. If the LLM extracts (B)-[???]->(A), the system needs to know that's equivalent to (A)-[MANAGES]->(B).
5. **Fact-carrying**: Edges carry a detailed `fact` text field beyond just the type label. The type is for categorization; the fact is for content.

### Common Edge Types for Agent Memory

| Edge Type | Meaning | Reverse |
|-----------|---------|---------|
| SERVES | An actor serves/is responsible for something | SERVED_BY |
| GOVERNS | A rule constrains or governs something | GOVERNED_BY |
| LIVES_ON | A feature/component exists within a surface | CONTAINS |
| DEPENDS_ON | One entity requires another | REQUIRED_BY |
| USES | An actor or system uses a tool/feature | USED_BY |
| CONTAINS | A container holds something | PART_OF |
| CONNECTS_TO | Integration/connection between systems | CONNECTED_FROM |
| PRODUCES | An actor/process creates an output | PRODUCED_BY |
| CONSTRAINS | One entity limits another | CONSTRAINED_BY |
| IMPLEMENTS | One entity realizes/implements another | IMPLEMENTED_BY |
| PREFERS | An actor has a preference | PREFERRED_BY |
| WORKS_FOR | Employment/role relationship | EMPLOYS |
| IS_FRIENDS_WITH | Social relationship (symmetric) | IS_FRIENDS_WITH |
| KNOWS_ABOUT | Knowledge/expertise relationship | KNOWN_BY |
| OWNS | Ownership relationship | OWNED_BY |
| TARGETS | A strategy targets a metric or audience | TARGETED_BY |
| PRIORITIZES | An actor or strategy prioritizes something | PRIORITIZED_BY |

### Bidirectional Edge Mapping

When the LLM extracts entities in either order, the system must produce consistent edge directions:

```python
EDGE_BIDIRECTIONAL_MAP = {
    "SERVES": "SERVED_BY",
    "SERVED_BY": "SERVES",
    "GOVERNS": "GOVERNED_BY",
    "GOVERNED_BY": "GOVERNS",
    "LIVES_ON": "CONTAINS",
    "CONTAINS": "LIVES_ON",
    "DEPENDS_ON": "REQUIRED_BY",
    "REQUIRED_BY": "DEPENDS_ON",
    "OWNS": "OWNED_BY",
    "OWNED_BY": "OWNS",
    "TARGETS": "TARGETED_BY",
    "TARGETED_BY": "TARGETS",
    # ... etc
}
```

If the LLM extracts (B)-[SERVED_BY]->(A), the system flips it to (A)-[SERVES]->(B). This ensures consistent graph structure regardless of extraction order.

### The Fact Field

Every edge carries a `fact` field — a natural language sentence that captures the specific relationship detail beyond what the type label alone conveys.

#### What the Fact Is

The fact is a self-contained, human-readable sentence. For example, for an edge of type `DEPENDS_ON` between "AI Chat" and "OpenAI API", the fact might be: "The AI Chat feature depends on the OpenAI API for streaming completions and tool-use responses."

#### How Facts Are Generated During Extraction

When Graphiti processes an episode (a text chunk), the LLM extraction prompt asks the model to:
1. Identify entities (nodes).
2. Identify relationships between entities (edges).
3. For each relationship, generate a `fact` sentence that captures the specific detail from the source text.

The LLM does not simply label the relationship — it writes a descriptive sentence grounded in the source content.

#### How Facts Are Used in Retrieval

During search, Graphiti can return facts as context. The fact text is embedded for semantic similarity search, meaning searches match against the descriptive content of relationships, not just their type labels. This makes retrieval much richer than a traditional triple store.

#### Fact Granularity Best Practices

- **One fact per relationship aspect**: If a single source passage describes multiple aspects of the same relationship, extract separate edges with separate facts rather than one edge with a compound fact.
- **Be specific**: "Speqq uses Supabase for Postgres database hosting and auth" is better than "Speqq uses Supabase."
- **Include context**: "As of Q1 2026, the onboarding flow requires email verification" is better than "Onboarding requires email verification" because the temporal context aids retrieval.
- **Avoid redundancy with the type**: The fact should add information beyond what the type label already says. "Feature X depends on Feature Y" adds nothing to a `DEPENDS_ON` edge.

---

## Domain/Range Constraints for Edge Types

Domain/range constraints specify which source (domain) and target (range) node type pairs are valid for each edge type. These constraints prevent nonsensical relationships like a `Rule` node having a `WORKS_FOR` edge to a `Feature` node.

### Constraint Table (Product Planning Example)

| Edge Type | Valid Source (Domain) | Valid Target (Range) |
|-----------|----------------------|----------------------|
| SERVES | Actor | Product, Feature, Strategy |
| GOVERNS | Rule | Feature, Surface, Flow, Product |
| LIVES_ON | Feature | Surface |
| DEPENDS_ON | Feature, Integration | Feature, Integration |
| USES | Actor | Feature, Product, Integration |
| CONTAINS | Product, Surface | Feature, Surface |
| CONNECTS_TO | Integration | Product, Integration |
| PRODUCES | Actor, Flow | Feature, Strategy |
| CONSTRAINS | Rule | Feature, Product, Flow |
| IMPLEMENTS | Feature | Strategy, Rule |
| PREFERS | Actor | Feature, Product, Concept |
| WORKS_FOR | Actor | Actor (org/team) |
| OWNS | Actor | Product, Feature, Strategy |
| TARGETS | Strategy | Actor, Concept |

### Relating to Graphiti's edge_type_map Parameter

Graphiti accepts an `edge_type_map` parameter that maps (source_type, target_type) pairs to allowed edge types. This is the mechanism for enforcing domain/range constraints at extraction time.

```python
EDGE_TYPE_MAP = {
    ("Actor", "Product"): ["SERVES", "USES", "OWNS"],
    ("Actor", "Feature"): ["SERVES", "USES", "OWNS", "PREFERS"],
    ("Actor", "Strategy"): ["SERVES", "OWNS"],
    ("Actor", "Actor"): ["WORKS_FOR"],
    ("Rule", "Feature"): ["GOVERNS", "CONSTRAINS"],
    ("Rule", "Surface"): ["GOVERNS"],
    ("Rule", "Flow"): ["GOVERNS", "CONSTRAINS"],
    ("Rule", "Product"): ["GOVERNS", "CONSTRAINS"],
    ("Feature", "Surface"): ["LIVES_ON"],
    ("Feature", "Feature"): ["DEPENDS_ON"],
    ("Feature", "Integration"): ["DEPENDS_ON"],
    ("Feature", "Strategy"): ["IMPLEMENTS"],
    ("Integration", "Product"): ["CONNECTS_TO"],
    ("Integration", "Integration"): ["CONNECTS_TO", "DEPENDS_ON"],
    ("Strategy", "Actor"): ["TARGETS"],
    ("Strategy", "Concept"): ["TARGETS"],
    ("Product", "Feature"): ["CONTAINS"],
    ("Surface", "Feature"): ["CONTAINS"],
    ("Surface", "Surface"): ["CONTAINS"],
}
```

When the LLM proposes an edge whose source/target pair is not in this map, Graphiti can either reject it or fall back to unconstrained extraction depending on configuration. See the section on constraining vs leaving open below.

---

## Cardinality Constraints

Cardinality constraints define how many relationships of a given type a node can participate in. While Graphiti does not enforce cardinality at the database level, designing with cardinality in mind improves extraction quality and downstream querying.

### One-to-Many vs Many-to-Many

| Relationship Pattern | Example | Cardinality |
|----------------------|---------|-------------|
| A product has one primary owner | Product -[OWNED_BY]-> Actor | Many-to-one (each product has one owner; one actor can own many products) |
| A feature lives on surfaces | Feature -[LIVES_ON]-> Surface | Many-to-many (a feature can appear on multiple surfaces; a surface contains multiple features) |
| A rule governs features | Rule -[GOVERNS]-> Feature | One-to-many (one rule can govern many features) |
| An actor uses features | Actor -[USES]-> Feature | Many-to-many |

### Per-Edge Cardinality Limits

Define expected limits to guide extraction validation:

```python
CARDINALITY_LIMITS = {
    # (edge_type, direction): max_count
    ("OWNED_BY", "outgoing"): 1,      # A product has at most 1 primary owner
    ("LIVES_ON", "outgoing"): 5,       # A feature should not live on more than 5 surfaces
    ("GOVERNS", "outgoing"): None,     # No limit — a rule can govern any number of features
    ("WORKS_FOR", "outgoing"): 1,      # An actor works for at most 1 organization
}
```

### Extraction Validation

After extraction, validate cardinality limits. When a new edge would violate a limit:

1. **Hard limits** (like OWNED_BY): The new edge should replace the existing one, with the old edge invalidated temporally (see Temporal Modeling).
2. **Soft limits** (like LIVES_ON capped at 5): Log a warning for review. The entity may need to be split or the model may be hallucinating relationships.
3. **Unlimited** (like GOVERNS): Accept all edges.

Cardinality validation is a post-extraction step — implement it as a check after the LLM produces candidate edges, before persisting to the graph.

---

## Temporal Modeling

Temporal metadata is central to Graphiti's design. Unlike static knowledge graphs, Graphiti treats the graph as a living record where facts change over time.

### Edge Temporal Metadata

Every edge in Graphiti carries temporal fields:

| Field | Type | Description |
|-------|------|-------------|
| `created_at` | datetime | When this edge was first extracted/created |
| `valid_from` | datetime | When the fact this edge represents became true in the real world |
| `valid_to` | datetime or null | When the fact stopped being true (null = still valid) |
| `invalidated_at` | datetime or null | When the system marked this edge as superseded (null = still active) |

### Bi-Temporal Model

Graphiti uses a **bi-temporal** model, distinguishing between:

1. **Transaction time** (`created_at`, `invalidated_at`): When the system recorded or invalidated the fact. This is system-controlled and immutable.
2. **Valid time** (`valid_from`, `valid_to`): When the fact was true in the real world. This is extracted from context and may be approximate.

This separation enables queries like:
- "What did we know about Feature X as of last Tuesday?" (transaction time query)
- "What was the product roadmap for Q3 2025?" (valid time query)
- "What facts were recorded this week that describe events from last month?" (cross-temporal query)

### Episode-Based Temporal Model

Graphiti organizes ingestion into **episodes** — discrete units of information (a conversation turn, a document section, a meeting transcript chunk). Each episode has a timestamp, and edges extracted from an episode inherit its timestamp as the default `valid_from`.

This means:
- Temporal resolution is episode-level, not sentence-level.
- If an episode contains facts with different valid times ("We launched Feature A last month and plan Feature B for next quarter"), the extraction prompt should parse these into edges with different `valid_from` / `valid_to` values.
- Episode ordering matters: later episodes can invalidate edges from earlier episodes when facts change.

### Designing for Entity State Changes Over Time

When an entity's state changes (e.g., a feature moves from "planned" to "shipped"), the graph should reflect this as:

1. The old edge (e.g., Feature -[HAS_STATUS]-> "planned") gets `valid_to` set to the change date and `invalidated_at` set to the current processing time.
2. A new edge (Feature -[HAS_STATUS]-> "shipped") is created with `valid_from` set to the change date.

This preserves history. A query for "current state" filters where `invalidated_at IS NULL`, while a historical query can look at any point in time.

#### Temporal Ontology Considerations

When designing your ontology with temporal semantics in mind:

- **Distinguish stable vs volatile relationships**: `WORKS_FOR` (changes rarely) vs `HAS_STATUS` (changes frequently). Volatile relationships generate more temporal edges and need careful invalidation logic.
- **Model planned vs actual time**: A strategy might have an edge `TARGETS` with `valid_from` in the future (planned date). Distinguish this from facts that describe the present.
- **Avoid embedding time in node names**: Do not create "Q3 2025 Roadmap" as a node name. Instead, create a "Roadmap" node with temporal edges to features. This prevents entity resolution failures when the same roadmap is referenced with different time qualifiers.
- **Use episodes for provenance**: Each edge links back to the episode that produced it. This provides an audit trail for when and why facts were added or changed.

---

## Entity Resolution

Entity resolution (ER) is the process of determining whether a newly extracted entity refers to the same real-world thing as an existing node in the graph. Ontology design directly affects ER quality.

### Canonical Naming Conventions per Type

Define naming conventions so the LLM extracts consistent names:

| Node Type | Naming Convention | Examples |
|-----------|-------------------|----------|
| Product | Official name, proper casing | "Speqq", "Linear", "GitHub" |
| Actor (person) | Full name, title case | "Alice Chen", "Bob Martinez" |
| Actor (role) | Title case, no articles | "Product Manager", "Tech Lead" |
| Feature | Short descriptive phrase, sentence case | "Real-time collaboration", "AI chat" |
| Surface | Product-specific label | "Workspace Editor", "Settings page" |
| Rule | Short label describing the constraint | "Workspace limit", "Rate limit policy" |
| Strategy | Goal-oriented phrase | "Beta launch to 5k users" |
| Integration | Service name | "GitHub", "Stripe", "OpenAI API" |

Include these conventions in the extraction prompt via the node type's docstring or field description. The more consistent the LLM's output, the better ER performs.

### Merge Rules

When Graphiti detects a potential match (by name similarity within the same type), it applies merge rules:

1. **Exact match**: Same name, same type -> merge (update properties if new data is more complete).
2. **Fuzzy match**: High similarity score (e.g., "Alice Chen" vs "Alice C.") within the same type -> merge with confidence threshold.
3. **Alias handling**: The LLM may extract "PM" when the canonical name is "Product Manager". Use the extraction prompt to instruct the LLM to expand abbreviations when possible.

### Identity Keys

For node types with strong identifiers, define identity keys — properties that uniquely identify an entity beyond name similarity:

```python
class Integration(BaseModel):
    """An external service, API, or connected system."""
    name: str = Field(description="Service name (e.g., 'GitHub', 'Stripe').")
    api_url: Optional[str] = Field(
        default=None,
        description="Base API URL if mentioned. Acts as identity key for disambiguation."
    )
```

When `api_url` is present on both a new extraction and an existing node, use it for exact match even if names differ slightly ("GitHub API" vs "GitHub").

### How Node Type Design Affects Resolution Quality

- **Fewer types = more candidates to compare**: With a flat `Actor` type, "Alice Chen" and "Product Manager" are both Actors and get compared. This is good when they refer to the same person.
- **More types = fewer false matches**: With separate `Person` and `Role` types, "Alice Chen" (Person) is never compared to "Product Manager" (Role). This is good when they are genuinely different entities.
- **Descriptive names reduce ambiguity**: A Feature named "AI Chat" is less ambiguous than one named "Chat". Prompt the LLM to use the most specific name available.

---

## Constraining vs Leaving Open (Resolving the Constraint Paradox)

There is an apparent tension between two valid approaches:

- **Constrain edge types**: Enumerate allowed edge types and domain/range pairs to ensure graph consistency.
- **Leave edge types open**: Let the LLM generate relationship types freely to capture unanticipated relationships.

### What Graphiti Actually Enforces vs Suggests

Graphiti's behavior depends on configuration:

- **`entity_types` (node types)**: When provided, these are treated as **suggestions with strong preference**. The LLM is prompted to use these types but may occasionally produce unlisted types. Set `strict_entity_types=True` to hard-enforce.
- **`edge_type_map`**: When provided, this **constrains** which edge types are valid for each (source, target) pair. Edges not matching the map are rejected or re-extracted.
- **Free-form edge types**: When `edge_type_map` is not provided, the LLM generates edge types freely. The bidirectional map normalizes direction but does not constrain vocabulary.

### When to Constrain

- **Constrain node types** when your domain is well-understood and you need predictable graph structure for downstream queries.
- **Constrain edge types** when you need consistent relationship vocabulary for aggregation, filtering, or reporting.
- **Constrain domain/range** when nonsensical relationships would degrade retrieval quality (e.g., preventing Rule -[WORKS_FOR]-> Feature).

### When to Leave Open

- **Leave edge types open** during early exploration when you do not yet know what relationships matter.
- **Leave domain/range open** when you want the LLM to discover unexpected but valid relationships.
- **Use a hybrid approach**: Constrain node types (which are fewer and more stable) but leave edge types partially open. Define the core edge types you need for querying, and let the LLM add additional relationship types that fall outside your map.

### Practical Hybrid Pattern

```python
# Core constrained edge types (used for queries and aggregation)
CORE_EDGE_TYPES = {
    ("Actor", "Product"): ["SERVES", "USES", "OWNS"],
    ("Feature", "Surface"): ["LIVES_ON"],
    ("Rule", "Feature"): ["GOVERNS", "CONSTRAINS"],
}

# Allow unconstrained edges for pairs not in the map
ontology = OntologyConfig(
    entity_types=ENTITY_TYPES,
    edge_type_map=CORE_EDGE_TYPES,
    allow_unconstrained_edges=True,  # Edges for unlisted pairs are allowed
)
```

This gives you consistency where it matters while preserving discovery for the rest of the graph.

---

## Ontology Configuration in Graphiti

### config.yaml (Expanded)

```yaml
graphiti:
  entity_types:
    - name: "Product"
      description: "A software product, platform, or named system"
      priority: high
    - name: "Actor"
      description: "A person, role, or user persona that interacts with systems"
      priority: high
    - name: "Feature"
      description: "A specific capability or behavior within a product"
      priority: normal
    - name: "Surface"
      description: "A UI surface, page, or view within a product"
      priority: normal
    - name: "Rule"
      description: "A business rule, constraint, or policy"
      priority: normal
    - name: "Integration"
      description: "An external service, API, or connected system"
      priority: normal
    - name: "Strategy"
      description: "A business or product strategy, objective, or goal"
      priority: normal
    - name: "Flow"
      description: "A user workflow, journey, or sequence of actions"
      priority: low
    - name: "Concept"
      description: "An abstract idea, methodology, or domain term"
      priority: low

  edge_types:
    - name: "SERVES"
      reverse: "SERVED_BY"
      description: "An actor serves or is responsible for something"
      domain: ["Actor"]
      range: ["Product", "Feature", "Strategy"]
    - name: "GOVERNS"
      reverse: "GOVERNED_BY"
      description: "A rule constrains or governs something"
      domain: ["Rule"]
      range: ["Feature", "Surface", "Flow", "Product"]
    - name: "LIVES_ON"
      reverse: "CONTAINS"
      description: "A feature or component exists within a surface"
      domain: ["Feature"]
      range: ["Surface"]
    - name: "DEPENDS_ON"
      reverse: "REQUIRED_BY"
      description: "One entity requires another"
      domain: ["Feature", "Integration"]
      range: ["Feature", "Integration"]
    - name: "USES"
      reverse: "USED_BY"
      description: "An actor or system uses a tool or feature"
      domain: ["Actor"]
      range: ["Feature", "Product", "Integration"]
    - name: "OWNS"
      reverse: "OWNED_BY"
      description: "Ownership or primary responsibility"
      domain: ["Actor"]
      range: ["Product", "Feature", "Strategy"]
    - name: "IMPLEMENTS"
      reverse: "IMPLEMENTED_BY"
      description: "One entity realizes or implements another"
      domain: ["Feature"]
      range: ["Strategy", "Rule"]
    - name: "TARGETS"
      reverse: "TARGETED_BY"
      description: "A strategy targets an audience or concept"
      domain: ["Strategy"]
      range: ["Actor", "Concept"]

  strict_entity_types: true
  allow_unconstrained_edges: false
```

### Python (Full Configuration)

```python
from graphiti_core import Graphiti
from graphiti_core.utils.ontology import OntologyConfig

ontology = OntologyConfig(
    entity_types=ENTITY_TYPES,       # Pydantic models defined above
    edge_type_map=EDGE_TYPE_MAP,     # Dict of (source, target) -> [edge_types]
    edge_bidirectional_map=EDGE_BIDIRECTIONAL_MAP,
    strict_entity_types=True,
    allow_unconstrained_edges=False,
)

graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    ontology=ontology,
)
```

---

## Community Nodes and Type Interaction

Graphiti supports **community nodes** — automatically generated higher-order nodes that represent clusters of densely connected entities. Understanding how your node type design interacts with community formation helps you get better summaries and retrieval.

### How Communities Form

Communities are detected through graph algorithms (typically Leiden or Louvain community detection) that group nodes with many mutual connections. The algorithm does not consider node types — it operates purely on edge density.

### How Node Types Interact with Communities

- **Homogeneous communities**: If your ontology produces clusters of same-type nodes (e.g., a group of Features that all DEPEND_ON each other), communities will be type-homogeneous. Community summaries then describe feature clusters.
- **Heterogeneous communities**: If your ontology produces cross-type relationships (Actor -[OWNS]-> Product -[CONTAINS]-> Feature), communities will span types. Community summaries describe product ecosystems.
- **Hub nodes**: Nodes with many edges (e.g., a Product node connected to many Features, Actors, and Rules) become community hubs. If your ontology creates too many edges to a single node type, communities will be dominated by those hubs.

### Design Implications

- Keep edge density balanced across types. If every node connects to a Product node, communities will all center on Products and lose granularity.
- Use specific relationship types (LIVES_ON, GOVERNS, IMPLEMENTS) rather than generic ones (RELATED_TO) so community summaries are meaningful.
- If you need type-specific communities (e.g., "all features in this area"), query by node type within a community rather than expecting the algorithm to produce type-pure clusters.

---

## Multi-Tenancy Scoping

### Global vs Per-Tenant Ontologies

The ontology (type definitions) is typically **global** — all tenants share the same node types and edge types. This is the recommended approach because:

- Ontology design is a domain modeling decision, not a tenant preference.
- Shared types enable cross-tenant analytics and benchmarking.
- Maintaining per-tenant ontologies creates version drift and operational complexity.

### group_id Interaction

Graphiti uses `group_id` to scope data to a tenant. All nodes and edges belong to a group. The ontology defines types; the group_id scopes data.

```python
# Same ontology, different tenants
await graphiti.add_episode(
    name="team-standup",
    episode_body="We decided to prioritize the AI chat feature...",
    group_id="workspace_abc123",  # Tenant A
)

await graphiti.add_episode(
    name="product-review",
    episode_body="The dashboard needs real-time updates...",
    group_id="workspace_def456",  # Tenant B
)
```

### Same Entity Across Groups

The same real-world entity (e.g., "GitHub" as an Integration) may appear in multiple groups. Graphiti treats these as **separate nodes** — one per group. Entity resolution operates within a group, not across groups.

If you need cross-group entity references (e.g., a shared product catalog), model them as:
- A reference group that holds canonical entities.
- Per-tenant groups that hold tenant-specific relationships to those entities.
- A REFERENCES edge type linking tenant-local nodes to canonical nodes.

---

## Multi-Domain Composition

When your system spans multiple domains (e.g., product planning + customer support + engineering), you need strategies for composing ontologies.

### Namespace Conventions

Prefix domain-specific types to avoid collision:

```python
# Product planning domain
class PlanningFeature(BaseModel):
    """A product feature in the planning domain."""
    name: str

# Engineering domain
class EngineeringService(BaseModel):
    """A backend service or microservice."""
    name: str
    language: Optional[str] = None
```

Alternatively, use a `domain` property on a shared type rather than separate types per domain:

```python
class Feature(BaseModel):
    """A capability or behavior, tagged by domain."""
    name: str
    domain: str = Field(description="Which domain: 'planning', 'engineering', 'support'.")
```

### Layering Multiple Domain Ontologies

Structure ontologies in layers:

1. **Core layer**: Types shared across all domains (Actor, Product, Organization).
2. **Domain layer**: Types specific to one domain (Feature, Surface for planning; Service, Endpoint for engineering).
3. **Integration layer**: Edge types that bridge domains (IMPLEMENTS linking a planning Feature to an engineering Service).

```python
CORE_TYPES = [Actor, Product, Organization]
PLANNING_TYPES = [Feature, Surface, Flow, Rule, Strategy]
ENGINEERING_TYPES = [Service, Endpoint, Repository]

# Compose for the full system
ALL_TYPES = CORE_TYPES + PLANNING_TYPES + ENGINEERING_TYPES
```

### Conflict Resolution

When composing ontologies, conflicts arise when:

- Two domains define types with the same name but different semantics. **Resolution**: Rename one or use namespace prefixes.
- Two domains define overlapping edge types. **Resolution**: Use the more general version and add a `domain` property to the edge fact.
- An entity could belong to types from different domains. **Resolution**: Choose the primary domain and use cross-domain edges rather than dual-typing.

---

## Ontology Evolution

### When to Add New Types
- When extraction consistently miscategorizes a common entity kind
- When retrieval needs to filter by a type that doesn't exist
- When a new domain area is being indexed (e.g., adding billing-related types)

### When NOT to Add New Types
- For one-off entities that fit an existing type with fallback
- When the distinction doesn't affect retrieval quality
- When it would create overlap with an existing type

### Migration Strategy

#### Type Renames

When a node type name changes (e.g., `Capability` -> `Feature`):

1. Add the new type to the ontology configuration.
2. New extractions use the new type name.
3. Existing nodes retain the old type name in the graph.
4. Run a migration query to update existing nodes:

```cypher
MATCH (n:Entity {entity_type: 'Capability'})
SET n.entity_type = 'Feature'
```

5. Remove the old type from the ontology configuration only after migration is complete.

#### Type Splits

When one type is split into multiple types (e.g., `Actor` -> `Person` + `Role`):

1. Add the new types to the ontology. Keep the old type temporarily.
2. New extractions will use the new types.
3. Migrate existing nodes based on a discriminator property or manual review:

```cypher
// Migrate actors with email addresses to Person
MATCH (n:Entity {entity_type: 'Actor'})
WHERE n.email IS NOT NULL
SET n.entity_type = 'Person'

// Migrate remaining actors to Role
MATCH (n:Entity {entity_type: 'Actor'})
SET n.entity_type = 'Role'
```

4. Update edge type constraints (domain/range) to reference the new types.
5. Remove the old type from configuration.

#### Backward Compatibility

During migration, your application may need to query across both old and new schemas:

```cypher
// Query that works during and after migration
MATCH (n:Entity)
WHERE n.entity_type IN ['Actor', 'Person', 'Role']
RETURN n
```

Maintain a version map in your application code:

```python
# Schema version compatibility
TYPE_ALIASES = {
    "Capability": "Feature",      # Renamed in v2
    "Actor": ["Person", "Role"],  # Split in v3
}

def resolve_type_query(type_name: str) -> list[str]:
    """Return all type names that should be included when querying for a type."""
    aliases = TYPE_ALIASES.get(type_name, [])
    if isinstance(aliases, str):
        return [type_name, aliases]
    return [type_name] + aliases
```

#### Querying Across Schema Versions

When the ontology evolves, older edges may reference types or relationship labels that no longer exist in the current schema. To handle this:

- Never delete old edges during migration. Invalidate them temporally (`invalidated_at`).
- Create new edges under the current schema that carry forward the still-valid facts.
- Use the `created_at` timestamp to distinguish old-schema vs new-schema edges when debugging.

#### Re-Ingestion Tooling Guidance

For critical ontology changes, re-ingestion may be necessary — replaying source episodes through the new ontology:

1. **Identify affected episodes**: Query edges by type to find episodes that produced entities of the changed type.
2. **Invalidate old extractions**: Set `invalidated_at` on edges from those episodes.
3. **Re-ingest**: Replay episodes through the current ontology. Graphiti will extract new entities and edges using the updated types.
4. **Validate**: Compare old and new extraction results for a sample of episodes. Check entity counts, type distributions, and edge patterns.
5. **Clean up**: Once validated, optionally archive invalidated edges.

Re-ingestion is expensive. Prefer additive changes (new types, new properties) over destructive changes (renames, splits) when possible.

---

## Ontology Evaluation

An ontology should be evaluated empirically, not just designed theoretically. Use these techniques to assess and improve ontology quality.

### Coverage Testing Against Sample Corpus

1. Assemble a representative sample of source documents (conversation transcripts, PRDs, meeting notes).
2. Run extraction with the current ontology.
3. Manually review a subset and annotate:
   - Entities that were correctly typed.
   - Entities that were miscategorized (wrong type).
   - Entities that were missed (not extracted at all).
   - Spurious entities (hallucinated by the LLM).

### Precision and Recall per Type

Calculate per-type metrics:

| Type | Extracted | Correct | Missed | Spurious | Precision | Recall |
|------|-----------|---------|--------|----------|-----------|--------|
| Product | 15 | 14 | 1 | 1 | 93% | 93% |
| Actor | 42 | 38 | 8 | 4 | 90% | 83% |
| Feature | 67 | 51 | 12 | 16 | 76% | 81% |
| Rule | 8 | 6 | 5 | 2 | 75% | 55% |

Low recall for a type suggests the description is too narrow or the type is not well-understood by the LLM. Low precision suggests the description is too broad or overlaps with another type.

### Type Distribution Monitoring

Track the distribution of extracted types over time:

```python
# Query type distribution
MATCH (n:Entity)
WHERE n.group_id = $group_id
RETURN n.entity_type AS type, count(*) AS count
ORDER BY count DESC
```

Watch for:
- **Dominant types** (one type has 60%+ of entities): The type may be too broad. Consider splitting.
- **Unused types** (zero extractions over significant corpus): Remove or revise the description.
- **Balanced distribution with a long tail**: Healthy ontology with fallback types catching edge cases.

### A/B Testing Ontology Changes

When making significant ontology changes:

1. Run extraction on the same corpus with both the old and new ontology.
2. Compare: entity counts per type, edge counts, fact quality (manual review of a sample).
3. Measure retrieval quality: run a set of test queries against both graphs and compare the relevance of returned facts.
4. Only deploy the new ontology if metrics improve or hold steady across the board.

---

## Complete Worked Example: Project Management SaaS

This example walks through ontology design end-to-end for a hypothetical project management SaaS called "TaskFlow."

### Step 1: Requirements Gathering

**Domain**: TaskFlow is a project management tool used by engineering teams. Key entities include projects, tasks, team members, sprints, and integrations. Teams discuss priorities in standups, file bugs, and track velocity.

**Key questions**:
- What entities do users talk about? Projects, tasks, people, sprints, tools.
- What relationships matter for retrieval? Who owns what, what blocks what, what's in which sprint.
- What changes over time? Task status, sprint assignments, team membership, priorities.

### Step 2: Node Type Design

```python
from datetime import datetime
from enum import Enum
from typing import Optional
from pydantic import BaseModel, Field, field_validator


class TaskStatus(str, Enum):
    BACKLOG = "backlog"
    TODO = "todo"
    IN_PROGRESS = "in_progress"
    IN_REVIEW = "in_review"
    DONE = "done"
    CANCELLED = "cancelled"


class TaskPriority(str, Enum):
    URGENT = "urgent"
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"


class Project(BaseModel):
    """A project or initiative within TaskFlow. Use the official project name."""
    name: str = Field(description="Project name as it appears in TaskFlow.")
    status: Optional[str] = Field(
        default=None, description="Active, completed, or archived."
    )

    @field_validator("name")
    @classmethod
    def clean_name(cls, v: str) -> str:
        return v.strip()


class Task(BaseModel):
    """A work item, bug, story, or ticket."""
    name: str = Field(
        description="Task title. Use the most specific name from context."
    )
    status: Optional[TaskStatus] = Field(
        default=None,
        description="Current task status. Infer from 'done', 'working on', 'blocked', etc."
    )
    priority: Optional[TaskPriority] = Field(
        default=None, description="Priority if mentioned."
    )
    task_type: Optional[str] = Field(
        default=None, description="Type: 'bug', 'story', 'spike', 'chore'."
    )


class TeamMember(BaseModel):
    """A person on the team. Use full name when available."""
    name: str = Field(description="Full name, title case.")
    role: Optional[str] = Field(
        default=None, description="Job title or team role."
    )

    @field_validator("name")
    @classmethod
    def title_case_name(cls, v: str) -> str:
        return v.strip().title()


class Sprint(BaseModel):
    """A time-boxed iteration or sprint."""
    name: str = Field(
        description="Sprint identifier (e.g., 'Sprint 14', '2026-Q1-W3')."
    )
    start_date: Optional[str] = Field(
        default=None, description="Sprint start date if mentioned."
    )
    end_date: Optional[str] = Field(
        default=None, description="Sprint end date if mentioned."
    )


class Tool(BaseModel):
    """An external tool, service, or integration."""
    name: str = Field(description="Tool or service name (e.g., 'Jira', 'GitHub').")


TASKFLOW_ENTITY_TYPES = [Project, Task, TeamMember, Sprint, Tool]
```

### Step 3: Edge Type Design with Domain/Range

| Edge Type | Domain | Range | Meaning |
|-----------|--------|-------|---------|
| OWNS | TeamMember | Project, Task | Ownership / primary responsibility |
| ASSIGNED_TO | Task | TeamMember | Task assignment |
| BLOCKED_BY | Task | Task | Blocking dependency |
| BELONGS_TO | Task | Project | Task is part of project |
| IN_SPRINT | Task | Sprint | Task is scheduled in sprint |
| INTEGRATES_WITH | Project | Tool | Project uses external tool |
| REPORTS_TO | TeamMember | TeamMember | Reporting relationship |

```python
TASKFLOW_EDGE_TYPE_MAP = {
    ("TeamMember", "Project"): ["OWNS"],
    ("TeamMember", "Task"): ["OWNS"],
    ("Task", "TeamMember"): ["ASSIGNED_TO"],
    ("Task", "Task"): ["BLOCKED_BY"],
    ("Task", "Project"): ["BELONGS_TO"],
    ("Task", "Sprint"): ["IN_SPRINT"],
    ("Project", "Tool"): ["INTEGRATES_WITH"],
    ("TeamMember", "TeamMember"): ["REPORTS_TO"],
}

TASKFLOW_EDGE_BIDIRECTIONAL_MAP = {
    "OWNS": "OWNED_BY",
    "OWNED_BY": "OWNS",
    "ASSIGNED_TO": "ASSIGNS",
    "ASSIGNS": "ASSIGNED_TO",
    "BLOCKED_BY": "BLOCKS",
    "BLOCKS": "BLOCKED_BY",
    "BELONGS_TO": "CONTAINS",
    "CONTAINS": "BELONGS_TO",
    "IN_SPRINT": "INCLUDES",
    "INCLUDES": "IN_SPRINT",
    "INTEGRATES_WITH": "INTEGRATED_BY",
    "INTEGRATED_BY": "INTEGRATES_WITH",
    "REPORTS_TO": "MANAGES",
    "MANAGES": "REPORTS_TO",
}
```

### Step 4: Temporal Considerations

Key state changes in this domain:
- **Task status changes**: `IN_PROGRESS` -> `IN_REVIEW` -> `DONE`. Each change produces a new edge with updated `valid_from`.
- **Sprint assignment changes**: A task moves from Sprint 14 to Sprint 15. The old `IN_SPRINT` edge gets `valid_to`; a new one is created.
- **Team membership changes**: A team member leaves or changes roles. `REPORTS_TO` edges are temporally bounded.
- **Blocking resolution**: When a blocker is resolved, the `BLOCKED_BY` edge gets `valid_to` set.

### Step 5: Sample Extraction

**Input episode** (standup transcript):

> "Alice said she finished the login page redesign yesterday. Bob mentioned that the API rate limiting task is blocked by the auth service migration, which Dana is working on. Carol is picking up the dashboard performance bug for Sprint 15."

**Extracted nodes**:

| Name | Type | Properties |
|------|------|------------|
| Alice | TeamMember | `{role: null}` |
| Bob | TeamMember | `{role: null}` |
| Dana | TeamMember | `{role: null}` |
| Carol | TeamMember | `{role: null}` |
| Login Page Redesign | Task | `{status: "done", task_type: "story"}` |
| Api Rate Limiting | Task | `{status: "in_progress", task_type: null}` |
| Auth Service Migration | Task | `{status: "in_progress", task_type: null}` |
| Dashboard Performance Bug | Task | `{status: "todo", task_type: "bug"}` |
| Sprint 15 | Sprint | `{start_date: null, end_date: null}` |

**Extracted edges**:

| Source | Edge Type | Target | Fact |
|--------|-----------|--------|------|
| Login Page Redesign | ASSIGNED_TO | Alice | "Alice completed the login page redesign, finishing it yesterday." |
| Api Rate Limiting | BLOCKED_BY | Auth Service Migration | "The API rate limiting task is blocked by the auth service migration." |
| Auth Service Migration | ASSIGNED_TO | Dana | "Dana is currently working on the auth service migration." |
| Dashboard Performance Bug | ASSIGNED_TO | Carol | "Carol is picking up the dashboard performance bug." |
| Dashboard Performance Bug | IN_SPRINT | Sprint 15 | "The dashboard performance bug is scheduled for Sprint 15." |

**Temporal metadata on the Login Page Redesign -> Alice edge**:
- `created_at`: 2026-03-22 (processing time)
- `valid_from`: 2026-03-21 (yesterday, when Alice finished)
- `valid_to`: null (still valid — she completed it)
- `invalidated_at`: null

### Step 6: Validation

After extraction, verify:
- All 5 node types received at least one entity (coverage check).
- No nonsensical edges (domain/range check): all ASSIGNED_TO edges go from Task to TeamMember.
- Cardinality check: each task has at most one ASSIGNED_TO (or flag for review if multiple).
- Entity resolution: "Alice" should merge with existing "Alice Chen" if present in the graph.

---

## Anti-Patterns

1. **Type explosion**: 30+ node types leads to extraction confusion and slow LLM calls. Keep it under 15.
2. **Overlapping types**: "Feature" and "Capability" will be used inconsistently. Pick one.
3. **Missing fallbacks**: Without Topic/Object fallbacks, entities get force-fit into wrong types.
4. **Unconstrained edges without a plan**: Leaving edge types fully unconstrained is appropriate during exploration but becomes a liability in production. Once you know your domain, define at least the core edge types you query against. See "Constraining vs Leaving Open" for the recommended hybrid approach.
5. **Ignoring direction**: Undirected relationships (IS_FRIENDS_WITH) are fine, but most relationships have a natural direction. Define it.
6. **Over-specifying descriptions**: Descriptions should be 1-2 sentences. Long descriptions confuse the extractor.
7. **Embedding time in node names**: "Q3 2025 Roadmap" as a node name causes entity resolution to fail when the same roadmap is referenced differently. Use temporal edges instead.
8. **Ignoring cardinality**: Without cardinality expectations, the graph accumulates duplicate or contradictory relationships silently.
9. **No evaluation loop**: Designing an ontology without testing it against real data leads to blind spots. Always run extraction on a sample corpus and measure.
