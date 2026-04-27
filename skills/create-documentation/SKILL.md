---
name: documentation-writer
description: >-
  Write and maintain internal technical documentation in the Nextra-powered
  content/ directory. Covers product architecture, feature specs, onboarding,
  and operational guides. Uses a write-validate-fix loop to ensure docs stay
  coherent, non-redundant, and readable as a whole. Use when a feature is
  complete and needs documentation, or when asked to "document", "write docs",
  or "update docs".
---

# Documentation Writer

Write and maintain documentation in the **`content/`** directory (Nextra-powered, served at `/docs`).

Documentation is not append-only. Every write must consider the existing docs as a whole and leave the site in a better state than it was found.

**Style guide:** Follow `.docs/technical-writing/technical-writing-style-guide.md` for voice, tone, and formatting. When in doubt, follow [Google developer documentation style](https://developers.google.com/style).

---

## Where docs live

```
content/
  index.mdx              ← docs home / overview
  internal/              ← technical docs (architecture, features, ops)
    architecture.mdx
    feature-specs/
      <feature-name>.mdx
    onboarding.mdx
  guides/                ← user-facing docs (for beta launch)
    getting-started.mdx
    ...
```

- **`content/internal/`** — technical docs for the team: architecture, feature specs, onboarding, operational guides
- **`content/guides/`** — user-facing docs for when the product goes public
- All files are `.mdx` with Nextra frontmatter (`title`, `description`)
- Nextra handles routing, sidebar, and navigation automatically from the file structure

---

## Process: Write-Validate-Fix Loop

This is not a one-shot process. Documentation goes through a loop until it passes validation.

### Step 1: Survey

Before writing anything:

1. **Read every file in `content/`** — understand what exists, how it's organized, what topics are covered
2. **Read the feature's source code** — every component, route, API endpoint, server action relevant to what you're documenting
3. **Read the feature's PRD/plan** if one exists in `.docs/`
4. **Decide placement** — where does this doc belong? Does it fit an existing page or need a new one? Would it make more sense as a section in an existing doc?

### Step 2: Write

1. Write the content following the structure and writing rules below
2. If creating a new page, add proper Nextra frontmatter:
   ```yaml
   ---
   title: Page title
   description: One-line description for SEO and sidebar
   ---
   ```
3. If updating an existing page, integrate the new content — don't just append to the bottom

### Step 3: Self-validate

After writing, run through these checks yourself. Every check must pass before moving to Step 4.

#### 3a. Self-coherence
- Re-read the page you just wrote/updated **in full**
- Does it read as a cohesive document from top to bottom?
- Would a new team member understand it without context?
- Are there sections that feel bolted-on or out of place?

#### 3b. Cross-doc coherence
- Read every other page in `content/` that touches related topics
- Does anything you wrote **conflict** with existing docs?
- Does anything you wrote **duplicate** content that already exists elsewhere?
- Are cross-references correct and pointing to the right pages?

#### 3c. Structural coherence
- Does the `content/` directory still make sense as a whole?
- Is the new page in the right location in the hierarchy?
- Would someone browsing the sidebar find this where they'd expect it?
- Does the `content/index.mdx` need updating to reflect the new content?

#### 3d. Information quality
- No placeholder text (TBD, TODO, coming soon, placeholder)
- No orphaned references (links to pages that don't exist)
- No stale information that contradicts the current codebase
- No redundant content — if something is said in two places, consolidate to one

### Step 4: Subagent review (mandatory)

After self-validation passes, **launch a subagent** to independently review the documentation. This is a separate reviewer — not the writer checking its own work.

**Spawn the subagent with this prompt:**

> You are a technical writing reviewer following the Google developer documentation style guide. You are reviewing documentation for Speqq, an AI-powered product planning tool.
>
> Your job is to review the documentation files I've written/updated and produce a pass/fail verdict with specific issues to fix.
>
> **Read these files:**
> 1. The style guide: `.docs/technical-writing/technical-writing-style-guide.md`
> 2. Every file in `content/` — understand the full docs site
> 3. Focus especially on: [list the files you wrote/updated]
>
> **Evaluate against two dimensions:**
>
> **Dimension 1: Google developer documentation style compliance**
> - Voice: second person, active voice, present tense
> - Sentence case headings
> - Code in backticks, UI elements in bold
> - Concise — no filler, no marketing language, no hedge words
> - Numbered steps for procedures, bulleted lists for options
> - Tables for structured data
> - Terms defined on first use
> - Consistent terminology across all pages
>
> **Dimension 2: Product communication quality**
> - Does it accurately describe what Speqq does?
> - Would someone unfamiliar with the product understand the feature after reading?
> - Is the level of detail appropriate for the audience (internal team vs. end user)?
> - Are capabilities described completely — not just the happy path?
> - Do the docs tell a coherent story when read across multiple pages?
> - Is anything confusing, contradictory, or missing?
> - Does the information architecture make sense — would you find things where you expect them?
>
> **Output format:**
> ```
> VERDICT: PASS or FAIL
>
> Style issues (if any):
> - [file:line] issue description → suggested fix
>
> Product communication issues (if any):
> - [file] issue description → suggested fix
>
> Cross-doc issues (if any):
> - [file1 vs file2] conflict/redundancy description → suggested resolution
> ```
>
> Be strict. If anything is unclear, inconsistent, or doesn't meet the standard, it's a FAIL.

**How to act on the review:**
- If **PASS** → proceed to Step 6
- If **FAIL** → fix every issue the reviewer flagged, then go back to Step 3 (self-validate) and run through the full loop again including a fresh subagent review
- Do NOT argue with the reviewer's findings — fix them

### Step 5: Fix and re-loop

If the subagent review returns FAIL:
1. Fix every flagged issue
2. Go back to **Step 3** — full self-validation
3. Launch a **new subagent review** (Step 4) — do not reuse the previous one
4. Repeat until the subagent returns PASS

Do NOT skip re-validation after a fix. A fix can introduce new coherence issues.

### Step 6: Present for user approval (mandatory gate)

After the subagent review passes, present the work to the user for final sign-off. **Documentation is NOT done until the user approves.**

**Present to the user:**
1. A summary of what was added/changed (files + one-line each)
2. Any existing docs that were updated for coherence
3. Any structural changes to `content/`
4. The subagent review result (PASS + any notes)
5. A direct ask: "Do you want to review any of these pages before I finalize?"

**User response handling:**
- If the user **approves** → documentation is complete. Proceed to Step 7.
- If the user **requests changes** → make the changes, then go back to **Step 3** and run the full loop again (self-validate → subagent review → present to user). Do not skip any steps.
- If the user **wants to review specific pages** → show them the content, wait for feedback, then act on it.

**Rules:**
- Do NOT mark documentation as done without explicit user approval
- Do NOT assume silence is approval — ask directly
- Do NOT push back on user feedback — incorporate it
- The user's judgment overrides both self-validation and the subagent review

### Step 7: Report

After user approval, provide a final report:
- What was added/changed (files + summary)
- Any existing docs that were updated for coherence
- Any structural changes to `content/`
- Subagent review result
- User approval confirmation

---

## What to document

### For internal/technical docs (`content/internal/`)

**Architecture docs:**
- System-level architecture and how components connect
- Data flow through the system
- Key technical decisions and why they were made
- Infrastructure and deployment topology

**Feature specs:**
- What the feature does (complete list of capabilities)
- How it works technically (data model, API surface, state management)
- How to access it in the UI
- Edge cases and limitations
- Error states and what they mean
- Integration points with other features

**Operational docs:**
- How to set up the dev environment
- Release process
- Debugging guides
- Monitoring and alerting

### For user-facing docs (`content/guides/`)

- What the feature is and does (from the user's perspective)
- Step-by-step instructions
- Inputs, outputs, and limits
- Troubleshooting
- No architecture, no database schemas, no implementation details

---

## Writing rules

**Voice & tone:**
- Second person — "you", not "the user" or "one"
- Active voice — "Click **Save**" not "The Save button should be clicked"
- Present tense — "The file saves" not "The file will save"
- Direct and concise — no filler, no marketing speak
- Friendly but professional

**Content:**
- Every capability documented — not just the happy path
- Steps are numbered and specific — "Click **+ New** in the top right" not "add an item"
- UI elements referenced by exact label
- Error messages documented with what they mean and how to fix them
- Limitations stated clearly
- No jargon without definition on first use

**Formatting:**
- Headings use sentence case — "Add a context file" not "Add A Context File"
- Code in backticks — `command`, `fileName`, `parameter`
- UI elements in bold — **Save**, **Settings**, **Add File**
- File paths in code — `content/internal/`
- Key combinations with plus — Ctrl+S, Cmd+Enter
- Tables for structured data (options, parameters, limits)
- Ordered lists for steps, unordered lists for options

---

## Anti-patterns (NEVER do these)

1. **NEVER append without reading** — always read existing docs first
2. **NEVER leave the docs in a worse state** — if you touch a page, it must be better when you're done
3. **NEVER duplicate content across pages** — if the same info exists in two places, consolidate
4. **NEVER create orphan pages** — every page must be reachable from navigation
5. **NEVER skip the validation loop** — write → validate → fix → re-validate is mandatory
6. **NEVER leave placeholder text** — no TBD, TODO, coming soon, or empty sections
7. **NEVER add a page without checking if the content belongs in an existing page** — prefer updating over creating
8. **NEVER ignore cross-doc conflicts** — if your new content contradicts another page, resolve it
9. **NEVER publish without re-reading the full page** — partial reads miss coherence issues
