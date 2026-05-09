# Claude in Chrome — Browser Extension MCP

Anthropic's first-party Chrome extension. Connects Claude Code to **your real Chrome browser** over native messaging. Inherits your logged-in profile.

**For Speqq journey QA, this is the fallback option.** `chrome-devtools-mcp` (see [`CHROME_CLI.md`](CHROME_CLI.md)) is primary because it has the full DevTools surface — screenshots, performance traces, dialog handling — that this extension lacks.

## Official Documentation

- **Claude Code + Chrome guide:** https://code.claude.com/docs/en/chrome
- **Getting started with Claude in Chrome:** https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome
- **Claude in Chrome extension (Chrome Web Store):** https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn
- **VS Code integration:** https://code.claude.com/docs/en/vs-code#automate-browser-tasks-with-chrome

When the canonical docs and this file disagree, **the canonical docs win.**

---

## What it is

A Chrome extension + native messaging host that lets Claude Code drive your everyday browser. Uses Chrome's `debugger` permission to issue CDP commands. Browser actions appear in real time in a visible Chrome window. Login state is shared — any site you're already signed into just works.

**Status:** Beta. Per official docs:
- Requires Claude Code v2.0.73+ and extension v1.0.36+
- Works on Google Chrome and Microsoft Edge only
- **Not supported** on Brave, Arc, other Chromium browsers, or WSL
- Requires a direct Anthropic plan (Pro, Max, Team, Enterprise) — not third-party providers

---

## Setup

### 1. Install the extension
From the [Chrome Web Store link above](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn). Sign in with your Claude account.

### 2. Enable in Claude Code
- One-shot: launch Claude Code with `claude --chrome`
- Toggle in session: run `/chrome` and select **Enable**
- Always-on: run `/chrome` → **Enabled by default** (note: increases context usage since browser tools are always loaded)

### 3. Verify
Run `/chrome` to see connection status. Or call `mcp__claude-in-chrome__tabs_context_mcp` — it should return your real tabs.

### 4. Native messaging host (auto-installed on first enable)

Per official docs, configuration files are placed at:

| Browser | OS | Path |
|---|---|---|
| Chrome | macOS | `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json` |
| Chrome | Linux | `~/.config/google-chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json` |
| Chrome | Windows | `HKCU\Software\Google\Chrome\NativeMessagingHosts\` (registry) |
| Edge | macOS | `~/Library/Application Support/Microsoft Edge/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json` |
| Edge | Linux | `~/.config/microsoft-edge/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json` |

If the extension isn't detected after first enable, **restart Chrome** so it picks up the new config.

---

## Tool Inventory

All tools below are exposed as `mcp__claude-in-chrome__<name>`. To see the canonical list at runtime, run `/mcp` in Claude Code and select `claude-in-chrome`.

### Tabs & Navigation
| Tool | Purpose |
|---|---|
| `tabs_context_mcp` | List current tabs in your real Chrome. **Call this first** at session start. |
| `tabs_create_mcp` | Open a new tab. |
| `navigate` | Go to a URL. |
| `switch_browser` | Switch between Chrome and Edge. |

### Read Page Content
| Tool | Purpose |
|---|---|
| `read_page` | Read the current page (text + structure). |
| `get_page_text` | Plain text of the page. |
| `find` | Locate an element by description. |

### Interaction
| Tool | Purpose |
|---|---|
| `computer` | Vision/description-based click and keyboard control. |
| `form_input` | Fill form fields. |

### Console & Network
| Tool | Purpose |
|---|---|
| `read_console_messages` | Bulk read of console output. Pass a regex `pattern` to filter (logs are verbose). |
| `read_network_requests` | Bulk read of network activity. |

### JavaScript
| Tool | Purpose |
|---|---|
| `javascript_tool` | Run arbitrary JS in the active page. |

### Recording
| Tool | Purpose |
|---|---|
| `gif_creator` | Record an interaction sequence as a GIF. Capture extra frames before/after for smooth playback. Name files meaningfully. |
| `upload_image` | Upload an image to the model. |

### Misc
| Tool | Purpose |
|---|---|
| `resize_window` | Change window size. |
| `shortcuts_list` | List configured browser shortcuts. |
| `shortcuts_execute` | Execute a configured shortcut. |
| `update_plan` | Update Claude's task plan in Chrome. |

---

## Capability Gaps vs `chrome-devtools-mcp`

These are the reasons this extension is **not** the primary tool for Speqq QA:

| Capability | `claude-in-chrome` | `chrome-devtools-mcp` |
|---|---|---|
| Screenshots (`take_screenshot`) | ❌ none | ✅ |
| Performance traces (LCP/CLS/INP) | ❌ none | ✅ |
| Lighthouse audit | ❌ none | ✅ |
| Memory/heap snapshots | ❌ none | ✅ |
| CPU / network throttling | ❌ (only `resize_window`) | ✅ `emulate` |
| **Handle JS dialogs** (`alert`/`confirm`/`prompt`) | ❌ — **dialogs FREEZE the extension** | ✅ `handle_dialog` |
| Snapshot+UID deterministic clicking | ❌ description/vision only | ✅ |
| Long-running session stability | ⚠️ service worker can go idle | ✅ direct CDP |

CLAUDE.md DoD § 10 explicitly requires LCP/CLS/INP/TTFB measurements and Lighthouse ≥ 90. **`claude-in-chrome` cannot produce those numbers.** If a journey needs perf evidence, switch to `chrome-devtools-mcp`.

---

## Stability gotchas (per official docs)

1. **Browser modal dialogs (alert/confirm/prompt) FREEZE the extension.** It will stop receiving any subsequent commands until you manually dismiss the dialog in the browser. Mitigations:
   - Avoid clicking elements that may trigger dialogs.
   - Warn the user first if you must.
   - Use `console.log` for debugging, then read it via `read_console_messages` instead of triggering visible dialogs.
   - If a dialog is already on screen, `javascript_tool` may help check / dismiss it before proceeding.
2. **Service worker can go idle during long sessions** → connection breaks. Run `/chrome` → **Reconnect extension** to restore.
3. **Login pages and CAPTCHAs pause the agent.** It will ask the user to handle them manually. Don't try to bypass.
4. **Site permissions** are inherited from the extension settings. Manage allow/block per-site in `chrome://extensions` → Claude → Details.
5. **Windows named-pipe conflict (`EADDRINUSE`)** if multiple Claude Code sessions try to use Chrome at once. Close other sessions and restart.
6. **Common errors:**

| Error | Cause | Fix |
|---|---|---|
| `Browser extension is not connected` | Native messaging host can't reach extension | Restart Chrome and Claude Code, run `/chrome` → Reconnect |
| `Extension not detected` | Extension disabled / not installed | Enable in `chrome://extensions` |
| `No tab available` | Acted before a tab was ready | Create a new tab and retry |
| `Receiving end does not exist` | Service worker idle | `/chrome` → Reconnect |

---

## When to use it (rare for Speqq QA)

Pick `claude-in-chrome` over `chrome-devtools-mcp` only when:

1. **You explicitly want to drive the user's everyday Chrome window** — they want to watch in real time and take over mid-flow.
2. **You're not in attach-mode for `chrome-devtools-mcp`** and can't easily set it up — and the journey doesn't need screenshots, perf, or dialog handling.
3. **You need GIF recording for a stakeholder demo.** This is the only tool with `gif_creator` exposed in this environment.

For everything else, prefer `chrome-devtools-mcp` per [`CHROME_CLI.md`](CHROME_CLI.md).

---

## Best practices (when used)

1. **Filter console output** with `pattern` regex. Don't dump everything.
   ```
   read_console_messages(pattern: "\\[Speqq\\]|Error|TypeError|Warning")
   ```
2. **Capture frames before and after** when using `gif_creator` for smooth playback. Name the file meaningfully (`qa-journey-4-create-prd.gif`).
3. **Stay focused.** Don't explore tangentially. If 2-3 tool calls fail, stop and ask the user — don't loop.
4. **Don't reuse tab IDs across sessions.** Always call `tabs_context_mcp` fresh; use `tabs_create_mcp` for a clean tab unless the user explicitly says reuse.
5. **At session start, always:**
   - Call `tabs_context_mcp` to see real tabs.
   - Create a fresh tab unless reusing was authorized.
6. **If browser commands stop working:** check for a blocking dialog → ask the user to dismiss → retry.
