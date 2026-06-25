---
name: native-review
description: >-
  Audit a commit, PR, or set of changes for native-first compliance. Verifies that
  every concern has one canonical owner using Next.js, React 19, Supabase, LangGraph,
  or other approved first-party SDK primitives — not custom code that reimplements
  what a framework already provides. Single source of truth per concern, one pattern
  per data flow. Produces a structured native-vs-custom verdict with concrete
  citations of which SDK/framework primitive should replace any custom code found.
  Use when reviewing PRs for paradigm compliance, when asked to "native review",
  "check for custom code", "audit for SDK adherence", or before merging any non-trivial
  change.
allowed-tools: Read, Grep, Glob, Bash
---

# Native Review

A single-purpose audit: **does this change use the canonical native primitive, or is it reinventing a framework/SDK concern?**

This is not a general code review. It checks paradigm compliance with `.agents/.claude/ENGINEERING.md`. If a change introduces custom code where Next.js, React 19, Supabase, LangGraph, Yjs, Zod, Sonner, date-fns, shadcn/ui, or another approved dependency already owns the concern, flag it.

Run on every PR before approving. Produces one of three verdicts:

- 🟢 **NATIVE** — every concern in this change maps to a first-party primitive. Ship it.
- 🟡 **CUSTOM-WITH-REASON** — custom code is present but documented justification exists (Speqq-specific product logic or no SDK answer). Acceptable.
- 🔴 **REINVENT** — custom code reimplements an SDK capability. Block until rewritten on native primitive.

---

## Step 1 — Establish what changed

```bash
git diff --stat <base>...HEAD
git diff <base>...HEAD
```

For each changed file, answer:
- What concern does this code own? (auth, data fetching, state, forms, mutations, live updates, navigation, validation, error handling, etc.)
- Is the concern covered by `.agents/.claude/ENGINEERING.md`?

If a concern isn't in that table, it's either out-of-scope for the audit (pure product logic) or the table is incomplete (flag).

---

## Step 2 — Audit against the source of truth

Use `.agents/.claude/ENGINEERING.md` as the canonical map. Do not duplicate the full map here. For each concern, answer:

1. What concern does this code own?
2. Which native owner does `ENGINEERING.md` assign?
3. Does the change use that owner?
4. If not, is this Speqq-specific product logic with a clear reason?

High-frequency checks:

- Data reads use Server Components and `lib/domains/<domain>/queries.ts`.
- UI mutations use Server Actions and `lib/domains/<domain>/mutations.ts`.
- External clients use Route Handlers and `lib/domains/<domain>/service.ts`.
- Components are presentation/interaction only.
- New domain logic lives under `lib/domains/<domain>`.
- Required config throws immediately.
- Errors are visible, typed, and debuggable.
- Independent I/O is parallelized.
- Broad list queries select columns deliberately.
- Existing legacy patterns are not copied into new work.
- Harmful legacy patterns are quarantined, not made reusable.

If a rule is not covered by `ENGINEERING.md`, treat that as a guide gap and surface it.

---

## Step 3 — Walk the diff with this checklist

For every changed file, ask in order:

1. **Identify the concern.** What does this code own? (Match against the tables above.)
2. **Find the native owner.** Which row in the table covers it?
3. **Compare.** Does the change use the native owner, or something else?
4. **If something else: classify.**
   - Speqq-specific product logic with no SDK answer → 🟡 acceptable, but check for an inline comment justifying it.
   - SDK answer exists and the change reinvents it → 🔴 REINVENT. Block.

### Specific reinvention patterns to grep for

```bash
# Custom fetch/data wrappers
grep -rn "function fetch\|async function get\|class.*Client" --include="*.ts" --include="*.tsx" <changed-files>

# Custom toast / notification
grep -rn "createToast\|showNotification\|notify(" <changed-files>

# Custom optimistic patterns
grep -rn "tempId\|optimisticId\|reconcile" <changed-files>

# Custom auth checks bypassing requireUser
grep -rn "getSession\|getUser" <changed-files>

# Custom polling instead of realtime
grep -rn "setInterval\|setTimeout.*fetch\|poll" <changed-files>

# Multiple onAuthStateChange subscribers
grep -rn "onAuthStateChange" <changed-files>

# Manual document.title
grep -rn "document.title\|<title>" <changed-files>

# Manual Suspense around route content
grep -rn "<Suspense" <changed-files>

# localStorage for app state
grep -rn "localStorage\.\(set\|get\)Item" <changed-files>

# SWR in new files
grep -rn "useSWR\|import.*from .swr" <changed-files>

# Zustand for things that map to URL/cookie
grep -rn "create<.*Store\|zustand" <changed-files>

# Broad selects or hidden config fallbacks
grep -rn "select(\"\\*\"\|process\\.env\\.[A-Z0-9_]* *||" <changed-files>
```

Each match needs justification or rewrite.

---

## Step 4 — Verdict format

Produce one report with this exact shape:

```
# Native Review — <PR title or commit SHA>

## Verdict
🟢 NATIVE | 🟡 CUSTOM-WITH-REASON | 🔴 REINVENT

## Concerns audited
- <concern 1>: <native owner> → ✅ used | ❌ reinvented | ⚠️ partial
- <concern 2>: ...

## Reinventions to fix (blocking)
1. **<file:line>** — <description of custom code>
   - Native answer: <SDK primitive + 1-line example>
   - Why this matters: <e.g., two cache layers, duplicated state>

## Custom-with-reason (acceptable)
1. **<file:line>** — <description>
   - Why no native answer: <justification, from inline comment or PR description>

## Quarantined legacy patterns
1. **<file:line>** — <legacy pattern touched for a stop-the-bleeding fix>
   - Short-term reason: <why the narrow change is acceptable>
   - Long-term fix: <canonical migration path>

## Recommendations
- <concrete file:line + native rewrite>

## Files audited
- path/a.ts (concern: data fetching)
- path/b.tsx (concern: form)
- ...
```

---

## Step 5 — Auto-block on REINVENT

If the verdict is 🔴 REINVENT, do **not** approve the PR. Report the verdict, cite the specific files/lines, and stop. The author must rewrite on native primitives or document a 🟡 justification.

If the verdict is 🟡 CUSTOM-WITH-REASON, surface the justifications so the user can confirm they're acceptable.

If the verdict is 🟢 NATIVE, the change is ready for the standard `code-review` skill (broader quality pass) or `pr-preflight` (build/lint/test gates).

---

## What this skill is NOT

- Not a general code-quality review (use `code-review`).
- Not a build/lint/test gate (use `pr-preflight`).
- Not an architectural review of features that don't exist yet (use `tech-design-doc`).
- Not a security audit (use `code-review` Step 5).

This skill answers exactly one question: **"Did we use the native primitive, or did we write custom code?"**

---

## Reference: paradigm sources

- `.agents/.claude/ENGINEERING.md` — canonical architecture and hard gates
- `.agents/.claude/CLAUDE.md` — operating priorities
- `.claude/SYSTEM.md` — services and SDKs in use
- `package.json` — full list of approved first-party SDKs and libraries

If a primitive appears in `package.json` and a concern in the diff doesn't use it, that's a candidate for 🔴 REINVENT.
