---
title: commit-submodule
description: Commit changes inside the .claude submodule and update the parent repo's submodule pointer. Use whenever you edit any file under .claude/ in a parent repo that consumes this submodule.
user_invocable: true
---

# commit-submodule

Two repos, two pushes. Run top to bottom from the parent repo root.

```yaml
context:
  submodule_repo: speqqai/claude
  submodule_path: .claude
  submodule_branch: main
  ci_actions:
    - wagoid/commitlint-github-action  # validates message
    - googleapis/release-please-action  # opens release PR

inputs:
  files:        list[str]   # paths under .claude/ to stage
  type:         enum        # feat | fix | docs | chore | refactor | test | ci | style | perf
  scope:        str?        # optional, e.g. skills, context, design-system
  description:  str         # imperative, lowercase, no trailing period

invariants:
  message_format: '^(feat|fix|docs|chore|refactor|test|ci|style|perf)(\(.+\))?: .+'
  enforced_by:
    local: .githooks/commit-msg
    ci:    wagoid/commitlint-github-action

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
    on_fail:
      hook_rejected: rewrite message to match invariants.message_format

  - id: 3_push
    run: git push origin main
    triggers:
      - commitlint CI
      - release-please opens or updates 'chore(main): release X.Y.Z' PR

  - id: 4_release   # optional; can batch multiple commits before merging
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
    note: if parent is on a feature branch, push then PR to dev as usual

validation:
  after_3:
    - "git -C .claude log -1 --oneline shows your commit"
    - "gh run list --repo speqqai/claude --limit 2 shows commitlint passed"
    - "release-please PR open at https://github.com/speqqai/claude/pulls"
  after_5:
    - "parent diff HEAD~1 -- .claude shows only submodule SHA change"
    - "parent CI green"

out_of_scope:
  - editing files (caller's job)
  - PR-based submodule flow (see claude-release-workflow skill)
  - rebases, stashes, force-pushes
```
