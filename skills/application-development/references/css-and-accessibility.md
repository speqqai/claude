# CSS and Accessibility Standards

Use this checklist for every UI change.

## CSS architecture

1. Centralize tokens.
- Keep color, spacing, typography, radius, and motion tokens in a single source.
- Prefer CSS custom properties for theme-level consistency.

2. Layer styles predictably.
- Global foundation: reset/base/tokens.
- Shared utilities: layout/spacing/text helpers.
- Component styles: local CSS Modules or equivalent scoped styles.

3. Avoid style drift.
- Avoid inline one-off values when a token exists.
- Avoid duplicate component styles across feature folders.
- Avoid deep selector chains that are hard to maintain.

## Accessibility defaults

1. Semantics first.
- Use native elements (`button`, `nav`, `main`, `label`, `table`) before ARIA fallbacks.

2. Keyboard support.
- Ensure all interactive controls are reachable and usable via keyboard.
- Ensure focus order is logical and visible.

3. Forms and feedback.
- Provide explicit labels and programmatic error associations.
- Announce loading/success/error states where needed.

4. Contrast and motion.
- Meet WCAG contrast expectations.
- Respect reduced-motion preferences for non-essential animation.

## Definition of done for UI tasks

- [ ] Uses shared design tokens.
- [ ] No unreachable controls via keyboard.
- [ ] Focus indicator is visible on interactive elements.
- [ ] Labels, helper text, and errors are screen-reader friendly.
- [ ] Contrast and motion preferences are respected.
