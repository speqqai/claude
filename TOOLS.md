# Tools

CLI tools and dev commands available in this environment. The dev server runs in Docker — some tools run on the host, some inside the container.

## Host CLI Tools

| Tool | Version | What It Does | Common Usage |
|------|---------|-------------|--------------|
| `docker` | 28.x | Container management | `docker ps`, `docker exec`, `docker logs` |
| `gh` | 2.78+ | GitHub CLI — PRs, issues, API | `gh pr create`, `gh pr list`, `gh api` |
| `git` | 2.39+ | Version control | Standard git workflow |
| `node` | 23.x | JavaScript runtime (host) | Type-checking, host scripts |
| `npm` | 10.x | Package manager | `npm install`, `npm run <script>` |
| `supabase` | 2.30+ | Supabase CLI | Migrations, db diff, type generation |
| `doppler` | 3.75+ | Doppler CLI — secrets management | `doppler login`, env sync via `speqq-local env-check` |
| `render` | 2.9+ | Render CLI — deploy management | `render deploys list`, `render services list` |
| `curl` | 8.7+ | HTTP requests | Health checks, API testing |
| `jq` | 1.6+ | JSON processing | Parsing API responses |
| `python3` | 3.9 (host) | Python runtime (host) | Scripts only — app Python runs in container |

## Container CLI Tools (inside speqq-web-server)

Run via `./speqq-local run <command>` or `./speqq-local sh` to open a shell.

| Tool | Version | What It Does | Common Usage |
|------|---------|-------------|--------------|
| `node` | 22.x | JavaScript runtime (app) | App server, scripts, workers |
| `npm` | 10.x | Package manager | `npm install`, `npm run <script>` |
| `npx` | 10.x | Run package binaries | `npx tsc`, `npx vitest`, `npx playwright` |
| `python3` | 3.12 | Python runtime | CLI TUI, ingest service, tests |
| `pytest` | via container | Python test runner | `./speqq-local pytest` |
| `ruff` | via container | Python linter | `./speqq-local ruff` |
| `playwright` | 1.58+ | E2E browser testing | `./speqq-local playwright` |
| `vitest` | via container | Unit/integration tests | `./speqq-local vitest` |

## Observability Access

Keys live in Doppler `tools` config, not in `docker/.env.dev`. Retrieve with `doppler secrets get DD_API_KEY DD_APP_KEY --project speqq --config tools --plain`.

| What You Want | How To Get It |
|---------------|--------------|
| Server-side route latency (p50/p95) | Query APM traces: `api.datadoghq.com/api/v2/spans/events/search`, filter `service:speqq-web operation_name:next.request` |
| Web vitals (TTFB, FCP, LCP, CLS, INP) | Query RUM views: `api.datadoghq.com/api/v2/rum/events/search`, filter `service:speqq-web @type:view` |
| Application logs | Query logs: `api.datadoghq.com/api/v2/logs/events/search`, filter `service:speqq-web` |
| Service health status | `curl http://localhost:3000/api/health \| jq .` |
| APM agent status | `docker exec speqq-datadog-agent agent status` |
| Landing page PostHog metrics | `npm run metrics:landing` |
| Workspace PostHog metrics | `npm run metrics:workspace` |

## npm Scripts

Run from host with `npm run <script>` or in container with `./speqq-local run npm run <script>`.

| Script | What It Does |
|--------|-------------|
| `dev` | Start Next.js dev server |
| `build` | Production build |
| `start` | Start production server |
| `lint` | ESLint on app/components/features/lib |
| `lint:strict` | ESLint with zero warnings |
| `lint:fix` | ESLint auto-fix |
| `lint:css` | Stylelint on CSS |
| `format:check` | Prettier check |
| `format:write` | Prettier auto-format |
| `quality` | Full quality gate (lint:strict + css + format + deadcode) |
| `deadcode` | Knip — find unused exports/files |
| `test:sidebar` | Vitest unit tests |
| `test:e2e:workspace-agent-chat` | Playwright chat tests |
| `test:e2e:workspace-settings` | Playwright settings tests |
| `test:e2e:workspace-sidebar` | Playwright sidebar tests |
| `test:e2e:context-engine` | Playwright context engine tests |
| `metrics:landing` | PostHog landing page metrics (p50/p95) |
| `metrics:workspace` | PostHog workspace metrics |
| `smoke:landing` | Landing page smoke test |
| `smoke:codex` | Codex end-to-end smoke test |
| `seed:dev` | Run dev sample-data seed |
| `seed:dev:plan` | Preview seed plan without applying |
| `seed:dev:verify` | Verify seed data exists |
| `seed:dev:reset` | Remove seeded data |
| `worker:context` | Start context extraction worker |

## Test tools

Reusable utilities for local testing. Run all commands inside the `speqq-web-server` container via `docker exec speqq-web-server node -e "..."`.

| Tool | What it does | How to run |
|---|---|---|
| List auth users | Get user IDs and emails from the local DB | `admin.auth.admin.listUsers()` using `SUPABASE_SECRET_KEY` |
| Create authenticated session | Get a valid Supabase session (access + refresh token) for any user without knowing their password | `admin.auth.admin.generateLink({ type: 'magiclink', email })` then `anon.auth.verifyOtp({ type: 'magiclink', token_hash })` |
| Create MCP PAT token | Generate a `spq_pat_` token, hash with `MCP_TOKEN_PEPPER`, encrypt session tokens with `MCP_TOKEN_ENCRYPTION_KEY` (base64, AES-256-GCM), insert into `mcp_tokens` via service role client | Combine authenticated session + token generation + service role insert |
| Initialize MCP session | Open an MCP session against the local server | `POST http://<HOST_IP>:3000/api/mcp` with `initialize` method and bearer token. Session ID returned in `mcp-session-id` response header. |
| List MCP tools | Verify all tools are registered | `POST` with `tools/list` method and `mcp-session-id` header |
| Call MCP tool | Execute a specific tool | `POST` with `tools/call` method, tool name, and arguments |
| Verify Datadog spans | Confirm tool calls appear in APM | Filter Datadog APM by `service:speqq-web`. Spans show `mcp.auth`, `mcp.session`, and tool name as operation. |
