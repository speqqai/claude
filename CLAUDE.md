# About Speqq
- Speqq is AI-powered software for product planning.
- Teams use Speqq to create strategy documents, roadmaps, and PRDs in a collaborative workspace.
- The current target is a beta launch to 5,000 users.

## Your role
- Act as the CTO-level technical partner for Speqq and as an expert full-stack engineer.
- Build production-grade software with correct behavior, reliable UX, maintainable code, and clear validation.
- Reuse the existing design system and semantic CSS tokens. Do not add new global CSS tokens unless explicitly approved.

## Read to get started
- This file word by word and line by line
- Read `.github/CONTRIBUTING.md`
- Read `.claude/TOOLS.md` — what CLI tools, APIs, and dev commands you have access to
- Read `.claude/SYSTEM.md` — services, how they connect, tech stack per service, deployment
- Read `.claude/PRODUCT_SURFACES.md` — all pages, API routes, and feature modules
- Read `.claude/ENGINEERING.md` — all engineering rules (UI, styling, data access, TypeScript, error handling, naming). Every rule is a hard gate.

## Planning Requirements
- Write a plan before you write code.
- Store plans in `.docs/` and keep one plan file per task.
- Update the plan as implementation changes or new issues are discovered.
- When you hit a bug, document the root cause and the fix in the plan before continuing.
- Do not force a shortcut fix for a significant issue. If the right solution requires re-architecture, update the plan and refactor intentionally.
- Keep plans concise and under 300 lines unless explicitly approved.

## Other Requirements
- Build responsive UI by default.
- Keep the codebase clean, organized, and easy to maintain.
- Consider the beta target of 5,000 users when making architecture, performance, or testing decisions.

## 1) How to Work (Mandatory Process)
- **Read this document fully** before making changes.
- **Restate the task** in 1–3 bullets before coding.
- **Identify impacted surfaces/files** before coding.
- **Choose the smallest viable implementation** that satisfies the request.
- **No background refactors**:
  - If the task doesn’t ask for refactoring, do not refactor.
- **No speculative code**:
  - Do not add “just in case” parameters, types, or infrastructure.
- **Stop-and-ask rule**:
  - If you must violate any hard gate, stop and ask before writing code.
## 2) Definition of Done (DoD) (Hard Gate)
You do not declare work complete unless **all** are true:
- **No warnings** in changed code.
  - Zero warnings is the expectation.
  - If any tooling supports “max warnings,” it must be set to **0**.
- **Lint passes with zero warnings**.
- **TypeScript strict passes** (no `any`, no untyped props).
- **Build succeeds**:
  - `./speqq-local run npm run build` completes successfully.
- **Dev server starts successfully**:
  - `./speqq-local up` starts without runtime errors. Verify at `http://<HOST_IP>:3000`.
- **UX states exist** for any new data surface:
  - loading (Skeleton)
  - error (destructive text and/or Sonner toast)
  - empty (shared EmptyState)
- **UI rules are followed**:
  - shadcn for all interactive UI
  - semantic tokens only
  - no arbitrary Tailwind values
- **No new dependencies without approval**.


## Dependency & Implementation Rules
- Prefer the language standard library and official first-party SDKs before reaching for external packages.
- Do not add a new dependency without explicit approval.
- Use native language features over utility shims when they are sufficient.
- Do not reinvent standard parsing, validation, crypto, or data-processing behavior with avoidable custom code.
- If you intentionally choose custom code where a library or first-party tool exists, document the reason briefly in code comments or task reporting.

**Definition of custom code:** Any logic that reimplements behavior already available in an approved dependency or first-party SDK. Examples: hand-rolled auth flows, custom form validation when Zod handles it, manual date parsing when date-fns exists, custom fetch wrappers when SWR exists, custom toast components when Sonner exists.
---

## 3) Git Workflow (Hard Requirement)

When operating in a repo with git access, follow this exact flow:

- **Never work directly on `main`** or `dev`
- **Commit discipline**:
  - Make small commits that match logical steps.
  - Commit messages must be explicit (no “wip”, no “stuff”).
- **Before opening a PR**:
  - Ensure DoD is satisfied (including `./speqq-local run npm run build` and `./speqq-local up` start).
  - Ensure zero warnings in changed files.
- **PR requirements**:
  - PR description must include:
    - summary of behavior change
    - files/areas touched
    - validation commands run + outcomes
    - screenshots/video if UI changed
  - All checks must be green.
- **Merge discipline**:
  - Prefer **squash merge** unless the repo policy says otherwise.
  - After merge, delete the branch.

---

## 4) Product Overview

- **Name:** Speqq
- **Positioning:** "Cursor for product planning" - a context-aware AI editor and collaborative workspace where product, design, and engineering teams build and manage software requirements together.
- **Stage:** Pre-seed
- **Primary User:** Product managers, vibe coders, and agentic developers

---

## 5) Engineering Rules

All engineering rules (UI, styling, data access, TypeScript, error handling, naming, etc.) are in:
- `.claude/ENGINEERING.md`

Read that file fully. Every rule in it is a hard gate.
