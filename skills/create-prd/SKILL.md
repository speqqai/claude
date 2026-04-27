---
name: prd
description: Produce production-grade Product Requirements Documents (PRDs) that are implementation-ready for PM and engineering review. Use when asked to create, rewrite, improve, or evaluate a PRD, feature spec, requirements document, or launch-ready product plan with clear user outcomes, testable requirements, explicit approvals, and deterministic validation gates.
---

# PRD

- Create or review PRDs using the mandatory standards in this skill.
- You MUST save all PRD artifacts in `.docs/`.
- As you work through the project, you MUST keep the PRD up to date.

## Required Resources

Always load and follow these files in this order:
1. `references/prd-standard.md`
2. `references/prd-template.md`
3. `references/validation-rubric.md`

`references/prd-standard.md` is the source of truth for workflow, gates, and definition of done.

## Execution Contract

1. Run Step 0 (Brief Alignment Loop) from `references/prd-standard.md` before drafting any PRD content.
2. Restate the user's raw brief as a structured interpretation and get explicit user confirmation.
3. Repeat the restatement/correction cycle until the user explicitly confirms scope.
4. Follow the remaining workflow and all non-negotiable rules in `references/prd-standard.md`.
5. Draft output using the exact section order from `references/prd-template.md`.
6. Requirements MUST be written only in the canonical two-tier table format: one Requirements Summary Table plus one Requirement Detail Table per requirement.
7. Do not write requirements as bullets, prose paragraphs, or mixed formatting outside those tables.
8. Every Requirement Detail Table MUST include: Requirement ID, Title, Requirement Statement, Who, Why, Priority, Capability IDs, Metric IDs, Interaction Notes, Cascade Notes, Acceptance Criteria, and Failure / Edge Criteria.
9. Users section MUST describe real-world personas, not system roles.
10. Every requirement's Who field MUST specify both roles AND plan tiers.
11. Run the five-pass expert review loop from the standard.
12. Run the final preflight checklist from the standard.
13. Score with the rubric in `references/validation-rubric.md`.
14. Do not mark `Ready for Implementation` unless all readiness gates pass.

## Output Rules

- Write for two audiences: PM/stakeholders (product decision) and engineers (implementation scope).
- Keep solution definition focused on user capabilities, not implementation details.
- Make requirements explicit, atomic, and testable.
- Present requirements only in the two-tier table format (summary table + detail tables).
- Keep IDs and traceability consistent (`G-xxx`, `M-xxx`, `CAP-xxx`, `REQ-xxx`, `A-xxx`).
- Record assumptions, approvals, and unresolved blockers explicitly.
- Resolve ambiguity before finishing: who gets this, how it interacts, how it cascades.
- If blockers remain, return `Draft - Blocked`.

## Failure Handling

If required inputs are missing:
- Ask concise follow-up questions.
- If asking is impossible, record explicit assumptions in the Assumption Register with owner, due date, impact level, and confidence.

If scope confirmation is not explicit:
- Keep the PRD in alignment mode and continue the brief restatement loop.
- Do not proceed to drafting or requirements decomposition.

If any validation or preflight check fails:
- Return `Draft - Needs Revision` (or `Draft - Blocked` for blocking issues).
- List failed checks and concrete fixes in the validation report.
