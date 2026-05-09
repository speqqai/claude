---
title: commit-submodule
description: Commit changes inside the .claude submodule and update the parent repo's submodule pointer. Use whenever you edit any file under .claude/ in a parent repo that consumes this submodule.
user_invocable: true
---

# commit-submodule

Two repos, two pushes. Run top to bottom from the parent repo root.

```yaml
context:
  submodule_repo:   speqqai/claude
  submodule_path:   .claude
  submodule_branch: main
  ci_actions:
    - wagoid/commitlint-github-action  # validates message
    - googleapis/release-please-action # opens release PR

inputs:
  files:        list[str]   # paths under .claude/ to stage
  type:         enum        # feat | fix | docs | chore | refactor | test | ci | style | perf
  scope:        str?        # optional, e.g. skills, context, design-system
  description:  str         # imperative, lowercase, no trailing period

invariants:
  message_format: '^(feat|fix|docs|chore|refactor|test|ci|style|perf)(\(.+\))?: .+'
  enforced_by:
    local: .githooks/commit-msg     # pure shell, regex match
    ci:    wagoid commitlint action # repeats the check
  changelog_bumps_on:               # release-please default rules
    - feat   # → minor bump
    - fix    # → patch bump
    # chore, docs, refactor, etc. land on main but do NOT appear in CHANGELOG

preconditions:
  - admin_on_submodule_repo: true   # if false → use claude-release-workflow skill (PR flow)

steps:

  - id: 1_enter
    run: |
      cd .claude
      git checkout main
      git pull origin main --ff-only
    why: submodule defaults to detached HEAD; main must be fresh before commit

  - id: 2_commit
    run: |
      git add <files>
      git commit -m "<type>(<scope>): <description>"
    examples:
      - 'feat(skills): add commit-submodule skill'
      - 'fix(context): correct broken link in TOOLS.md'
      - 'chore: remove vestigial package.json'

  - id: 3_push
    run: git push origin main
    triggers:
      - commitlint CI
      - release-please opens or updates 'chore(main): release X.Y.Z' PR (only if a feat or fix was pushed since last release)

  - id: 4_release   # optional; safe to batch multiple commits before merging
    run: |
      gh pr list  --repo speqqai/claude --search 'release-please'
      gh pr merge --repo speqqai/claude <PR_NUMBER> --merge
      git pull origin main --ff-only
    effect: bumps version.txt, updates CHANGELOG.md, publishes GitHub Release

  - id: 5_bump_parent
    run: |
      cd ..
      git add .claude
      git commit -m "chore: bump .claude submodule"
      git push
    note: parent repo does not enforce conventional commits; any clear message works. If parent is on a feature branch, push then PR to dev as usual.

errors:

  - id: HOOK_REJECTED
    step: 2_commit
    cause: commit message does not match invariants.message_format
    fix:  rewrite message with a valid type prefix; re-run step 2

  - id: CI_COMMITLINT_FAILED
    step: 3_push
    cause: |
      commitlint config file fails to load — typically a .js config
      parsed as ESM because the surrounding package.json (or the
      wagoid container's /package.json) sets "type": "module".
    fix: |
      rename commitlint.config.js → commitlint.config.cjs,
      OR delete the config file entirely (wagoid defaults to
      @commitlint/config-conventional, which is what most repos want).

  - id: BRANCH_PROTECTION_BLOCKED
    step: 3_push
    cause: not admin on speqqai/claude; required status checks haven't passed yet
    fix:  use claude-release-workflow skill (PR-based flow) instead

  - id: RELEASE_PR_MISSING
    step: 4_release
    cause: only chore/docs/refactor/etc. commits since last release; release-please does not bump for those types
    fix:  push at least one feat or fix commit, OR skip step 4 (no release needed yet)

  - id: PARENT_PUSH_REJECTED
    step: 5_bump_parent
    cause: parent branch protected, or PR-only flow required for parent's branch
    fix:  open a PR for the bump commit per the parent repo's normal flow

validation:
  after_3:
    - "git -C .claude log -1 --oneline shows your commit"
    - "gh run list --repo speqqai/claude --limit 2 shows commitlint passed"
    - "release-please PR open at https://github.com/speqqai/claude/pulls (if a feat/fix was pushed)"
  after_5:
    - "parent diff HEAD~1 -- .claude shows only submodule SHA change"
    - "parent CI green"

out_of_scope:
  - editing files (caller's job)
  - PR-based submodule flow (see claude-release-workflow skill)
  - rebases, stashes, force-pushes
```
