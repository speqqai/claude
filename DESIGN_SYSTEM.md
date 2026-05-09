# Speqq Design System

Reference spec: **Fluent 2** (https://fluent2.microsoft.design)
Components: **shadcn/ui** (Radix + Tailwind) — we use Fluent 2 values, not Fluent 2 components.

This document is the single source of truth for auditing visual consistency.
Every token in `styles/globals.css` must trace back to a rule in this file.

---

## Font

Platform-native system fonts. No custom web font loaded.

```css
--font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
```

| Platform | Font | Fluent 2 Ramp |
|----------|------|---------------|
| macOS / iOS | San Francisco | macOS / iOS ramp |
| Windows | Segoe UI | Windows ramp |
| Android | Roboto | Android ramp |

Each platform gets its native font, which the Fluent 2 type ramps are designed for. No font files to download. Faster load times. Feels native on every device.

---

## Typography

Fluent 2 Web type ramp. We map their roles to our token names.

### Adopted roles (used in Speqq)

| Fluent 2 Role | Size | Line Height | Weight | Our Token | Use in Speqq |
|---------------|------|-------------|--------|-----------|--------------|
| Caption 1 | 12px | 16px | 400 | `--font-text-xs` | Timestamps, captions, badges, tertiary info |
| Body 1 | 14px | 20px | 400 | `--font-text-sm` | Primary UI text: menus, sidebar, tables, chat, buttons |
| Body 1 Strong | 14px | 20px | 600 | `--font-text-sm` + `font-semibold` | Emphasized labels, active nav items |
| Subtitle 2 | 16px | 22px | 600 | `--font-text-md` | Panel titles, section headings, editor prose |
| Subtitle 1 | 20px | 26px | 600 | `--font-heading-sm` | Page headings, dialog titles |
| Title 3 | 24px | 32px | 600 | `--font-heading-lg` | Large page titles |
| Title 2 | 28px | 36px | 600 | `--font-heading-xl` | Hero headings (if needed) |

### NOT adopted (too large for IDE, reserved for marketing)

Caption 2 (10px), Title 1 (32px), Large Title (40px), Display (68px).

### Rules

- **Primary UI text is 14px (`--font-text-sm`), not 12px.** This is the single biggest change.
- 12px (`--font-text-xs`) is only for captions, timestamps, and tertiary info.
- Same type scale on all screen sizes. No font inflation on mobile.
- Weight variants use Tailwind utilities (`font-medium`, `font-semibold`), not separate tokens.

### Token definitions

```css
--font-text-xs-size: 0.75rem;        /* 12px — Fluent Caption 1 */
--font-text-xs-line-height: 1rem;     /* 16px — Fluent Caption 1 */
--font-text-sm-size: 0.875rem;        /* 14px — Fluent Body 1 */
--font-text-sm-line-height: 1.25rem;  /* 20px — Fluent Body 1 */
--font-text-md-size: 1rem;            /* 16px — Fluent Subtitle 2 */
--font-text-md-line-height: 1.375rem; /* 22px — Fluent Subtitle 2 */
--font-heading-sm-size: 1.25rem;      /* 20px — Fluent Subtitle 1 */
--font-heading-sm-line-height: 1.625rem; /* 26px — Fluent Subtitle 1 */
--font-heading-lg-size: 1.5rem;       /* 24px — Fluent Title 3 */
--font-heading-lg-line-height: 2rem;  /* 32px — Fluent Title 3 */
--font-heading-xl-size: 1.75rem;      /* 28px — Fluent Title 2 */
--font-heading-xl-line-height: 2.25rem; /* 36px — Fluent Title 2 */
```

### Tokens to delete

`--font-text-3xs`, `--font-text-2xs`, `--font-text-lg`,
`--font-heading-xs`, `--font-heading-md` (duplicate of sm),
`--font-heading-2xl`, `--font-heading-3xl`, `--font-heading-4xl`, `--font-heading-5xl`.

---

## Spacing

Fluent 2 global spacing ramp. Base unit: **4px**.

### Adopted scale

| Fluent 2 Token | Value | Our Token | Use |
|----------------|-------|-----------|-----|
| size20 | 2px | `--space-0-5` | Hairline gaps, border offsets |
| size40 | 4px | `--space-1` | Tight gaps (icon-to-label) |
| size60 | 6px | `--space-1-5` | Compact padding |
| size80 | 8px | `--space-2` | Standard gap |
| size100 | 10px | `--space-2-5` | Medium padding |
| size120 | 12px | `--space-3` | Section padding |
| size160 | 16px | `--space-4` | Card/panel padding |
| size200 | 20px | `--space-5` | Large section gap |
| size240 | 24px | `--space-6` | Between major sections |
| size320 | 32px | `--space-8` | Page-level margins |

### Token definitions

```css
--space-0-5: 0.125rem;  /* 2px  — Fluent size20 */
--space-1: 0.25rem;      /* 4px  — Fluent size40 */
--space-1-5: 0.375rem;   /* 6px  — Fluent size60 */
--space-2: 0.5rem;       /* 8px  — Fluent size80 */
--space-2-5: 0.625rem;   /* 10px — Fluent size100 */
--space-3: 0.75rem;      /* 12px — Fluent size120 */
--space-3-5: 0.875rem;   /* 14px — (off-grid, IDE density) */
--space-4: 1rem;         /* 16px — Fluent size160 */
--space-5: 1.25rem;      /* 20px — Fluent size200 */
--space-6: 1.5rem;       /* 24px — Fluent size240 */
--space-8: 2rem;         /* 32px — Fluent size320 */
```

### Notes

- `--space-3-5` (14px) is off the Fluent grid but useful for IDE panel headers. Keep for now.
- Values beyond 32px (size360–size560) are not needed for our UI density.

---

## Control Sizes

Interactive element heights (buttons, inputs, rows).

### Adopted scale

| Our Token | Value | Fluent 2 Mapping | Use |
|-----------|-------|------------------|-----|
| `--control-size-2xs` | 24px | — | Badges, tiny indicators |
| `--control-size-xs` | 28px | Small | Sidebar rows, menu items, compact buttons |
| `--control-size-sm` | 32px | Medium | Standard buttons, inputs |
| `--control-size-md` | 36px | — | Prominent controls |
| `--control-size-lg` | 40px | Large | Panel headers, large actions |

### Mobile overrides (Fluent 2: 44px minimum touch target for web/iOS)

```css
@media (max-width: 479px) {
  :root {
    --control-size-xs: 2.75rem;  /* 44px — meets Fluent web touch target */
    --control-size-sm: 2.75rem;  /* 44px */
  }
}
```

### Tokens to delete

`--control-size-3xs` (22px), `--control-size-xl` (40px — merged into lg),
`--control-size-2xl` (44px), `--control-size-3xl` (48px).

Also delete the duplicate set: `--size-control-sm`, `--size-control-md`, `--size-control-lg` (lines 1279-1281 in globals.css).

---

## Icon Sizes

### Adopted scale

| Our Token | Value | Fluent 2 Ref | Use |
|-----------|-------|--------------|-----|
| `--control-icon-size-xs` | 14px | — | Icons in compact controls |
| `--control-icon-size-sm` | 16px | System icon small | Icons in standard controls |
| `--control-icon-size-md` | 20px | System icon default | Icons in prominent controls, standalone |

### Mobile overrides

```css
@media (max-width: 479px) {
  :root {
    --control-icon-size-xs: 1.125rem;  /* 18px */
    --control-icon-size-sm: 1.25rem;   /* 20px */
  }
}
```

### Tokens to delete

`--control-icon-size-lg` (20px — merged into md),
`--control-icon-size-xl` (22px), `--control-icon-size-2xl` (24px).

---

## Border Radius (Shape)

Fluent 2 shape scale mapped to our tokens.

### Adopted scale

| Fluent 2 Shape | Value | Our Token | Use |
|----------------|-------|-----------|-----|
| Extra Small | 4px | `--radius-xs` | Badges, checkboxes |
| Small | 6px | `--radius-sm` | Small controls |
| Small | 8px | `--radius-md` | Buttons, inputs, menu items |
| Medium | 12px | `--radius-xl` | Cards, dialogs, panels |
| Large | 16px | `--radius-2xl` | Large surfaces, sheets |
| Full | 9999px | `--radius-full` | Pills, avatars, round buttons |

### Tokens to delete

`--radius-2xs` (2px), `--radius-lg` (10px — too close to xl),
`--radius-3xl` (20px), `--radius-4xl` (24px).

Also delete the duplicate set: `--radius-control`, `--radius-surface`, `--radius-panel`, `--radius-pill` (lines 1282-1285).

---

## Responsive Breakpoints

Fluent 2 breakpoints. Replaces our single 768px cut.

| Fluent 2 Class | Breakpoint | Our Token | Use |
|----------------|------------|-----------|-----|
| small | < 480px | `--breakpoint-sm` | Phone portrait |
| medium | < 640px | `--breakpoint-md` | Phone landscape / small tablet |
| large | < 1024px | `--breakpoint-lg` | Tablet |
| x-large | >= 1024px | `--breakpoint-xl` | Laptop (default desktop) |
| xx-large | >= 1366px | `--breakpoint-2xl` | Desktop |
| xxx-large | >= 1920px | `--breakpoint-3xl` | Ultra-wide |

### CSS media queries

```css
/* Phone */
@media (max-width: 479px) { }
/* Tablet */
@media (max-width: 1023px) { }
/* Desktop (default — no query needed) */
/* Wide */
@media (min-width: 1366px) { }
/* Ultra-wide */
@media (min-width: 1920px) { }
```

### Mobile override strategy

- Typography: **no change** (same scale on all sizes per Fluent 2)
- Control sizes: bump to 44px touch targets below 480px
- Icon sizes: bump proportionally below 480px
- Layout: reflow panels (sidebar → sheet, editor/chat toggle)

---

## Color Palette

### Semantic roles (keep)

These are the colors components actually reference. They stay.

| Category | Tokens | Purpose |
|----------|--------|---------|
| Text | `--color-text`, `--color-text-secondary`, `--color-text-tertiary`, `--color-text-emphasis`, `--color-text-inverse`, `--color-text-disabled` | Text hierarchy |
| Surface | `--color-surface`, `--color-surface-secondary`, `--color-surface-tertiary`, `--color-surface-elevated`, `--color-surface-elevated-secondary` | Background layers |
| Border | `--color-border`, `--color-border-subtle`, `--color-border-strong`, `--color-border-disabled` | Edge treatment |
| Alpha | `--alpha-02` through `--alpha-70` | Transparency overlays |
| Status | `--color-text-danger`, `--color-text-success`, `--color-text-caution`, `--color-text-warning`, `--color-text-info`, `--color-text-discovery` + matching `--color-background-*` and `--color-border-*` | Semantic feedback |
| Brand | `--color-brand`, `--color-brand-light`, `--color-brand-dark`, `--color-brand-subtle`, `--color-brand-muted` | Brand identity |

### Primitive palettes (keep as foundation)

Gray (0–1000), Blue, Red, Green, Yellow, Orange, Purple, Pink scales.
These feed into semantic tokens. Do not reference primitives directly in components.

### Tokens to audit and consolidate

| Issue | Tokens | Action |
|-------|--------|--------|
| Duplicate surface tokens | `--color-bg`, `--color-bg-subtle`, `--color-bg-muted`, `--color-bg-inset` vs `--color-surface-*` | Remove `--color-bg-*`, use `--color-surface-*` |
| Duplicate text tokens | `--color-text-muted` vs `--color-text-tertiary` | Remove `--color-text-muted`, use `--color-text-tertiary` |
| shadcn built-ins | `--background`, `--foreground`, `--card`, `--popover`, `--muted`, `--accent`, `--destructive`, `--border`, `--input`, `--ring`, `--primary`, `--secondary` | Keep — shadcn components reference these internally |
| OpenAI duplicates (lines 1323-1336) | `--chat-background-color`, `--menu-item-background-color`, etc. with hardcoded values | Remove — these duplicate semantic tokens with different values |
| Unused component aliases | `--button-background-color`, `--button-text-color`, `--radio-group-indicator-*`, `--segmented-control-*`, `--slider-*`, `--switch-*`, etc. | Audit usage. Keep if referenced, delete if dead. |
| Unused status combos | Many `--color-ring-*-ghost`, `--color-ring-*-outline`, `--color-ring-*-soft`, `--color-ring-*-solid` variants | Most are identical to parent. Flatten. |
| Hardcoded hex in dark mode | `--pill-warning-bg: #945e0c` | Replace with semantic token reference |

---

## Token Summary

### What we keep (31 core tokens)

| Category | Count | Tokens |
|----------|-------|--------|
| Text size | 3 | `text-xs` (12), `text-sm` (14), `text-md` (16) |
| Heading size | 3 | `heading-sm` (20), `heading-lg` (24), `heading-xl` (28) |
| Control size | 5 | `2xs` (24), `xs` (28), `sm` (32), `md` (36), `lg` (40) |
| Icon size | 3 | `xs` (14), `sm` (16), `md` (20) |
| Spacing | 11 | `0.5` (2) through `8` (32) |
| Radius | 6 | `xs` (4) through `full` (9999) |

### What we delete (17 tokens)

| Category | Tokens to remove |
|----------|-----------------|
| Text size | `3xs`, `2xs`, `lg` |
| Heading size | `xs`, `md`, `2xl`, `3xl`, `4xl`, `5xl` |
| Control size | `3xs`, `xl`, `2xl`, `3xl` |
| Icon size | `lg`, `xl`, `2xl` |
| Radius | `2xs`, `lg`, `3xl`, `4xl` |
| Duplicate sets | `--size-control-*`, `--radius-control/surface/panel/pill` |

---

## Audit Checklist

When reviewing any component, verify:

1. **Primary text uses `--font-text-sm` (14px)**, not `--font-text-xs` (12px)
2. **12px (`--font-text-xs`) is only used for** timestamps, captions, badges, tertiary labels
3. **No hardcoded sizes** — no `size-3`, `h-4 w-4`, `text-sm` (Tailwind). Use token vars.
4. **Spacing values are from the scale** — no arbitrary values like `p-[13px]`
5. **Touch targets are >= 44px on mobile** — buttons, links, interactive rows
6. **Radius uses the adopted scale** — no `rounded-lg` (Tailwind). Use token vars.
7. **Colors reference semantic tokens** — no hex, no primitive palette tokens in components
