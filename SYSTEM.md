# System

How Speqq is built, what services run, and how they connect.

## Services

All services run in Docker containers on a shared `speqq-dev` network.

| Service | Container | Local Port | Runtime | Source | Purpose |
|---------|-----------|------------|---------|--------|---------|
| Web App | `speqq-web-server` | 3000 | Node.js 22 / Next.js 16 | `app/`, `components/`, `features/`, `lib/` | Main application — SSR, API routes, UI |
| Collab Server | `speqq-collab-server` | 4444 | Node.js / Yjs | `services/collab-server/` | Real-time document collaboration via WebSocket |
| Context Worker | `speqq-context-worker` | — | Node.js 22 | Same image as web app | Background context extraction jobs |
| Ingest API | `speqq-ingest-api` | 8100 | Python 3.12 / FastAPI | `ingest/` | Knowledge graph ingestion, Graphiti schema/search |
| Neo4j | `speqq-neo4j` | 7474, 7687 | Neo4j | Docker image | Graph database for context engine |
| Postgres | `supabase-db` | 5432, 5433 | PostgreSQL | Supabase stack | Primary database |
| Supabase Gateway | `supabase-kong` | 8000 | Kong | Supabase stack | API gateway for Supabase services |
| Datadog Agent | `speqq-datadog-agent` | — | Datadog Agent 7 | `docker/docker-compose.datadog.dev.yml` | APM traces, log collection |
| Dev Tools | `speqq-dev-tools` | — | Node.js / Python | `docker/speqq-tools/` | DB dump/restore, admin tooling |
| Inbucket Mail | `supabase-mail` | 9000 | Inbucket | Supabase stack | Local email testing — access at `http://<HOST_IP>:9000`, not `localhost`, to stay on the same origin as the app |

> **Note:** Ports above are local Docker mappings. On Render (dev/prod), services use Render's internal networking — ports are assigned by the platform and differ from local.

## How Services Connect

| From | To | Protocol | Purpose |
|------|-----|----------|---------|
| Web App | Postgres | TCP (via Supabase client) | All database reads/writes |
| Web App | Collab Server | WebSocket | Real-time document sync |
| Web App | Ingest API | HTTP | Context graph operations |
| Web App | Neo4j | Bolt | Direct graph queries |
| Web App | Datadog Agent | TCP :8126 | APM trace delivery |
| Collab Server | Postgres | TCP (direct) | Yjs document persistence |
| Collab Server | Web App | HTTP | Room authorization callbacks (`/api/collab/authorize`) |
| Ingest API | Neo4j | Bolt | Graph read/write via Graphiti |
| Browser | Web App | HTTP/HTTPS | Page loads, API calls |
| Browser | Collab Server | WebSocket | Live collaboration |
| Browser | Datadog Cloud | HTTPS | RUM telemetry (client-side) |

## Docker Compose Files

| File | What It Creates |
|------|----------------|
| `docker/docker-compose.supabase.dev.yml` | Supabase stack (Postgres, Auth, Kong, Realtime, Storage) — creates the `speqq-dev` network |
| `docker/docker-compose.neo4j.dev.yml` | Neo4j graph database — joins `speqq-dev` network |
| `docker/docker-compose.dev.yml` | Web app, collab server, context worker, ingest API, dev tools — joins `speqq-dev` network |
| `docker/docker-compose.datadog.dev.yml` | Datadog agent — joins `speqq-dev` network |
| `docker/docker-compose.render.dev.yml` | Render-parity web container for production testing |

## Tech Stack by Service

### Web App (Next.js)

| Category | Packages |
|----------|----------|
| Framework | Next.js 16, React 19, TypeScript (strict) |
| Styling | Tailwind CSS 4 (CSS-first, no tailwind.config.ts), shadcn/ui (Radix + Tailwind) |
| Editor | CodeMirror 6, y-codemirror.next (Yjs binding) |
| Data | SWR (client fetching), @supabase/ssr (Postgres), neo4j-driver |
| AI | Vercel AI SDK (ai, @ai-sdk/react, @ai-sdk/openai) |
| Collaboration | Yjs, y-websocket |
| MCP | @modelcontextprotocol/sdk, mcp-handler |
| Observability | dd-trace (APM), @datadog/browser-rum (RUM), posthog-js (analytics) |
| Billing | Stripe (@stripe/react-stripe-js, stripe) |
| GitHub | @octokit/app, @octokit/core, @octokit/webhooks |
| Docs | Nextra (MDX documentation site at /docs) |
| Icons | lucide-react (exclusively) |
| Dates | date-fns (exclusively) |
| Toasts | Sonner (exclusively, not shadcn Toast) |
| Tables | @tanstack/react-table |
| File Tree | react-arborist |
| Themes | next-themes |
| Markdown | react-markdown (non-streaming), streamdown (streaming, response.tsx only) |
| Layout | react-resizable-panels |
| i18n | i18next, react-i18next |
| Email | Resend (transactional email via React Email templates) |

### Collab Server

| Category | Packages |
|----------|----------|
| Runtime | Node.js / TypeScript |
| Collaboration | Yjs, y-websocket |
| Database | Direct Postgres connection (pg) |
| Auth | Delegates to web app via `/api/collab/authorize` callback |

### Ingest API (Python)

| Category | Packages |
|----------|----------|
| Runtime | Python 3.12 / FastAPI |
| Graph | Graphiti (knowledge graph framework) |
| Database | Neo4j (via Graphiti) |
| LLM | OpenAI (extraction agents) |

## Deployment

| Environment | Platform | Notes |
|-------------|----------|-------|
| Local | Docker (macOS/Linux) | `./speqq-local up` starts everything |
| Development | Render | Deploys from `dev` branch, env synced from Doppler |
| Production | Render | Deploys from `main` branch |

## Key Environment Variables

Env vars are managed via Doppler and synced to `docker/.env.dev` locally. Doppler project: `speqq`, configs: `local`, `dev`, `prod`, `tools`. Observability keys (`DD_API_KEY`, `DD_APP_KEY`) live in the `tools` config — retrieve with `doppler secrets get DD_API_KEY DD_APP_KEY --project speqq --config tools --plain`.

| Variable | Service | Purpose |
|----------|---------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Web App | Supabase API URL |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Web App | Supabase anon/public key |
| `SUPABASE_SECRET_KEY` | Web App | Supabase service role key |
| `NEXT_PUBLIC_APP_URL` | Web App, Collab | App base URL (uses HOST_IP locally). Use for redirect origins in route handlers and middleware — never use `request.url` which resolves to `localhost` inside Docker. |
| `SPEQQ_PUBLIC_APP_URL` | Web App | Server-side app URL. Generated by `genenv.sh` from HOST_IP locally. |
| `NEXT_SERVER_SUPABASE_URL` | Web App | Server-side Supabase URL (used by `lib/supabase/server.ts` and middleware) |
| `COLLAB_SERVER_CALLBACK_SECRET` | Web App, Collab | Shared secret for collab auth callbacks |
| `DD_API_KEY` | Datadog Agent | Datadog API key (in Doppler `tools` config, not `local`) |
| `DD_APP_KEY` | Datadog Agent | Datadog application key (in Doppler `tools` config, not `local`) |
| `NEXT_PUBLIC_DD_RUM_APPLICATION_ID` | Web App | Datadog RUM app ID |
| `NEXT_PUBLIC_DD_RUM_CLIENT_TOKEN` | Web App | Datadog RUM client token |
| `OPENAI_API_KEY` | Web App, Ingest API | OpenAI API access |
| `NEO4J_URI` / `NEO4J_USER` / `NEO4J_PASSWORD` | Web App, Ingest API | Neo4j graph database |
| `STRIPE_SECRET_KEY` | Web App | Stripe billing |
| `NEXT_PUBLIC_POSTHOG_KEY` | Web App | PostHog analytics |
| `RENDER_API_KEY` | CI/CD | Render deployment API |
| `GITHUB_APP_ID` / `GITHUB_APP_PRIVATE_KEY_PEM` | Web App | GitHub App integration |
| `RESEND_API_KEY` | Web App | Resend transactional email |
| `RESEND_FROM_EMAIL` | Web App | Resend sender address (e.g. `Team Speqq <noreply@speqq.com>`) |
| `SPEQQ_INVITE_CODE` | Web App | Invite code required for sign-up gating |
