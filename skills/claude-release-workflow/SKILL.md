---
title: github-release-workflow
description: Ship changes to a git submodule using conventional commits, commitlint, and Release Please.
user_invocable: true
---

# GitHub Release Workflow

The `.claude` directory is a git submodule pointing to `speqqai/claude`. Changes to skills, context files, or settings go through this workflow.

## Steps

1. Enter the submodule: `cd .claude`
2. Pull latest: `git checkout main && git pull origin main`
3. Create a branch: `git checkout -b <type>/<short-name>`
4. Make changes and commit using conventional commits: `<type>(<scope>): <description>`
5. Push and open a PR: `gh pr create --repo speqqai/claude --base main`
6. Wait for commitlint to pass
7. Merge: `gh pr merge <number> --repo speqqai/claude --merge --subject "<type>(<scope>): <description>"`
8. Release Please auto-opens a release PR — merge it to publish
9. Return to parent repo: `cd ..`
10. Update the submodule pointer: `git add .claude && git commit -m "chore: update .claude to vX.Y.Z"`

## Commit Types

`feat` `fix` `docs` `chore` `refactor` `test` `ci` `style` `perf`

## Validation

- [ ] Commitlint passed
- [ ] Release PR merged
- [ ] GitHub Release published at speqqai/claude/releases
- [ ] Parent repo submodule pointer updated
