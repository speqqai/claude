# Test Plan Template

## 1. Document Control
- Test Plan Title:
- Feature/Initiative:
- PRD Reference: [Link to PRD file]
- TDD Reference: [Link to TDD file]
- Status: Draft | In Progress | Complete
- Last Updated:

## 2. PRD Requirements Coverage

### 2.1 Requirements-to-Tests Traceability

Every PRD requirement must have at least one test. Map them here:

| PRD Requirement | Test Category | Test ID | Test Description | Priority |
|---|---|---|---|---|
| REQ-001: [title from PRD] | Integration / Component / Security / Performance / E2E | T-001 | [What the test verifies] | P0/P1/P2 |
| REQ-001: [title from PRD] | Integration | T-002 | [Second test for same requirement] | P0/P1/P2 |
| REQ-002: [title from PRD] | Component | T-003 | [What the test verifies] | P0/P1/P2 |

### 2.2 Coverage Gaps

| PRD Requirement | Missing Test Category | Reason | Action |
|---|---|---|---|
| [Any requirement not fully covered] | [Which category is missing] | [Why] | [What to do] |

## 3. Integration Tests

Tests that verify the feature works end-to-end: API route handlers, server actions, data persistence.

| Test ID | Test Name | PRD Requirement | Input | Expected Output | Failure Behavior |
|---|---|---|---|---|---|
| T-001 | [Descriptive test name] | REQ-xxx | [Request/action] | [Expected response/state] | [What happens on failure] |
| T-002 | | | | | |

### 3.1 Auth & Access Control Tests

| Test ID | Test Name | PRD Requirement (Who column) | Actor | Action | Expected Result |
|---|---|---|---|---|---|
| T-xxx | [e.g., rejects unauthenticated access to feature] | REQ-xxx (Admin, Business plan) | Unauthenticated user | [Action] | 401 Unauthorized |
| T-xxx | [e.g., allows admin on Business plan] | REQ-xxx | Admin (Business plan) | [Action] | Success |
| T-xxx | [e.g., rejects viewer on Free plan] | REQ-xxx | Viewer (Free plan) | [Action] | 403 Forbidden |

## 4. Component Tests

Tests that verify UI renders correctly and responds to interaction.

| Test ID | Test Name | PRD Requirement | Component | User Action | Expected Render/Behavior |
|---|---|---|---|---|---|
| T-xxx | [e.g., renders loading skeleton while fetching] | REQ-xxx | [Component path] | [Load page] | [Skeleton visible] |
| T-xxx | [e.g., shows empty state when no data] | REQ-xxx | [Component path] | [Load with no data] | [EmptyState visible] |
| T-xxx | [e.g., shows error toast on API failure] | REQ-xxx | [Component path] | [Trigger error] | [Sonner toast with error message] |

### 4.1 UX State Tests

Every data surface must have all three states tested:

| Component | Loading Test | Error Test | Empty Test | PRD Requirement |
|---|---|---|---|---|
| [Component name] | T-xxx | T-xxx | T-xxx | REQ-xxx |

## 5. Security Tests

Tests that verify access controls, RLS policies, and input validation.

| Test ID | Test Name | PRD Requirement (Who) | Attack Vector | Expected Defense |
|---|---|---|---|---|
| T-xxx | [e.g., RLS blocks cross-tenant data access] | REQ-xxx | User A queries User B's data | Zero rows returned |
| T-xxx | [e.g., rejects malicious input] | REQ-xxx | SQL injection in field X | Zod validation rejects |
| T-xxx | [e.g., no data leakage in error response] | REQ-xxx | Invalid ID in request | Generic error, no IDs/emails exposed |

## 6. Performance Tests

Tests that verify response times and query efficiency at scale.

| Test ID | Test Name | PRD Requirement | Metric | Threshold | Measurement |
|---|---|---|---|---|---|
| T-xxx | [e.g., list endpoint p95 < 300ms] | REQ-xxx | Response time (p95) | < 300ms | [How measured] |
| T-xxx | [e.g., no sequential scan on table X] | REQ-xxx | Query plan | No Seq Scan | EXPLAIN ANALYZE |

## 7. E2E Tests

Tests that verify critical user flows in a real browser.

| Test ID | Test Name | PRD Requirements Covered | User Flow Steps | Expected Outcome |
|---|---|---|---|---|
| T-xxx | [e.g., full feature creation flow] | REQ-001, REQ-003 | 1. Navigate to X → 2. Click Y → 3. Fill Z → 4. Submit | [Feature created, visible in list] |

## 8. Acceptance Criteria Coverage

Map each PRD requirement's acceptance criteria to specific tests:

### REQ-001: [title from PRD]

| Acceptance Criterion (from PRD) | Test ID | Test Category | Status |
|---|---|---|---|
| [First criterion from PRD detail table] | T-xxx | Integration | Planned / Written / Passing |
| [Second criterion] | T-xxx | Component | Planned / Written / Passing |
| [Third criterion] | T-xxx | Security | Planned / Written / Passing |

### REQ-002: [title from PRD]

| Acceptance Criterion (from PRD) | Test ID | Test Category | Status |
|---|---|---|---|
| [First criterion] | T-xxx | Integration | Planned / Written / Passing |

## 9. TDD Execution Order

The order tests should be written and run, following the TDD cycle:

| Phase | Test IDs | Category | PRD Requirements | Depends On |
|---|---|---|---|---|
| 1 | T-001, T-002 | Integration | REQ-001 | None |
| 2 | T-003, T-004 | Security | REQ-001 | Phase 1 |
| 3 | T-005, T-006 | Component | REQ-001, REQ-002 | Phase 1 |
| 4 | T-007 | Performance | REQ-001 | Phase 1 |
| 5 | T-008 | E2E | REQ-001, REQ-002, REQ-003 | Phase 1-4 |

## 10. Test Plan Summary

| Category | Total Tests | P0 Tests | P1 Tests | P2 Tests | PRD Requirements Covered |
|---|---|---|---|---|---|
| Integration | | | | | |
| Component | | | | | |
| Security | | | | | |
| Performance | | | | | |
| E2E | | | | | |
| **Total** | | | | | |

### Coverage Check

- [ ] Every PRD requirement (REQ-xxx) has at least one test
- [ ] Every PRD acceptance criterion maps to a test
- [ ] Every PRD Who column (roles/plans) has auth/access control tests
- [ ] Every data surface has loading/error/empty state tests
- [ ] All three UX states tested for every new component
- [ ] Security tests cover RLS, auth, input validation
- [ ] No PRD requirement is untested
