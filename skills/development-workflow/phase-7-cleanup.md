# Phase 6: Cleanup

## Purpose
The agent removes all dead code, orphaned files, and old patterns by invoking `/code-review` to identify issues and `/pr-preflight` to verify the final state. The codebase must be cleaner after the feature ships than before the feature started.

## Input
The agent must read the following before invoking the skills:
- The status file — `.docs/{project-name}-status.md`.
- The design document — `.docs/{project-name}-design.md`.
- The git diff of all changes on the feature branch.

## Work

### Step 1: Identify issues
The agent must invoke `/code-review` with the following context:
- The full git diff of the feature branch.
- The instruction to focus on dead code, orphaned files, unused imports, old patterns, and debug logging.

The `/code-review` skill handles the review process. The agent must fix every issue the skill identifies.

### Step 2: Delete dead code
Based on the `/code-review` output, the agent must delete:
- Functions, methods, hooks, or components that are no longer called by any file in the codebase.
- Variables or constants that are no longer referenced by any file in the codebase.
- Types or interfaces that are no longer imported by any file in the codebase.
- Commented-out code in any file that was touched during this workflow.
- `console.log`, `console.debug`, or debug logging statements added during development.

### Step 3: Delete orphaned files
The agent must delete:
- Files that are no longer imported by any other file in the codebase.
- Test files that test functions or components that no longer exist.
- Documentation files that describe features or behavior that no longer exist.

### Step 4: Delete old patterns
If the new feature replaces an old pattern or old implementation:
- The agent must delete the old implementation files.
- The agent must update every file that imported from the old implementation to import from the new implementation.
- The agent must verify that no references to the old implementation remain in the codebase.

### Step 5: Final preflight
The agent must invoke `/pr-preflight` to run the full pre-PR checklist. The `/pr-preflight` skill handles the verification process. Every check must pass.

## Output
- All dead code, orphaned files, old patterns, unused imports, and debug logging removed.
- `/pr-preflight` passes all checks.
- The status file updated with Phase 6 marked as IN PROGRESS.

## Gate
The `/code-review` skill completes and all identified issues are fixed. The `/pr-preflight` skill passes all checks. `npm run build` succeeds. All E2E tests pass. Lint passes with zero warnings. The user approves the cleanup and marks the project as DONE.
