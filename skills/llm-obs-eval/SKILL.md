---
name: llm-obs-eval
description: >-
  Run agent evaluations via Datadog LLM Observability Experiments.
  Use when testing agent tool selection, verifying cost/token tracking,
  running experiments, querying results, or when asked to "run eval",
  "check datadog", "verify llm obs", "test agent turns", or "check cost".
allowed-tools: Read, Grep, Glob, Bash
---

# Agent Evaluation Runbook

This runbook covers how to run evaluations against the Speqq agent using
Datadog's LLM Observability Experiments workflow, and how to query results
programmatically. Both agents and humans should be able to follow this end-to-end.

## How It Works

```
Python experiment runner (scripts/datadog/experiment.py)
  │
  ├─ Creates DD Dataset with test records (prompts + expected behavior)
  ├─ Creates DD Experiment linked to the dataset
  ├─ For each record, runs a task that:
  │     ├─ Creates throwaway Supabase user + workspace
  │     ├─ Calls the live agent API (POST /api/agent/turns)
  │     ├─ Parses SSE stream → response text, tool calls, token usage
  │     ├─ Creates a child LLM span (model + tokens → DD computes cost)
  │     └─ Returns structured JSON output
  ├─ Runs evaluators against each output (tool_selection, turn_completed)
  ├─ Cleans up test data
  └─ Prints experiment URL

All data lands in DD under project "speqq":
  DD Experiments UI → experiment → records → traces → spans
  Export API        → spans/events (with tokens, cost, evaluations)
```

## Step 1: Load Keys

Always load from Docker containers, never from .env files (Supabase keys go stale on restart).

```bash
export EVAL_SERVICE_ROLE_KEY=$(docker exec supabase-kong printenv SUPABASE_SERVICE_KEY)
export EVAL_ANON_KEY=$(docker exec supabase-kong printenv SUPABASE_ANON_KEY)
export DD_API_KEY=$(docker exec speqq-web-server printenv DD_API_KEY)
export DD_APP_KEY=$(docker exec speqq-web-server printenv DD_APP_KEY)
export OPENAI_API_KEY=$(docker exec speqq-web-server printenv OPENAI_API_KEY)
```

Verify: `echo "SRK=${#EVAL_SERVICE_ROLE_KEY} ANON=${#EVAL_ANON_KEY} API=${#DD_API_KEY} APP=${#DD_APP_KEY} OAI=${#OPENAI_API_KEY}"`

All five must be non-zero. If any are empty, the container is down — check `docker ps`.

## Step 2: Verify Containers

```bash
docker ps --filter "name=speqq-web-server" --format "{{.Names}} {{.Status}}"
docker ps --filter "name=supabase-kong" --format "{{.Names}} {{.Status}}"
```

Both must show "Up". The web server must have your latest code — restart it after code changes:
```bash
docker restart speqq-web-server && sleep 15
```

## Step 3: Run the Experiment

```bash
python3 scripts/datadog/experiment.py
```

This runs all scenarios (greeting, product_question, feature_question), creates evaluations,
and prints an experiment URL like:
```
Experiment URL: https://app.datadoghq.com/llm/experiments/<id>
```

Open that URL to see results in DD's Experiments UI.

### What the experiment tests

| Scenario | Prompt | Expected tools | Evaluators |
|----------|--------|---------------|------------|
| greeting | "Hello, how are you?" | None | turn_completed, tool_selection (no_tools_expected) |
| product_question | "What is the architecture of my product?" | search_product_context | turn_completed, tool_selection (correct/incorrect) |
| feature_question | "What features does my product have?" | search_product_context | turn_completed, tool_selection (correct/incorrect) |

### What DD captures per record

| Data | Source | How |
|------|--------|-----|
| Input prompt | Experiment task | Passed as input_data |
| Output response | Experiment task | Returned from task |
| Tools called | SSE stream | `tool.updated` events |
| Token usage | SSE stream | `token.usage` event (emitted by route.ts) |
| Estimated cost | DD auto-calculation | From LLM span: model + provider + tokens |
| Model name | Experiment config + LLM span | `gpt-5.3-codex` etc. |
| Duration | DD | Per-span timing |
| Evaluations | Experiment evaluators | tool_selection (pass/fail), turn_completed (pass/fail) |
| Trace/span tree | DD | Experiment task span → child LLM span |

## Step 4: Query Results Programmatically

### List experiments

```bash
curl -s "https://api.datadoghq.com/api/v2/llm-obs/v1/experiments" \
  -H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

### Get experiment spans via Export API

This is the primary way to query per-record data (tokens, cost, model, input/output):

```bash
curl -s "https://api.datadoghq.com/api/v2/llm-obs/v1/spans/events?\
filter%5Bquery%5D=%40ml_app%3Aspeqq&\
filter%5Bfrom%5D=$(date -u -v-1H +%Y-%m-%dT%H:%M:%SZ)&\
filter%5Bto%5D=$(date -u +%Y-%m-%dT%H:%M:%SZ)&\
page%5Blimit%5D=50&sort=-timestamp" \
  -H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY"
```

Filter by experiment:
```
filter[query]=@ml_app:speqq @experiment_id:<id>
```

Each span includes:
- `attributes.metrics.input_tokens`, `output_tokens`, `total_tokens`
- `attributes.metrics.estimated_total_cost` (nanodollars, divide by 1e9 for USD)
- `attributes.metrics.estimated_input_cost`, `estimated_output_cost`
- `attributes.model_name`, `attributes.model_provider`
- `attributes.input`, `attributes.output` (full content)
- `attributes.span_kind` (`llm`, `experiment`, etc.)
- `attributes.tags` (experiment_id, dataset_name, run_id, etc.)

### Summarize cost/tokens for an experiment

```bash
curl -s "https://api.datadoghq.com/api/v2/llm-obs/v1/spans/events?\
filter%5Bquery%5D=%40ml_app%3Aspeqq&\
filter%5Bfrom%5D=<ISO_FROM>&filter%5Bto%5D=<ISO_TO>&\
page%5Blimit%5D=50&sort=-timestamp" \
  -H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -c "
import json, sys
data = json.load(sys.stdin)
spans = data.get('data', [])
exp_id = '<EXPERIMENT_ID>'
total_cost = 0
total_tokens = 0
for s in spans:
    a = s['attributes']
    tags = a.get('tags', [])
    if not any(exp_id in t for t in tags):
        continue
    if a.get('span_kind') != 'llm':
        continue
    m = a.get('metrics', {})
    cost = m.get('estimated_total_cost', 0) / 1e9
    tokens = m.get('total_tokens', 0)
    total_cost += cost
    total_tokens += tokens
    inp = (a.get('input', {}).get('value', ''))[:60]
    print(f'  {inp:60s}  tokens={tokens:>6}  cost=\${cost:.6f}')
print(f'\n  TOTAL:  tokens={total_tokens:>6}  cost=\${total_cost:.6f}')
"
```

### Aggregated metrics (time-window, not per-experiment)

```bash
# Cost in last hour
curl -s "https://api.datadoghq.com/api/v1/query?\
from=$(( $(date +%s) - 3600 ))&to=$(date +%s)&\
query=sum:ml_obs.span.llm.total.cost{ml_app:speqq}.as_count()" \
  -H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY"
```

## Step 5: Adding New Scenarios

Edit `scripts/datadog/experiment.py`. Add a new record to the `records` list in `create_dataset()`:

```python
{
    "input_data": {
        "prompt": "Create a PRD for a login page",
        "scenario": "prd_creation",
        "expect_tools": True,
        "expected_tool_names": ["create_prd"],
    },
    "expected_output": None,
    "metadata": {
        "test_type": "tool_selection",
        "expect_tools": "true",
        "expected_tools": "create_prd",
        "description": "PRD creation — should trigger create_prd tool",
    },
}
```

The existing evaluators (`tool_selection_evaluator`, `turn_completed_evaluator`) handle any
scenario automatically based on `expect_tools` and `expected_tool_names` in the input.

## Step 6: Adding New Evaluators

Add a function with signature `(input_data, output_data, expected_output) -> EvaluatorResult`:

```python
from ddtrace.llmobs._experiment import EvaluatorResult

def response_quality_evaluator(input_data, output_data, expected_output):
    response = output_data.get("response", "")
    has_content = len(response) > 50
    return EvaluatorResult(
        value="sufficient" if has_content else "too_short",
        assessment="pass" if has_content else "fail",
        reasoning=f"Response length: {len(response)} chars",
    )
```

Then add it to the `evaluators` list in `LLMObs.experiment(...)`.

## Key Facts

1. **Experiments are Python-only.** The `ddtrace` Python SDK (`>=3.14`) has `LLMObs.experiment()`. The Node.js dd-trace SDK does NOT support experiments.

2. **Cost requires an LLM span.** DD computes cost from `kind: "llm"` + `model_name` + `model_provider` + token metrics. The experiment task creates a child LLM span for this.

3. **Token usage comes from the SSE stream.** Route.ts emits a `token.usage` event with `inputTokens`, `outputTokens`, `totalTokens` before the terminal event.

4. **Cost is in nanodollars.** Divide by 1e9 for USD.

5. **Keys come from Docker containers.** Never trust `.env` files — Supabase keys go stale on restart.

6. **Export API is the query tool.** `GET /api/v2/llm-obs/v1/spans/events` returns individual spans with full metrics. Filter by `@ml_app:speqq`, experiment tags, time range.

7. **DD project is `speqq`** (id: `7a39195b-485b-4aec-b23c-86f4cd36d07d`). Always pass `project_name="speqq"` to `create_dataset()` and `experiment()`.

8. **The experiment tests the real agent.** The task calls `/api/agent/turns` — same API the UI uses. Auth, MCP tools, workspace context, tool execution all run for real.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `ERROR: DD_API_KEY is not set` | Keys not loaded | Run Step 1 |
| `failed to send, dropping 1 traces` | No local DD agent (harmless) | Ignore — experiment uses agentless mode |
| Cost N/A in DD Experiments UI | LLM span missing or model not recognized | Check `cost_estimate_status` tag on LLM span via Export API |
| Token count N/A | `token.usage` SSE event not emitted | Check route.ts has the `token.usage` emit block; restart web server |
| Experiment in wrong project | Missing `project_name="speqq"` | Pass it to both `create_dataset()` and `experiment()` |
| `AttributeError: LLMObs has no attribute` | Wrong ddtrace version | `pip3 install "ddtrace>=3.14"` |
| Agent turn fails 401 | Stale Supabase keys | Re-export keys from Kong (Step 1) |
| Agent turn fails 500 | Web server crashed or not rebuilt | `docker restart speqq-web-server` |

## Files

| File | Purpose |
|------|---------|
| `scripts/datadog/experiment.py` | Experiment runner — the main entry point |
| `app/api/agent/turns/route.ts` | Agent turn handler with LLM Obs instrumentation + token.usage SSE event |
| `.claude/skills/llm-obs-eval/SKILL.md` | This runbook |
| `.claude/skills/llm-obs-eval/references/keys.md` | Key sources and loading commands |
| `.claude/skills/llm-obs-eval/references/data.md` | Complete data inventory and DD API reference |
