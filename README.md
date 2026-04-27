# Speqq Claude Config

Centralized Claude Code skills and project context for the Speqq team.

## Setup

Clone the parent repo with the submodule:

```bash
git clone --recurse-submodules https://github.com/speqqai/speqq.git
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init
```

Then activate the commit message hook:

```bash
cd .claude
git config core.hooksPath .githooks
```

## Making changes

All commits must follow [Conventional Commits](https://www.conventionalcommits.org):

```
<type>(<scope>): <description>
```

**Types:** `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `style`, `perf`

**Examples:**

```
feat(skills): add supabase-rls skill
fix: correct broken frontmatter in development-workflow
docs(context): update SYSTEM.md with new service
chore: remove deprecated codex skill
```

The local hook rejects non-conforming messages at commit time. CI enforces the same check on PRs.

## Release process

1. Branch off `main`, make changes, open a PR
2. CI validates commit messages — PR is blocked if any fail
3. Merge the PR
4. Release Please auto-opens a Release PR with a version bump and changelog
5. Merge the Release PR — a GitHub Release is published automatically

View all releases: https://github.com/speqqai/claude/releases

## Pulling updates into speqqai/speqq

```bash
cd .claude
git pull origin main
cd ..
git add .claude
git commit -m "chore: update .claude to latest"
```

## What's in this repo

```
CLAUDE.md              — project instructions and engineering gates
ENGINEERING.md         — UI, styling, data access, TypeScript, error handling rules
SYSTEM.md              — services, connections, tech stack, deployment
TOOLS.md               — CLI tools, APIs, and dev commands
PRODUCT_SURFACES.md    — pages, API routes, and feature modules
settings.local.json    — Claude Code settings
skills/                — 22 reusable Claude Code skills
```
