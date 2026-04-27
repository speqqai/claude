# Product Surfaces

All user-facing pages, API routes, and feature modules in the Speqq codebase.

## Pages

| Route | What Users See | Source |
|-------|---------------|--------|
| `/` | Landing page — product marketing, CTAs | `app/page.tsx` |
| `/pricing` | Pricing plans | `app/pricing/` |
| `/early-access` | Early access signup | `app/early-access/` |
| `/manifesto` | Company manifesto | `app/manifesto/` |
| `/docs` | Documentation site (Nextra/MDX) | `app/docs/`, `content/` |
| `/auth/sign-in` | Sign in | `app/auth/` |
| `/auth/sign-up` | Sign up | `app/auth/` |
| `/auth/forgot-password` | Password reset request | `app/auth/` |
| `/auth/reset-password` | Password reset form | `app/auth/` |
| `/auth/callback` | Auth callback — handles email verification, password recovery, and OAuth code exchange | `app/auth/` |
| `/workspace` | Workspace shell — sidebar, editor, chat | `app/workspace/` |
| `/workspace/settings` | Workspace settings | `app/workspace/settings/` |
| `/cli/authorize` | CLI/TUI authorization flow | `app/cli/` |

## API Routes

### Core Product

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/workspaces` | GET, POST | List/create workspaces |
| `/api/workspace-members` | GET, POST | Workspace membership |
| `/api/documents` | GET, POST | Document CRUD |
| `/api/folders` | GET, POST | Folder management |
| `/api/requirements` | GET, POST | Requirements CRUD |
| `/api/roadmap-items` | GET, POST | Roadmap items |
| `/api/releases` | GET, POST | Release management |
| `/api/shipping-evidence` | GET, POST | Delivery evidence |
| `/api/table-preferences` | GET, PUT | User table preferences |
| `/api/user-profile` | GET, PUT | User profile |

### AI & Context

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/agent` | POST | AI agent chat (streaming) |
| `/api/context` | GET | Context retrieval |
| `/api/context-engine` | POST | Context extraction/ingestion |
| `/api/context-files` | GET, POST | Context file management |
| `/api/workspace-context` | GET | Workspace-scoped context |
| `/api/audio` | POST | Audio transcription |

### Integrations

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/mcp` | POST | MCP server endpoint |
| `/api/mcp-changes` | GET | MCP change polling |
| `/api/github` | POST | GitHub webhook receiver |
| `/api/collab/authorize` | POST | Collab room authorization |
| `/api/collab/access` | POST | Collab access token |
| `/api/billing` | GET, POST | Stripe billing |
| `/api/cli` | POST | CLI auth endpoints |

### Infrastructure

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/health` | GET, HEAD | Service health checks |
| `/api/auth` | GET, POST | Auth session management |
| `/api/early-access` | POST | Early access signup (sends waitlist confirmation email via Resend) |
| `/api/early-access/invite` | POST | Send invite emails to batch of waitlisted users |
| `/api/auth/validate-invite` | POST | Validate invite code for sign-up gating |

## Feature Modules

Located in `features/`. Each module owns its UI components, hooks, and tests.

| Module | What It Owns |
|--------|-------------|
| `workspaceAgentChat/` | Chat UI, message handling, streaming |
| `workspaceSidebar/` | Sidebar navigation, file tree, document list |
| `editor/` | CodeMirror document editor, collaboration bindings |
| `workspace-settings/` | Settings pages and forms |
| `account-management/` | User account, profile settings |
| `cli/` | CLI TUI integration logic |
| `ide-extension/` | IDE extension support |
| `agentContext/` | Agent context retrieval and display |
| `new-user/` | New user onboarding, email templates (confirmation code, password reset, waitlist confirmation, waitlist invite) |
| `manifesto/` | Manifesto page components |
