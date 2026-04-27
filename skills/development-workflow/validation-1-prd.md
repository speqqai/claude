# Phase 1: PRD — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 1 PRD output. You must return PASS or FAIL with specific reasons.

## What to review
Read the PRD at `.docs/{project-name}-prd.md`.
Read the Phase 0 status file at `.docs/{project-name}-status.md` to confirm the PRD covers the stated project scope.

## Validation checklist

### Problem statement:
- [ ] The PRD contains a clear problem statement.
- [ ] The problem statement describes a user pain point, not a technical gap.

### Personas:
- [ ] The PRD describes target personas as real-world people, not system roles.
- [ ] Each persona description includes what the persona does, why the persona uses Speqq, and what the persona needs from this feature.

### Requirements:
- [ ] Every functional requirement has a testable acceptance criterion.
- [ ] Each acceptance criterion can be verified by observing user-visible behavior — not by reading code or database state.
- [ ] The requirements specify which roles and plan tiers the feature applies to.

### User flows:
- [ ] The PRD contains step-by-step user flows.
- [ ] Each user flow starts with a user action and ends with a user-visible result.

### Edge cases:
- [ ] The PRD lists edge cases and error scenarios.
- [ ] Each edge case describes what happens when the user encounters the edge case.

### Out of scope:
- [ ] The PRD contains an out-of-scope section.
- [ ] The out-of-scope section explicitly names what the feature does not do.

### No implementation details:
- [ ] The PRD does not contain database column names or table schemas.
- [ ] The PRD does not contain API endpoint designs or route signatures.
- [ ] The PRD does not contain component names or file paths.
- [ ] The PRD does not contain architecture decisions.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
