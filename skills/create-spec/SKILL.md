---
name: create-spec
description: This skill writes, refines, and structures software **specs** for spec-driven development.
---

ALWAYS READ THIS SKILL LINE BY LINE AND WORD BY WORD

## When to use this skill

- The user wants to write a new spec or any of its parts.
- The user wants to refine or improve an existing spec.
- The user wants to review a spec against a quality bar.
- The user wants to convert needs, notes, or user stories into spec statements.
- The user wants functional or nonfunctional requirements, quality attributes, interfaces, or constraints written.

## Process

Follow these stages in order. Each stage has an exit condition — do not advance until it is met.

### 1. Request alignment

- Treat the user as a stakeholder. Your job is to understand their desired outcome.
- Use Mermaid diagrams to confirm understanding when a flow, state model, or architecture relationship is clearer visually. See `references/spec-image-types.md` for diagram selection.
- If the user is unclear on what they want, direct the conversation. Act as a software consultant: ask pointed questions, suggest directions, surface tradeoffs.
- **Exit:** You and the user agree on what the spec will cover.

### 2. Ground yourself in the product

- Assess what you know. Do you understand what the product does, who uses it, and what the user is working on? If not, fill the gap before proceeding.
- Use the product graph tools to search the codebase. This is your primary source of truth.
- Use the internet for market context, competitors, and landscape. Web is supplementary — it does not replace product-specific grounding.
- **Match your voice to the domain.** When discussing user needs, capabilities, and product decisions — speak like a product manager or UX designer. When discussing system behavior, data flow, and technical constraints — speak like an engineer.
- **Exit:** You can describe the product, its users, and the current state of the area you are speccing.

### 3. Research solutions

- Research common solutions to the problem. Look at how comparable products solve it.
- Consider what innovative approaches exist.
- Understand the current state of the product relative to the goal. Is this expanding an existing feature, replacing something, or building net-new? The answer changes the spec.
- **Exit:** You have a short list of viable approaches with tradeoffs.

### 4. Align on the solution

This is one of the most important stages. Use more visuals than words here.

- Present the user with a Mermaid diagram or concise flow outline that shows the proposed solution in a form a human can digest quickly.
- The user cannot absorb information at the rate you can produce it. Favor one clear visual over three paragraphs of explanation.
- If multiple approaches are viable, present them with tradeoffs and ask the user to choose. Use directional questions to narrow the decision.
- **Exit:** You and the user are clear on the solution. You can describe exactly what will be built and what is out of scope.

### 5. Write the spec

- Follow the template in `references/spec-template.md` to structure the spec. Always use the template unless the user explicitly tells you to do otherwise.
- Use the style guide and rules below for every requirement you write.
- Use terms from `references/spec-glossary.md` precisely — do not invent synonyms.
- Elicit requirements from: the aligned solution, the product grounding, user goals, operational scenarios, constraints, and quality needs.
- Capture constraints, business rules, use cases, interface needs, and quality attributes — not just functional requirements.
- **Exit:** The spec is written with all sections populated. No empty sections.

### 6. Validate the spec

- Re-read every section of the spec you wrote. Do not work from memory.
- Check every requirement against the quality bar below. Fix failures — do not list them.
- Check traceability: every requirement traces up to a user need and down to testable acceptance criteria.
- Check completeness: walk the progression from nothing exists → first use → normal operation → edge cases → maintenance. Fill gaps.
- Check consistency: no contradictions, no duplicate requirements, one term per concept.
- Confirm the spec defines the right system (validation) and is well-formed (verification).
- **Exit:** The spec passes the quality bar and is ready for review.

## Reference files:

- **`references/spec-glossary.md`** — when you need the precise definition of a requirement type or term.
- **`references/spec-template.md`** — when scaffolding a whole spec document. Holds the document outlines and the per-requirement metadata schema.
- **`references/spec-image-types.md`** — when deciding whether and which diagram to add.

## Spec style guide and rules

1. **Separate the what from the how.** A requirement states what the system must do or be — not how to build it.
2. **One requirement, one statement.** Each requirement is a single, atomic, testable statement, so a single test can confirm it.
3. **The set must hold together before approval.** Before a spec is approved, its requirements must be complete, consistent, and within scope. A working draft may still contain TBDs.
4. **Validate and verify before building.** Confirm the spec defines the right system (validation) and is well-formed (verification) before it drives implementation.

---

### Write requirements at the right level

Requirements exist in tiers. Write each at its level, and trace it up to the need it serves and down to the requirements derived from it.

| Level         | Captures                                           | Lives in                         | Audience            |
| ------------- | -------------------------------------------------- | -------------------------------- | ------------------- |
| Business      | Why the product exists; objectives and scope       | Stakeholder spec (StRS)          | Sponsors, execs     |
| User          | Goals and tasks users must accomplish              | Stakeholder spec (StRS)          | Users, PMs/BAs      |
| System        | Behavior of the whole system (hardware + software) | System spec (SyRS)               | Systems engineers   |
| Software      | Behaviors the software must exhibit                | Software spec (SRS)              | Developers, testers |
| Nonfunctional | Quality attributes and properties                  | Across the system/software specs | All                 |
| Constraint    | Restrictions on the solution                       | Constraints / design constraints | Architects          |

Keep traceability bidirectional: every lower-tier requirement points up to a parent need, and every parent points down to its children.

---

### How to write a single requirement

### Keywords

- Use **shall** for every mandatory requirement. Pick one keyword and use it consistently — never mix shall, must, and will for the same purpose.
- Use **should** for preferences or goals (non-binding).
- Use **may** for options or allowances (non-binding).
- Avoid **will** — it reads as a statement of fact and can be construed as legally binding.
- Avoid **must** — it is easily misread as a requirement.
- For descriptive text that is not a requirement, use is, are, or was.

### Grammar and style

- Write in active voice with an explicit subject: "the system shall...". Avoid passive constructions like "shall be able to be selected."
- State requirements positively. Avoid "shall not."
- Put the key point first; add supporting detail after it.
- Keep it to one requirement per statement. Conjunctions (and, or, also, unless, except, but) and slashes (and/or) usually mean two requirements are hiding in one — split them.
- Be explicit at numeric boundaries: write "5 or fewer," not "less than 5."
- Cover every path. Specify all outcomes of conditional logic and all exception and error cases — not just the happy path.
- Use the same term for the same concept everywhere. Define each term once, as a plain declarative statement.
- Record any assumptions in the requirement's rationale.

### Level of detail

- Give enough detail that a developer can build it and a tester can verify it.
- Add more detail when the work goes to an external client, the team is distributed, testing is based on the requirements, or estimates must be accurate. Use less when customers are closely involved or the team knows the domain well.
- Rule of thumb: if a small number of tests can confirm it, the detail is about right.
- Use a table when a single requirement varies only by a small detail.

---

## Requirement syntax patterns

Use a consistent template for every requirement. Pick the one that fits:

- **System behavior:** `[Condition] the [system] shall [action] [object] [constraint or value].`
  e.g., _When a trigger signal is received, the system shall set the status bit within 2 seconds._
- **User capability:** `The [user or actor] shall be able to [action] [object] [qualifying conditions, response time, or quality].`
  e.g., _The cashier shall be able to void a line item before payment is taken._

**EARS patterns** (use for clear, parseable phrasing):

- **Ubiquitous:** `The <system> shall <response>.`
- **Event-driven:** `When <trigger>, the <system> shall <response>.`
- **State-driven:** `While <state>, the <system> shall <response>.`
- **Unwanted behavior:** `If <unwanted condition>, then the <system> shall <response>.`
- **Optional feature:** `Where <feature is included>, the <system> shall <response>.`
- **Complex:** combine the patterns above.

A **condition** is a measurable qualifier that makes a requirement verifiable. A **constraint** restricts the design; it may apply to one requirement, to many, or stand alone. Condition-action tables and use cases are also valid ways to capture behavior.

---

## Words and phrases to avoid

A word that can't be turned into a pass/fail test does not belong in a requirement. Replace every vague qualifier with a measurable value, range, condition, or named standard.

Avoid:

- **Superlatives:** best, most.
- **Subjective terms:** user-friendly, easy to use, cost-effective, fast, robust, flexible, seamless.
- **Vague pronouns:** it, this, that.
- **Vague adverbs/adjectives:** almost always, significant, minimal, several, some.
- **Open-ended phrases:** provide support, including but not limited to, as a minimum, etc.
- **Comparatives without a baseline:** better than, higher quality.
- **Loopholes:** if possible, as appropriate, as applicable, when necessary, normally, ideally.
- **Unbounded verbs:** optimize, maximize, minimize, support.
- **Hidden compounds:** and, or, also, unless, except, but, and/or.
- **Incomplete references:** citing a document without its date and version, or without naming the applicable parts.
- **Placeholders left in an approved spec:** to be determined (TBD).

---

## Functional requirements

A functional requirement describes a behavior the system exhibits under specific conditions. Write each one with the syntax patterns above. Group related functional requirements under the feature or function they support. Pair every functional requirement with its triggering condition and expected response, and add companion requirements for the exception and error paths.

---

## Nonfunctional requirements and quality attributes

Quality attributes — the "-ilities" — are nonfunctional requirements, alongside constraints and interfaces. Decide which attributes matter before drafting, tailor them to the system, and make every one measurable.

- **User-facing attributes:** Availability, Installability, Integrity, Interoperability, Performance, Reliability, Robustness (correct behavior despite invalid input), Safety, Security, Usability.
- **Developer-facing attributes:** Efficiency, Modifiability, Portability, Reusability, Scalability, Verifiability (testability).

To make them testable:

- Never write "the system shall be user-friendly." Replace it with a measurable target. Apply SMART — Specific, Measurable, Attainable, Relevant, Time-bound — and confirm a tester can verify it.
- Avoid absolutes like 100%. Even life-critical systems specify high-but-finite targets (e.g., 99.99999% available).
- Specify performance quantitatively: how well, and under what conditions, a function runs.

---

## Constraints and business rules

- **Constraint:** a restriction on the design or construction of the product — e.g., an interface to an existing system that can't change, a physical size limit, a law or regulation, a fixed budget or schedule, a mandated technology platform, or a user/operator limitation. State it explicitly; it bounds the designer's options.
- **Business rule:** a policy, standard, or regulation that constrains the business. It is not itself a software requirement — it is the source of them. Trace each business rule to the functional requirements that enforce it rather than implementing it directly.

---

## External interface requirements

Specify every connection between the software and a user, another system, or a device. Cover four kinds:

1. **User interfaces** — standards, screen layouts, common controls, input validation, and error behavior. Mockups and screenshots do not replace written requirements; keep them separate from the spec body.
2. **Software interfaces** — APIs, services, and data formats exchanged with other systems.
3. **Hardware interfaces** — devices, ports, signals, supported hardware.
4. **Communications interfaces** — protocols, message formats, and transmission security.

---

## Requirement attributes

Give every requirement descriptive attributes in the repository, not just the sentence:

- **ID** — unique, persistent, never changed and never reused, even if the requirement is edited or deleted.
- **Priority** — High/Medium/Low or 1-5, set by stakeholder consensus.
- **Source / owner** — who originated it.
- **Rationale** — why it's needed, plus the analysis or assumptions behind it.
- **Type** — functional, performance, usability, interface, constraint, etc.
- **Dependency** — other requirements it relies on.
- **Risk** and **difficulty** — to support trade-offs and estimates.
- **Status / version** — under configuration control.

Emit each requirement as a structured record:
`{ id, Title, Description, Type, Priority, Phase/Version (eg, Phase 1), Status,`

See `references/spec-template.md` for the document outlines this lives inside.

---

## Labeling and traceability

- Give every requirement, table, and diagram a unique ID for cross-referencing. Don't rely on auto-numbering — inserting an item renumbers everything. Use a stable label like `RQ-1` of Fiq 1: Image of Front End
- Reference requirements by ID and text
- Maintain bidirectional traceability: each requirement links up to its source need and down to derived requirements, design elements, and the tests that verify it.

---

---

## Quality bar

### Every requirement must be

- **Necessary** — remove it and a real gap appears.
- **Correct** — accurately describes a capability that meets a stakeholder need.
- **Unambiguous** — reads the same way to everyone; can be interpreted only one way.
- **Complete** — carries all information needed to understand and implement it.
- **Singular** — one requirement, no conjunctions.
- **Implementation-free** — says what, not how.
- **Feasible** — achievable within known capabilities and constraints, at acceptable risk.
- **Verifiable** — a test can prove it was met; it is measurable.
- **Consistent** — does not conflict with any other requirement.
- **Prioritized** — its relative importance is agreed.
- **Traceable** — linked up to its source and down to its children.

### The set must be

- **Complete** — nothing missing; no TBD/TBS/TBR clauses at baseline.
- **Consistent** — no contradictions, no duplication, one term per concept throughout.
- **Modifiable** — DRY and version-controlled; edit once, propagate everywhere.
- **Traceable** — bidirectional links throughout.
- **Affordable** — satisfiable within cost, schedule, technical, and legal constraints.
- **Bounded** — held to the agreed scope; no creep.

---

## Reject these anti-patterns

- Compound requirements (multiple "shall"s or conjunctions in one statement).
- Design or implementation detail leaking into a requirement.
- Unmeasurable qualities ("user-friendly," "fast," "robust").
- Negative requirements ("shall not...") where a positive statement is clearer.
- Happy-path-only specs with no exception handling.
- Requirements without an ID, rationale, or traceability.
- Mockups or screenshots standing in for written functional or data requirements.
