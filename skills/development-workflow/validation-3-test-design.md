# Phase 2: Test Design — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 2 test design output. You must return PASS or FAIL with specific reasons.

## What to review
Read the PRD at `.docs/{project-name}-prd.md`.
Read every new or modified test file created during this phase.
Read the existing test infrastructure to verify the new tests follow existing patterns.

## Validation checklist

### PRD coverage:
- [ ] Every functional requirement in the PRD has at least one working and corresponding E2E test.
- [ ] Each test name maps clearly to a specific PRD requirement — the mapping is obvious from the test name alone.

### Unauthenticated tests:
- [ ] The test suite includes tests that run without an authenticated session.
- [ ] Unauthenticated tests verify that protected pages redirect to login.
- [ ] Unauthenticated tests verify that public pages render correctly without a session.

### Authenticated tests:
- [ ] The test suite includes tests that run with an authenticated session.
- [ ] Authenticated tests cover the happy path for every PRD requirement.
- [ ] Authenticated tests cover error states described in the PRD.
- [ ] Authenticated tests cover empty states described in the PRD.
- [ ] Authenticated tests cover edge cases described in the PRD.

### Test quality:
- [ ] Test file names are descriptive and domain-qualified — not generic names like `test1` or `feature-test`.
- [ ] Test descriptions are descriptive — a reader can understand what the test validates without reading the test body.
- [ ] Tests do not use mocked services — tests run against the real application stack.
- [ ] Tests do not use custom test utilities or helpers that duplicate existing test infrastructure.

### No bloat:
- [ ] No new test dependencies were added to package.json (or the additions were explicitly approved by the user).
- [ ] Test files live in the existing test directory structure — no new test directory pattern was created.
- [ ] No custom test abstractions were created that duplicate existing test utilities.

### Test execution:
- [ ] All tests can be run with the existing test runner command.
- [ ] All tests fail when run (expected — implementation does not exist yet).

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
