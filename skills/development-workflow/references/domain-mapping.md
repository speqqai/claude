# Domain-to-tool mapping

Before writing any code, identify which domain the work touches and use the correct approved tool. **Do not write custom code when an approved SDK, framework, or first-party API handles it.**

## Core framework

| Domain | Approved tool | Skill to invoke | Read BEFORE coding | Official docs |
|--------|--------------|-----------------|-------------------|---------------|
| Application architecture | Next.js 16 App Router + React 19 | `/application-development` | `.claude/skills/application-development/references/official-sources.md`, `.claude/skills/application-development/references/scalability-checklist.md` | https://nextjs.org/docs, https://react.dev |
| Next.js patterns | App Router, SSR, Route Handlers | `/next-js-framework` | `.claude/skills/next-js-framework/SKILL.md` | https://nextjs.org/docs |
| TypeScript | Strict mode, generics, Zod validation | `/mastering-typescript` | `.claude/skills/mastering-typescript/SKILL.md` | https://www.typescriptlang.org/docs/, https://zod.dev |
| Styling / CSS | Tailwind CSS 4 (semantic tokens only) | `/application-development` | `.claude/skills/application-development/references/css-and-accessibility.md` | https://tailwindcss.com/docs |

## Data and backend

| Domain | Approved tool | Skill to invoke | Read BEFORE coding | Official docs |
|--------|--------------|-----------------|-------------------|---------------|
| Database / data layer | Supabase (`@supabase/ssr`, direct calls in Route Handlers) | `/supabase-postgres-best-practices` | `.claude/skills/supabase-postgres-best-practices/SKILL.md` | https://supabase.com/docs |
| Client data fetching | SWR (all client-side data fetching and caching) | — | CLAUDE.md Section 9.2 | https://swr.vercel.app |
| Context engine / knowledge graph | Graphiti + Neo4j | `/context-engine` | `.claude/skills/context-engine/references/graphiti.md`, `.claude/skills/context-engine/references/neo4j.md`, `.claude/skills/context-engine/references/ontology-design.md`, `.claude/skills/context-engine/references/retrieval-patterns.md`, `.claude/skills/context-engine/references/temporal-knowledge-graphs.md` | https://neo4j.com/docs/ |

## Python TUI (CLI)

| Domain | Approved tool | Constraint | Official docs |
|--------|--------------|------------|---------------|
| TUI framework | Textual (`textual[syntax]`) | All TUI layout, widgets, and screens | https://textual.textualize.io |
| HTTP client | httpx | Async HTTP for Speqq API (streaming SSE) | https://www.python-httpx.org |
| Linting / formatting | Ruff | Linting and formatting for all Python code | https://docs.astral.sh/ruff/ |
| Testing | pytest | All Python tests | https://docs.pytest.org |
| Package management | UV or pip | — | https://docs.astral.sh/uv/ |

**Python-specific rules:**
- The TUI is a standalone Python package at `poc/cli/` — it is NOT part of the Next.js app
- HTTP-only integration — talks to Speqq via existing REST API endpoints
- No shared code with the main Next.js app
- Two external dependencies only: `textual` and `httpx`
- Use Textual built-in widgets before building custom ones (same zero-custom-code principle)

## AI and integrations

| Domain | Approved tool | Skill to invoke | Read BEFORE coding | Official docs |
|--------|--------------|-----------------|-------------------|---------------|
| AI / chat / streaming | Vercel AI SDK (`ai`, `@ai-sdk/react`) + OpenAI provider (`@ai-sdk/openai`) | — | CLAUDE.md Section 6 | https://sdk.vercel.ai/docs |
| MCP integration | `@modelcontextprotocol/sdk` | `/mcp-builder` | `.claude/skills/mcp-builder/SKILL.md` | https://modelcontextprotocol.io/docs |
| Codex app server | Codex protocol (JSON-RPC 2.0) | `/codex` | `.claude/skills/codex/SKILL.md` | — |

## UI components and libraries

| Domain | Approved tool | Constraint | Official docs |
|--------|--------------|------------|---------------|
| Interactive UI | shadcn/ui + Radix primitives (ALL interactive elements) | No raw `<button>`, `<input>`, `<select>`, `<textarea>` | https://ui.shadcn.com, https://www.radix-ui.com |
| Icons | lucide-react (**exclusively**) | No other icon libraries | https://lucide.dev |
| Toasts / notifications | Sonner (**exclusively**) | No shadcn Toast, no custom toast components | https://sonner.emilkowal.ski |
| Date formatting | date-fns (**exclusively**) | No manual date parsing, no moment, no dayjs | https://date-fns.org |
| Dark mode / themes | next-themes | No custom theme switching | https://github.com/pacocoursey/next-themes |
| Document editor | CodeMirror 6 | — | https://codemirror.net/docs/ |
| Requirements table | @tanstack/react-table | — | https://tanstack.com/table |
| File tree | react-arborist | — | https://react-arborist.netlify.app |

## Location-locked packages

These packages may ONLY be used in the specified files. If you import them elsewhere, it is a violation.

| Package | ONLY allowed in | Purpose |
|---------|----------------|---------|
| `use-stick-to-bottom` | `components/chat/elements/conversation.tsx` | Auto-scroll in chat |
| `streamdown` | `components/chat/elements/response.tsx` | Streaming markdown |
| `react-markdown` | Any file, but for **non-streaming** markdown only | Static markdown rendering |

## Process skills

| Domain | Skill to invoke | Read BEFORE coding |
|--------|-----------------|-------------------|
| PRD writing | `/prd` | `.claude/skills/prd/references/prd-standard.md`, `.claude/skills/prd/references/prd-template.md`, `.claude/skills/prd/references/validation-rubric.md` |
| Technical design | `/tech-design-doc` | `.claude/skills/tech-design-doc/SKILL.md` |
| Testing | `/test-driven-development` | `.claude/skills/test-driven-development/SKILL.md` |
| Documentation | `/documentation-writer` | `.claude/skills/documentation-writer/SKILL.md`, `.docs/technical-writing/technical-writing-style-guide.md` |
| Frontend design | `/frontend-design` | `.claude/skills/frontend-design/SKILL.md` |

## Zero custom code — specific violations

If you write any of these, you are in violation. Use the approved tool instead.

| Violation | Use instead |
|-----------|-------------|
| Hand-rolled date parsing or formatting | `date-fns` |
| Custom fetch wrappers or data hooks | `SWR` |
| Custom icon components or imported icon SVGs | `lucide-react` |
| Custom toast/alert/notification components | `Sonner` |
| ORM wrappers, service classes, repository patterns | Direct Supabase calls in Route Handlers |
| Custom theme toggle or dark mode logic | `next-themes` |
| Custom markdown renderer for streaming | `streamdown` (in `response.tsx` only) |
| Custom markdown renderer for static content | `react-markdown` |
| Custom form validation logic | `Zod` |
| Raw `<button>`, `<input>`, `<select>`, `<textarea>` | shadcn/ui components |
| Arbitrary Tailwind values (`w-[347px]`, `bg-[#fff]`) | Tailwind scale utilities + semantic tokens |
| `dark:` CSS variants | CSS variables via `next-themes` |

## Rules

- If your task touches a domain in these tables, you MUST read the files in the "Read BEFORE coding" column before writing any code
- If you find yourself writing logic that reimplements something an approved tool already does, stop and use the tool instead
- If no approved tool covers the need, STOP and ask the user before writing custom code
- See CLAUDE.md Section 6 (Tech Stack) and Section 12 (First-Party SDK First) for the full policy
