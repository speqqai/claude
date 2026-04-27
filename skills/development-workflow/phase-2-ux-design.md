# Phase 2: UX Design

## Purpose
The agent produces Figma mocks by invoking the `/ux-design` skill. The Figma mocks serve as the exact visual specification for implementation — when the user approves the mock, the next engineer builds exactly what the mock shows.

## Input
The agent must read the following before invoking the skill:
- The status file — `.docs/{project-name}-status.md`.
- The PRD — `.docs/{project-name}-prd.md`.
- The existing UI surfaces that the feature will modify — read the current components and pages.

## Work
The agent must invoke `/ux-design` with the following context:
- The PRD requirements, user flows, edge cases, and error scenarios.
- The affected surfaces from the Phase 0 status file.
- The existing UI patterns found in the codebase.

The `/ux-design` skill handles the full Figma design process — reading the design system, creating screens, creating all states, creating interaction variants, creating responsive variants, and generating code connect mappings. The agent must not duplicate the skill's work.

## Phase-specific constraints
The agent must verify the `/ux-design` output meets these additional constraints:
- Every PRD user flow has a complete set of step-by-step Figma frames.
- Every data-driven surface has loading, error, empty, and populated states in Figma.
- Every interactive element has default, hover, active, focused, and disabled variants in Figma.
- Every screen has mobile (375px), tablet (768px), and desktop (1280px) responsive variants.
- All colors in the Figma file use Speqq design tokens from `styles/globals.css` lines 12–1202.
- No custom interactive components were created when the Figma design system has an existing component.

## Output
- A Figma file with all screens, states, interactions, and responsive variants.
- `.docs/{project-name}-ux-design.md` — summary document with the Figma file URL and a list of every screen and state.
- Code Connect mappings linking Figma components to codebase components.
- The status file updated with Phase 2 marked as IN PROGRESS.

## Gate
The `/ux-design` skill completes successfully. Every PRD requirement has a corresponding Figma screen or state. All design tokens match the Speqq design system. The user approves the Figma mocks before the agent proceeds to Phase 3.
