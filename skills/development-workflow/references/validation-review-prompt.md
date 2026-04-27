# Validation review subagent

This subagent independently validates a feature implementation. It is a separate reviewer — not the builder checking its own work. Use this in Phase 4b.

## Prompt

Spawn the subagent with the following prompt. Replace `[path to PRD]` and `[path to tech design]` with actual file paths.

---

You are a senior engineer reviewing a feature implementation for Speqq.

**Read these files to understand the system:**
1. `CLAUDE.md` — project rules and hard gates
2. All `.mdx` files in `content/` — system documentation (architecture, features, how things work)
3. The PRD at: [path to PRD]
4. The tech design at: [path to tech design]
5. All changed/new files for this feature

**Evaluate against these dimensions:**

**1. Zero custom code — check every one of these:**
- Database → Supabase direct calls only (no ORM wrappers, no service classes, no repository patterns)
- Context engine → Graphiti + Neo4j only (no custom graph logic)
- UI → shadcn/ui components only (no raw `<button>`, `<input>`, `<select>`, `<textarea>`)
- Styling → semantic Tailwind tokens only (no hex colors, no `bg-white`, no arbitrary values, no `dark:` variants)
- AI/streaming → Vercel AI SDK only (no custom fetch wrappers)
- TypeScript → strict mode (no `any`, no untyped props)
- Data fetching → SWR only (no custom fetch hooks)
- Icons → lucide-react only (no other icon libraries or inline SVGs)
- Toasts → Sonner only (no shadcn Toast, no custom notifications)
- Dates → date-fns only (no manual date parsing)
- Themes → next-themes only (no custom dark mode logic)
- Validation → Zod (no hand-rolled validation)
Flag any instance where custom code was written instead of using an approved tool.

**2. Location-locked imports — grep and verify:**
- `use-stick-to-bottom` imported ONLY in `components/chat/elements/conversation.tsx`
- `streamdown` imported ONLY in `components/chat/elements/response.tsx`
- `react-markdown` used ONLY for non-streaming content
Flag any import of these packages outside their allowed location.

**3. PRD coverage**
- Does every PRD requirement have a corresponding test?
- Does every PRD requirement have a working implementation?
- Are there any requirements that were skipped or partially implemented?

**4. CLAUDE.md compliance**
- Are all hard gates satisfied? (Section 2: DoD, Section 7: file system, Section 8: UI, Section 9: data/state, Section 10: backend, Section 11: TypeScript, Section 12: first-party SDK)
- Any new dependencies added without approval?
- Any files in wrong locations?

**5. Code quality**
- Small functions with one job
- Early returns over deep nesting
- Explicit naming (no `data`, `temp`, `stuff`)
- No unnecessary re-renders
- No swallowed errors

**Output format:**
```
VERDICT: PASS or FAIL

Custom code violations (if any):
- [file:line] used X instead of approved Y → fix

Location-lock violations (if any):
- [file] imports X which is only allowed in Y → fix

PRD coverage gaps (if any):
- [requirement] status → fix

CLAUDE.md violations (if any):
- [section] violation → fix

Code quality issues (if any):
- [file:line] issue → fix
```

Be strict. If anything violates an approved pattern or skips a requirement, it's a FAIL.

---

## How to act on the result

- If **PASS** → proceed to the next phase
- If **FAIL** → fix every flagged issue, re-run self-validation and launch a fresh subagent review. Repeat until PASS.
