---
name: chrome
description: >-
  Walk through user journeys in a live Chrome tab and methodically validate each
  step against the running Speqq app. Use when the user asks to "test journeys",
  "walk through flows", "QA the app", "check user flows in Chrome", "test in
  browser", or shares a list of user journeys to verify. Captures console logs,
  network errors, screenshots, and performance traces when a journey breaks;
  correlates with the codebase to produce issue, root-cause hypothesis, and
  proposed fix; tracks progress across the full journey list one item at a time.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, mcp__chrome-devtools__list_pages, mcp__chrome-devtools__select_page, mcp__chrome-devtools__navigate_page, mcp__chrome-devtools__new_page, mcp__chrome-devtools__close_page, mcp__chrome-devtools__take_snapshot, mcp__chrome-devtools__take_screenshot, mcp__chrome-devtools__click, mcp__chrome-devtools__fill, mcp__chrome-devtools__fill_form, mcp__chrome-devtools__hover, mcp__chrome-devtools__drag, mcp__chrome-devtools__press_key, mcp__chrome-devtools__type_text, mcp__chrome-devtools__upload_file, mcp__chrome-devtools__handle_dialog, mcp__chrome-devtools__wait_for, mcp__chrome-devtools__list_console_messages, mcp__chrome-devtools__get_console_message, mcp__chrome-devtools__list_network_requests, mcp__chrome-devtools__get_network_request, mcp__chrome-devtools__evaluate_script, mcp__chrome-devtools__lighthouse_audit, mcp__chrome-devtools__performance_start_trace, mcp__chrome-devtools__performance_stop_trace, mcp__chrome-devtools__performance_analyze_insight, mcp__chrome-devtools__take_memory_snapshot, mcp__chrome-devtools__emulate, mcp__chrome-devtools__resize_page, mcp__claude-in-chrome__tabs_context_mcp, mcp__claude-in-chrome__tabs_create_mcp, mcp__claude-in-chrome__navigate, mcp__claude-in-chrome__find, mcp__claude-in-chrome__computer, mcp__claude-in-chrome__form_input, mcp__claude-in-chrome__get_page_text, mcp__claude-in-chrome__read_page, mcp__claude-in-chrome__read_console_messages, mcp__claude-in-chrome__read_network_requests, mcp__claude-in-chrome__javascript_tool, mcp__claude-in-chrome__gif_creator, mcp__claude-in-chrome__resize_window
---

# Chrome — User Journey QA

Drive a live Chrome tab. Walk a user-supplied journey list one item at a time. Capture browser evidence + read the codebase to produce precise bug reports.

**Decision: `chrome-devtools` MCP is the primary tool.** It has every QA capability needed — screenshots, performance traces, Lighthouse, dialog handling, deterministic UID-based clicking. The `claude-in-chrome` extension is only used when the user explicitly opts into it for some reason.

---

## When to Use

Trigger when the user says any of:
- "test journeys" / "walk through journeys" / "QA the app"
- "check user flows in Chrome" / "test in browser"
- Shares a checklist or numbered list of user flows to verify
- "let's go through the list"

If no journey list is supplied yet, **ask for it first**. Do not improvise journeys.

---

## Reference Files

Read these on demand. Do not duplicate their contents in this file.

- **[`references/CHROME_CLI.md`](references/CHROME_CLI.md)** — `chrome-devtools-mcp` setup, `--browser-url` attach to real Chrome, full tool inventory, gotchas. **This is the primary tool — read this first.**
- **[`references/CLAUDE_IN_CHROME.md`](references/CLAUDE_IN_CHROME.md)** — `claude-in-chrome` extension setup, tool inventory, capability gaps, when to use it as fallback.
- **[`references/TESTING_PROCESS.md`](references/TESTING_PROCESS.md)** — the per-session bootstrap, per-journey loop, four-signal verify, diagnose protocol, bug log format, performance check.

---

## Setup Preflight (run every session, in order)

Every command below has been verified on macOS. Run them in sequence. Don't skip a step because "it probably works."

### Step 1 — Probe both drivers in parallel

Call both, see what answers:

```
mcp__chrome-devtools__list_pages
mcp__claude-in-chrome__tabs_context_mcp
```

Then choose a driver from this matrix. **You only need ONE working driver** — don't gate on both.

| `chrome-devtools` result | `claude-in-chrome` result | Driver to use |
|---|---|---|
| Returns ≥1 page (any URL) | (anything) | **`chrome-devtools`** (primary — has snapshots, screenshots, perf, dialog handling) |
| Returns only `about:blank` / chrome-internal pages | Returns real tabs | **`chrome-devtools`** — but you must navigate to your env URL and (re)sign in. The MCP runs Chrome in its own managed profile (`~/.cache/chrome-devtools-mcp/shared-debug-profile`); login persists across sessions but tabs do not. |
| Errors / not in tool list | Returns real tabs | **`claude-in-chrome`** (fallback — no screenshots/perf/dialogs, but uses real Chrome with real login). See § Troubleshooting. |
| Errors / not in tool list | Errors | **STOP.** Neither driver works. Run § Troubleshooting → "No driver available". Do not improvise. |

**Profile sanity check** (only when chrome-devtools returns sparse pages):
```bash
PID=$(lsof -i :9222 -sTCP:LISTEN -t | head -1)
[ -n "$PID" ] && ps -p $PID -o command= | grep -oE 'user-data-dir=[^ ]+'
```
- Output ends in `/shared-debug-profile` → MCP-managed Chrome (expected default).
- Output ends in `…/Google/Chrome` → user-launched real-profile Chrome (rare; only if user opted into real-profile attach).
- No output → Nothing on `:9222`. The MCP server isn't connected to a Chrome. See § Troubleshooting.

---

### Step 2 — Verify `SPEQQ_LOCAL_DOPPLER_CONFIG` (HARD GATE)

```bash
echo "SPEQQ_LOCAL_DOPPLER_CONFIG=${SPEQQ_LOCAL_DOPPLER_CONFIG:-UNSET}"
```

If `UNSET` → **throw error and stop**. Do not guess. Do not default to `local`. Ask the user to:
```bash
export SPEQQ_LOCAL_DOPPLER_CONFIG=local_<yourname>
```
…and re-launch Claude Code so child processes inherit the var.

Per `feedback_no_fallbacks_error_instead.md`.

---

### Step 3 — Resolve test credentials from Doppler (secrets — no fallback)

```bash
doppler secrets get TEST_SPEQQ_USER1_EMAIL TEST_SPEQQ_USER1_PASSWORD \
  --config "$SPEQQ_LOCAL_DOPPLER_CONFIG" --project speqq --plain
```
Verified: outputs two lines — email, then password.

**If this command fails for any reason** (Doppler not installed, not logged in, config name wrong, secrets missing in that config) → **stop and surface the error to the user**. Do not read secrets from `docker/.env.dev`. Do not hardcode. Do not improvise.

Failure → user action:
- `doppler: command not found` → `brew install dopplerhq/cli/doppler`
- `You must provide a token` → `doppler login` (opens browser; user action)
- `Could not find requested config` → `SPEQQ_LOCAL_DOPPLER_CONFIG` value is wrong; ask user for correct slug
- `Could not find requested secret: TEST_SPEQQ_USER1_*` → secret missing from that personal config; ask user to add or point to a config that has it

**Never paste the password into the QA log or into chat.**

---

### Step 3b — Local URL

| Env | URL |
|---|---|
| `local` | `http://192.168.0.112:3000` |
| `dev` | `https://dev.speqq.com` |
| `prod` | `https://speqq.com` |

---

### Step 4 — Get the journey list

Verbatim from the user. **Do not improvise journeys.** If none was supplied, ask first.

---

### Step 5 — Create the QA log

Path: `.docs/qa/journey-test-<YYYY-MM-DD>.md`. Format defined in `references/TESTING_PROCESS.md` § QA Log Format. Seed every journey from the user's list as a checkbox.

---

### Step 6 — Open a clean tab, navigate, sign in

Driver-specific:

| Driver | Tools |
|---|---|
| `chrome-devtools` | `new_page(url)` → `take_snapshot` → check post-nav state (see below) |
| `claude-in-chrome` | `tabs_create_mcp` → `navigate(url)` → `read_page` |

After navigation, the snapshot/page tells you which path you're on:

| State | Action |
|---|---|
| Already inside the app (workspace UI visible, URL contains `/workspace`) | Managed profile had a persisted session. Skip sign-in. |
| Marketing landing page with "Log in" link | `click(Log in)` → re-snapshot → if redirected straight into app, skip. Otherwise fill the form. |
| Sign-in form visible | `fill_form` with email + password (retrieve password fresh from `doppler secrets get` — never hardcode) → `click(submit)` → `wait_for("expected post-login text")` |
| SSO / OAuth / CAPTCHA blocks the flow | Pause. Ask user to complete manually. Do not bypass. |

Then drain prior console noise: call `list_console_messages` once and discard, so Journey 1's reads are scoped clean.

---

### Step 7 — Confirm with the user before Journey 1

Print this summary:
```
Driver:   <chrome-devtools | claude-in-chrome>
Env:      <local | dev | prod>
URL:      <full URL>
Account:  <email only — never password>
Journeys: <count>
Log:      .docs/qa/journey-test-YYYY-MM-DD.md
```
Wait for "ready" / "go" / "yes". Don't start Journey 1 until then.

---

## Step 8 — Run the per-journey loop

Follow `references/TESTING_PROCESS.md` § Per-journey loop. One journey at a time. Wait for "next" before advancing. On break → § Diagnose Protocol. Never retry blindly.

---

## Troubleshooting

Each row is verified — the "Diagnostic" command actually runs and returns the listed signal. If a fix requires user action (extension install, Chrome quit), say so and stop.

### "No driver available" (both probes failed)

| Symptom | Diagnostic | Fix |
|---|---|---|
| `mcp__chrome-devtools__*` not in tool list | Check `~/.claude.json` for `mcpServers.chrome-devtools` | Add config block from `references/CHROME_CLI.md` § Install. **Restart Claude Code after editing.** Agent can edit, but cannot self-restart. |
| `mcp__claude-in-chrome__*` not in tool list | Run `/chrome` in Claude Code and select Enable | Or launch with `claude --chrome`. **User action required.** |

### chrome-devtools returns only `about:blank`

Means MCP launched its own managed-profile Chrome and no tabs exist yet. Expected first-run behavior.

| Diagnostic | Fix |
|---|---|
| `lsof -i :9222 -sTCP:LISTEN -t` returns a PID **and** that PID's `--user-data-dir` ends in `/shared-debug-profile` | Just `new_page` + navigate. Sign in inside this Chrome. Login state persists across sessions. |
| Same diagnostic, but `--user-data-dir` is the user's real Chrome profile | User opted into real-profile attach. Tabs may have been closed manually — open a new one. |
| `lsof` returns nothing | The MCP failed to spawn Chrome. Check Node ≥ 20.19, restart Claude Code. |

### Port `:9222` already in use by something else

```bash
PID=$(lsof -i :9222 -sTCP:LISTEN -t | head -1)
ps -p $PID -o command=
```
- If it's a Chrome process you don't recognize, ask the user before killing it.
- If it's not Chrome, the port is genuinely contested. Ask the user — don't `kill -9` blindly.

### claude-in-chrome: "Browser extension is not connected"

Cannot fix from agent side. Print this to the user:
1. Open `chrome://extensions` in Chrome.
2. Confirm "Claude" is enabled.
3. Run `/chrome` in Claude Code → **Reconnect**.
4. If still failing: quit Chrome (Cmd+Q), reopen, then `/chrome` → Reconnect.

### claude-in-chrome: "Receiving end does not exist" mid-session

Service worker went idle. `/chrome` → **Reconnect**. **User action required.**

### `SPEQQ_LOCAL_DOPPLER_CONFIG` unset

**Hard error. Stop the session.** Do not default to `local`. Do not read from `docker/.env.dev`. Print to user:
```bash
export SPEQQ_LOCAL_DOPPLER_CONFIG=local_<yourname>
```
…then re-launch Claude Code so child processes inherit the var.

### `doppler` CLI errors / not authenticated

```bash
doppler whoami 2>&1 | head -3
```
- Not logged in → `doppler login` (opens browser — user action)
- Not installed → `brew install dopplerhq/cli/doppler`

**Do not fall back to `docker/.env.dev` for secrets.** Stop and surface the error.

### Two instances of `chrome-devtools-mcp` registered

Check `~/.claude.json` for both `chrome-devtools` and `chrome-devtools-mcp` entries. They will compete for `:9222`. Keep one. (Speqq's current setup has both — harmless duplicate, but worth noting if you see weird behavior.)

### Chrome version mismatch / CDP errors after a Chrome update

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --version
```
Update Chrome to current stable. The MCP requires it.

---

## Hard Rules

1. **One journey at a time.** Wait for explicit confirmation before advancing.
2. **No fixes without approval.** Diagnose, log, ask. Implementation is a separate task.
3. **No guessing.** Read the code before forming a root cause. Cite `file:line`. (`feedback_no_guessing.md`)
4. **No fallback values.** Surface errors clearly. (`feedback_no_fallbacks_error_instead.md`)
5. **Respect auth boundaries.** Pause for login/CAPTCHA. Never bypass.
6. **Snapshot-first interaction.** Use `take_snapshot` → click by UID. Re-snapshot after navigation or DOM change.
7. **Filter logs.** Console and network output is verbose — always filter by pattern, status, or paginate.
8. **Save heavy artifacts to `tmp/`.** Screenshots, traces, GIFs go to `tmp/qa-*` (already gitignored).
9. **DoD perf gates apply.** When journey involves performance-sensitive flows, run `lighthouse_audit` and `performance_analyze_insight` against CLAUDE.md DoD § 10 thresholds.
10. **No background browsing.** No side-quests outside the journey list without authorization.
