---
name: code-review
description: >-
  Comprehensive pre-PR code review for Next.js + Supabase + TypeScript projects.
  Eight-step review process covering architecture, code quality, library-over-custom,
  TypeScript strictness, security (including Supabase RLS), performance (including
  Next.js patterns), testing validity, and documentation. Produces a structured
  PR-readiness verdict. If passing, submits a PR to dev automatically.
  Use when reviewing pull requests, checking code quality, identifying bugs,
  auditing security, analyzing performance, or when asked to "review", "check",
  or "audit" code changes.
allowed-tools: Read, Grep, Glob, Bash
---

# Code Review

Run this review on every set of changes before approving a PR. Work through each step in order. Do not skip steps.

**If the review produces a 🟢 READY TO SUBMIT verdict, automatically create a PR targeting the `dev` branch.**

---

## Step 1: Understand the Context

**Read the PR/commit description:**

- What is the goal of this change?
- Which issue or ticket does it address?
- Are there special considerations or tradeoffs?

**Check the scope:**

- How many files changed? (`git diff --stat` against target branch)
- What type of change? (feature, bugfix, refactor, chore)
- Are tests included?
- Are migrations included?

**Read every changed file completely.** No skimming. Partial reviews miss real issues.

---

## Step 2: High-Level Architecture Review

**Architecture & design:**

- [ ] Approach makes sense for the problem
- [ ] Consistent with existing patterns in the codebase
- [ ] No simpler alternative that achieves the same result
- [ ] Code is in the right place (right module, right layer)
- [ ] Appropriately scoped — not over-engineered, not too narrow

**Code organization:**

- [ ] Clear separation of concerns (business logic ≠ UI ≠ data access)
- [ ] Appropriate abstraction levels (no 300-line god components)
- [ ] Logical file/folder structure following project conventions
- [ ] Data flows in one direction; no prop drilling through 4+ levels

**Solution validity:**

- [ ] Implementation addresses the actual requirement (re-read the ticket)
- [ ] Breaking changes are flagged
- [ ] Mutations are idempotent (safe to retry, no duplicate records)
- [ ] Error boundaries isolate failures (one crash ≠ page down)

---

## Step 3: Detailed Code Review

**Naming:**

- [ ] Variables: descriptive, meaningful names
- [ ] Functions: verb-based, clear purpose (`getUserById` not `get`)
- [ ] Types/interfaces: noun-based, single responsibility
- [ ] Constants: `UPPER_CASE` for true constants
- [ ] No abbreviations unless universally understood (`id`, `url` — fine; `usr`, `mgr` — not fine)

**Functions:**

- [ ] Single responsibility
- [ ] Reasonable length (< 40 lines ideally)
- [ ] Clear inputs and outputs
- [ ] Explicit return types on exported functions
- [ ] Minimal side effects
- [ ] Proper error handling
- [ ] Early returns over deep nesting

**SOLID principles:**

- [ ] Single responsibility — each function/class/module does one thing
- [ ] Open/closed — extensible without modifying existing code
- [ ] Liskov substitution — subtypes are substitutable for their base types
- [ ] Interface segregation — no forced dependencies on unused interfaces
- [ ] Dependency inversion — depend on abstractions, not concretions

**Error handling:**

- [ ] All errors caught and handled at appropriate boundaries
- [ ] Meaningful error messages with context
- [ ] Proper logging (structured, not `console.log`)
- [ ] No silent failures — async errors never swallowed
- [ ] User-friendly errors for UI; technical details in logs only

**Code quality:**

- [ ] No code duplication (DRY)
- [ ] No dead code or commented-out code
- [ ] No magic numbers or strings — use named constants
- [ ] Consistent formatting with codebase conventions
- [ ] Loading, error, and empty states handled for every async operation

### Library Over Custom Code

LLM-generated code defaults to writing things from scratch. **Flag it.**

For every new utility, ask: does an established library already handle this?

See [references/library-checks.md](references/library-checks.md) for the full flag list and when to accept custom code.

### TypeScript Strictness

- [ ] `strict: true` enabled in `tsconfig.json`
- [ ] No `any` without a justifying comment; prefer `unknown`
- [ ] No unsafe `as` casts — use type guards or discriminated unions
- [ ] No `@ts-ignore` / `@ts-expect-error` without comment + linked issue
- [ ] No `!` non-null assertions — use null checks or optional chaining
- [ ] Return types on exported functions
- [ ] Types derived from Zod schemas (`z.infer<>`) not parallel definitions
- [ ] Database types from `supabase gen types typescript`, not manual
- [ ] `as const` or string unions over `enum`
- [ ] Discriminated unions over optional fields

See [references/typescript.md](references/typescript.md) for anti-patterns and examples.

---

## Step 4: Security Review

**Input validation:**

- [ ] All user inputs validated server-side (type, length, format, range)
- [ ] Server action inputs validated with Zod before processing
- [ ] No trust of client-side validation alone

**Authentication & authorization:**

- [ ] Every protected endpoint/action verifies authentication
- [ ] Resource access scoped to requesting user's permissions; no IDOR
- [ ] Auth checked in middleware or layout, not individual page components
- [ ] Session management via `@supabase/ssr`, not custom flows

**Data protection:**

- [ ] No hardcoded secrets, API keys, or credentials in source code
- [ ] Server secrets use `process.env.SECRET`; client vars use `NEXT_PUBLIC_`
- [ ] Sensitive data never logged, in error messages, or in API responses
- [ ] SQL injection prevented (parameterized queries / ORM)
- [ ] XSS prevented (user content escaped; `dangerouslySetInnerHTML` justified)
- [ ] CSRF protection via server actions (don't bypass built-in protection)

**Supabase security:**

- [ ] RLS enabled on every table
- [ ] Policies exist for SELECT, INSERT, UPDATE, DELETE
- [ ] `auth.uid()` used for authorization — never `raw_user_meta_data`
- [ ] RLS on join/junction tables
- [ ] No `service_role` key in client bundles
- [ ] Correct Supabase client per context (client/server/route handler)

**Dependencies:**

- [ ] New packages from trusted, maintained sources
- [ ] No known CVEs
- [ ] Minimal — don't add a package for one function

See [references/supabase.md](references/supabase.md) for Supabase-specific patterns and examples.

---

## Step 5: Performance Review

**Algorithms:**

- [ ] Appropriate algorithm choice for data size
- [ ] No O(n²) when O(n) or O(n log n) exists
- [ ] No unnecessary loops or redundant computation
- [ ] Expensive calculations cached/memoized

**Database:**

- [ ] No N+1 queries — use batched/joined calls
- [ ] All list queries paginated (`LIMIT` or cursor-based)
- [ ] Select only needed columns (flag `.select('*')` on wide tables)
- [ ] Queries filter/sort on indexed columns
- [ ] RLS subqueries filter by `auth.uid()` first

**Caching & resources:**

- [ ] Appropriate caching strategy with invalidation
- [ ] Subscriptions/listeners/timers cleaned up on unmount
- [ ] Supabase realtime channels unsubscribed in `useEffect` cleanup
- [ ] No memory leaks in long-running processes
- [ ] Bundle size checked — tree-shakeable imports, no barrel file bloat

**Next.js specific:**

- [ ] Server Components by default; `'use client'` pushed to leaf components
- [ ] No server-only code (DB, secrets) in client components
- [ ] Server Actions for mutations; API routes only for webhooks/external
- [ ] `<Suspense>` with fallbacks for long-loading data
- [ ] `revalidatePath` / `revalidateTag` after mutations
- [ ] `next/image` for all images; `next/font` for fonts
- [ ] `layout.tsx` for shared UI; `loading.tsx` + `error.tsx` per route group
- [ ] Pages export `metadata` or `generateMetadata`
- [ ] Props from server → client are serializable

See [references/nextjs.md](references/nextjs.md) for Next.js patterns and examples.

---

## Step 6: Testing Review

**Test coverage:**

- [ ] Unit tests for new code
- [ ] Integration tests for critical paths (API → DB → response)
- [ ] Happy path + validation error + unexpected failure tested
- [ ] Edge cases covered (empty, null, zero, negative, boundary values)
- [ ] Auth-scoped features test both authenticated and unauthenticated paths
- [ ] Bug fixes include regression test reproducing the original bug

**Test quality:**

- [ ] Tests are readable (Arrange-Act-Assert pattern)
- [ ] Test names describe scenario + expected outcome
- [ ] Tests are deterministic — no timing dependencies or shared state
- [ ] No test interdependencies — each test stands alone
- [ ] Realistic test data, not just `"test"` and `123`
- [ ] Only external boundaries mocked (network, DB, filesystem)

**Test validity:**

- [ ] Tests assert behavior, not implementation details
- [ ] Tests would fail if the feature broke (mentally delete the impl — would tests catch it?)
- [ ] No snapshot-only coverage — snapshots don't prove behavior
- [ ] No false-positive tests that pass regardless of correctness

---

## Step 7: Documentation Review

**Code comments:**

- [ ] Complex logic explained
- [ ] No obvious/redundant comments (`// increment counter` above `counter++`)
- [ ] TODOs have linked tickets or issues
- [ ] Comments are accurate and current

**API documentation:**

- [ ] Public functions/types have JSDoc with param descriptions
- [ ] Breaking changes documented with migration notes
- [ ] README updated if public API or setup changed

---

## Step 8: Provide Feedback & Verdict

### Giving Feedback

**Be constructive:**

```
✅ Good:
"Consider extracting this validation into a shared utility.
The same pattern exists in `lib/validators.ts` — reusing it
would keep things DRY and easier to test."

❌ Bad:
"This is wrong. Rewrite it."
```

**Be specific:**

```
✅ Good:
"Line 45: this Supabase query inside a `.map()` creates an N+1 pattern.
Consider using `.in('id', ids)` to batch into a single query."

❌ Bad:
"Performance issues here."
```

**Acknowledge good work:**

```
"Nice use of discriminated unions here — makes the error handling
exhaustive and type-safe. Clean."
```

### Severity Levels

Classify every finding:

| Level | Label | Meaning | Blocks? |
|-------|-------|---------|---------|
| Critical | `[CRITICAL]` | Security vulnerability, data loss, crash | **Yes** |
| Major | `[MAJOR]` | Bug, logic error, missing coverage | **Yes** |
| Minor | `[MINOR]` | Reduces future maintenance cost | No |
| Nit | `[NIT]` | Style preference, trivial cleanup | No |

### Output Template

```
# Code Review Report

## Summary
<2-3 sentences: what changed, scope, files touched, change type>

## Architecture (Step 2)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

## Code Quality / TypeScript / Libraries (Step 3)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

## Security (Step 4)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

## Performance / Next.js (Step 5)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

## Tests (Step 6)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

## Documentation (Step 7)
| Finding | File:Line | Severity | Recommendation |
|---------|-----------|----------|----------------|

**Verdict:** ✅ Pass | ⚠️ Issues | ❌ Blocking

---

## PR Readiness

🟢 **READY TO SUBMIT** — Submitting PR to dev.
🟡 **NEEDS FIXES** — Fix issues before submitting.
🔴 **NOT READY** — Blocking issues must be resolved.

| Metric | Count |
|--------|-------|
| 🔴 Critical | <n> |
| 🟠 Major | <n> |
| 🟡 Minor | <n> |
| ⚪ Nit | <n> |

## Action Items (priority order)
1. <fix>
2. <fix>
...
```

### Verdict Rules

- Any `[CRITICAL]` → **🔴 NOT READY**
- 2+ `[MAJOR]` across any steps → **🔴 NOT READY**
- 1 `[MAJOR]` → **🟡 NEEDS FIXES**
- Only `[MINOR]` and `[NIT]` → **🟢 READY TO SUBMIT**
- No findings → `✅ Pass`

### Submit PR (on 🟢 only)

1. Commit with clear, conventional commit message
2. Push the branch
3. Create PR:
   ```bash
   gh pr create --base dev --title "<title>" --body "<summary + review results>"
   ```
4. If verdict is 🟡 or 🔴 → **do not submit**. List action items and stop.

---

## NEVER Do

1. **NEVER skip a step** — all steps run even for small changes
2. **NEVER approve custom crypto, auth, or hashing** — always `[CRITICAL]`
3. **NEVER approve RLS disabled or `service_role` on client** — always `[CRITICAL]`
4. **NEVER produce the report without reading every changed file**
5. **NEVER soften severity to be polite** — accuracy over feelings
6. **NEVER count snapshot-only tests as adequate coverage**
7. **NEVER let `any` pass without a justifying comment in the code**
8. **NEVER submit a PR if verdict is not 🟢**

---

## References

- [Library Over Custom Code checklist](references/library-checks.md)
- [TypeScript anti-patterns & examples](references/typescript.md)
- [Next.js patterns & examples](references/nextjs.md)
- [Supabase patterns & examples](references/supabase.md)
- [Common anti-patterns with code examples](references/anti-patterns.md)
