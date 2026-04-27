# Phase 4: Implementation and Testing

## Purpose
The agent implements the feature by invoking the `/application-development` skill, following the design from Phase 3. The agent runs all migrations, confirms the build passes, and verifies all E2E tests from Phase 2 pass.

## Input
The agent must read the following before invoking the skill:
- The status file — `.docs/{project-name}-status.md`.
- The PRD — `.docs/{project-name}-prd.md`.
- The test files from Phase 2.
- The design document from Phase 3 — `.docs/{project-name}-design.md`.
- The rules directory — every rule file.

## Work

### Database migrations:
If the design document specifies database changes, the agent must run migrations before writing application code:
- The agent must write and run the migration SQL against the development database.
- The agent must verify the migration completed successfully by querying the new tables or columns.
- The agent must verify RLS policies are active on every new table.
- The agent must not proceed with implementation until migrations are confirmed.

### Implementation:
The agent must invoke `/application-development` with the following context:
- The design document as the implementation plan.
- The E2E test files as the acceptance criteria.
- The rules directory as the coding constraints.

The `/application-development` skill handles the implementation process. The agent must not duplicate the skill's work.

### Phase-specific constraints:
The agent must verify the implementation meets these additional constraints:
- The agent must implement one PRD requirement at a time.
- After implementing each requirement, the agent must run the corresponding E2E test from Phase 2.
- The agent must not move to the next requirement until the current requirement's test passes.
- The agent must run lint after each requirement and fix all warnings — zero warnings at all times.
- The agent must commit after each passing requirement with a descriptive commit message.
- The agent must not deviate from the design document without updating the design document first and getting user approval.

### Final verification:
After all requirements are implemented:
- `npm run build` must complete with zero errors.
- `npm run dev` must start without runtime errors.
- All E2E tests from Phase 2 must pass.
- TypeScript strict compilation must pass with zero errors.
- Lint must pass with zero warnings in all changed files.

## Output
- Working feature code committed to the feature branch.
- All migrations run and confirmed.
- All E2E tests passing.
- Build and dev server confirmed working.
- The status file updated with Phase 4 marked as IN PROGRESS.

## Gate
The `/application-development` skill completes successfully. All E2E tests pass. `npm run build` succeeds. `npm run dev` starts without errors. TypeScript strict passes. Lint passes with zero warnings. All migrations are confirmed. The user approves the implementation before the agent proceeds to Phase 5.
