---
name: application-development
description: Build or refactor production-grade web applications with scalable architecture using official framework guidance first. Use when implementing or reviewing Next.js, React, TypeScript, Node.js, CSS architecture, accessibility, and performance decisions for maintainable app development with minimal ad hoc patterns.
---

# Application Development

Build robust apps by defaulting to official framework patterns, strong typing, accessible UI, and measurable performance.

## Workflow

1. Load references in this order.
- Read `references/official-sources.md` first.
- For UI work, also read `references/css-and-accessibility.md`.
- For architecture or scaling concerns, also read `references/scalability-checklist.md`.

2. Set architecture boundaries before coding.
- Define feature boundaries, ownership, and data flow.
- Keep domain logic out of UI components.
- Keep server-only and client-only modules explicit.

3. Apply framework-first implementation.
- In Next.js, prefer App Router conventions and built-in primitives before custom infrastructure.
- In React, minimize effects and derive state when possible.
- In TypeScript, use strict types and typed interfaces at API boundaries.

4. Centralize styling.
- Define design tokens once (colors, spacing, typography, radius, motion).
- Reuse shared primitives and component-level styles; avoid one-off inline patterns.
- Keep CSS architecture predictable (tokens -> utilities -> components).

5. Enforce accessibility by default.
- Use semantic HTML first.
- Ensure keyboard navigation, visible focus states, and accessible labels.
- Validate color contrast and announce async/status updates correctly.

6. Build for performance and operations.
- Choose rendering/data-fetching strategy intentionally.
- Reduce JavaScript payload and avoid unnecessary client components.
- Monitor Core Web Vitals and fix regressions before release.

7. Validate and ship.
- Run lint/type/test/build checks.
- Confirm key user flows manually.
- Document tradeoffs, risks, and follow-up tasks.

## Non-Negotiable Rules

1. Prefer official docs and first-party APIs over custom abstractions.
2. Avoid introducing new dependencies without clear justification.
3. Do not duplicate business logic across routes/components.
4. Do not ship UI that fails keyboard navigation or basic WCAG checks.
5. Do not add styling primitives outside the centralized token system.

## Output Contract

Return work in this order:

1. Architecture summary.
- Boundaries, data flow, and rendering choices.

2. Implementation map.
- What official framework features were used.
- What custom code remains and why it is necessary.

3. Quality report.
- Accessibility checks, performance considerations, and test coverage.

4. Release notes.
- Commands run, results, known limitations, and next actions.
