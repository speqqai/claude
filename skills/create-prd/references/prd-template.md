# Production PRD Template

## 1. Document Control
- PRD Title:
- Feature/Initiative:
- Owner:
- Reviewers:
- Status: Draft | In Review | Ready for Implementation | Draft - Blocked
- Last Updated:
- Target Release:

## 2. Executive Summary
What this feature is in plain language. Why it matters. Who benefits. One paragraph.

## 3. Scope Alignment Brief (User Confirmed)
- User raw brief (verbatim):
- Agent interpreted brief (structured):
  - What is being built:
  - Who it is for:
  - Why it matters:
  - Intended user outcomes:
  - In-scope items:
  - Out-of-scope items or unknowns:
  - Constraints:
  - Ambiguities/questions:
- User confirmation statement (verbatim):
- Confirmation timestamp:

## 4. Product Context
- Product Name:
- Tagline:
- Product Description:
- Category/Domain:
- Surfaces:
- Lifecycle Stage:
- Relevant Existing User Journey:

How this feature fits into the bigger picture. Help the reader understand where this feature lives within the product.

## 5. Users and Needs

Describe the real people who will be affected by this feature. Not system roles — real-world personas.

### 5.1 Primary Persona
- Persona Name: (real-world description, e.g., "Deal Coordinator / Project Manager")
- Role/Environment: Who they are in the real world
- Jobs-to-be-Done: What they are trying to accomplish
- Pain Points: What's their pain today without this feature
- How this feature changes their experience:

### 5.2 Secondary Persona(s)
- Persona Name:
- Role/Environment:
- Why impacted:
- Pain Points:
- How this feature changes their experience:

### 5.3 Concrete User Scenarios
- Scenario 1:
  - Trigger:
  - Current workaround:
  - Cost of failure:
- Scenario 2 (optional):

## 6. Problem Statement
- Current state:
- Pain and consequence:
- Why now:

## 7. Evidence and Source Notes
- Claim:
- Source type: Analytics | User Research | Support Tickets | Sales Feedback | Assumption
- Evidence detail:
- Confidence: High | Medium | Low

## 8. Goals, Non-Goals, and Success Metrics
### 8.1 Goals
- Goal G-001:
- Goal G-002:

### 8.2 Non-Goals
- Explicit out-of-scope list:

### 8.3 Success Metrics
- Metric M-001: Definition, baseline, target, window.
- Metric M-002: Definition, baseline, target, window.
- Leading indicators:
- Lagging indicators:
- Guardrail/counter-metrics:

## 9. Solution Overview (User Capabilities)
### Capability CAP-001: [Name]
- Target persona(s):
- Trigger/context:
- User action:
- User-visible result:
- Constraints/guardrails:

### Capability CAP-002: [Name]
- Target persona(s):
- Trigger/context:
- User action:
- User-visible result:
- Constraints/guardrails:

## 10. Requirements

All requirements must be written in tables only. Do not add freeform requirement bullets or prose outside the summary and detail tables.

### 10.1 Requirements Summary Table

| ID | Title | Requirement | Who (roles/plans) | Why | Priority |
|---|---|---|---|---|---|
| REQ-001 | [Short scan-friendly title] | [One clear sentence from user's perspective] | [Roles + plan tiers] | [User need or business rule] | P0/P1/P2 |
| REQ-002 | | | | | |
| REQ-003 | | | | | |

---

### 10.2 Requirement Detail Tables

#### REQ-001: [short title]

| Field | Details |
|---|---|
| **Requirement ID** | REQ-001 |
| **Title** | [Short scan-friendly title] |
| **Requirement Statement** | [What the product needs to do] |
| **Who** | [Roles and plans this applies to] |
| **Why** | [User need or business rule] |
| **Priority** | P0/P1/P2 |
| **Capability IDs** | CAP-001 |
| **Metric IDs** | M-001 |
| **Interaction Notes** | [How this interacts with existing features, settings, or controls] |
| **Cascade Notes** | [How this applies to related or nested objects, or `Not applicable`] |
| **Acceptance Criteria** | 1. [First testable condition] |
| **Failure / Edge Criteria** | 1. [First failure or edge condition, or `Not applicable`] |

---

#### REQ-002: [short title]

| Field | Details |
|---|---|
| **Requirement ID** | REQ-002 |
| **Title** | [Short scan-friendly title] |
| **Requirement Statement** | [What the product needs to do] |
| **Who** | [Roles and plans this applies to] |
| **Why** | [User need or business rule] |
| **Priority** | P0/P1/P2 |
| **Capability IDs** | CAP-001 |
| **Metric IDs** | M-001 |
| **Interaction Notes** | [How this interacts with existing features, settings, or controls] |
| **Cascade Notes** | [How this applies to related or nested objects, or `Not applicable`] |
| **Acceptance Criteria** | 1. [First testable condition] |
| **Failure / Edge Criteria** | 1. [First failure or edge condition, or `Not applicable`] |

---

## 11. Scope Boundaries
**In-scope:**
- What this feature covers

**Out-of-scope:**
- What it explicitly does NOT cover, and why

**Deferred Edge Cases:**
- Edge cases acknowledged but deferred

**Release Constraints:**
- Timeline, resource, or platform constraints

## 12. Technical Considerations
High-level notes for the engineering team — not implementation details, but things they should know:
- **Platforms:** What platforms this needs to work on
- **Surfaces touched:** What existing product areas this touches
- **Data model:** What data the feature needs to read or write (conceptual, not schema)
- **Integrations:** What integrations are relevant
- **Business rules:** What existing rules or constraints the feature must respect

## 13. Risks and Mitigations

| Risk | Severity | Owner | Mitigation |
|---|---|---|---|
| [What could go wrong] | High/Medium/Low | [Owner] | [How to mitigate] |
| | | | |

## 14. Dependencies
- Dependency:
- Owner:
- Critical-path status: Blocking | Non-blocking
- Due date (YYYY-MM-DD):
- Impact if delayed:

## 15. Assumption Register
- Assumption ID (A-xxx):
- Statement:
- Impact if false:
- Impact level: High | Medium | Low
- Owner:
- Due date (YYYY-MM-DD):
- Confidence: High | Medium | Low

## 16. Traceability Matrix
- Goal [G-xxx] -> Metric [M-xxx] -> Capability [CAP-xxx] -> Requirement [REQ-xxx]

## 17. Safety, Privacy, and Policy Requirements
- Safety risk summary:
- Misuse/abuse risk summary:
- Privacy/data handling constraints:
- Policy/compliance requirements:
- Mitigations and controls:
- Incident response owner:
- Not-applicable justification (if applicable):

## 18. Rollout and Validation Plan
- Phases:
- Phase entry criteria:
- Phase exit criteria:
- Eligibility:
- Monitoring signals:
- Monitoring cadence:
- Monitoring owner:
- Rollback triggers:
- Rollback owner:
- Rollback execution steps:
- Validation activities:
- Pre-launch evaluation matrix:
  - Happy path checks:
  - Failure path checks:
  - Edge case checks:
  - Policy/safety checks:
- Go/No-go criteria:

## 19. Approval Matrix
- PM approver: Pending | Approved | Not Required
- Engineering approver: Pending | Approved | Not Required
- Design approver: Pending | Approved | Not Required
- Security approver: Pending | Approved | Not Required
- Legal/Privacy/Policy approver(s): Pending | Approved | Not Required
- Final ship/no-ship decision owner:

## 20. Open Questions
- Question:
- Owner:
- Due date (YYYY-MM-DD):
- Impact if unresolved:

## 21. Implementation Readiness Verdict
- Verdict: Ready for Implementation | Draft - Needs Revision | Draft - Blocked
- Blockers (if any):
- Unresolved high-impact, low-confidence assumptions:

## 22. Validation Report
- Pass 1 (Structure and Completeness): Pass | Fail
- Pass 2 (Requirement Quality): Pass | Fail
- Pass 3 (Scope and Prioritization): Pass | Fail
- Pass 4 (Risk and Delivery Readiness): Pass | Fail
- Pass 5 (Sign-Off Simulation): Pass | Fail
- Pass 5 answers:
  - Problem clarity: Yes | No
  - Capability focus: Yes | No
  - Implementation consistency: Yes | No
  - Requirement testability: Yes | No
  - Risks/dependencies explicit: Yes | No

## 23. Final Preflight Checklist
- ID consistency: Pass | Fail
- Required dates use `YYYY-MM-DD`: Pass | Fail
- Approval matrix complete: Pass | Fail
- Scope confirmation recorded: Pass | Fail
- No blocker-level open question remains: Pass | Fail
- No unresolved high-impact, low-confidence assumption remains: Pass | Fail
- No placeholder text remains in required sections: Pass | Fail
- P0 ratio check passes or justification exists: Pass | Fail
- Ambiguity violations count is zero: Pass | Fail
- Requirements Summary Table and Detail Tables are consistent: Pass | Fail
- Every requirement's Who field includes roles and plan tiers: Pass | Fail

## 24. Review Iteration Log
- Pass 1 summary:
- Pass 2 summary:
- Pass 3 summary:
- Pass 4 summary:
- Pass 5 summary:
  - Metrics quality: Yes | No
  - Scope control: Yes | No
  - Ambiguity resolution: Yes | No
- Rubric score (0-18):
- P0 ratio and justification:
- Ambiguity violations count:
- Weak areas + fixes:
