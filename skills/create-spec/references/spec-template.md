# Spec Template

This template maps the Wiegers SRS structure to Speqq tabs. Follow this when scaffolding a spec. Always create the required tabs. Add optional tabs based on the feature.

---

## Required tabs

These tabs exist on every spec.

### Overview (page tab)

Covers Wiegers §1 Introduction and §2 Overall Description. Write this as narrative prose with H2/H3 sections:

- **Purpose** — What is this spec for? One sentence.
- **Product perspective** — Is this a new product, an extension, or a replacement?
- **User classes** — Who uses this? Name each user class with their goals, frequency, domain knowledge, and permissions.
- **Operating environment** — Platforms, infrastructure, deployment model.
- **Scope** — What is in. What is explicitly out.
- **Constraints** — Technology mandates, regulatory requirements, platform limitations, budget, timeline.
- **Assumptions and dependencies** — What we are assuming is true. What this depends on.
- **Open issues** — Questions that need answers before or during implementation.

### Requirements (table tab)

Covers Wiegers §3 System Features, §6 Quality Attributes, and constraints. Every requirement is a row. Child rows are acceptance criteria nested under the parent requirement.

**Required columns:**

| Column      | Description                                                                       |
| ----------- | --------------------------------------------------------------------------------- |
| ID          | Unique, persistent, never reused (e.g., `RQ-001`, `RQ-002`)                       |
| Title       | Short name for the requirement                                                    |
| Description | The full requirement statement using EARS syntax                                  |
| Type        | functional, performance, usability, interface, constraint, quality, business-rule |
| Priority    | High, Medium, Low                                                                 |
| Phase       | Phase 1, Phase 2, Future                                                          |
| Status      | Draft, Approved, Deferred                                                         |

**Optional columns** (add when the feature needs them):

| Column              | When to add                                            |
| ------------------- | ------------------------------------------------------ |
| Rationale           | When the "why" is not obvious from the requirement     |
| Difficulty          | When effort estimation matters                         |
| Risk                | When failure consequences vary across requirements     |
| Verification method | test, analysis, inspection, demonstration              |
| Surface             | When requirements map to specific UI surfaces          |
| Actor               | When multiple user classes have different requirements |

Create columns first, then write rows. Every row must have all required columns populated.

---

## Optional tabs

Add these based on the feature. Not every spec needs all of them.

### User Journeys (page tab)

Add when user-facing flows exist. Assign each journey an ID (UJ-001, UJ-002). For each journey:

- Actor and entry point
- Trigger — what initiates this journey
- Preconditions — what must be true before
- Happy path — numbered steps
- Alternate paths — variations
- Failure paths — what goes wrong, what the user sees
- Terminal state — where the user ends up

### System Design (page tab)

Add when the feature involves architecture decisions, data flow, or multi-service coordination. Use Mermaid diagrams and prose. See `references/spec-image-types.md` for diagram types and examples.

- Architecture decisions with rationale
- Data flow between services or components
- State machines for entity lifecycles
- Sequence diagrams for multi-system interactions
- Technical constraints and tradeoffs

### Data Model (page tab)

Add when entities and relationships are core to the feature. Covers Wiegers §4 Data Requirements.

- Entities and their attributes
- Relationships between entities
- Lifecycle — created, modified, archived, deleted
- Integrity rules — validation, constraints, cascading behavior
- Retention — how long data is kept

### Interface Contracts (page tab)

Add when APIs or integrations are core. Covers Wiegers §5 External Interface Requirements.

- Endpoints and methods
- Authentication and authorization
- Request/response payloads
- Error responses and codes
- Rate limits and quotas

### Business Rules (page tab)

Add when complex rules drive behavior. Assign each rule an ID (BR-001, BR-002).

- Facts, Constraints, Action enablers, Inferences, Computations
- Each rule atomic — one per statement
- Trace each rule to the functional requirements that enforce it

---

## Per-requirement metadata schema

Every requirement in the Requirements table carries these attributes:

```
{
  id:                 // unique, persistent, never changed, never reused (e.g., "RQ-001")
  text:               // the single "shall" statement
  type:               // functional | performance | usability | interface | constraint | quality | business-rule
  priority:           // High / Medium / Low
  rationale:          // why the requirement is needed; assumptions recorded here
  difficulty:         // Easy | Nominal | Difficult
  risk:               // consequence grade (financial, safety/health, legal/standards)
  status:             // draft | reviewed | approved | implemented | verified
  verification_method:// test | analysis | inspection | demonstration
}
```

---

## Wiegers SRS reference outline

For reference, the full Wiegers SRS template that the tab structure above is derived from:

1. **Introduction** — Purpose, Document Conventions, Intended Audience, Project Scope, References
2. **Overall Description** — Product Perspective, User Classes, Operating Environment, Constraints, Assumptions
3. **System Features** — per feature: Description, Functional Requirements
4. **Data Requirements** — Logical Data Model, Data Dictionary, Reports, Data Integrity/Retention
5. **External Interface Requirements** — User, Software, Hardware, Communications
6. **Quality Attributes** — Usability, Performance, Security, Safety, others
7. **Internationalization and Localization Requirements**
8. **Other Requirements**

- **Appendixes** — Glossary, Analysis Models
