# Chrome CLI — `chrome-devtools-mcp`

Primary browser-automation tool for the `chrome` skill. Direct CDP (Chrome DevTools Protocol) via Puppeteer. Mature, deterministic, full DevTools surface.

## Official Documentation

- **Repo & docs:** https://github.com/ChromeDevTools/chrome-devtools-mcp
- **Tool reference:** https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md
- **Configuration:** https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/configuration.md
- **Troubleshooting:** https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md
- **Chrome DevTools docs:** https://developer.chrome.com/docs/devtools

When the canonical docs and this file disagree, **the canonical docs win**. Re-read them when you suspect drift.

---

## What it is

An MCP server that drives Chrome over CDP. Two run modes:

| Mode | Behavior | Speqq default |
|---|---|---|
| **Browser-URL** (`--browser-url=http://127.0.0.1:9222`) | The MCP itself spawns a Chrome instance bound to `:9222`, using its **own managed user-data-dir** (`~/.cache/chrome-devtools-mcp/shared-debug-profile`). Login state persists in this profile across sessions, but it's separate from your daily Chrome. | ✅ **This is what's configured in `~/.claude.json`.** |
| **Real-profile attach** (advanced) | You launch Chrome yourself with `--remote-debugging-port=9222` AND `--user-data-dir` pointing at your real Chrome profile. The MCP attaches via the same `--browser-url`. You then drive your daily browser. | Not the default. Requires quitting your real Chrome (Cmd+Q) first. Has tradeoffs (every test mutates your real profile state). |

For the standard Speqq QA workflow, **the default Browser-URL mode is what runs**. You sign in to Speqq inside the MCP-managed Chrome the first time; the cookie persists for subsequent sessions.

---

## One-time setup (the default — verified working)

### 1. Confirm the MCP server is installed

Verified: if `mcp__chrome-devtools__*` tools appear in your tool list, you're done. The current Speqq environment already has both `chrome-devtools` and `chrome-devtools-mcp` registered in `~/.claude.json` (harmless duplicate, both pointing at `:9222`).

If neither registration exists, add ONE of them to `~/.claude.json`:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest", "--browser-url=http://127.0.0.1:9222"],
      "env": {}
    }
  }
}
```

**Requirements (verified):** Node.js v20.19+ (this machine: v23.11.0), Chrome current stable, macOS or Linux.

Then restart Claude Code so the MCP loads.

### 2. Verify connectivity

```
mcp__chrome-devtools__list_pages
```

| Result | Meaning | Action |
|---|---|---|
| Returns `about:blank` and/or omnibox-popup pages | MCP spawned its managed Chrome — empty profile, no Speqq session yet | `new_page` → `navigate_page("https://speqq.com" or local URL)` → sign in. Login persists in `~/.cache/chrome-devtools-mcp/shared-debug-profile`. |
| Returns Speqq tab(s) you don't recognise | The managed Chrome has prior persisted login | Reuse if testing the same account. Otherwise sign out and back in. |
| Errors | MCP not connected | See SKILL.md § Troubleshooting → "No driver available". |

### 3. (Optional) Real-profile attach mode

Only if you want to drive your daily Chrome browser:

1. **Quit Chrome completely** (Cmd+Q on macOS — closing windows is not enough).
2. Launch it manually with both flags:
   ```bash
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
     --remote-debugging-port=9222 \
     --user-data-dir="$HOME/Library/Application Support/Google/Chrome"
   ```
3. Now `mcp__chrome-devtools__list_pages` will return your real tabs.

**Caveats:** Every test action mutates your real profile state. Browser updates, sync, and extensions can interfere. Not recommended for routine QA.

---

## Tool Inventory

All tools below are exposed as `mcp__chrome-devtools__<name>`. See the [official tool reference](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md) for parameter schemas — they may evolve.

### Navigation & Tabs
| Tool | Purpose |
|---|---|
| `list_pages` | List all open tabs/pages with IDs. |
| `select_page` | Switch active context to a specific page. |
| `new_page` | Open a new tab. |
| `navigate_page` | Go to URL, reload, or navigate history. |
| `close_page` | Close a tab. |
| `wait_for` | Pause until specific text appears. Use this instead of arbitrary sleeps. |

### Element Interaction (snapshot-first)
| Tool | Purpose |
|---|---|
| `take_snapshot` | Returns the accessibility tree with **UIDs**. **Always call this before clicking/filling.** |
| `click` | Click an element by UID. |
| `fill` | Type into a single input by UID. |
| `fill_form` | Fill multiple fields at once. |
| `hover` | Trigger hover state. |
| `drag` | Drag-and-drop between elements. |
| `press_key` | Send a key or chord (e.g. `"Enter"`, `"Control+C"`). |
| `type_text` | Type into the currently focused element. |
| `upload_file` | Upload a file via a file input. |
| `handle_dialog` | Accept or dismiss `alert` / `confirm` / `prompt`. **Only this tool can handle dialogs.** |

### Inspection & Debugging
| Tool | Purpose |
|---|---|
| `take_screenshot` | Visual capture (full page or element). Save to `tmp/qa-*`. |
| `list_console_messages` | All console output. Filter aggressively. |
| `get_console_message` | Fetch a single message by ID. |
| `list_network_requests` | All network activity. Filter by status/type. |
| `get_network_request` | Full request/response (headers, body, timing) by ID. |
| `evaluate_script` | Run arbitrary JS in page context. Use for DOM/global state inspection. |

### Performance & Profiling
| Tool | Purpose |
|---|---|
| `performance_start_trace` | Begin a performance trace (often `reload=true, autoStop=true`). |
| `performance_stop_trace` | End trace, return data. |
| `performance_analyze_insight` | Extract LCP, CLS, INP, long tasks, layout shifts from the trace. |
| `lighthouse_audit` | Full Lighthouse run (perf, a11y, SEO, best practices). |
| `take_memory_snapshot` | Heap snapshot for memory-leak debugging. |

### Emulation
| Tool | Purpose |
|---|---|
| `resize_page` | Set viewport size. |
| `emulate` | Throttle CPU/network, emulate device, set geolocation. |

---

## Snapshot-first interaction pattern

The deterministic Speqq pattern. Use this every time:

```
1. take_snapshot                  → returns a11y tree with UIDs
2. (find the UID of the target)
3. click(uid="...")               → or fill, hover, press_key, etc.
4. wait_for("expected text")      → confirm next state
5. (re-snapshot only if DOM changed enough that prior UIDs are stale)
```

**Why:** UIDs are stable for the life of a snapshot and point to exactly one element. Description-based clicking guesses; UIDs don't.

**Re-snapshot triggers:**
- Navigation
- Modal/sheet open or close
- Major list update (sort, filter, pagination)
- Any "I expected element X to be there but click failed"

**Don't re-snapshot for every click.** One snapshot covers many subsequent interactions on the same view.

---

## Common pitfalls

1. **Forgot to quit Chrome before launching with `--remote-debugging-port`.** Chrome will silently launch a second instance attached to a different profile. Always `pkill -x "Google Chrome"` first.
2. **Wrong `--user-data-dir`.** If you see no logged-in state, you pointed at a fresh profile dir. Use the real one (see One-time setup).
3. **Using `evaluate_script` when `take_snapshot` would do.** Snapshot is cheaper and more reliable. Reserve `evaluate_script` for global state, framework internals, or DOM queries the a11y tree doesn't expose.
4. **Not filtering console / network.** Output is verbose. Filter by pattern, status, or page through results.
5. **Triggering dialogs without preparing.** `confirm()` and `prompt()` block the page until handled. Plan to call `handle_dialog` immediately after the action that triggers them.
6. **Stale UIDs.** UIDs invalidate after re-render. If a click "didn't take", re-snapshot before retrying.
7. **Performance tracing without `reload=true`.** A trace started mid-page captures only the tail. For LCP and load-time metrics, always trace from a fresh navigation.

---

## Privacy / telemetry knobs (per official docs)

- `--no-usage-statistics` — disable usage telemetry
- `--no-performance-crux` — don't send URLs to Google CrUX API for real-user performance data
- `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS=1` — disable update checks

For the full flag list, run `npx chrome-devtools-mcp@latest --help` or read the [configuration docs](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/configuration.md).
