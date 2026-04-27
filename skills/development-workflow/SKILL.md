---
name: development-workflow
description: >-
  End-to-end development workflow orchestrator. The agent must follow this
  workflow phase-by-phase when starting any new feature or project.
  Overview → PRD → UX Design → Test Design → Architecture → Implementation → Documentation → Cleanup.
  Each phase is independent with its own instructions and validation file.
  Use when starting any new feature or project.
---

# Development Workflow

## Purpose
This skill orchestrates a feature from idea to release by running eight phases in order. Each phase has its own instructions file, its own deliverable, and its own independent validation file. The agent does not skip phases. The agent does not skip validation. The user is the final authority on all decisions.

## Phases

| Phase | Instructions | Skill Invoked | Deliverable | Validation |
|---|---|---|---|---|
| 0 — Overview | `phase-0-overview.md` | None — agent does this directly | Status file with project understanding, affected surfaces, domains touched | Subagent validates against `validation-0-overview.md` |
| 1 — PRD | `phase-1-prd.md` | `/create-prd` | Product requirements document with testable acceptance criteria | Subagent validates against `validation-1-prd.md` |
| 2 — UX Design | `phase-2-ux-design.md` | `/ux-design` | Figma mocks with all screens, states, interactions, and responsive variants | Subagent validates against `validation-2-ux-design.md` |
| 3 — Test Design | `phase-3-test-design.md` | `/test-driven-development` | E2E test suite — unauthenticated and authenticated paths — using existing test infrastructure | Subagent validates against `validation-3-test-design.md` |
| 4 — Architecture | `phase-4-architecture.md` | `/tech-design-doc` | Technical design document — how the feature fits into the existing app with zero bloat | Subagent validates against `validation-4-architecture.md` |
| 5 — Implementation | `phase-5-implementation.md` | `/application-development` | Working code, passing tests, migrations run, build confirmed | Subagent validates against `validation-5-implementation.md` |
| 6 — Documentation | `phase-6-documentation.md` | `/create-documentation` | Nextra docs updated, all project docs updated | Subagent validates against `validation-6-documentation.md` |
| 7 — Cleanup | `phase-7-cleanup.md` | `/code-review` + `/pr-preflight` | Dead code deleted, orphaned files removed, old patterns removed | Subagent validates against `validation-7-cleanup.md` |

## How to execute

### Before starting any phase:
- read `/Users/delaghetto/Speqq/speqq_v1/.github/CONTRIBUTING.md`
The agent must read the phase instructions file (e.g., `phase-1-prd.md`) before starting the phase. The instructions file contains what to read, what skill to invoke, what to produce, and what the output must contain.

### After completing any phase:
The agent must launch a subagent to validate the phase output. The subagent must read the validation file (e.g., `validation-1-prd.md`). The subagent must return PASS or FAIL. If the subagent returns FAIL, the agent must fix every issue the subagent identified and re-run validation until the subagent returns PASS.

### After validation passes:
The agent must present the phase output to the user. The agent must not proceed to the next phase until the user explicitly approves. Silence is not approval.

### Phase execution loop:
```
For each phase (0 through 7):
  1. Read phase-{N}-{name}.md
  2. Execute the work described in the instructions — invoke the skill listed in the phases table
  3. Launch subagent with validation-{N}-{name}.md to validate output
  4. If FAIL → fix issues → re-run validation
  5. If PASS → present output to user → wait for user approval
  6. Update status file
  7. Proceed to next phase
```

## Status file

The agent must create and maintain `.docs/{project-name}-status.md` throughout all phases.

The status file must contain:
- Current phase number and name.
- Completed phases with file paths to each deliverable.
- Next phase and what the agent will do next.
- Any blockers or open questions.

The agent must update the status file after every phase transition.

## Recovery after context loss

If the agent loses context mid-workflow, the agent must follow this sequence before writing any code:
1. Read AGENTS.md and the rules directory.
2. Glob `.docs/*-status.md` to find the active status file, then read the status file.
3. Read every artifact listed in the status file.
4. Read the git log to see what has been committed.
5. Tell the user which phase the agent is on and what the agent will do next.
6. The agent must not write code until steps 1–5 are complete.

## Hard rules

- The agent must not skip any phase.
- The agent must not skip validation for any phase.
- The agent must not proceed to the next phase without user approval.
- The agent must not write production code without completing the PRD phase and the test design phase first.
- The agent must not add a new dependency without user approval.
- The agent must not commit directly to `main` or `dev`.
- The agent must not leave placeholder text in any artifact.
- The agent must not ask the user for information that exists in a prior phase artifact.
- The agent must use `npm` not `pnpm`.
- The agent must update the status file after every phase transition.
- The agent must delete all dead code and orphaned files during the cleanup phase.
