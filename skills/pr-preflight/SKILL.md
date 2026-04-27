---
name: pr-preflight
description: >-
  Pre-PR preflight checklist for Next.js + Supabase + TypeScript projects.
  Thirteen-step pass/fail gate: env, security, types, build, lint, Next.js,
  Supabase, multi-tenant, custom code audit, performance, tests, diff review, submit.
  Every step must pass. If passing, submits PR to dev automatically.
  Triggers on: "submit PR", "open PR", "create PR", "push this", "ship it",
  "preflight", "ready to merge", or when implementation is complete.
allowed-tools: Read, Grep, Glob, Bash
---

# PR Preflight

Run every step. Fix failures before moving on. Do not skip steps. Do not submit until all pass.

---

## Step 1: Environment

```bash
ls .env.example
```
- [ ] `.env.example` exists
- [ ] New env vars from this PR are added to `.env.example`

```bash
git diff --cached --name-only | grep -E "^\.env($|\.local|\.production)"
```
- [ ] No `.env` files staged in git

---

## Step 2: Security

```bash
grep -rn "NEXT_PUBLIC_.*SECRET\|NEXT_PUBLIC_.*SERVICE_ROLE\|NEXT_PUBLIC_.*PRIVATE\|NEXT_PUBLIC_.*PASSWORD" --include="*.env*" --include="*.ts" --include="*.tsx" . 2>/dev/null
```
- [ ] No secrets exposed via `NEXT_PUBLIC_`

```bash
grep -rn "service_role\|SERVICE_ROLE\|serviceRole" --include="*.ts" --include="*.tsx" app/ components/ hooks/ lib/ 2>/dev/null | grep -v "\.server\.\|server-only\|// server"
```
- [ ] No `service_role` in client code

```bash
grep -rn "sk_live\|sk_test\|api_key.*=.*['\"].*[a-zA-Z0-9]\{20,\}\|password.*=.*['\"]" --include="*.ts" --include="*.tsx" . | grep -v node_modules | grep -v ".env" | grep -v "\.test\.\|\.spec\."
```
- [ ] No hardcoded credentials

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -l "^'use client'" 2>/dev/null | xargs grep -n "process\.env\." 2>/dev/null | grep -v "NEXT_PUBLIC_"
```
- [ ] No server secrets in `'use client'` files

**Any failure here = stop and fix immediately.**

---

## Step 3: Type Check

```bash
npx tsc --noEmit
```
- [ ] Zero errors

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n ": any\| as any\|<any>" 2>/dev/null | grep -v "// any:" | grep -v node_modules
```
- [ ] No unjustified `any` (each must have `// any: <reason>`)

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "@ts-ignore\|@ts-expect-error" 2>/dev/null
```
- [ ] No `@ts-ignore` without explanation + issue link

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "as [A-Z]" 2>/dev/null | grep -v "as const\|as React\|as string\|as number\|as boolean" | grep -v node_modules
```
- [ ] No unsafe `as` casts on external data — use Zod

---

## Step 4: Build

```bash
npm run build 2>&1 | tee /tmp/build-output.txt
```
- [ ] Exit code 0

```bash
changed_files=$(git diff dev --name-only -- '*.ts' '*.tsx'); for f in $changed_files; do grep -i "warning\|warn" /tmp/build-output.txt | grep "$f" || true; done
```
- [ ] Zero warnings in your changed files — fix them, don't ship them

---

## Step 5: Lint

```bash
npm run lint 2>&1 | tee /tmp/lint-output.txt
```
- [ ] Zero errors

```bash
changed_files=$(git diff dev --name-only -- '*.ts' '*.tsx'); for f in $changed_files; do grep -A1 "$f" /tmp/lint-output.txt | grep -i "warning" || true; done
```
- [ ] Zero warnings in your changed files

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "eslint-disable" 2>/dev/null
```
- [ ] No new `eslint-disable` without justifying comment + issue link

---

## Step 6: Next.js

```bash
find app -name "page.tsx" -o -name "layout.tsx" | xargs grep -l "^'use client'" 2>/dev/null
```
- [ ] No `'use client'` on pages or layouts — push to leaf components

```bash
grep -rn "<img " --include="*.tsx" app/ components/ 2>/dev/null
```
- [ ] No raw `<img>` tags — use `next/image`

```bash
grep -rn "fonts.googleapis.com\|fonts.gstatic.com" --include="*.tsx" --include="*.css" app/ components/ 2>/dev/null
```
- [ ] No external font CDN links — use `next/font`

```bash
find app -name "page.tsx" -exec sh -c 'if ! grep -q "metadata\|generateMetadata" "$1"; then echo "❌ $1"; fi' _ {} \;
```
- [ ] Every page exports `metadata` or `generateMetadata`

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -l "'use server'" 2>/dev/null | while read -r f; do if ! grep -q "\.parse\|\.safeParse\|z\.\|zod" "$f"; then echo "❌ $f"; fi; done
```
- [ ] Every server action validates input with Zod

- [ ] `loading.tsx` and `error.tsx` exist for route groups with async data
- [ ] API routes used only for webhooks/external — not internal data fetching

---

## Step 7: Supabase

*(Skip if no Supabase code changed)*

```bash
for migration in $(git diff dev --name-only | grep "supabase/migrations.*\.sql"); do if grep -q "CREATE TABLE" "$migration"; then echo "--- $migration ---"; grep -c "ENABLE ROW LEVEL SECURITY" "$migration"; grep -c "CREATE POLICY" "$migration"; fi; done
```
- [ ] Every new table has RLS enabled
- [ ] Every new table has RLS policies (SELECT, INSERT, UPDATE, DELETE)

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "supabase.*\.from\|supabase.*\.rpc" 2>/dev/null | head -20
```
- [ ] Every Supabase call handles `{ data, error }` — error never ignored

```bash
grep -rn "interface.*Row\s*{\|type.*Row\s*=" --include="*.ts" --include="*.tsx" . | grep -v "node_modules\|supabase/\|generated\|Database\[" | head -5
```
- [ ] Database types from `supabase gen types typescript` — no manual row types

- [ ] Correct client per context (browser/server/route handler)
- [ ] Realtime subscriptions have cleanup in `useEffect` return
- [ ] `auth.uid()` for authorization — never `raw_user_meta_data`

---

## Step 8: Multi-Tenant / New-User Readiness

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "user_id.*=.*['\"].*-.*-\|org_id.*=.*['\"].*-.*-\|tenant.*=.*['\"].*-.*-" 2>/dev/null
```
- [ ] No hardcoded user IDs, org IDs, or tenant IDs

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "admin@\|@example.com\|isAdmin.*true\|role.*===.*['\"]admin" 2>/dev/null
```
- [ ] No hardcoded emails or admin-only gates

```bash
git diff dev --name-only -- '*.tsx' | xargs grep -l "\.map(" 2>/dev/null | while read -r f; do if ! grep -q "length.*===.*0\|\.length.*0\|empty\|no.*found\|EmptyState\|no.*data\|!data" "$f"; then echo "⚠️ $f"; fi; done
```
- [ ] UI handles zero records (empty state, not crash)

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "\.upload\|\.storage\|bucket" 2>/dev/null | grep -v "auth\.\|user\.\|userId\|user_id"
```
- [ ] File uploads scoped to user (not shared paths)

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "\.channel\|\.on('postgres_changes'" 2>/dev/null | grep -v "filter\|eq\."
```
- [ ] Realtime subscriptions filtered to user's data

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "localhost\|127\.0\.0\.1\|:3000\|:5432\|:8080" 2>/dev/null | grep -v "node_modules\|\.test\.\|\.spec\.\|\.env"
```
- [ ] No localhost URLs or hardcoded ports in production code

- [ ] Queries scoped to current user (via RLS or explicit filter)
- [ ] No assumptions about pre-existing data — works on first signup
- [ ] Feature works for all roles, not just admin
- [ ] Cache keys include user/tenant ID
- [ ] Error messages don't leak other users' data

**Any multi-tenant failure = stop and fix. This is a blocker.**

---

## Step 9: Custom Code Audit

**Do not build anything your stack already provides.** Not just the items listed below — anything. If React, Next.js, Supabase, shadcn/ui, Radix, SWR, usehooks-ts, or any package in `package.json` offers it, use it. The examples below are common traps, not the full list.

**For every new function, component, hook, or utility in the diff:**

1. Check the framework (React, Next.js, Supabase) — built-in?
2. Check the UI library (shadcn/ui, Radix) — component exists?
3. Check `package.json` — installed package does this?
4. Check hooks libraries (usehooks-ts, SWR) — hook exists?
5. Check platform APIs (browser, Node) — native API?
6. Check npm — package with >1K weekly downloads?
7. If nothing exists → state: "Removing this cuts [feature] because [reason]"

**Skip any of these checks = audit fails.**

```bash
# List all new functions/components/hooks in the diff
git diff dev -- '*.ts' '*.tsx' | grep "^+" | grep -E "(export\s+)?(function|const|class)\s+[A-Z]|function use[A-Z]" | grep -v "node_modules\|\.test\.\|\.spec\." | sed 's/^+//'
```

**Scan for common traps:**

```bash
# Hand-rolled date logic
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "new Date\|\.getTime\|\.toLocaleDateString\|getMonth\|getFullYear" 2>/dev/null | grep -v "node_modules\|date-fns\|dayjs"
```
- [ ] No custom date logic — use `date-fns` or `Intl.DateTimeFormat`

```bash
# Hand-rolled validation instead of Zod
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "\.test(\|\.match(\|RegExp\|typeof.*===\|instanceof" 2>/dev/null | grep -v "node_modules\|\.test\.\|\.spec\.\|zod\|\.safeParse"
```
- [ ] No custom validation — use Zod

```bash
# Custom hooks that duplicate usehooks-ts
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "function use[A-Z]\|const use[A-Z]" 2>/dev/null | grep -v "node_modules\|usehooks-ts"
```
- [ ] Every custom hook checked against usehooks-ts catalog

```bash
# Custom UI components that duplicate shadcn/Radix
git diff dev --name-only -- '*.tsx' | xargs grep -n "function [A-Z]\|const [A-Z].*=.*(" 2>/dev/null | grep -vi "page\|layout\|section\|feature" | head -20
```
- [ ] No custom components for things shadcn/Radix already provides

```bash
# Custom fetch/data hooks that duplicate SWR
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "useFetch\|useQuery\|useData\|useState.*fetch\|useEffect.*fetch" 2>/dev/null | grep -v "node_modules\|swr\|@tanstack"
```
- [ ] No custom data fetching — use SWR

```bash
# Custom debounce/throttle/retry
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "debounce\|throttle\|setTimeout.*clear\|retry.*await\|attempts.*<" 2>/dev/null | grep -v "node_modules\|lodash\|usehooks-ts\|p-retry"
```
- [ ] No custom debounce/throttle/retry — use library

```bash
# Custom ID generation
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "Math\.random\|generateId\|makeId\|createId" 2>/dev/null | grep -v "node_modules\|nanoid\|cuid\|uuid\|crypto\."
```
- [ ] No custom IDs — use `nanoid`, `cuid2`, or `crypto.randomUUID()`

```bash
# Custom deep clone/merge
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "JSON\.parse(JSON\.stringify\|deepClone\|deepMerge\|deepEqual" 2>/dev/null | grep -v "node_modules"
```
- [ ] No custom deep clone/merge — use `structuredClone` or library

```bash
# Custom auth/session instead of Supabase
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "jwt\|token.*decode\|session.*get\|cookie.*parse" 2>/dev/null | grep -v "node_modules\|supabase\|@supabase"
```
- [ ] No custom auth — use Supabase Auth / `@supabase/ssr`

```bash
# Custom error boundary
git diff dev --name-only -- '*.tsx' | xargs grep -n "componentDidCatch\|getDerivedStateFromError" 2>/dev/null | grep -v "node_modules\|react-error-boundary"
```
- [ ] No custom error boundary — use `react-error-boundary`

**Don't rebuild anything these provide (including but not limited to):**

**React:**
- [ ] `useState`, `useReducer`, `createContext`, `createPortal`, `memo`, `useMemo`, `useCallback`, `forwardRef`, `lazy` + `Suspense`, `useId`, `useTransition`, `useDeferredValue`

**Next.js:**
- [ ] `next/navigation` (useRouter, usePathname, useSearchParams), `next/link`, `next/image`, `next/font`, `next/script`
- [ ] `metadata` / `generateMetadata`, `loading.tsx`, `error.tsx`, `not-found.tsx`
- [ ] Server Actions (`'use server'`), `revalidatePath` / `revalidateTag`, `generateStaticParams`
- [ ] `middleware.ts`, `next.config.js` redirects/rewrites/headers, `Suspense` streaming, parallel routes, route groups

**Supabase:**
- [ ] `supabase.auth` (login, signup, reset, OAuth, session), `@supabase/ssr` middleware, `supabase.auth.getUser()`
- [ ] RLS policies (not app-level auth filters), Supabase Realtime, Supabase Storage, Supabase Edge Functions
- [ ] `supabase gen types typescript`, `.from().select().eq()` chain, `.textSearch()`, Database Webhooks / `pg_net`

**shadcn/ui + Radix:**
- [ ] Button, Input, Textarea, Label, Select, Checkbox, RadioGroup, Switch, Slider
- [ ] Dialog, Sheet, Popover, HoverCard, Tooltip, DropdownMenu, ContextMenu, NavigationMenu, Menubar
- [ ] Tabs, Accordion, Collapsible, Card, Alert, Badge, Avatar, Skeleton, Progress, Separator
- [ ] Toast (sonner), Command (cmdk), Calendar, Form (react-hook-form + Zod), Table (@tanstack/react-table)
- [ ] ScrollArea, AspectRatio, Breadcrumb

**SWR:**
- [ ] `useSWR` (not custom fetch hooks), `mutate` with `optimisticData`, `errorRetryCount`, `refreshInterval`, built-in deduplication

**usehooks-ts:**
- [ ] useDebounce, useDebouncedCallback, useLocalStorage, useMediaQuery, useOnClickOutside, useWindowSize
- [ ] useCopyToClipboard, useIntersectionObserver, useEventListener, useInterval, useTimeout
- [ ] useToggle, useHover, useScrollLock, useIsClient, usePrevious, useCountdown

**Utilities:**
- [ ] Date/time → `date-fns` or `Intl.DateTimeFormat`
- [ ] Validation → `zod`
- [ ] Deep clone → `structuredClone`
- [ ] Deep merge → `deepmerge` or `lodash/merge`
- [ ] IDs → `crypto.randomUUID()` or `nanoid`
- [ ] Debounce/throttle → `lodash/debounce` or `usehooks-ts`
- [ ] Retry → `p-retry`
- [ ] Slugify → `slugify`
- [ ] Crypto/hashing → platform crypto — **never custom**
- [ ] Array groupBy → `Object.groupBy`
- [ ] Query strings → `URLSearchParams`
- [ ] URL construction → `new URL()`
- [ ] Case conversion → `change-case`
- [ ] Markdown → `react-markdown`
- [ ] HTML sanitization → `dompurify`
- [ ] CSV → `papaparse`
- [ ] Form state → `react-hook-form`
- [ ] Error boundary → `react-error-boundary`
- [ ] Keyboard shortcuts → `react-hotkeys-hook`
- [ ] Drag and drop → `@dnd-kit/core`
- [ ] Virtual scrolling → `@tanstack/react-virtual`
- [ ] Animations → `framer-motion` or CSS transitions

**Produce the table:**

```
| Custom Code | What It Does | Built-in / Library | Verdict |
|-------------|-------------|-------------------|---------| 
| useDebounce() | Debounces value | usehooks-ts | ❌ REPLACE |
| <LoadingDots/> | Animated loader | shadcn Skeleton | ❌ REPLACE |
| fetchWithRetry() | Fetch + retries | SWR errorRetryCount | ❌ REPLACE |
| calculateScore() | Speqq scoring | None — domain logic | ✅ KEEP — cuts scoring |
```

- **❌ REPLACE** = stack handles this → use it
- **✅ KEEP** = nothing handles this → state what feature gets cut without it
- **⚠️ SIMPLIFY** = partially covered → use the library for the covered part

**Any ❌ REPLACE = blocker. Fix before submitting.**

---

## Step 10: Performance

**If performance tests don't exist yet, write them first (TDD). Then run them. Then confirm they pass.**

### Code Pattern Checks

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "\.map(.*await\|\.forEach(.*await\|for.*await.*supabase\|for.*await.*fetch" 2>/dev/null
```
- [ ] No N+1 patterns (await inside loops)

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "from 'lodash'\|from \"lodash\"" 2>/dev/null
```
- [ ] No full library imports — use subpath

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -c "await " 2>/dev/null | awk -F: '$2 > 3 {print "⚠️ " $1 " has " $2 " sequential awaits"}'
```
- [ ] Sequential awaits parallelized where independent (`Promise.all`)

```bash
git diff dev --name-only -- '*.tsx' | xargs grep -l "useEffect" 2>/dev/null | while read -r f; do if grep -q "subscribe\|addEventListener\|setInterval\|setTimeout" "$f"; then if ! grep -q "return.*=>\|cleanup\|unsubscribe\|removeEventListener\|clearInterval\|clearTimeout" "$f"; then echo "❌ $f"; fi; fi; done
```
- [ ] Subscriptions, listeners, timers cleaned up on unmount
- [ ] List queries have pagination or limits

### Performance Metrics — Must Have Tests

**If these tests don't exist, write them before submitting. Test-driven: write the test with the threshold first, then make it pass.**

**Page Load & Web Vitals:**

| Metric | Pass | Fail |
|--------|------|------|
| Largest Contentful Paint (LCP) | < 2.5s | > 4s |
| First Input Delay (FID) | < 100ms | > 300ms |
| Cumulative Layout Shift (CLS) | < 0.1 | > 0.25 |
| Interaction to Next Paint (INP) | < 200ms | > 500ms |
| Time to First Byte (TTFB) | < 800ms | > 1.8s |
| Lighthouse Performance Score | ≥ 90 | < 70 |

- [ ] Web vitals test exists and passes thresholds
- [ ] If no web vitals test exists → write one now

**Bundle Size:**

| Metric | Pass | Fail |
|--------|------|------|
| First Load JS (shared) | < 100KB gzip | > 150KB |
| Per-route JS chunk | < 50KB gzip | > 80KB |
| No single dependency | > 50KB gzip alone | — |

```bash
npm run build 2>&1 | grep -E "First Load JS|└|├"
```
- [ ] Bundle size test or build check exists and passes
- [ ] If no bundle size check exists → add assertion to CI or test suite

**API Response Times:**

| Endpoint Type | p95 Pass | p95 Fail |
|--------------|----------|----------|
| Simple read (get item) | < 150ms | > 300ms |
| List query (25 items, paginated) | < 300ms | > 500ms |
| Search query | < 400ms | > 800ms |
| Write (create/update) | < 250ms | > 500ms |
| Server action (mutation + revalidate) | < 500ms | > 1s |

- [ ] API response time tests exist for every new endpoint
- [ ] If no response time tests exist → write them now with these thresholds

**Database Query Performance:**

| Query Type | Pass | Fail |
|-----------|------|------|
| Simple SELECT (indexed) | < 5ms | > 20ms |
| SELECT with RLS | < 10ms | > 50ms |
| SELECT with JOIN | < 15ms | > 50ms |
| List query (25 rows) | < 10ms | > 30ms |
| Full-text search | < 30ms | > 100ms |
| INSERT / UPDATE single row | < 5ms | > 20ms |
| Seq Scan on tables > 1K rows | Never | Any |

- [ ] Every new query has been run with `EXPLAIN ANALYZE`
- [ ] No sequential scans on tables expected to exceed 1K rows
- [ ] If no query performance tests exist → write them now

**Client-Side Rendering:**

| Metric | Pass | Fail |
|--------|------|------|
| Component render | < 16ms (60fps) | > 33ms |
| List render (25 items) | < 50ms | > 100ms |
| List render (100+ items) | Virtualized | Not virtualized |
| Route transition | < 300ms perceived | > 1s |
| Search/filter response | < 100ms after debounce | > 300ms |

- [ ] React Profiler shows no component > 16ms render time
- [ ] Lists over 50 items use `@tanstack/react-virtual`

**Memory:**

| Metric | Pass | Fail |
|--------|------|------|
| Initial JS heap | < 30MB | > 50MB |
| After 30 min session | < 60MB | > 100MB |
| Heap growth (steady state) | < 0.5MB/min | > 2MB/min |
| Detached DOM nodes after nav | 0 | > 10 |

- [ ] No memory leaks — heap doesn't grow unbounded during normal use
- [ ] If no memory test exists → manually verify with Chrome DevTools Memory tab

**Concurrent Load (5,000 users target):**

| Metric | Pass | Fail |
|--------|------|------|
| DB connection pool usage | < 60% | > 80% |
| Concurrent realtime connections | < 500 | > 1000 |
| Auth token refresh | 1/user/hour | > 1/user/min |

- [ ] Supabase plan supports estimated connection + bandwidth at 5K users
- [ ] Auth middleware refreshes once per session, not per request

### Performance Test Enforcement

- [ ] Every new API endpoint has a response time test with thresholds above
- [ ] Every new database query has been `EXPLAIN ANALYZE`'d
- [ ] Every new list/table component has been tested with 100+ items
- [ ] If a performance test didn't exist before this PR and this PR adds code in that area → **write the test now**

**No performance test = no PR. Write the test first, set the threshold, then make it pass.**

---

## Step 11: Tests

**Write tests BEFORE building. If tests don't exist for the area you touched, write them now. Then run the full suite before submitting.**

### Required Test Categories

**Every feature must have tests in these categories. If they're missing, write them first.**

**1. Integration Tests (Vitest — real DB, no mocks)**
- [ ] API route handlers — request in, correct response out, data persisted
- [ ] Server actions — mutation + revalidation + correct DB state
- [ ] Data integrity — create, read, update, delete all work correctly
- [ ] Auth flows — protected routes reject unauthenticated, allow authenticated

**2. Component Tests (Vitest + Testing Library)**
- [ ] Components render without crashing
- [ ] Loading, error, and empty states all display correctly
- [ ] User interactions work (click, type, submit)
- [ ] Forms validate and submit correctly
- [ ] Conditional rendering (role-based UI, feature flags)

**3. Security Tests (Vitest — real DB)**
- [ ] RLS: user A's query returns zero of user B's records
- [ ] Unauthenticated requests to protected endpoints return 401
- [ ] Invalid/malicious input rejected by Zod validation
- [ ] No data leakage in error responses (no IDs, emails, counts)

**4. Performance Tests (Vitest + EXPLAIN ANALYZE)**
- [ ] API response times under threshold (see Step 10 metrics)
- [ ] DB queries have no sequential scans on large tables
- [ ] Lists with 100+ items render under 100ms

**5. E2E Tests (Playwright)**
- [ ] Critical user flows: signup → onboard → create → edit → delete
- [ ] Navigation: main routes load without error
- [ ] Auth: login, logout, session persistence, protected redirect
- [ ] Multi-tenant: two users in parallel, neither sees the other's data

### TDD Order — When to Write What

| When | Write |
|------|-------|
| Before building any feature | Integration test for the API/data layer |
| Before building any UI | Component test for the component |
| Before any DB migration | Security test for RLS policies |
| Before submitting PR | Performance test for new endpoints/queries |
| After feature complete | E2E test for the critical user flow |

### Run the Full Suite

**Before submitting, run ALL test categories — not just the ones you wrote.**

```bash
# 1. Unit + Integration + Component + Security + Performance tests
npm run test
```
- [ ] All tests pass

```bash
# 2. E2E tests
npx playwright test
```
- [ ] All E2E tests pass

```bash
# 3. Check for skipped or .only tests
git diff dev --name-only -- '*.test.*' '*.spec.*' | xargs grep -n "\.skip\|xit\|xdescribe\|\.only" 2>/dev/null
```
- [ ] No skipped or `.only` tests

```bash
# 4. Verify new files have test coverage
git diff dev --name-only -- '*.ts' '*.tsx' | grep -v "\.test\.\|\.spec\.\|__test" | while read -r src; do base=$(basename "$src" | sed 's/\.tsx\?$//'); if ! git diff dev --name-only | grep -q "${base}\.test\.\|${base}\.spec\."; then echo "⚠️ $src — no test file"; fi; done
```
- [ ] New source files have corresponding test files
- [ ] Tests cover happy path + error path + edge case
- [ ] Multi-tenant: tests verify data isolation between users

**All 5 categories must pass. A passing PR means: integration ✅ components ✅ security ✅ performance ✅ E2E ✅**

---

## Step 12: Diff Review

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "console\.log\|console\.debug\|debugger\b\|alert(" 2>/dev/null | grep -v "// keep\|logger\.\|\.test\.\|\.spec\."
```
- [ ] No debug code

```bash
git diff dev --name-only -- '*.ts' '*.tsx' | xargs grep -n "TODO\|FIXME\|HACK\|XXX" 2>/dev/null | grep -v "\.test\.\|\.spec\."
```
- [ ] No leftover TODOs (unless tracked with issue number)

```bash
git diff dev --name-only | grep -E "\.env$|\.env\.local|\.DS_Store|\.pem|\.key|node_modules"
```
- [ ] No accidental files

```bash
git diff dev --name-only
```
- [ ] All changed files are related to this task
- [ ] No commented-out code blocks

---

## Step 13: Submit

```bash
git fetch origin dev
echo "Commits behind dev: $(git rev-list HEAD..origin/dev --count)"
```
- [ ] Branch rebased on latest `dev`

```bash
echo "Branch: $(git branch --show-current)"
```
- [ ] Branch follows `<type>/<description>` naming

```bash
git add -A
git diff --cached --stat
git commit -m "<type>(<scope>): <description>"
git push origin HEAD
```

```bash
gh pr create \
  --base dev \
  --title "<type>(<scope>): <short description>" \
  --body "## What
<1-2 sentence summary>

## Why
<Link to issue + context>

## How
<Key decisions, tradeoffs>

## Multi-Tenant Readiness
- [x] Data scoped to current user via RLS
- [x] Empty states handled for new users
- [x] No hardcoded IDs, emails, or dev-only config
- [x] Cross-tenant data leakage verified

## Checklist
- [x] Build passes (zero warnings in changed files)
- [x] Lint passes (zero warnings in changed files)
- [x] Tests pass (new tests included)
- [x] Type check passes (no unjustified any)
- [x] Security scan clean
- [x] Next.js patterns followed
- [x] Supabase RLS verified
- [x] Self-reviewed diff"
```

---

## Preflight Report

Output before submitting:

```
# Preflight Report

| Step | Status |
|------|--------|
| 1. Env | ✅ / ❌ |
| 2. Security | ✅ / ❌ |
| 3. Types | ✅ / ❌ |
| 4. Build | ✅ / ❌ |
| 5. Lint | ✅ / ❌ |
| 6. Next.js | ✅ / ❌ |
| 7. Supabase | ✅ / ⏭️ |
| 8. Multi-tenant | ✅ / ❌ |
| 9. Custom code | ✅ / ❌ |
| 10. Performance | ✅ / ❌ |
| 11. Tests | ✅ / ❌ |
| 12. Diff review | ✅ / ❌ |
| 13. Submit | ✅ / ❌ |

🟢 ALL CLEAR — PR submitted.
🔴 BLOCKED — <what failed>
```

---

## NEVER

1. **NEVER submit with build/lint errors or warnings in your files**
2. **NEVER skip the security scan**
3. **NEVER skip multi-tenant checks**
4. **NEVER leave debug code in the PR**
5. **NEVER commit `.env` files or credentials**
6. **NEVER delete or skip tests to pass the suite**
7. **NEVER include unrelated file changes**
8. **NEVER skip steps for "small changes"**
9. **NEVER hardcode user IDs, org IDs, or emails**
10. **NEVER assume pre-existing data — must work on first login**
