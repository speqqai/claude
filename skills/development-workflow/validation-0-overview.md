# Phase 0: Overview — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 0 overview output. You must return PASS or FAIL with specific reasons.

## What to review
Read the status file at `.docs/{project-name}-status.md`.

## Validation checklist

### Project understanding:
- [ ] The status file contains a 2–3 sentence summary of what the feature does.
- [ ] The summary explains why the feature matters to Speqq users.
- [ ] The summary identifies the user problem the feature solves.
- [ ] The summary lists the product surfaces the feature will touch.

### Affected files and directories:
- [ ] The status file lists existing files that the feature will modify.
- [ ] The status file lists new files that the feature will create.
- [ ] The status file lists existing database tables the feature will query or modify.
- [ ] The status file lists new database tables or columns the feature will require.
- [ ] Every listed file path is a real path that exists in the codebase (verify with glob or ls).

### Domains touched:
- [ ] The status file lists every domain the feature involves.
- [ ] Each domain has a specific framework, SDK, or library named — not a generic term like "approved tool."
- [ ] Each named framework, SDK, or library exists in package.json (verify by reading package.json).

### Phase tracking:
- [ ] The status file contains a checklist of all 7 phases (0 through 6).
- [ ] Each phase has a status value (NOT STARTED, IN PROGRESS, BLOCKED, PASS, or DONE).
- [ ] Phase 0 is marked as IN PROGRESS or PASS.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
