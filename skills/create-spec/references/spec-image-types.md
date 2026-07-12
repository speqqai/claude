# Spec Image Types

Use visuals to communicate structure, flow, and relationships. A spec should combine well-structured natural language with visual models when prose alone is too slow to parse.

---

## Mermaid Diagrams

Use ````mermaid` fenced code blocks. Mermaid handles system flows, data models, state machines, business processes, and decision logic.

### Picking The Right Diagram

| What you need to show                                       | Diagram type            | Use it when                                          |
| ----------------------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| Service flows, API interactions, multi-system communication | Sequence diagram        | Multiple actors exchange messages in a defined order |
| Business processes, decision logic, user flow branching     | Flowchart               | A process has branching paths and decision points    |
| Data entities and relationships                             | ER diagram              | Defining the logical data model                      |
| State transitions, lifecycle flows                          | State diagram           | An entity has a meaningful lifecycle                 |
| Cross-role processes                                        | Swimlane flowchart      | Showing who does what in a multi-role workflow       |
| Many condition/outcome combinations                         | Markdown decision table | Multiple conditions have distinct outcomes           |

### Examples

**Sequence diagram — service flow:**

```mermaid
sequenceDiagram
    participant App as Your App
    participant Queue as Message Queue
    participant Worker as Delivery Worker
    participant Endpoint as Customer Endpoint

    App->>Queue: Enqueue webhook event
    Queue->>Worker: Dequeue event
    Worker->>Endpoint: POST signed payload
    alt 2xx response
        Worker->>Queue: Acknowledge delivery
    else 5xx or timeout
        Worker->>Queue: Schedule retry
    end
```

**State diagram — lifecycle:**

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Delivering
    Delivering --> Delivered : 2xx response
    Delivering --> Retrying : 5xx / timeout
    Retrying --> Delivering : retry attempt
    Retrying --> Failed : max attempts exceeded
    Failed --> [*]
    Delivered --> [*]
```

**ER diagram — data model:**

```mermaid
erDiagram
    ENDPOINT ||--o{ SUBSCRIPTION : has
    SUBSCRIPTION }o--|| EVENT_TYPE : subscribes_to
    ENDPOINT ||--o{ DELIVERY : receives
    DELIVERY }o--|| EVENT : delivers
```

**Flowchart — business process:**

```mermaid
flowchart TD
    A[Event fires] --> B{Matching subscriptions?}
    B -->|Yes| C[Enqueue delivery]
    B -->|No| D[Discard]
    C --> E[Sign payload with HMAC]
    E --> F[POST to endpoint]
    F --> G{Response?}
    G -->|2xx| H[Mark delivered]
    G -->|5xx/timeout| I[Schedule retry]
    G -->|4xx| J[Mark permanently failed]
```

Use notes and labels to call out important behavior. Keep diagrams focused: show the key flow, not every edge case.

---

## General Principles

- Visualize the complex or risky parts of the feature, not everything.
- Diagrams complement text; they do not replace written requirements.
- Give every diagram a unique identifier so requirements can cross-reference it.
- Use markdown tables for matrices, comparisons, and decision logic when a diagram would be harder to scan.
