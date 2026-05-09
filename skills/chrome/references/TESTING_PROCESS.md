# Testing Process — Speqq Journey QA

The deterministic loop the `chrome` skill follows. Tool-agnostic in spirit, but written assuming `chrome-devtools-mcp` is the active driver (see [`CHROME_CLI.md`](CHROME_CLI.md)). If the session is using `claude-in-chrome` instead (see [`CLAUDE_IN_CHROME.md`](CLAUDE_IN_CHROME.md)), substitute the equivalent tool names — the workflow logic is identical.

---

## Per-Session Bootstrap

Run **once** at the start of a QA session.

### 1. Confirm setup
- `mcp__chrome-devtools__list_pages` should return your real tabs (Speqq, Gmail, etc.). If not → run `references/CHROME_CLI.md` § One-time setup.
- If the user wants `claude-in-chrome` instead → `mcp__claude-in-chrome__tabs_context_mcp`.

### 2. Capture session inputs (ask the user, don't guess)

**Target environment** — ask which one for this session:

| Env | URL | Notes |
|---|---|---|
| `local` | `http://${HOST_IP}:3000` | `HOST_IP` from `docker/.env.dev` (set by `./speqq-local up`) |
| `dev` | `https://dev.speqq.com` | |
| `prod` | `https://speqq.com` | Be cautious — real production data, real RLS |

**Test account** — Doppler-only (no fallbacks). Retrieve at session start:
```bash
doppler secrets get TEST_SPEQQ_USER1_EMAIL TEST_SPEQQ_USER1_PASSWORD \
  --config "$SPEQQ_LOCAL_DOPPLER_CONFIG" --project speqq --plain
```
Verified: outputs two lines — email, then password.

If `SPEQQ_LOCAL_DOPPLER_CONFIG` is unset → **throw error and stop**. Do not default to `local`. Do not read from `docker/.env.dev`. Per `feedback_no_fallbacks_error_instead.md`. Ask the user to:
```bash
export SPEQQ_LOCAL_DOPPLER_CONFIG=local_<yourname>
```
…and re-launch Claude Code so child processes inherit the var.

**If `doppler` itself fails** (CLI missing, not authenticated, secret missing in that config) → also stop. Do not read secrets from `docker/.env.dev`. Surface the error to the user.

**Never paste the password into any file or into chat.**

If the user names a different test account, capture that instead.

> Multi-tenant note: wrong account = wrong RLS scope = false bugs. Always confirm before running journeys.

**Journey list** — verbatim from the user. If none provided, **ask first** — do not improvise journeys.

### 3. Create the QA log file

Path: `.docs/qa/journey-test-<YYYY-MM-DD>.md`

Seed it with the format below (see § QA Log Format). Reflect every journey from the user's list as a checkbox.

### 4. Open a clean tab and sign in
- `new_page` → navigate to the env URL.
- If a sign-in page is shown, fill `TEST_SPEQQ_USER1_EMAIL` and password (or whichever account was supplied), submit.
- If SSO/OAuth/CAPTCHA blocks → pause and ask the user to complete it manually.
- Confirm post-login state (workspace loaded) before declaring bootstrap done.
- Drain prior console noise: call `list_console_messages` once, ignore results, so the next read is scoped to Journey 1.

### 5. Stop. Confirm with the user.
Show: env, URL, account, journey count, log path, driver in use (chrome-devtools attached / claude-in-chrome). Ask "ready to start Journey 1?" Wait for explicit confirmation.

---

## Per-Journey Loop

Run for **each** journey, one at a time. Do not advance until the user confirms.

### A. Restate
One line. *"Journey 3: User creates a new PRD from the sidebar and renames it."* Catches misinterpretation early.

### B. Prepare
- Navigate to the journey's starting state.
- Login/CAPTCHA → pause and ask the user to handle. Never bypass.
- If the prior journey left state behind (modal open, toast lingering), reset before starting.

### C. Snapshot
`take_snapshot` to get the a11y tree with UIDs. **One snapshot covers many subsequent clicks** on the same view — don't re-snapshot per click.

Re-snapshot when:
- Navigation
- Modal/sheet opens or closes
- Major list update (sort, filter, pagination)
- A click "didn't take" → UID may be stale

### D. Walk the steps
- Click by UID, fill by UID — `chrome-devtools__click(uid=...)`, `fill`, `fill_form`.
- After async actions, use `wait_for("expected text")` instead of arbitrary sleeps.
- If a flow may trigger a JS dialog (`confirm`, `prompt`), prepare to call `handle_dialog` immediately after the trigger.

### E. Verify (after each meaningful step) — Four-Signal Check
All four must be clean before marking the step ✅.

| Signal | Tool | Filter |
|---|---|---|
| **UI state** matches expectation | `take_snapshot` (or `evaluate_script` for global state) | Look for the expected text/element/role |
| **No console errors** | `list_console_messages` | Filter to `Error\|TypeError\|ReferenceError\|Warning:.*server\|Hydration` |
| **No failed network calls** | `list_network_requests` | Filter to status ≥ 400, plus CORS/timeouts |
| **Expected data present** | `evaluate_script` if needed | Read DOM/global state (e.g. `window.__NEXT_DATA__`) |

Visual evidence (`take_screenshot`) is only required for bug reports, not every step — keep it cheap.

### F. On clean → mark ✅, advance
Update the QA log checklist, output a one-line status to the user, and ask "next?"

### G. On break → run the Diagnose Protocol (do NOT retry blindly)

---

## Diagnose Protocol

When a step fails. In parallel where possible:

### 1. Capture evidence
- `take_screenshot` → save to `tmp/qa-<YYYY-MM-DD>-j<N>.png`
- `list_console_messages` (no filter, full dump for the report)
- `list_network_requests` filtered to failures (status ≥ 400, CORS, timeouts)
- For the suspect failed call: `get_network_request(id)` to get full headers + request/response body
- `take_snapshot` if the DOM looks unexpected
- (Optional) `evaluate_script` to read global state like `window.__NEXT_DATA__`, Supabase client state, current user

### 2. Locate the code path
- **Failed URL** → `Grep` for the route in `app/api/**` or for the server action handler.
- **Error message / stack frame** → `Grep` for the symbol, error string, or file in the stack.
- `Read` the relevant file(s) **before** forming a hypothesis. No guessing — cite `file:line` (per `feedback_no_guessing.md`).

### 3. Form a root-cause hypothesis
- One sentence.
- Cite supporting `file:line`.
- State confidence: **high / medium / low**.
- If confidence < high, list what would raise it (e.g. "verify `org_id` is in session by calling `evaluate_script` for `supabase.auth.getUser()`").

### 4. Propose a fix
- Smallest viable change (per CLAUDE.md § 1).
- If the proper fix needs re-architecture, **say so** — don't propose a shortcut (per CLAUDE.md § Planning).
- Prefer SDK / framework / library solution over custom code (per `feedback_use_frameworks_natively.md`).

### 5. Log it in the QA file
Append the bug entry under the failing journey using the template below.

### 6. Stop and ask
Three options for the user:
- **Fix now** — exit the skill, implement the fix, return to QA.
- **Log and continue** — leave the bug logged, move to the next journey.
- **Escalate** — needs deeper investigation or product decision.

**Never apply a fix without explicit approval. This skill is QA, not implementation.**

---

## Performance Check (when DoD § 10 applies)

Run when:
- Journey involves a perf-sensitive flow (page load, large list render, search, route transition).
- User asks for performance numbers.
- A journey "feels slow" — quantify it.

```
performance_start_trace(reload=true, autoStop=true)
  → wait for autoStop
performance_analyze_insight
```

Capture against CLAUDE.md DoD § 10 thresholds:

| Metric | Pass | Fail |
|---|---|---|
| LCP | < 2.5s | > 4s |
| CLS | < 0.1 | > 0.25 |
| INP | < 200ms | > 500ms |
| TTFB | < 800ms | > 1.8s |
| Lighthouse Performance | ≥ 90 | < 70 |

For Lighthouse: `lighthouse_audit`. For memory leaks suspected: `take_memory_snapshot` before and after a flow, compare heap.

Record numbers in the QA log under the journey.

---

## Bug Entry Template

Append under the failing journey in the QA log:

```markdown
### ❌ Journey N — <name>

**Symptom**
<One-line description of what the user sees / what failed.>

**Reproduction**
1. <step>
2. <step>
3. <step that breaks>

**Browser Evidence**

Console errors (top 3 frames each):
\`\`\`
TypeError: Cannot read property 'id' of undefined
  at createPrd (app/api/prd/route.ts:42:18)
  at handler (next/dist/server/...)
\`\`\`

Failed network calls:

| Method | URL | Status | Response |
|--------|-----|--------|----------|
| POST | /api/prd | 500 | `{"error":"missing org_id"}` |

Screenshot: `tmp/qa-2026-05-05-j4.png`
Snapshot: <pasted accessibility tree fragment if relevant>

**Code Path**
- Entry: `app/(workspace)/sidebar/new-prd-button.tsx:23`
- Failing call: `app/api/prd/route.ts:42`
- Relevant logic: `lib/prd/create.ts:15` — expects `orgId` but receives `undefined`

**Root Cause Hypothesis** (confidence: high)
Sidebar button submits without org context; route requires it. The `useCurrentOrg()` hook is not consumed in `new-prd-button.tsx:23`.

**Proposed Fix**
Smallest viable: read org from server-side session in `route.ts` rather than passing client-side. Aligns with CLAUDE.md hard gate "no hardcoded org IDs" and removes a class of multi-tenant bugs.

**Status**
- [ ] Fix approved
- [ ] Fix implemented
- [ ] Re-tested in browser
```

---

## QA Log Format (top of `.docs/qa/journey-test-<DATE>.md`)

```markdown
# Journey QA — <YYYY-MM-DD>

**Environment:** <local | dev | prod> — <URL>
**Account:** <email or test user id>
**Branch:** <git branch>
**Driver:** chrome-devtools-mcp (attached) | claude-in-chrome
**Tester:** Claude (chrome skill) + <user>

## Journeys

- [ ] 1. <journey name>
- [ ] 2. <journey name>
- [x] 3. <journey name>  ✅
- [ ] 4. <journey name>  ❌ — see [bug](#-journey-4--<slug>)
...

## Summary

| | Count |
|---|---|
| Passed | X / N |
| Failed | Y |
| Blocked | Z |
| Open bugs | <count> |

## Performance Snapshot (if collected)

| Page / Flow | LCP | CLS | INP | TTFB | Lighthouse Perf |
|---|---|---|---|---|---|
| /workspace | 1.8s | 0.04 | 120ms | 540ms | 94 |

---

## Per-Journey Entries

(Bug entries appended below as journeys fail.)
```

Update the checklist after every journey so the user can scan progress at a glance.

---

## Quick Patterns

**Filter console for app-specific errors only:**
```
list_console_messages → filter pattern: "\\[Speqq\\]|Error|TypeError|ReferenceError|Hydration|did not match|Warning:.*server"
```

**Inspect a single failed API call end-to-end:**
```
list_network_requests → filter status >= 400
get_network_request(id) → full request + response body
Grep "/api/<route>" in app/api/**
Read the route handler
```

**Catch hydration / SSR mismatches early:**
```
list_console_messages → filter "Hydration|did not match|Warning:.*server"
```

**Performance suspicion (slow page):**
```
performance_start_trace(reload=true, autoStop=true)
performance_analyze_insight  → LCP, CLS, INP, long tasks
lighthouse_audit  → score + actionable items
```

**Record a repro for a stakeholder (claude-in-chrome only):**
```
gif_creator with capture-before/after frames, named qa-journey-<N>-<slug>.gif
```

**When 2 description-based clicks miss in claude-in-chrome:**
Switch to `chrome-devtools` snapshot+UID for that interaction.

---

## Hard Rules (referenced from SKILL.md)

1. One journey at a time. Wait for explicit "next" before advancing.
2. No fixes without approval — diagnose, log, ask.
3. No guessing — read the code, cite `file:line`.
4. No fallback values — surface errors clearly.
5. Respect auth boundaries — pause for login/CAPTCHA.
6. Snapshot-first — `take_snapshot` before clicking.
7. Filter logs aggressively.
8. Save artifacts to `tmp/`.
9. DoD perf gates apply when relevant.
10. No background browsing.
