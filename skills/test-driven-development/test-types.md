# Test Types — What to Write, When, and Where

## 5 Required Test Categories

### 1. Integration Tests (Vitest — real DB, no mocks)
**"Does the feature actually work end-to-end?"**
- API route handlers — request in, correct response out, data persisted
- Server actions — mutation + revalidation + correct DB state
- Data integrity — create, read, update, delete all work correctly
- Auth flows — protected routes reject unauthenticated, allow authenticated

**When:** Before building any feature — write the API/data test first.

### 2. Component Tests (Vitest + Testing Library)
**"Does the UI render correctly and respond to interaction?"**
- Components render without crashing
- Loading, error, and empty states all display correctly
- User interactions work (click, type, submit)
- Forms validate and submit correctly
- Conditional rendering (role-based UI, feature flags)

**When:** Before building any UI — write the component test first.

### 3. Security Tests (Vitest — real DB)
**"Can someone do something they shouldn't?"**
- RLS: user A's query returns zero of user B's records
- Unauthenticated requests to protected endpoints return 401
- Invalid/malicious input rejected by Zod validation
- No data leakage in error responses (no IDs, emails, counts)

**When:** Before any DB migration — write the RLS/auth test first.

### 4. Performance Tests (Vitest + EXPLAIN ANALYZE)
**"Is it fast enough at scale?"**
- API response times under threshold (p95 < 300ms reads, < 500ms writes)
- DB queries have no sequential scans on large tables
- Lists with 100+ items render under 100ms

**When:** Before submitting PR — write performance test with threshold, then verify it passes.

### 5. E2E Tests (Playwright)
**"Does the full user flow work in a real browser?"**
- Critical user flows: signup → onboard → create → edit → delete
- Navigation: main routes load without error
- Auth: login, logout, session persistence, protected redirect
- Multi-tenant: two users in parallel, neither sees the other's data

**When:** After feature complete — write E2E for the critical path.

---

## TDD Order

| When | Write |
|------|-------|
| Before building any feature | Integration test for the API/data layer |
| Before building any UI | Component test for the component |
| Before any DB migration | Security test for RLS policies |
| Before submitting PR | Performance test for new endpoints/queries |
| After feature complete | E2E test for the critical user flow |

## Before Submitting PR — Run ALL Categories

```bash
# All unit + integration + component + security + performance
npm run test

# All E2E
npx playwright test
```

All 5 categories must pass: integration ✅ components ✅ security ✅ performance ✅ E2E ✅
