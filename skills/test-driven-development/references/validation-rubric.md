# Test Plan Validation Rubric

Use this rubric after completing the test plan and before beginning TDD execution.

## Critical Checks (Must Pass)

### 1. PRD Traceability
- Every PRD requirement (REQ-xxx) has at least one row in the Requirements-to-Tests Traceability table (Section 2.1).
- Every PRD acceptance criterion maps to a specific test in Section 8.
- No test exists that doesn't trace to a PRD requirement.

Fail if: Any REQ-xxx is missing from the traceability table, or any acceptance criterion is untested.

### 2. Auth & Access Control Coverage
- Every unique role/plan combination from the PRD Who column has at least one auth test.
- Tests verify both allowed AND denied access (positive and negative cases).
- Unauthenticated access is tested for every protected endpoint.

Fail if: Any role/plan from PRD Who column is untested, or only positive access tests exist.

### 3. Test Category Coverage
- Every PRD requirement has tests across the appropriate categories (integration, component, security, performance, E2E).
- Integration tests exist for every API route and server action.
- Component tests exist for every new UI component.
- Security tests exist for every new RLS policy and auth check.

Fail if: A PRD requirement only has one test category when multiple are relevant.

### 4. UX State Coverage
- Every new data surface has three tests: loading, error, empty.
- These are documented in the UX State Tests table (Section 4.1).

Fail if: Any new data surface is missing a loading, error, or empty state test.

### 5. Failure Behavior Defined
- Every integration test specifies what happens on failure (Section 3, Failure Behavior column).
- Error responses are tested (not just happy paths).
- No silent failure scenarios are untested.

Fail if: Any integration test is missing failure behavior, or only happy paths are tested.

### 6. TDD Execution Order
- Section 9 defines the order tests should be written.
- Integration tests come before component tests.
- Security tests come before E2E tests.
- Dependencies between phases are explicit.

Fail if: No execution order defined, or order violates TDD principles (e.g., E2E before integration).

### 7. Test Naming Quality
- Every test has a descriptive name that states the behavior being tested.
- No vague names like "test1", "it works", "handles data".
- Test names read as behavior specifications.

Fail if: Any test has a vague or non-descriptive name.

### 8. No Mocks Where Avoidable
- Integration tests use real database, not mocks.
- Security tests use real RLS policies, not mocks.
- Mocks are only used for external services (email, payment, etc.) with justification.

Fail if: Tests mock internal behavior that can be tested with real code.

## Quality Scoring (0-12)
Score each area 0-2:
- 0 = missing or unusable.
- 1 = present but incomplete.
- 2 = clear, complete, and covers edge cases.

Areas:
- PRD Traceability (every requirement and acceptance criterion mapped)
- Auth & Access Control (every role/plan tested, positive and negative)
- Integration Tests (every API route, success and failure paths)
- Component Tests (every new UI, all three UX states)
- Security Tests (RLS, input validation, data leakage)
- Test Execution Order (clear phases, correct dependencies)

Interpretation:
- 10-12: Comprehensive test plan, ready for TDD execution.
- 7-9: Good base, fill gaps before starting TDD.
- <=6: Not ready, major revision required.

## Common Rewrite Triggers

Rewrite a test plan entry if it:
- Doesn't trace to a PRD requirement.
- Only tests happy paths without failure behavior.
- Has a vague test name.
- Mocks internal code that can be tested directly.
- Missing auth/access control for a role/plan in the PRD Who column.
- Missing UX state tests (loading/error/empty) for a new data surface.
- Tests implementation details instead of behavior.

## Final Readiness Gate

Set test plan to `Complete` only when:
- All 8 critical checks pass.
- Rubric score >=10/12.
- Every PRD requirement has tests across all relevant categories.
- Every acceptance criterion maps to a test.
- No coverage gaps remain unaddressed.
