# Data Directory

Every piece of data available for eval, where to get it, and how.

All DD API requests use these auth headers:
```
-H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY"
```

---

## Per-Record Data (from DD Export API)

Source: `GET https://api.datadoghq.com/api/v2/llm-obs/v1/spans/events`

This is the primary query tool for eval results. Each span contains:

| Data | Field path | Notes |
|------|-----------|-------|
| Input prompt | `attributes.input.value` | Full user message |
| Output response | `attributes.output.value` | Full assistant response |
| Input tokens | `attributes.metrics.input_tokens` | |
| Output tokens | `attributes.metrics.output_tokens` | |
| Total tokens | `attributes.metrics.total_tokens` | |
| Est. input cost | `attributes.metrics.estimated_input_cost` | Nanodollars (÷ 1e9 = USD) |
| Est. output cost | `attributes.metrics.estimated_output_cost` | Nanodollars |
| Est. total cost | `attributes.metrics.estimated_total_cost` | Nanodollars |
| Cost estimate status | tag `cost_estimate_status` | `success` or `unsupported_model` |
| Model name | `attributes.model_name` | e.g. `gpt-5.3-codex` |
| Model provider | `attributes.model_provider` | e.g. `openai` |
| Span kind | `attributes.span_kind` | `llm`, `experiment`, `agent`, `tool` |
| Duration | `attributes.duration` | Nanoseconds |
| Trace ID | `attributes.trace_id` | Links related spans |
| Span ID | `attributes.span_id` | Unique span identifier |
| Parent span | `attributes.parent_id` | For span tree navigation |
| Experiment ID | tag `experiment_id` | Filter by experiment |
| Dataset name | tag `dataset_name` | Filter by dataset |
| Run ID | tag `run_id` | Filter by run |

### Query examples

```bash
# All spans for ml_app:speqq in last hour
filter[query]=@ml_app:speqq
filter[from]=2026-04-25T15:00:00Z
filter[to]=2026-04-25T16:00:00Z
page[limit]=50
sort=-timestamp

# LLM spans only (have cost/token data)
filter[query]=@ml_app:speqq @span_kind:llm

# Filter by experiment (use tag in query)
# Note: filter in Python after fetching, matching experiment_id in tags array
```

---

## Per-Record Data (from SSE stream during experiment)

Source: `scripts/datadog/experiment.py` task function parsing `/api/agent/turns` SSE

| Data | SSE event | Notes |
|------|-----------|-------|
| Turn ID | Header `x-agent-turn-id` | Unique per turn |
| Trace ID | `trace.context` event | LLM Obs trace ID from `exportSpan()` |
| Tools called | `tool.updated` events (status=completed) | Tool name in `label`, prefixed `speqq:` |
| Response text | `message.delta` events concatenated | Full assistant response |
| Token usage | `token.usage` event | `inputTokens`, `outputTokens`, `totalTokens`, `model` |
| Turn status | `turn.completed` or `turn.failed` | |

---

## Experiment Management (DD REST API)

Source: `https://api.datadoghq.com/api/v2/llm-obs/v1/...`

| Endpoint | Method | What it does |
|----------|--------|-------------|
| `/experiments` | GET | List experiments (filter by project_id) |
| `/experiments` | POST | Create experiment |
| `/projects` | GET | List projects |
| `/projects` | POST | Create project |
| `/{project_id}/datasets` | GET | List datasets |
| `/{project_id}/datasets` | POST | Create dataset |
| `/{project_id}/datasets/{id}/records` | GET | List records |
| `/{project_id}/datasets/{id}/records` | POST | Append records |
| `/annotation-queues` | GET | List annotation queues |
| `/spans/events` | GET | **Export API** — query individual spans with full data |

### DD Project

- Name: `speqq`
- ID: `7a39195b-485b-4aec-b23c-86f4cd36d07d`
- ml_app: `speqq`

---

## Aggregated Metrics (time-window, not per-experiment)

Source: `GET https://api.datadoghq.com/api/v1/query`

These are aggregate metrics across all traffic, not per-experiment. Use narrow time windows to isolate.

| Metric | Query |
|--------|-------|
| Total cost | `sum:ml_obs.span.llm.total.cost{ml_app:speqq}.as_count()` |
| Input cost | `sum:ml_obs.span.llm.input.cost{ml_app:speqq}.as_count()` |
| Output cost | `sum:ml_obs.span.llm.output.cost{ml_app:speqq}.as_count()` |
| Total tokens | `sum:ml_obs.span.llm.total.tokens{ml_app:speqq}.as_count()` |
| Input tokens | `sum:ml_obs.span.llm.input.tokens{ml_app:speqq}.as_count()` |
| Output tokens | `sum:ml_obs.span.llm.output.tokens{ml_app:speqq}.as_count()` |
| Span counts | `sum:ml_obs.span{ml_app:speqq}by{span_kind}.as_count()` |

Cost values are in **nanodollars** (divide by 1e9 for USD).

Filterable tags: `ml_app`, `span_kind`, `model_name`, `model_provider`.
NOT filterable: `turn_id`, `span_name`, `experiment_id`.

---

## Python SDK Methods (experiment runner)

Source: `ddtrace.llmobs.LLMObs` (Python, ddtrace >= 3.14)

| Method | What it does |
|--------|-------------|
| `enable()` | Initialize LLM Obs with api_key, app_key, site, ml_app |
| `create_dataset()` | Create dataset with records (input_data, expected_output, metadata) |
| `experiment()` | Create experiment with task, dataset, evaluators, config |
| `experiment.run()` | Execute the experiment, returns ExperimentResult |
| `llm()` | Context manager — create an LLM span (needed for cost tracking) |
| `annotate()` | Set input/output/metrics on current span |
| `flush()` | Force-flush pending data to DD |

---

## Node.js SDK Methods (route.ts instrumentation)

Source: `tracer.llmobs.*` on speqq-web-server (dd-trace Node)

| Method | What it does |
|--------|-------------|
| `enable()` | Enable LLM Obs with mlApp |
| `trace()` | Create LLM Obs span (agent/tool/llm) |
| `wrap()` | Wrap function with span |
| `annotate()` | Add input/output/metrics to active span |
| `exportSpan()` | Returns `{ traceId, spanId }` for active span |
| `flush()` | Force-flush pending spans |

Note: Node SDK does NOT support experiments, datasets, or evaluators.
