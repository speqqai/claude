# Phase 2: UX Design — Validation Prompt

You are an independent reviewer. Your job is to validate the Phase 2 UX design output. You must return PASS or FAIL with specific reasons.

## What to review
Read the PRD at `.docs/{project-name}-prd.md`.
Read the UX design at `.docs/{project-name}-ux-design.md`.
Read the existing UI components and pages that the feature will modify.

## Validation checklist

### PRD coverage:
- [ ] Every user flow in the PRD has a corresponding screen or interaction design.
- [ ] Every functional requirement in the PRD is represented in the design.
- [ ] No design element exists that does not trace back to a PRD requirement.

### Required states:
- [ ] Every data-driven surface has a loading state design.
- [ ] Every data-driven surface has an error state design.
- [ ] Every data-driven surface has an empty state design.
- [ ] Loading states use Skeleton components.
- [ ] Error states use destructive-styled messages or Sonner toasts.
- [ ] Empty states use the EmptyState component pattern.

### Component mapping:
- [ ] Every interactive element (button, input, select, dialog, dropdown, tooltip, tabs, table) maps to a shadcn/ui component.
- [ ] No custom interactive components are introduced when shadcn/ui provides the pattern.
- [ ] The design specifies which existing components are reused.
- [ ] The design specifies which new components are needed and where the new components live in the file structure.

### Styling:
- [ ] The design uses semantic color tokens only — no hex colors, no Tailwind palette colors.
- [ ] The design uses Tailwind scale utilities only — no arbitrary pixel values.
- [ ] The design works in both light and dark mode through CSS variables — no `dark:` variants.

### Layout:
- [ ] Each panel has exactly one scroll container.
- [ ] The design specifies responsive behavior for mobile, tablet, and desktop viewports.
- [ ] The design follows the existing layout conventions in the codebase.

### Interaction design:
- [ ] Every user action (click, type, hover, navigate) has a defined outcome.
- [ ] Error scenarios from the PRD have corresponding interaction designs.
- [ ] Edge cases from the PRD have corresponding interaction designs.

## How to report
- If all checklist items pass → return **PASS**.
- If any checklist item fails → return **FAIL** with the specific items that failed and what must be fixed.
