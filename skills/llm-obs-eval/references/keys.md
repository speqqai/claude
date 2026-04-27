# Keys

How to get every key needed for eval runs.

## Source: Docker containers (always use these, never .env files)

| Key | Command | Used for |
|-----|---------|----------|
| EVAL_SERVICE_ROLE_KEY | `docker exec supabase-kong printenv SUPABASE_SERVICE_KEY` | Supabase admin API (create users, workspaces) |
| EVAL_ANON_KEY | `docker exec supabase-kong printenv SUPABASE_ANON_KEY` | Supabase sign-in (get auth tokens) |
| DD_API_KEY | `docker exec speqq-web-server printenv DD_API_KEY` | DD API authentication, experiment submission |
| DD_APP_KEY | `docker exec speqq-web-server printenv DD_APP_KEY` | DD API queries (Export API, metrics, experiments) |
| OPENAI_API_KEY | `docker exec speqq-web-server printenv OPENAI_API_KEY` | Not used by experiment directly, but needed in env for ddtrace auto-instrumentation |

## Load all at once

```bash
export EVAL_SERVICE_ROLE_KEY=$(docker exec supabase-kong printenv SUPABASE_SERVICE_KEY)
export EVAL_ANON_KEY=$(docker exec supabase-kong printenv SUPABASE_ANON_KEY)
export DD_API_KEY=$(docker exec speqq-web-server printenv DD_API_KEY)
export DD_APP_KEY=$(docker exec speqq-web-server printenv DD_APP_KEY)
export OPENAI_API_KEY=$(docker exec speqq-web-server printenv OPENAI_API_KEY)
```

## Verify

```bash
echo "SRK=${#EVAL_SERVICE_ROLE_KEY} ANON=${#EVAL_ANON_KEY} API=${#DD_API_KEY} APP=${#DD_APP_KEY} OAI=${#OPENAI_API_KEY}"
```

All five must be non-zero. If any are empty, the container is down — check `docker ps`.

## Why not .env files?

Supabase keys go stale when containers restart. The container environment is the only reliable source.
