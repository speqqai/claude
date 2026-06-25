# Speqq Engineering Guide

## Purpose

These rules make agents write code that is native-first, easy to debug, reusable, efficient, and safe to operate. Generic concerns belong to frameworks and SDKs. Speqq-specific behavior belongs in typed domain modules.

## Native-First Hierarchy

Before writing custom code, check in this order:

1. Current repo pattern that already follows this guide.
2. Next.js / React primitive.
3. First-party SDK for the service involved.
4. Approved package already in `package.json`.
5. Speqq-specific custom code.

Custom code is only for product behavior that no provider owns: workspace rules, document workflows, domain policies, agent workflows, and product-specific transformations.

Thin domain adapters are allowed when they encode Speqq policy around a native SDK. Generic wrappers are not. A good adapter has domain language, typed inputs, typed outputs, and no hidden retries/cache/auth behavior.

Forbidden without explicit approval:

- New dependencies.
- Custom auth/session systems.
- Custom validation libraries.
- Custom fetch/data clients for internal UI.
- Custom toast/modal/form primitives when shadcn/Sonner/React already cover it.
- Manual Supabase storage URL/auth flow construction.
- Duplicate implementations of the same concern.

## Product-First Code Quality

Agents must preserve the product architecture, not just make the immediate request pass.

Before implementing, answer:

1. What user outcome or product contract does this change affect?
2. What is the canonical owner for this logic?
3. What existing pattern should this reuse?
4. What future maintenance cost could this introduce?

Do not ship a quick fix that creates:

- a second source of truth
- duplicated validation or formatting
- hidden state
- UI-owned business logic
- broad defensive guards
- untraceable errors
- one-off components or utilities

If the prompt asks for a local fix but the correct solution requires moving logic to the canonical owner, do the canonical implementation or stop and ask if the scope is too large.

## Canonical Data Flow

Use this decision tree for new code:

| Use case | Boundary | Domain owner |
|---|---|---|
| Initial page data | Server Component | `lib/domains/<domain>/queries.ts` |
| UI mutation | Server Action | `lib/domains/<domain>/mutations.ts` |
| External client, MCP, CLI, webhook, PAT, streaming | Route Handler | `lib/domains/<domain>/service.ts` |
| Browser-only state/events | Client Component | typed props from server boundary |
| Live cross-client updates | Supabase realtime | one owner per concern |
| Collaborative document content | Yjs WebSocket | document content only |

Rules:

- Server Components by default.
- Add `'use client'` only for browser APIs, React state, event handlers, realtime subscriptions, editor bindings, or imperative UI.
- Client Components do not query databases, construct app URLs, check permissions, validate business rules, or call internal `/api/*` for UI mutations.
- API routes are thin external transport controllers: parse input, call a domain function, return a response.
- Legacy SWR/API/hook patterns are not precedent for new work. Migrate legacy patterns only when directly modifying that surface.

## Legacy Pattern Policy

Existing code is not automatically precedent. This guide is the source of truth for new work.

- Prefer current canonical architecture over nearby legacy examples.
- When touching older `lib/*`, route, SWR, or hook-based code, migrate only the directly edited concern.
- Do not broad-migrate unrelated files to satisfy the target architecture.
- If local code conflicts with this guide, follow this guide and note the conflict in your summary.
- If you must make a short-term fix inside a harmful pattern, quarantine it: keep the edit narrow, do not add new users of the pattern, and identify the long-term canonical migration.
- Never extract, wrap, document, or promote a harmful legacy pattern as reusable infrastructure.

## Domain And File Ownership

One concern has one home:

| Concern | Home |
|---|---|
| Route composition, layouts, pages, loading/error files | `app/**` |
| Server Actions | colocated under `app/**` or feature route boundary |
| External transport | `app/api/**` |
| Domain queries/mutations/services/permissions/schemas/constants | `lib/domains/<domain>/**` |
| Shared generated DB types | `lib/types/supabase-generated.ts` |
| Shared app types | `lib/types/index.ts` |
| Feature-local types | same feature file/folder |
| Domain UI | `features/<domain>/components/**` |
| shadcn primitives | `components/ui/**` |
| App layout primitives | `components/layout/**` |
| Product-agnostic shared components | `components/common/**` |

Do not add new root-level folders without approval. Do not add barrel files for local convenience; package public APIs are the only acceptable exception.

`lib/domains/<domain>` is the target for new domain code. Existing domain logic outside that structure may remain until directly touched.

## Component Rules

- Search `components/ui`, `components/layout`, `components/common`, and the relevant feature folder before creating a component.
- Every interactive primitive uses shadcn/ui or an approved domain library. No raw `<button>`, `<input>`, `<select>`, `<textarea>`, `<dialog>`, `<details>`, or `<summary>` for app UI.
- Components own presentation and local interaction only.
- Layouts own outer spacing. Components own inner spacing. UI primitives own internal spacing.
- Data-driven UI has loading, error, and empty states at the correct boundary.
- Split components when mode flags or branching make behavior hard to test.

VS Code/Cursor extension webviews are an exception because shadcn is not available there. Webviews may use native HTML, but they must be minimal, accessible, escaped, CSP-safe, and isolated from app UI patterns.

## Design System

- Fluent 2 values, not Fluent components.
- shadcn/ui for standard UI primitives.
- Use only tokens from the top section of `styles/globals.css` above the `LEGACY` marker.
- Migrate legacy tokens only when directly modifying that component.
- Primary UI text: `--font-text-sm`.
- Captions/timestamps/tertiary info: `--font-text-xs`.
- Spacing: `--space-*` on a 4px grid.
- Touch targets: 44px minimum on mobile.
- Colors: semantic tokens only. No hex, Tailwind palette colors, or `dark:` variants.
- No arbitrary Tailwind values except established editor height cases such as CodeMirror/react-arborist `calc()`.

## TypeScript And Naming

- No `any`, `@ts-ignore`, `object`, `Function`, or `{}`.
- Every exported function has typed params and return value.
- Database rows use generated Supabase types.
- Prefer `satisfies` for object maps/config tables so keys stay precise without unsafe assertions.
- Prefer discriminated unions for state machines, result objects, and event payloads.
- Use `unknown` at unsafe boundaries, then narrow with Zod or a focused type guard.
- Type guards must be small, named by domain, and tested if they protect runtime data.
- Do not use type assertions (`as SomeType`) to silence compiler errors. Use assertions only at well-defined external boundaries after validation, and keep them local.
- Domain nouns are explicit: `workspaceId`, `documentTitle`, `memberRole`.
- Booleans start with `is`, `has`, `can`, `should`.
- Handlers start with `handle` or `on` plus domain/action.
- Avoid ambiguous nouns: no bare `id`, `name`, `status`, `data`, `error` in broad scopes.
- Avoid unclear abbreviations like `req`, `res`, `cfg`, `ctx` unless industry-standard in the local context.

## TypeScript Control Flow

Keep TypeScript code boring and readable:

- Prefer guard clauses and early returns over nested `if`/`else`.
- Keep normal function bodies to two indentation levels. Extract named helpers when branching grows.
- Use exhaustive `switch` for discriminated unions. The default branch should assign to `never` when practical.
- Avoid boolean flag soup. If behavior has modes, model it as a discriminated union.
- Avoid optional chaining chains that hide missing invariants. If a value is required, validate once and proceed with a non-null type.
- Keep regex at boundaries only. Name complex regex constants, document what they parse, and add tests. Prefer URL, URLSearchParams, Intl, date-fns, Zod, or SDK parsers over regex.
- Avoid broad `try/catch`. Catch at route/action/job boundaries or around a specific fallible SDK call. Convert errors to typed, user-safe failures there.
- Do not catch and continue unless continuing is the explicit product behavior.
- Split parsing, validation, domain decision, and side effect into separate named functions when a flow becomes hard to scan.

## Config And Security

- Missing required config throws immediately.
- `process.env.X || "fallback"` is forbidden for required values.
- Every config value has one source.
- No UUIDs, tenant IDs, secrets, service URLs, or timeouts hardcoded in source.
- Secrets and service URLs are server-side only. Client Components receive safe config as props.
- RLS is respected. Service role bypass requires a comment explaining why it is safe and necessary.
- Middleware runs in Edge Runtime: no Node modules such as `fs`, `crypto`, or `dd-trace`.
- Route handlers that need Node APIs declare `export const runtime = "nodejs"`.

## Debuggability And Errors

- Errors are visible to users and diagnosable by engineers.
- No empty `catch`, no console-only handling, no vague `"Something went wrong"` for actionable failures.
- Server errors return safe JSON and proper status. Never expose stack traces or DB internals.
- Retryable UI errors use Sonner. Field errors use inline destructive text. Unexpected route failures use `error.tsx`.
- Important server-side branches are logged/traced/metered with enough context to debug without leaking secrets.
- Do not add fallbacks that hide broken invariants. If the server guarantees data, do not null-check it away client-side.

Every Server Action and Route Handler should make these easy to identify during debugging:

- operation name
- relevant domain ids, never secrets
- authenticated actor or source when available
- validation failure reason
- downstream SDK/service error category
- final status/result

## State, Session, And Freshness

One concern, one source:

| Concern | Owner |
|---|---|
| Workspace/document/thread identity | URL segment |
| Filters/views | URL search params |
| Panel visibility | parallel routes where practical |
| Tiny durable prefs | cookies, one cookie per concern |
| Ephemeral UI | component-local `useState` |
| Cold-start recovery | DB column |
| Server request memoization | React `cache()` |
| Tab-return freshness | `router.refresh()` in `startTransition` on `visibilitychange` |

Rules:

- No Zustand for state that maps to URL, cookie, server data, or DB.
- No app-owned `localStorage`/`sessionStorage`; reserve browser storage for third-party SDKs such as Supabase auth or Yjs persistence.
- At most one app-level `onAuthStateChange` subscriber.
- Browser Supabase SDK uses `autoRefreshToken: false`; middleware owns token refresh.
- Auth decisions use `getUser()`/server auth boundaries, not `getSession()`.
- Use `redirect()`, `notFound()`, `unauthorized()`, or `forbidden()` for route/auth boundaries when stable. Default to redirect for sign-in flows until stability is verified.

## Forms And Mutations

- Forms use Server Actions with `useActionState` and `useFormStatus`.
- Validation runs server-side with Zod.
- Optimistic UI uses `useOptimistic`.
- Mutation invalidation uses `revalidateTag()` or `revalidatePath()`.
- Non-blocking post-response work uses `after()` from `next/server`.
- No custom client fetch wrappers, temp-ID reconciliation systems, or manual form-state machines for generic form concerns.

## Performance

- Server-render and stream by default.
- Minimize client bundle and client-only components.
- Run independent I/O concurrently with `Promise.all`.
- Avoid N+1 DB/API calls; batch or join at the domain layer.
- Use select lists deliberately; do not fetch whole rows for broad lists unless needed.
- Use stable cache tags and revalidation boundaries for server data.
- Any loop that performs async work must batch/parallelize or explain why sequential execution is required.
- Any broad list query must specify selected columns and ordering.
- For hot paths, unbounded loops, nested data processing, or large client lists, choose the lowest-complexity algorithm that fits and document non-obvious tradeoffs.
- Do not add dependencies or large client libraries without approval and bundle/performance justification.

## Testing And Validation

Match testing to risk:

- Pure helper/domain function: focused unit tests.
- Server Action/route/domain integration: integration tests with typed fixtures or local service harness.
- Database migration/RLS: migration and access tests.
- User-facing workflow: browser/manual or E2E validation.
- Performance-sensitive path: prove no obvious N+1 or serial I/O regression.

Do not declare completion with introduced TypeScript errors, lint warnings, failing tests, or unvalidated DB/API contract changes.

## Definition Of Ready

Before non-trivial code:

1. Identify the canonical data flow.
2. Identify the domain owner.
3. Identify existing components/patterns to reuse.
4. Identify validation needed for the risk tier.
5. Ask before violating a hard gate.

## Definition Of Done

For changed behavior:

- Implementation uses canonical architecture.
- Native/SDK-first checklist passed.
- Types are strict.
- User-visible states exist where needed.
- Errors are visible and debuggable.
- Tests match risk.
- Relevant lint/type/build commands pass.
- Docs updated if user-facing behavior, setup, or API contracts changed.

## Stop And Ask

Stop before adding dependencies, new data layers, new root folders, service-role bypasses, generic utilities, non-shadcn primitives, hex colors, or broad refactors.
