# Phase 4: Implementation and Testing — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 4 implementation output. You must return PASS or FAIL with specific reasons.

## What to review
Read the PRD at `.docs/{project-name}-prd.md`.
Read the design document at `.docs/{project-name}-design.md`.
Read the rules directory — every rule file in rules/README.md.
Read every file that was created or modified during this phase (check git diff against the branch base).
Run the validation commands listed below.

## Validation checklist

### PRD completeness:
- [ ] Every functional requirement in the PRD has been implemented.
- [ ] Every acceptance criterion in the PRD can be verified by running the corresponding E2E test.

### Design compliance:
- [ ] The implementation matches the design document — no files, routes, components, or database changes exist that are not in the design document.
- [ ] The implementation uses the framework, SDK, or library function specified in the design document for each requirement.

### Database:
- [ ] All migrations from the design document have been run.
- [ ] New tables and columns exist in the database (verify by querying).
- [ ] RLS policies are active on every new table (verify by querying `pg_policies` or equivalent).

### Build and runtime:
- [ ] `npm run build` completes with zero errors.
- [ ] `npm run dev` starts without runtime errors.
- [ ] All E2E tests from Phase 2 pass.

### TypeScript:
- [ ] TypeScript strict compilation passes with zero errors.
- [ ] No `any` types exist in changed files.
- [ ] No `@ts-ignore` or `@ts-expect-error` exists without an explanatory comment.
- [ ] Every function parameter and component prop has an explicit type.

### Lint:
- [ ] Lint passes with zero warnings in all changed files.

### Custom Code Policy:
- [ ] No custom code exists for behavior that a framework, SDK, or library in package.json already provides.
- [ ] Every custom implementation is Speqq-specific product logic that no dependency handles.

### Hardcoding Policy:
- [ ] No hardcoded UUIDs, database IDs, URLs, API endpoints, workspace IDs, organization IDs, timeouts, retry counts, page sizes, secrets, API keys, or tokens exist in the implementation.
- [ ] Configuration values come from environment variables, the database, or named constants.

### Naming Policy:
- [ ] Every function, variable, component, and file name is descriptive and domain-qualified.
- [ ] No bare nouns (`id`, `name`, `title`, `status`, `type`, `value`, `data`, `error`) exist without a domain qualifier.

### UI/UX Policy:
- [ ] Every new data-driven surface has a loading state (Skeleton), error state (destructive text or Sonner toast), and empty state (EmptyState component).
- [ ] Every interactive element uses a shadcn/ui component — no raw HTML interactive elements.
- [ ] Every color uses a semantic token — no hex colors, no Tailwind palette colors, no `dark:` variants.

### Error Handling Policy:
- [ ] Every expected failure produces a user-visible message — no silent catches, no empty catch blocks, no console-only logging.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
