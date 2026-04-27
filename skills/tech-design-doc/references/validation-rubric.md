# TDD Validation Rubric

Use this rubric after drafting and before final verdict.

## Critical Checks (Must Pass)

### 1. PRD Traceability
- Every PRD requirement (REQ-xxx) has a row in the PRD-to-Design Traceability table.
- Every design section references which PRD requirement it fulfills.
- No design element exists that doesn't trace to a PRD requirement.
- The Who column from the PRD is reflected in authorization design.

Fail if: Any REQ-xxx is missing from the traceability table, or design includes features not in the PRD.

### 2. Research References
- Every SDK/framework used in the design has documentation referenced in Section 3.
- References are specific (doc URL or section name), not generic ("see Next.js docs").
- Key findings demonstrate the agent actually read the docs, not guessed.
- Version compatibility is confirmed for every SDK/framework.

Fail if: Any SDK/framework usage lacks a documentation reference, or references are generic/unverifiable.

### 3. Existing Codebase Reuse
- Section 4 documents existing capabilities that are reused.
- The design does not recreate patterns, components, or utilities that already exist.
- Existing auth flows, data hooks, UI patterns, and database structures are extended rather than replaced.

Fail if: The design creates something new when an equivalent already exists in the codebase.

### 4. Zero Custom Code
- Section 10 (Custom Code Audit) is complete.
- Every piece of logic uses SDK/framework built-in capabilities.
- Glue code (connecting two framework features) is acceptable IF documented with justification.
- Any custom code has explicit justification: what SDK capability is missing, with a doc reference showing the gap.
- No reimplementation of behavior available in an approved dependency.

Fail if: Custom code exists without explicit justification citing the SDK gap, or behavior reimplements an existing SDK capability.

### 5. Architecture Diagram
- An ASCII architecture diagram exists in Section 6.
- The diagram shows component relationships and system boundaries.
- Data flow direction is clear (arrows showing request/response paths).
- External services, databases, and client/server boundaries are labeled.

Fail if: No architecture diagram, or diagram is too vague to understand component relationships.

### 6. Error Handling and Logging
- Every failure point has an entry in the Error Points table (Section 8).
- Every error is both logged AND surfaced to the user.
- No silent failures or swallowed errors.
- No fallback logic — errors are reported, not worked around.
- Error patterns follow existing codebase conventions.

Fail if: Any failure point lacks error logging, errors are swallowed, or fallback logic exists.

### 7. Security
- Authentication approach is documented.
- Authorization is mapped from PRD Who column (roles AND plan tiers).
- RLS policies are defined for new/modified tables.
- Input validation is server-side with framework/library specified.
- Sensitive data handling is addressed.

Fail if: Security section is missing, authorization doesn't match PRD Who column, or input validation is missing.

### 8. Server-Side Default
- All logic is server-side unless explicitly justified.
- Any client-side logic has a specific reason documented in Section 7.4.
- "User interactivity" is not a blanket justification — only specific interaction patterns that cannot be server-rendered.

Fail if: Client-side logic exists without explicit justification, or justification is generic.

### 9. No Fallbacks
- The design does not include fallback implementations.
- No "if X fails, try Y" patterns (except for user-visible error states).
- No feature flags for "just in case" scenarios.
- No backward-compatibility shims.

Fail if: Fallback logic, defensive workarounds, or "just in case" feature flags exist in the design.

### 10. No Removed Sections
- The TDD does not include: Test Strategy, Monitoring & Observability, or Rollback Plan.
- These are handled by separate processes (TDD, ops) and must not be in the TDD.

Fail if: Test strategy, monitoring/observability, or rollback plan sections are present.

## Quality Scoring (0-16)
Score each area 0-2:
- 0 = missing or unusable.
- 1 = present but incomplete or unclear.
- 2 = clear, complete, and implementation-ready.

Areas:
- PRD Traceability (every requirement mapped)
- Research Quality (specific doc references, key findings)
- Existing Code Reuse (thorough audit of what exists)
- Architecture Clarity (diagram + data flow + component responsibilities)
- API Contracts (complete request/response shapes, auth, error responses)
- Error Handling (every failure point covered, no silent failures)
- Security (auth, authz, RLS, validation, sensitive data)
- Custom Code Discipline (zero custom code or justified minimal glue)

Interpretation:
- 14-16: Production-grade and implementation-ready.
- 11-13: Good base, revise weak areas before implementation.
- <=10: Not ready, major revision required.

## Common Rewrite Triggers
Rewrite a design section if it:
- References an SDK/framework without citing specific documentation.
- Introduces custom logic where a built-in SDK capability exists.
- Designs a new component/pattern when one already exists in the codebase.
- Includes fallback or defensive logic.
- Puts logic on the client without justification.
- Swallows or ignores errors.
- Doesn't map back to a PRD requirement.
- Describes authorization that doesn't match the PRD Who column.

## Final Readiness Gate
Set `Ready for Implementation` only when:
- All 10 critical checks pass.
- Rubric score >=14/16.
- No blocker-level open questions remain.
- PRD-to-Design traceability is complete.
- Custom Code Audit verdict is "Zero custom code" or "Minimal glue code only."
- Founder approval is recorded.
