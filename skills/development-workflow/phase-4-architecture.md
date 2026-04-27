# Phase 3: Architecture Design

## Purpose
The agent produces a technical design document by invoking the `/tech-design-doc` skill. The design must describe how the feature fits into the existing application with zero unnecessary bloat.

## Input
The agent must read the following before invoking the skill:
- The status file — `.docs/{project-name}-status.md`.
- The PRD — `.docs/{project-name}-prd.md`.
- The test files from Phase 2 — to understand what behavior must be supported.
- The rules directory — every rule file, especially Custom Code Policy, Hardcoding Policy, and Data and Backend Policy.
- The existing codebase files listed in the Phase 0 status file as affected files.

## Work
The agent must invoke `/tech-design-doc` with the following context:
- The PRD requirements and acceptance criteria.
- The affected files and domains from the Phase 0 status file.
- The existing codebase patterns found by reading the affected files.

The `/tech-design-doc` skill handles the design document creation process — PRD traceability, architecture decisions, and framework research. The agent must not duplicate the skill's work by writing design content manually.

## Phase-specific constraints
The agent must verify the `/tech-design-doc` output meets these additional constraints:

**Fit check — the agent must answer these questions before accepting the design:**
- Can this feature be built entirely with existing patterns in the codebase? If yes, the design must use those patterns.
- Does this feature require a new dependency? If yes, the agent must stop and ask the user before proceeding.

**Zero bloat check — the design must pass all of these:**
- Every new file in the design maps to a specific PRD requirement.
- No new abstraction layer is introduced (no service classes, no repositories, no ORM wrappers).
- No new utility files are created for one-time operations.
- No new directory structure is introduced when existing directories can hold the new files.
- No new dependency is added when existing dependencies handle the need.

**Database changes:**
- If the feature requires database changes, the design must include migration SQL.
- If the feature requires new tables, the design must include RLS policies.

## Output
- `.docs/{project-name}-design.md` — the technical design document produced by `/tech-design-doc`.
- The status file updated with Phase 3 marked as IN PROGRESS.

## Gate
The `/tech-design-doc` skill completes successfully. Every PRD requirement maps to a design decision. The zero bloat check passes. Database migrations are specified if needed. The user approves the design before the agent proceeds to Phase 4.
