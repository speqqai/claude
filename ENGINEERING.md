# Speqq Engineering Guide

## Purpose
Software-development rules for AI Agents

## Code Style

- All data flows through API routes. Component → Hook → API → Database.
- Client components never query databases, construct URLs, or validate business rules.
- Validation lives on the server only. Do not duplicate it client-side.
- Use React 19 built-ins (`useActionState`, `useFormStatus`, `useOptimistic`) and SWR before creating custom hooks.
- Custom hooks only when native hooks cannot do it AND the logic appears in 3+ components.
- Use Supabase SDK methods. Do not construct storage URLs or auth flows manually.
- No `any`. No `@ts-ignore`. No untyped props.
- Functions do one thing. Max 30 lines per function. Max 200 lines per component file.
- Delete the old pattern before writing the new one. No dead imports, unused props, or orphaned fallbacks.
- No fallbacks that mask broken states. If the server guarantees data, do not null-check it client-side.
- Business logic values (roles, bucket names, TTLs) go in named constants. No magic strings.
- Layouts own outer spacing. Components own inner spacing. Do not override shadcn component spacing.
- Every component is independently testable with mocked props or hooks.
- Search shadcn/ui before building any UI component. Search the codebase before creating a new pattern.
- No custom code that reimplements what a framework, SDK, or library in package.json already provides.
- No custom auth, validation, fetch wrappers, retry helpers, date formatting, toast systems, or UI primitives.
- Before writing any code, check in order: existing repo patterns → framework primitives → first-party SDKs → approved libraries in package.json.
- Custom code is only acceptable for Speqq-specific product logic that no dependency provides — workspace rules, document workflows, domain types.
- If the problem sounds generic instead of product-specific, stop and ask before writing custom code.
- Do not repeat logic. If it exists, reuse it.
- Missing configuration = throw immediately. `process.env.VAR || "default"` is forbidden.
- Every configuration value has exactly one source. No duplicates across files.
- Multi-step flows (auth, email, payments) must use the same origin throughout. Mixing origins breaks cookies and CORS.
- Server-side branches must be observable — tagged in a dd-trace span, logged, or metered.
- Multi-step flows must be tested end-to-end, not one endpoint in isolation.
- Middleware runs in Edge Runtime — no Node.js modules (`dd-trace`, `fs`, `crypto`). Route handlers run in Node.js Runtime with `export const runtime = "nodejs"`.
- `NEXT_PUBLIC_*` variables are baked at build time. Server-side `process.env` reads happen at runtime.
- Server Components by default. Only add `'use client'` for browser APIs, event handlers, or React state.
- Data fetching happens in Server Components and passes data as props to client components. Do not use SWR or client-side fetch for initial data loads.
- Use Server Actions (`"use server"`) for mutations (form submissions, plan changes, state updates). Prefer Server Actions over API route calls from client components.
- **Migration note:** The codebase is moving from client-side SWR fetching to Server Components + Server Actions. Older pages may still use SWR in client components. New code must use Server Components for data fetching. Migrate older patterns only when directly modifying them.
- Names include the domain noun: `workspaceId` not `id`, `documentTitle` not `title`, `memberRole` not `role`.
- Booleans start with `is`, `has`, `can`, `should`: `isDocumentLocked` not `locked`.
- Handlers start with `handle` or `on` + domain + action: `handleDocumentSave` not `handleSave`.
- No bare nouns (`id`, `name`, `status`, `data`, `error`) — always domain-qualify.
- No abbreviations that lose meaning: `req`, `res`, `cfg`, `ctx` are forbidden unless industry-standard.
- No UUIDs, URLs, secrets, timeouts, or tenant IDs hardcoded in source. Use env vars, DB config, or named constants.
- UI copy (labels, headings, button text, error messages) is the only acceptable hardcoded value.
- Secrets and service URLs from `process.env` server-side only. Client Components receive config as props.
- Every data-driven component has three states: loading (Skeleton), error (Sonner toast or destructive text), empty (muted text).
- Every interactive element uses shadcn/ui. No raw `<button>`, `<input>`, `<select>`, `<textarea>`.
- Colors use semantic tokens only. No hex, no Tailwind palette colors, no `dark:` variants.
- No arbitrary Tailwind values (`w-[347px]`). Exception: `calc()` for CodeMirror/react-arborist heights.
- **Design system:** Fluent 2 values. Use only tokens from the top section of `styles/globals.css` (above the LEGACY marker). See `.claude/DESIGN_SYSTEM.md` for the full spec and audit checklist.
- **Typography:** Primary UI text = `--font-text-sm` (14px). `--font-text-xs` (12px) = captions/timestamps only. No font inflation on mobile.
- **Spacing:** `--space-*` tokens on a 4px grid. No arbitrary pixel values for padding/margin/gap.
- **Touch targets:** 44px minimum on mobile. Auto-handled by CSS overrides on `--control-size-*`.
- One scroll container per panel. Scroll element is direct child of flex column with `flex-1`. Headers/footers `shrink-0`.
- No unnecessary wrapper `<div>`. Max 4 levels of DOM nesting. No `z-index` in app code.
- Route handlers are thin controllers: parse the request, call domain functions from `lib/`, return the response. No constants, no business logic, no SDK calls defined inline in route files.
- Domain logic (queries, Stripe operations, metering) lives in `lib/` as typed, testable functions. Route handlers import and compose them.
- Constants (column selections, config values, error messages) live in `lib/constants/`. Zero string literals or config values defined in route files.
- Types come from `lib/types/supabase-generated.ts` (auto-generated from the database schema). Do not hand-write types for database rows.
- **Migration note:** The codebase is moving toward this pattern. Older routes may still have inline queries and constants. New code must follow this pattern. Migrate older routes only when directly modifying them.
- RLS is always respected. Service role bypasses require an explicit comment explaining why.
- Simple single-table queries may be written directly in `lib/` domain functions. Multi-service composition (e.g., Supabase + Stripe) must be in `lib/` — never inline in a route handler.
- No `any`, `@ts-ignore`, `object`, `Function`, `{}`. Every param, prop, and return has a type.
- Database types come from `lib/types/supabase-generated.ts`. Shared application types in `lib/types/index.ts`. Feature-local types in the same file.
- Errors are always visible to the user. No empty `catch`, no `console.log`-only, no “Something went wrong.”
- Server errors return proper HTTP status + JSON error message. Never expose DB columns or stack traces.
- Sonner toast for retryable errors. Inline destructive text for field errors. `error.tsx` for unhandled exceptions.
- Only touch files the task requires. No background refactors, renames, or reformats.
- No barrel files (`index.ts` re-exports). No new root-level folders without approval.
- Stop and ask before: adding dependencies, building non-shadcn components, adding data layers, using hex colors, bypassing SDKs, modifying 10+ files, or making product decisions.
- Pick the lowest-complexity algorithm that fits. State the Big-O of the solution in a comment, and if it's worse than O(n log n) on a hot path, justify why or rewrite." — This forces the agent to actually think about complexity instead of defaulting to nested loops, and the comment makes it auditable.
"Run independent work concurrently. If two operations don't depend on each other (API calls, file reads, DB queries), use async/await with Promise.all / asyncio.gather / goroutines — never await them serially." — This catches the single biggest real-world perf killer in modern code: sequential I/O that should be parallel.
"Flatten control flow. Use early returns and guard clauses instead of nesting. No function should have more than 2 levels of indentation inside its body." — This is your "no nested ifs" rule, made concrete. The 2-level cap is the trick — it gives the agent a measurable target instead of a vibe.

## Process

### Definition of Ready (before writing any code)

Do not write code until the design is confirmed with the user.

1. **Identify the layers.** For every change, name: the API route, the hook, the component, and the layout. If a layer doesn't exist, build it first.
2. **Identify the data flow.** For each piece of data: where does it come from? Every answer must be “API route via hook” or “Server Component prop.” If the answer is “computed in the component,” move it to the API.
3. **Identify the components.** Search shadcn/ui and the codebase. List every component by name. If creating a new one, explain why nothing existing works.
4. **Remove before you add.** If replacing a pattern, delete every reference to the old pattern first.
5. **Check for masking fallbacks.** If the server guarantees data, do not null-check client-side. Every `?? fallback` needs a documented reason.
6. **Present the design.** Describe the layers, data flow, and components to the user. Do not code until confirmed.

### Definition of Done (before declaring work complete)

- `./speqq-local run npm run build` passes.
- TypeScript strict passes with zero errors.
- Lint passes with zero warnings in changed files.
- Every data operation goes through an API route.
- Every UI element uses shadcn. No raw HTML interactive elements.
- No client-side validation duplicating server validation.
- No unused imports, props, or dead code in changed files.
- No hardcoded business logic values. Named constants for roles, bucket names, TTLs.
- Spacing follows ownership: layouts own outer, components own inner.
- Each component is independently testable.
- Tests updated for changed behavior.
- Documentation updated if the change affects user-facing behavior, setup, or API contracts.
