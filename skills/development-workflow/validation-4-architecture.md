# Phase 3: Architecture Design — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 3 architecture design output. You must return PASS or FAIL with specific reasons.

## What to review
Read the PRD at `.docs/{project-name}-prd.md`.
Read the design document at `.docs/{project-name}-design.md`.
Read the rules directory — especially Custom Code Policy, Hardcoding Policy, and Data and Backend Policy.
Read the existing codebase files referenced in the design document to verify the design uses existing patterns.

## Validation checklist

### PRD traceability:
- [ ] Every functional requirement in the PRD maps to a specific design decision in the design document.
- [ ] No PRD requirement is missing from the design document.
- [ ] No design decision exists that does not trace back to a PRD requirement.

### Existing pattern usage:
- [ ] The design uses existing file structure and directory patterns — no new directory structure is introduced unnecessarily.
- [ ] The design uses existing UI component patterns — no new component patterns are introduced when existing patterns handle the need.
- [ ] The design uses existing data access patterns — database queries follow the same style as existing Route Handlers and Server Components.
- [ ] The design uses existing API route patterns — new routes follow the same conventions as existing routes.

### Zero bloat:
- [ ] No new abstraction layer is introduced (no service classes, no repositories, no ORM wrappers, no data-access helpers).
- [ ] No new utility file is created for a one-time operation.
- [ ] No new dependency is added when an existing dependency handles the need.
- [ ] Every new file in the design maps to a specific PRD requirement.

### Database changes:
- [ ] New tables, columns, indexes, and RLS policies are specified if the feature requires database changes.
- [ ] Migration SQL is included in the design document.
- [ ] RLS policies are specified for every new table.

### API route changes:
- [ ] New routes specify the route path, HTTP method, request shape, and response shape.
- [ ] New routes follow existing route conventions in the codebase.

### Data flow:
- [ ] The design describes how data moves from database to API route to Server Component to Client Component.
- [ ] Client Components receive data as props from Server Components — Client Components do not query the database directly.

### Custom Code Policy compliance:
- [ ] The design does not introduce custom code for behavior that a framework, SDK, or library in package.json already provides.
- [ ] If the design includes custom code, the custom code implements Speqq-specific product behavior that no dependency provides.

### What was rejected:
- [ ] The design explicitly states patterns, abstractions, or infrastructure that were considered and rejected.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
