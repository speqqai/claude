# Phase 5: Documentation

## Purpose
The agent updates all project documentation by invoking the `/create-documentation` skill. Documentation includes Nextra content pages, project docs, and any configuration or setup documentation that changed.

## Input
The agent must read the following before invoking the skill:
- The status file — `.docs/{project-name}-status.md`.
- The PRD — `.docs/{project-name}-prd.md`.
- The design document — `.docs/{project-name}-design.md`.
- The existing documentation in `content/` — browse all `.mdx` files to understand current documentation structure.

## Work
The agent must invoke `/create-documentation` with the following context:
- The PRD — every capability, user flow, edge case, and error scenario.
- The existing documentation structure in `content/`.
- The instruction that documentation must be written for users, not engineers.

The `/create-documentation` skill handles the documentation creation process including its own write-validate-fix loop. The agent must not duplicate the skill's work by writing documentation content manually.

## Phase-specific constraints
The agent must verify the `/create-documentation` output meets these additional constraints:
- Every capability described in the PRD has corresponding user-facing documentation.
- Every user flow described in the PRD has a step-by-step guide.
- No placeholder text ("TODO", "TBD", "coming soon") exists in any documentation file.
- No implementation details appear in user-facing documentation — no database schemas, no API details, no component names, no file paths.
- If the feature changed environment variables or setup steps, README.md or the relevant setup guide was updated.
- If the feature changed API contracts, API documentation was updated (if API documentation exists).

## Output
- Updated or new `.mdx` files in `content/`.
- Updated project documentation files if applicable.
- The status file updated with Phase 5 marked as IN PROGRESS.

## Gate
The `/create-documentation` skill completes successfully. Every PRD capability has corresponding documentation. No placeholder text exists. The user approves the documentation before the agent proceeds to Phase 6.
