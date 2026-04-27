# Phase 2: Test Design

## Purpose
The agent writes end-to-end tests by invoking the `/test-driven-development` skill. The tests must cover every PRD requirement with both unauthenticated and authenticated paths, using the existing test infrastructure.

## Input
The agent must read the following before invoking the skill:
- The status file — `.docs/{project-name}-status.md`.
- The PRD — `.docs/{project-name}-prd.md`.
- The existing test infrastructure — find existing test files, test config, and test utilities already in the codebase.

## Work
The agent must invoke `/test-driven-development` with the following context:
- The PRD requirements and acceptance criteria.
- The existing test infrastructure and patterns found in the codebase.
- The instruction to write E2E tests only — not unit tests, not integration tests.

The `/test-driven-development` skill handles the test creation process. The agent must not duplicate the skill's work by writing test logic manually.

## Phase-specific constraints
The agent must verify the `/test-driven-development` output meets these additional constraints:

**Unauthenticated tests must exist:**
- Protected pages redirect unauthenticated users to the login page.
- Public pages render correctly without a session.
- Public API routes return appropriate responses without a session.

**Authenticated tests must exist:**
- The user can log in and reach the feature surface.
- Every PRD requirement works as described in the acceptance criteria.
- Error states display the correct message when operations fail.
- Empty states display correctly when no data exists.
- Loading states appear while data is being fetched.

**No extra bloat:**
- No new test dependencies were added if the existing test infrastructure handles the need.
- No custom test utilities were created that duplicate existing test infrastructure.
- No mock services were created — E2E tests run against the real application stack.
- Test files live in the existing test directory structure.

## Output
- E2E test files in the existing test directory.
- All tests fail when run (expected — implementation code does not exist yet).
- The status file updated with Phase 2 marked as IN PROGRESS.

## Gate
The `/test-driven-development` skill completes successfully. Every PRD requirement has a corresponding E2E test. Every test suite includes unauthenticated and authenticated paths. All tests fail (expected). No new test dependencies were added without user approval. The user approves the test design before the agent proceeds to Phase 3.
