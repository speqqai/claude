# PRD Writing Standard (Production Grade)

## Purpose
Use this guide to produce Product Requirements Documents (PRDs) that are ready for engineering implementation and PM approval at a high bar.

This document defines exactly how an LLM agent must work, what it must output, and the validation loop it must run before calling a PRD ready.

## Your Audience

Two groups will read this PRD:

1. **PM and stakeholders** — they need to understand what this feature is, why it matters, and who it impacts. They are not engineers. They need to walk away understanding the product decision.

2. **Engineers starting implementation** — they need to know exactly WHAT needs to be built so they can begin. Not HOW to build it — that comes in the Technical Design Doc. But they need enough detail to know the full scope of what the product needs to do.

## Your Job

Write a PRD that documents **what needs to happen within the product for users**. In great detail.

You are NOT writing a technical design document. The TDD comes later and covers how to implement things — API routes, database schemas, system architecture. Your job is the WHAT and the WHY:

- **What** is being built
- **Why** it matters (the justification)
- **Who** it impacts
- **What** the product needs to do for those people

## When to Use This Standard
Use this standard when asked to:
- Create a new PRD.
- Rewrite or improve an existing PRD.
- Evaluate whether a PRD is implementation-ready.
- Convert feature notes into a structured requirements document.

## Input Contract (Required Before Drafting)
Collect or infer the following inputs first:
- Product and feature name.
- Business objective and user outcome objective.
- Target users/personas.
- Known constraints (time, policy, legal, platform, dependencies).
- Launch expectation (alpha, beta, GA, internal only).

If any required input is missing:
1. Ask concise follow-up questions.
2. If asking is impossible, create explicit assumptions in an Assumption Register.

## Core Principles
1. Be outcome-first: describe what users can do, not how systems are built.
2. Be testable: each requirement must be objectively verifiable.
3. Be explicit: mark every assumption, dependency, and risk.
4. Be scoped: goals and non-goals must be clear and non-overlapping.
5. Be decision-ready: document must stand on its own without hidden context.

## Non-Negotiable Rules
1. Follow the workflow in order. Do not skip gates.
2. Do not mark a PRD `Ready for Implementation` until all validation passes.
3. Do not leave placeholders (`TBD`, `TBA`, `to be determined`) in required sections.
4. Keep implementation details out unless explicitly requested.
5. Every requirement must have a summary row in the Requirements Summary Table AND its own Requirement Detail Table.
6. Requirements must be written only in tables. Do not write requirement bullets or freeform requirement prose outside the summary and detail tables.
7. Every requirement must map to capability and metric IDs.
8. Every Requirement Detail Table must include: Requirement ID, Title, Requirement Statement, Who, Why, Priority, Capability IDs, Metric IDs, Interaction Notes, Cascade Notes, Acceptance Criteria, and Failure / Edge Criteria.
9. Record unknowns as assumptions with owner and due date.
10. Back problem and outcome claims with evidence or mark them as assumptions.
11. Record explicit approvers and sign-off status for PM, Engineering, Design (if applicable), and required control functions (Security/Legal/Privacy/Policy when applicable).
12. Do not proceed past Step 0 until the user explicitly confirms the scope-aligned brief.

## Writing About Users: Personas, Not Roles

Describe the real people who will be affected by this feature. Not system roles — real-world personas:
- Who are they in the real world? (e.g., "a sales leader sharing a pitch deck with investors", not "Team Admin")
- What are they trying to accomplish?
- What's their pain today without this feature?
- How does this feature change their experience?

Use user and role data from context to inform this, but write about people, not system classifications.

**In the requirements tables**, use system roles and plan tiers in the **Who** column — engineers need this to implement access controls. But in the Users section, write about personas.

## Requirements Format: Two-Tier Tables

Requirements are the heart of the PRD. They use a two-tier table format:

### Tier 1: Requirements Summary Table

A scannable overview of all requirements:

| ID | Title | Requirement | Who (roles/plans) | Why | Priority |
|---|---|---|---|---|---|
| REQ-001 | Short scan-friendly title | What the product needs to do — one clear sentence from the user's perspective | Which roles can use this and which plan tiers have access | The user need or business rule that drives this | P0/P1/P2 |

### Tier 2: Requirement Detail Tables

Each requirement gets its own detail table with everything in one place:

#### REQ-001: [short title]

| Field | Details |
|---|---|
| **Requirement ID** | REQ-001 |
| **Title** | Short scan-friendly title |
| **Requirement Statement** | What the product needs to do |
| **Who** | Roles and plans this applies to |
| **Why** | User need or business rule |
| **Priority** | P0/P1/P2 |
| **Capability IDs** | CAP-001 |
| **Metric IDs** | M-001 |
| **Interaction Notes** | How this interacts with existing features, settings, or controls |
| **Cascade Notes** | How this applies to related or nested objects, or `Not applicable` |
| **Acceptance Criteria** | 1. First testable condition |
| **Failure / Edge Criteria** | 1. First failure or edge condition, or `Not applicable` |

Rules for writing requirements:
- Write from the user's perspective. "Users can set an expiration date on any shared link" — not "The PATCH endpoint should accept an expiry timestamp field."
- **Who** must be explicit about roles AND plan tiers. Engineers need this to implement access controls correctly.
- Each acceptance criterion must be testable — a QA engineer can write a pass/fail test from the text.
- Each requirement must be unambiguous — only one interpretation is possible.
- Each requirement must be justified — tied to a user need or business rule.
- **Interaction Notes** must explain how the requirement behaves relative to existing features, settings, or controls.
- **Cascade Notes** must explain how the requirement applies to related or nested objects, or state `Not applicable`.
- Priority: P0 (must-have), P1 (should-have), P2 (nice-to-have).

Derive requirements from:
- What users need to accomplish (from user and feature data)
- Business rules and constraints that the product enforces today (especially plan limits and role-based permissions)
- Existing product capabilities that this feature extends or interacts with

## Output Language Constraints (Required)
- Use precise, testable language.
- Replace vague words unless quantified: `fast`, `easy`, `intuitive`, `robust`, `seamless`, `user-friendly`.
- Prefer single-sentence requirement statements.
- Use explicit numbers, thresholds, or time windows where relevant.
- Do not invent data. If the product context doesn't support a requirement, don't write it. A focused PRD with fewer well-grounded requirements is better than a padded one.

## Ambiguity Kill List (Rewrite Before Finalization)
- Bad: "The system should be fast."
- Good: "P95 response time for action X must be <=2.0s under Y load."
- Bad: "Users can easily complete onboarding."
- Good: "At least 85% of new users complete onboarding within 5 minutes on first attempt."
- Bad: "The feature should handle errors gracefully."
- Good: "When API timeout occurs, user sees retry CTA within 1s and no data loss occurs."

## Resolve Ambiguity

A good PRD makes decisions so engineers don't have to guess. Before you finish writing, pressure-test every requirement:

- **Who gets this?** If the product has different plans or roles, be explicit about which ones have access to this capability. An engineer should never have to guess whether to gate a feature.
- **How does this interact with what already exists?** The product already has features, rules, and controls. When the new feature touches existing ones, state what happens. If two settings could conflict, say which one wins.
- **If this applies to related or nested objects, how does it cascade?** Products have hierarchies (e.g., a parent object containing child objects). If the feature applies at one level, state whether it cascades down, and what happens when parent and child have different settings.

The test: if two engineers read this PRD independently and would build different things, the PRD has a gap. Find those gaps and close them.

If you can't resolve an ambiguity from the product context — say so. Add an **Open Questions** section for things that need a decision but don't have enough data to make one. An honest open question is better than a hidden assumption.

## ID Conventions (Required)
- Goal IDs: `G-001`, `G-002`, ...
- Metric IDs: `M-001`, `M-002`, ...
- Capability IDs: `CAP-001`, `CAP-002`, ...
- Requirement IDs: `REQ-001`, `REQ-002`, ...
- Assumption IDs: `A-001`, `A-002`, ...

## Mandatory Workflow

### Step 0: Brief Alignment Loop (Mandatory Before Drafting)
Take the user's raw input and create a structured interpretation before drafting any PRD content.

Process:
- Capture the raw user brief verbatim.
- Rewrite the brief into a structured interpretation that includes:
  - What is being built.
  - Who it is for.
  - Why it matters.
  - Intended user outcomes.
  - Explicit in-scope items.
  - Explicit out-of-scope items (or unknowns).
  - Key constraints already stated.
  - Key ambiguities/questions.
- Restate this interpretation back to the user in plain language.
- Ask for explicit confirmation or corrections.
- If corrections are provided, update the interpretation and restate again.
- Repeat until the user explicitly confirms the brief and scope.

Required confirmation pattern:
- The user must explicitly confirm (for example: "Yes, this is the brief and scope").
- Implicit agreement is not sufficient.

Gate:
- A user-confirmed scope-aligned brief exists.

### Step 1: Intake and Scope Framing
Capture:
- Request summary (1 sentence).
- Feature/initiative name.
- Release context (new feature, migration, optimization, launch).
- Constraints already known (timeline, legal, policy, platform, resourcing).

Gate:
- Scope is clear enough to judge what is in and out.

### Step 2: Understand the Product
Capture:
- Product name.
- Product tagline.
- Product one-line description.
- Product domain/category.
- Product surfaces (web, mobile, API, admin, extension, etc.).
- Product lifecycle stage (0->1, growth, maturity).
- Relevant entry points in the current user journey.

Rule:
- If data is missing, ask for it.
- If asking is impossible, record explicit assumptions with owner and due date.

Gate:
- Every field is either confirmed fact or labeled assumption.

### Step 3: Understand the Users
Identify:
- Primary persona (real-world person, not system role).
- Secondary persona(s), if materially impacted.
- Persona goals/jobs-to-be-done.
- Current pain points.
- Context of use (when, where, frequency, urgency, device).

Add at least one concrete persona scenario with:
- Role and environment.
- Trigger event.
- Current workaround.
- Cost of failure.

Gate:
- A specific unmet need is clear for the primary persona.

### Step 4: Define Problem and Outcomes
Write:
- Problem statement: current state -> pain -> consequence.
- Evidence notes for problem claims (analytics, user research, support data, or explicit assumption).
- Why now (user/business urgency).
- Desired user outcomes.
- Desired business outcomes.
- Success metrics with baseline (if known), target, and measurement window.

Metric requirements:
- Quantitative where possible.
- Use measurable units (time, conversion, completion, retention, error rate, etc.).
- Separate leading vs lagging indicators.
- Define at least one guardrail/counter-metric to detect harmful regressions.

Gate:
- Every goal maps to at least one measurable metric and at least one guardrail/counter-metric is defined for the PRD.

### Step 5: Define Solution as User Capabilities
Describe capabilities only in user-observable terms.

For each capability include:
- Capability ID and name.
- Target persona(s).
- Trigger/context.
- User action.
- User-visible result.
- Guardrails/constraints.

Do not include:
- API design.
- Database/schema design.
- Framework/library choices.
- Internal architecture.

Gate:
- Capabilities are implementation-agnostic and user-visible.

### Step 6: Convert Capabilities into Requirements (Two-Tier Tables)

First, create the **Requirements Summary Table** with all requirements in a single scannable table:

| ID | Title | Requirement | Who (roles/plans) | Why | Priority |
|---|---|---|---|---|---|

Then, create a **Requirement Detail Table** for each requirement. All requirement-specific content must live in these tables.

#### REQ-001: [short title]

| Field | Details |
|---|---|
| **Requirement ID** | REQ-001 |
| **Title** | Short scan-friendly title |
| **Requirement Statement** | What the product needs to do |
| **Who** | Roles and plans this applies to |
| **Why** | User need or business rule |
| **Priority** | P0/P1/P2 |
| **Capability IDs** | CAP-001 |
| **Metric IDs** | M-001 |
| **Interaction Notes** | How this interacts with existing features, settings, or controls |
| **Cascade Notes** | How this applies to related or nested objects, or `Not applicable` |
| **Acceptance Criteria** | 1. First testable condition |
| **Failure / Edge Criteria** | 1. First failure or edge condition, or `Not applicable` |

Quality rules:
- No vague terms unless quantified.
- Include actor, condition, action, expected result.
- Keep each requirement independently testable.
- Keep each requirement atomic (one behavior per requirement statement).
- Keep P0 scope disciplined: if more than 40% of requirements are P0, provide explicit justification.
- **Who** column must specify both roles AND plan tiers explicitly.
- Acceptance criteria must be objective pass/fail checks.
- Failure / Edge Criteria must be explicit where relevant, and `Not applicable` where not relevant.
- Interaction and cascade behavior must be captured in the table, not left to prose elsewhere.

Gate:
- Every requirement has a summary row AND a detail table, and all requirement-specific content lives in those tables.

### Step 7: Scope Boundaries
Define:
- In-scope items.
- Non-goals.
- Out-of-scope edge cases.
- Release assumptions/constraints.

Gate:
- Non-goals prevent scope creep and do not conflict with goals.

### Step 8: Technical Considerations
High-level notes for the engineering team — not implementation details, but things they should know:
- What platforms this needs to work on.
- What existing product areas this touches.
- What integrations are relevant.
- What data the feature needs to read or write.

Gate:
- Engineers have enough context to begin technical design without guessing about product surface or data needs.

### Step 9: Risks, Dependencies, and Open Questions
Document:
- Top risks (product, legal, compliance, adoption, operational) in a table format.
- Severity and mitigation for each risk.
- Dependency owners, impact, and critical-path status (`Blocking` or `Non-blocking`).
- Open questions that block sign-off.

Gate:
- High-severity risks have owner + mitigation path.

### Step 10: Safety, Privacy, and Policy Requirements
For user-facing or data-processing features, document:
- User harm risks and misuse/abuse risks.
- Privacy/data handling constraints (collection, retention, deletion, access).
- Policy/compliance constraints and review requirements.
- Safety mitigations and abuse prevention controls.
- Incident response owner for post-launch safety/privacy issues.

If the feature is not safety/privacy/policy relevant, explicitly state why.

Gate:
- Safety/privacy/policy impact is explicitly assessed and either addressed or justified as not applicable.

### Step 11: Rollout and Validation Plan
Define:
- Rollout phases (alpha/beta/GA).
- Eligibility and controls.
- Monitoring signals.
- Rollback triggers.
- Rollback execution owner and rollback execution steps.
- Validation activities (user testing, analytics checks, support signals).
- Pre-launch evaluation matrix (happy path, failure path, edge cases, policy/safety checks).
- Phase entry/exit criteria per rollout phase.
- Post-launch monitoring cadence and owner.

Gate:
- Includes explicit go/no-go criteria.
- Includes phase entry/exit criteria and named owner for monitoring cadence.

### Step 12: Approval and Sign-Off Readiness
Document:
- Required approvers by role.
- Approval status (`Pending`, `Approved`, `Not Required`) per role.
- Blocking approvals still open.
- Decision owner for final ship/no-ship call.

For pre-seed teams where the founder holds all roles, a single explicit founder approval satisfies all approver slots in the approval matrix.

Gate:
- No required approver is missing from the approval matrix.

## Output Contract (Required Sections)
Use this exact order:
1. Document Control
2. Executive Summary
3. Scope Alignment Brief (User Confirmed)
4. Product Context
5. Users and Needs
6. Problem Statement
7. Evidence and Source Notes
8. Goals, Non-Goals, and Success Metrics
9. Solution Overview (User Capabilities)
10. Requirements (Summary Table + Detail Tables)
11. Scope Boundaries
12. Technical Considerations
13. Risks and Mitigations
14. Dependencies
15. Assumption Register
16. Traceability Matrix
17. Safety, Privacy, and Policy Requirements
18. Rollout and Validation Plan
19. Approval Matrix
20. Open Questions
21. Implementation Readiness Verdict
22. Validation Report
23. Final Preflight Checklist
24. Review Iteration Log (5 passes)

Use `references/prd-template.md` when drafting.

## Five-Pass Expert Review Loop (Mandatory)
Run these five passes in order. Revise after each pass.

### Pass 1: Structure and Completeness
Check:
- All required sections exist.
- No placeholder text remains.
- Product, user, and problem context are specific.
- Scope Alignment Brief includes explicit user confirmation text.
- Users section describes real-world personas, not system roles.

Fail examples:
- Missing product surfaces.
- Missing persona scenario trigger/workaround/cost.
- Users described only as system roles ("Admin", "Viewer") without real-world context.

### Pass 2: Requirement Quality
Check each requirement for:
- Summary row exists in the Requirements Summary Table.
- Detail table exists with all fields (Requirement ID, Title, Requirement Statement, Who, Why, Priority, Capability IDs, Metric IDs, Interaction Notes, Cascade Notes, Acceptance Criteria, Failure / Edge Criteria).
- Requirement content appears only in the summary/detail tables, not in freeform bullets or prose.
- Who column specifies both roles AND plan tiers.
- Acceptance criteria are objective and independently testable.
- Failure / Edge Criteria are explicit where relevant.
- Traceability to capability and metric.

Fail examples:
- Vague adjectives without numbers.
- Multiple behaviors in one requirement.
- Missing Who column or Who column without plan tier.
- Requirement content written outside the requirement tables.
- Requirement in summary table but no detail table (or vice versa).

### Pass 3: Scope and Prioritization
Check:
- P0/P1/P2 priorities are coherent.
- Non-goals prevent likely scope creep.
- Deferred items are explicit.
- P0 ratio is <=40%, or explicit justification is documented.

Fail examples:
- Priority inflation (too many P0s).
- Non-goals that conflict with goals.

### Pass 4: Risk and Delivery Readiness
Check:
- Critical risks have owners and mitigations in table format.
- Dependencies have owners, due dates, and critical-path status.
- Rollout has measurable go/no-go and rollback criteria with rollback owner.
- Evaluation matrix covers happy path, failure path, edge cases, and policy/safety checks.
- Technical Considerations section gives engineers enough context.

Fail examples:
- High-severity risk with no owner.
- Dependency with unknown owner, unknown impact, or missing critical-path status.
- No Technical Considerations section.

### Pass 5: Sign-Off Simulation
All answers must be `Yes`:
- Is the problem clear and evidence-based?
- Is the solution capability-based and implementation-agnostic?
- Could independent teams implement equivalent behavior from this PRD?
- Are requirements testable without private context?
- Are risks/dependencies explicit and actionable?
- Are success metrics measurable and mapped?
- Do non-goals control scope drift?
- Are ambiguities resolved (who gets this, how it interacts, how it cascades)?

If any pass fails, revise and restart from the failed pass.

## Final Preflight Checklist (Deterministic, Mandatory)
All checks must pass:
- ID consistency: no duplicate IDs and no broken traceability references.
- Required dates use `YYYY-MM-DD`.
- Approval matrix: no required approver remains `Pending` (founder approval satisfies all roles for pre-seed teams).
- Scope brief confirmation check: explicit user confirmation is recorded.
- No unresolved blocker-level open question.
- No unresolved high-impact, low-confidence assumption.
- No placeholder text in required sections.
- P0 ratio check passes (or justification exists).
- Ambiguity violations count is zero.
- Requirements Summary Table and Detail Tables are consistent (same IDs, same titles).
- Every requirement's Who column includes both roles and plan tiers.

If any preflight check fails, set verdict to `Draft - Needs Revision` or `Draft - Blocked`.

## Validation Scoring
Score using `references/validation-rubric.md`.

Readiness threshold:
- Must score `>=16/18`.

## Definition of Done
A PRD is implementation-ready only when:
- All workflow gates pass.
- All five review passes pass.
- Final preflight checklist passes.
- Rubric score is `>=16/18`.
- No blocker-level open questions remain.
- No high-impact, low-confidence assumptions remain unresolved.
- All required approvers are explicitly marked `Approved` (founder approval satisfies all roles for pre-seed teams).

If blockers remain, set verdict:
- `Draft - Blocked`
