---
name: ux-design
description: >-
  Produce production-grade UX mocks in Figma using the Figma MCP server.
  Creates all screens, states, and interactions defined in a PRD using the
  Speqq design system tokens. Outputs a validated Figma file that serves as
  the exact spec for implementation. Use when a PRD is approved and the
  feature needs visual design before architecture or implementation.
---

# UX Design Skill

## Purpose
The agent produces Figma mocks that serve as the exact visual specification for implementation. Every screen, state, and interaction from the PRD must be represented in the Figma file. When the user approves the Figma mock, the next engineer builds exactly what the mock shows — no interpretation, no guessing.

## Figma MCP Tools

The agent uses the Figma MCP server to create and read designs. The available tools are:

### Write tools:
| Tool | Purpose |
|---|---|
| `use_figma` | Primary write tool. Creates and edits pages, frames, components, variants, variables, styles, text, images, and auto layout. |
| `create_new_file` | Creates a blank Figma Design file in drafts. |

### Read tools:
| Tool | Purpose |
|---|---|
| `get_design_context` | Returns structured code representation (React + Tailwind) of a Figma selection. |
| `get_variable_defs` | Extracts variables and styles (color, spacing, typography tokens) from a selection. |
| `search_design_system` | Queries connected design libraries for components, variables, and styles by search text. |
| `get_metadata` | Returns layer IDs, names, types, positions, and sizes for a selection. |
| `get_screenshot` | Captures a screenshot of a selection. |
| `get_code_connect_map` | Retrieves Figma node-to-code component mappings. |

### Design system tools:
| Tool | Purpose |
|---|---|
| `create_design_system_rules` | Generates a rules file guiding agents to produce consistent code from designs. |
| `add_code_connect_map` | Maps a Figma node ID to a codebase component path. |

## Input
The agent must read the following before creating any Figma designs:
- The PRD — `.docs/{project-name}-prd.md` — every requirement, user flow, edge case, and error scenario.
- The Speqq design tokens — `styles/globals.css` lines 12–1202 (the approved token set). Lines after 1202 are deprecated and must not be used.
- The existing Figma design system — use `search_design_system` to find existing Speqq components, variables, and styles before creating new ones.
- The existing UI — read the current components and pages in the codebase that the feature will modify.

## Speqq Design Token Reference

The agent must use the Speqq design tokens defined in `styles/globals.css`. The agent must not use hex colors directly — the agent must reference the token names so the implementation uses the same CSS variables.

### Surface colors:
| Token | Light | Dark | Usage |
|---|---|---|---|
| `--color-surface` | `#fff` | `#212121` | Primary background |
| `--color-surface-elevated` | `#fff` | `#303030` | Cards, modals, elevated surfaces |
| `--color-surface-elevated-secondary` | `#f9f9f9` | `#414141` | Secondary elevated surfaces |
| `--color-surface-secondary` | `#f9f9f9` | `#181818` | Secondary background |
| `--color-surface-tertiary` | `#f3f3f3` | `#131313` | Tertiary background |

### Text colors:
| Token | Light | Dark | Usage |
|---|---|---|---|
| `--color-text` | `#282828` | `#dcdcdc` | Primary text |
| `--color-text-emphasis` | `#0d0d0d` | `#dcdcdc` | Headings, emphasized text |
| `--color-text-secondary` | `#282828` | `#b9b9b9` | Secondary text |
| `--color-text-tertiary` | `#8f8f8f` | `#8f8f8f` | Placeholder, hint text |
| `--color-text-disabled` | `#8f8f8f` | `#5d5d5d` | Disabled text |
| `--color-text-inverse` | `#fff` | `#0d0d0d` | Text on solid backgrounds |

### Intent colors (solid backgrounds):
| Token | Light | Dark | Usage |
|---|---|---|---|
| `--color-background-primary-solid` | `#181818` | `#f3f3f3` | Primary buttons, CTAs |
| `--color-background-info-solid` | `#0285ff` | `#339cff` | Info states |
| `--color-background-success-solid` | `#00a240` | `#008635` | Success states |
| `--color-background-warning-solid` | `#e25507` | `#b9480d` | Warning states |
| `--color-background-danger-solid` | `#e02e2a` | `#ba2623` | Error/destructive states |
| `--color-background-discovery-solid` | `#924ff7` | `#6b3ab4` | Discovery, new feature highlights |

### Border colors:
| Token | Light | Dark | Usage |
|---|---|---|---|
| `--color-border` | `black 10%` | `white 12%` | Default borders |
| `--color-border-strong` | `black 15%` | `white 20%` | Emphasized borders |
| `--color-border-subtle` | `black 5%` | `white 8%` | Subtle borders |

### Focus ring:
| Token | Value | Usage |
|---|---|---|
| `--color-ring` | `#0169cc` | Default focus ring |
| `--color-ring-danger` | `#ff8583` | Error state focus ring |

## Work

### Step 1: Read the existing design system
Before creating any frames, the agent must:
- Use `search_design_system` to find existing Speqq components in the connected Figma library.
- Use `get_variable_defs` on existing Speqq design files to extract the current variable set.
- Use `get_design_context` on existing screens that the feature will modify to understand current layout patterns.
- Record which existing components can be reused and which new components are needed.

### Step 2: Create the Figma file structure
The agent must use `create_new_file` to create a new Figma file named `{project-name} — UX Design`.

The agent must use `use_figma` to create the following page structure:
- **Page 1: Screens** — every unique screen the user sees, organized by user flow.
- **Page 2: States** — every data state (loading, error, empty, populated) for every data-driven surface.
- **Page 3: Interactions** — every interaction state (hover, focus, active, disabled) for every interactive element.
- **Page 4: Responsive** — every screen at mobile (375px), tablet (768px), and desktop (1280px) widths.

### Step 3: Create screens for every PRD user flow
For each user flow in the PRD, the agent must use `use_figma` to create:
- A frame for each step in the user flow.
- The frame must use the Speqq design tokens for all colors, spacing, and typography.
- The frame must use existing Speqq components from the design system wherever possible.
- The frame must include all UI elements described in the PRD — buttons, inputs, labels, navigation, content areas.
- Each frame must be named with the flow name and step number: `{Flow Name} / Step {N} — {Description}`.

### Step 4: Create all required states
For every data-driven surface in the design, the agent must use `use_figma` to create:

**Loading state:**
- Skeleton placeholders that match the layout shape of the loaded content.
- The skeleton must use `--color-surface-tertiary` for the placeholder background.

**Error state:**
- Destructive-styled error message using `--color-background-danger-solid` for the error indicator.
- Error text using `--color-text` with a clear description of what failed.
- A retry action if the error is retryable.

**Empty state:**
- Centered empty state with an icon, a heading, a description, and a primary action button.
- The heading must tell the user what the surface will contain.
- The action button must tell the user how to add the first item.

**Populated state:**
- The normal state with real-looking data — not "Lorem ipsum" or "Test data."
- Use realistic Speqq domain data: workspace names, document titles, requirement descriptions.

### Step 5: Create interaction states
For every interactive element, the agent must use `use_figma` to create variants showing:
- **Default** — the resting state.
- **Hover** — using the `-hover` token variant (e.g., `--color-background-primary-solid-hover`).
- **Active/Pressed** — using the `-active` token variant.
- **Focused** — showing the focus ring using `--color-ring`.
- **Disabled** — using `--color-text-disabled` and reduced opacity.

### Step 6: Create responsive variants
For every screen, the agent must use `use_figma` to create three viewport variants:
- **Mobile** — 375px width. Stack layout, full-width elements, collapsed navigation.
- **Tablet** — 768px width. Adjusted layout, sidebar may collapse.
- **Desktop** — 1280px width. Full layout with all panels visible.

### Step 7: Validate against the PRD
The agent must verify:
- Every PRD requirement has a corresponding screen or state in the Figma file.
- Every PRD user flow has a complete set of step-by-step frames.
- Every PRD edge case has a corresponding state.
- Every PRD error scenario has a corresponding error state.
- No screen or element uses colors outside the Speqq design token set.
- No screen introduces a custom interactive element when the design system has an existing component.

### Step 8: Generate code connect mappings
After the design is complete, the agent must:
- Use `add_code_connect_map` to map every Figma component to its corresponding codebase component path.
- Use `create_design_system_rules` to generate a rules file that ensures the implementation matches the design.

## Output
- A Figma file named `{project-name} — UX Design` with all pages, screens, states, interactions, and responsive variants.
- `.docs/{project-name}-ux-design.md` — a summary document listing every screen, every state, and the Figma file URL.
- Code Connect mappings linking Figma components to codebase components.

## Gate
Every PRD requirement has a corresponding Figma screen or state. Every data-driven surface has loading, error, empty, and populated states. Every interactive element has default, hover, active, focused, and disabled variants. Every screen has mobile, tablet, and desktop responsive variants. All colors use Speqq design tokens. The user approves the Figma mocks before the design is considered complete.

## Hard rules
- The agent must not use hex colors directly in Figma — the agent must use the Speqq design token names so the implementation maps exactly.
- The agent must not create custom interactive components when `search_design_system` returns an existing component.
- The agent must not use deprecated tokens from `styles/globals.css` lines after 1202.
- The agent must not use placeholder content ("Lorem ipsum", "Test", "Sample") — all content must be realistic Speqq domain data.
- The agent must not skip any data state (loading, error, empty, populated) for any data-driven surface.
- The agent must not skip responsive variants for any screen.
- The agent must check the existing design system with `search_design_system` before creating any new component.
