# Phase 6: Cleanup — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 6 cleanup output. You must return PASS or FAIL with specific reasons.

## What to review
Read the git diff of all changes on the feature branch.
Read every file that was modified or deleted during this phase.
Run the verification commands listed below.

## Validation checklist

### Dead code:
- [ ] No functions, methods, hooks, or components exist in changed files that are not called by any other file in the codebase (verify with grep).
- [ ] No variables or constants exist in changed files that are not referenced by any other file in the codebase (verify with grep).
- [ ] No types or interfaces exist in changed files that are not imported by any other file in the codebase (verify with grep).
- [ ] No commented-out code exists in any file that was touched during this workflow.
- [ ] No `console.log`, `console.debug`, or debug logging statements exist in any file that was touched during this workflow (production error logging is acceptable).

### Orphaned files:
- [ ] No files exist that are not imported by any other file in the codebase (verify with grep for the file name or exports).
- [ ] No test files exist that test functions or components that no longer exist.

### Old patterns:
- [ ] If the feature replaced an old implementation, the old implementation files have been deleted.
- [ ] No references to deleted files or old implementation names remain in the codebase (verify with grep).

### Imports:
- [ ] No unused imports exist in any file touched during this workflow.
- [ ] No duplicate imports exist in any file touched during this workflow.

### Build and runtime:
- [ ] `npm run build` completes with zero errors.
- [ ] `npm run dev` starts without runtime errors.
- [ ] All E2E tests from Phase 2 pass.
- [ ] TypeScript strict compilation passes with zero errors.
- [ ] Lint passes with zero warnings in all changed files.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
