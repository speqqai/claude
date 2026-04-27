# Phase 1: PRD

## Purpose
The agent produces a Product Requirements Document by invoking the `/create-prd` skill. This phase defines the input the agent must provide to the skill and the gate the output must pass.

## Input
The agent must read the following before invoking the skill:
- The status file from Phase 0 — `.docs/{project-name}-status.md`.
- The existing product documentation in `content/` — to understand current product behavior.

## Work
The agent must invoke `/create-prd` with the following context:
- The project summary from the Phase 0 status file.
- The user problem the feature solves.
- The affected surfaces and domains from the Phase 0 status file.

The `/create-prd` skill handles the full PRD creation process — structure, requirements, acceptance criteria, and validation. The agent must not duplicate the skill's work by writing PRD content manually.

## Phase-specific constraints
The agent must verify the `/create-prd` output meets these additional constraints before presenting to the user:
- The PRD specifies which roles and plan tiers the feature applies to.
- The PRD describes personas as real-world people, not system roles.
- The PRD contains no implementation details — no database schemas, no API designs, no component names, no file paths.

## Output
- `.docs/{project-name}-prd.md` — the product requirements document produced by `/create-prd`.
- The status file updated with Phase 1 marked as IN PROGRESS.

## Gate
The `/create-prd` skill completes successfully. The phase-specific constraints pass. The user approves the PRD before the agent proceeds to Phase 2.
