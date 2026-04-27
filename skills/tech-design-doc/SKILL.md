---
name: tech-design-doc
description: Create production-grade Technical Design Documents that map directly to PRD requirements. Enforces zero custom code, SDK/framework-first design, PRD traceability, architecture diagrams, and framework research with documentation references. Use when designing a feature's architecture before implementation.
---

# Technical Design Doc

- Create or review TDDs using the mandatory standards in this skill.
- You MUST save all TDD artifacts in `.docs/`.
- A PRD is REQUIRED before writing a TDD. If no PRD exists, you MUST create one first using the `prd` skill.
- You MUST read the PRD in full before designing. Do not ask the user for information that is already in the PRD.
- As you work through the project, you MUST keep the TDD up to date.

## Required Resources

Always load and follow these files in this order:
1. `references/tdd-standard.md`
2. `references/tdd-template.md`
3. `references/validation-rubric.md`

`references/tdd-standard.md` is the source of truth for workflow, gates, and definition of done.

## Execution Contract

1. Read the PRD first. Every design decision must trace back to a PRD requirement.
2. Gather framework/SDK documentation for every technology the design will use.
3. Identify what already exists in the codebase that can be reused.
4. Design using ONLY first-party SDK/framework capabilities — zero custom code unless explicitly justified.
5. Include an ASCII architecture diagram.
6. Reference official documentation for every framework/SDK decision.
7. Prefer server-side execution for all logic.
8. Draft output using the exact section order from `references/tdd-template.md`.
9. Run the validation rubric from `references/validation-rubric.md`.
10. Do not mark `Ready for Implementation` unless all readiness gates pass.

## Output Rules

- Every design section must trace to PRD requirement IDs.
- No custom code: use SDK/framework built-in capabilities. Glue code for integration is acceptable.
- No fallbacks: design for the correct path, not defensive workarounds.
- Error logging must be explicit for every failure point.
- Security considerations are mandatory for every TDD.
- Server-side by default for all logic.
- Reference official docs (URLs or doc section names) for every SDK/framework usage.

## Failure Handling

If no PRD exists:
- You MUST create one first using the `prd` skill before proceeding with the TDD.
- Do not design without requirements. A TDD without a PRD is not valid.

If the PRD is incomplete:
- Stop and ask the user to complete the PRD before proceeding.

If a framework/SDK cannot fulfill a requirement:
- Document the gap explicitly with the specific capability that is missing.
- Reference the official docs showing the limitation.
- Propose minimal glue code with justification.

If any validation check fails:
- Return `Draft - Needs Revision` (or `Draft - Blocked` for blocking issues).
- List failed checks and concrete fixes in the validation report.
