# Phase 0: Overview

## Purpose
The agent builds a complete understanding of the project before any planning or coding begins. The agent produces a status file that tracks work across all phases until the feature is launched.

## Input
The agent must read the following before starting this phase:
- AGENTS.md — agent behavior and thinking process.
- The rules directory — every rule file listed in rules/README.md.
- The task description or feature request from the user.
- The existing codebase structure — run `ls` on the project root, `app/`, `components/`, `lib/`, and `content/` directories.
- The existing documentation in `content/` — browse all `.mdx` files to understand what the product does today.
- The git log — read recent commits to understand what has been changed recently.

## Work
The agent must produce a status file at `.docs/{project-name}-status.md` that contains:

### Project understanding:
- A 2–3 sentence summary of what the feature does and why the feature matters to Speqq users.
- The user problem the feature solves.
- The surfaces the feature will touch (sidebar, editor, chat, settings, landing page, API routes, database).

### Affected files and directories:
- A list of existing files the feature will likely modify.
- A list of new files the feature will likely create.
- A list of existing database tables the feature will likely query or modify.
- A list of new database tables or columns the feature will likely require.

### Domains touched:
- Which domains this feature involves (auth, data fetching, UI components, AI/chat, editor, navigation, database schema, API routes).
- For each domain, the framework, SDK, or library that handles that domain (read from package.json, not guessed).

### Phase tracking:
- A checklist of all 7 phases (0 through 6) with status: NOT STARTED, IN PROGRESS, BLOCKED, PASS, or DONE.
- The current phase marked as IN PROGRESS.

## Output
- `.docs/{project-name}-status.md` — the status file described above.

## Gate
The status file exists. The status file contains all four sections (project understanding, affected files, domains, phase tracking). The user approves the overview before the agent proceeds to Phase 1.
