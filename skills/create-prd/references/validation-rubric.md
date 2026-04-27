# PRD Validation Rubric

Use this rubric after drafting and before final verdict.

## Critical Checks (Must Pass)
0. Scope alignment loop is completed with explicit user confirmation before drafting.
1. Product definition is explicit (name, tagline, description, surfaces, lifecycle).
2. User specificity is concrete — primary persona described as a real-world person (not system role) with scenario trigger/workaround/cost.
3. Problem statement links pain to measurable consequence.
4. Solution is described as user capabilities, not implementation design.
5. Requirements Summary Table exists with all requirements in a single scannable table (ID, Title, Requirement, Who, Why, Priority).
6. Every requirement has a Requirement Detail Table with all fields: Requirement ID, Title, Requirement Statement, Who, Why, Priority, Capability IDs, Metric IDs, Interaction Notes, Cascade Notes, Acceptance Criteria, Failure / Edge Criteria.
7. Requirements are written only in the summary/detail tables, not as freeform bullets or prose elsewhere.
8. Who field in both summary and detail tables specifies both roles AND plan tiers explicitly.
9. Acceptance Criteria are objective and independently testable.
10. Failure / Edge Criteria are explicit where relevant.
11. Non-goals are explicit and prevent obvious scope creep.
12. Goals map to measurable metrics.
13. Guardrail/counter-metrics are defined to detect negative side effects.
14. Evidence/source notes support key problem and outcome claims (or are labeled assumptions).
15. Traceability matrix is complete (Goal -> Metric -> Capability -> Requirement).
16. Risks are documented in table format with severity, owner, and mitigation.
17. Dependencies include critical-path status (`Blocking`/`Non-blocking`).
18. Rollback plan includes trigger, owner, and executable steps.
19. Assumptions are tracked with owner, due date, and confidence.
20. Priority distribution is coherent (P0 inflation is justified if present).
21. Language is precise and avoids unquantified vague terms in requirements.
22. No unresolved high-impact, low-confidence assumptions remain.
23. Safety/privacy/policy impact is explicitly assessed and addressed (or justified as not applicable).
24. All required approvers are listed with explicit approval status.
25. Rollout phases include explicit entry/exit criteria.
26. Pre-launch evaluation matrix covers happy path, failure path, edge cases, and policy/safety checks.
27. Post-launch monitoring cadence and owner are explicit.
28. Technical Considerations section provides platform, surface, data, and integration context for engineers.
29. Ambiguity resolution is complete — every requirement answers: who gets this (roles + plans), how it interacts with existing features, and how it cascades to related objects.
30. Requirements Summary Table and Detail Tables are consistent (same IDs, same titles, no orphans).
31. Final preflight checklist passes with no failed checks.
32. Review Iteration Log documents all five review passes.

If any critical check fails, PRD is not ready.

## Quality Scoring (0-18)
Score each area 0-2:
- 0 = missing or unusable.
- 1 = present but ambiguous/incomplete.
- 2 = clear, complete, and decision-ready.

Areas:
- Product Context
- User Insight (real-world personas, not system roles)
- Problem Definition
- Capability Definition
- Requirement Quality (summary table + one detail table per requirement, with all requirement metadata in tables)
- Scope Boundaries
- Metrics and Traceability
- Risks/Dependencies (table format with severity/owner/mitigation)
- Rollout Plan

Interpretation:
- 16-18: Production-grade and sign-off ready.
- 13-15: Good base, revise weak areas before sign-off.
- <=12: Not ready, major rewrite required.

## Common Rewrite Triggers
Rewrite a requirement if it:
- Uses subjective language without quantification.
- Omits actor, condition, or expected result.
- Bundles multiple unrelated behaviors in one statement.
- Has no Failure / Edge Criteria where one is expected.
- Cannot be validated by PM/engineering/QA using only the PRD.
- Has a Who column that is missing roles or plan tiers.
- Is written outside the requirement tables.
- Exists in the summary table but has no detail table (or vice versa).
- Has Acceptance Criteria that are not objective pass/fail checks.
- Omits interaction notes or cascade notes where those behaviors matter.

## Final Readiness Gate
Set `Ready for Implementation` only when:
- All critical checks pass.
- Rubric score >=16/18.
- All five review passes pass.
- No blocker-level open question remains.
- No placeholders remain in required sections.
- Requirements Summary Table and Detail Tables are consistent.
- Every Who column includes both roles and plan tiers.
- Requirements are written only in the canonical tables.
- Ambiguity resolution is complete for every requirement.
