# 13 — Verbose Logging Standard (zerolog)

Verbosity is a first-class project property (D-013). This doc defines exactly what verbose means so every developer and every agent produces the same shape. Library: **`github.com/rs/zerolog` only** — no `hlog`, no `alice` (D-018/D-019: there is no HTTP server in the default stack). Verified against Context7 `/rs/zerolog`.

## 13.1 Logger setup (contract, not code to ship yet)
- **Production:** structured JSON to `os.Stdout` via `zerolog.New(os.Stdout).With().Timestamp().Logger()` plus core fields `role`, `host`, `version`, `env`.
- **Development:** `zerolog.ConsoleWriter{Out: os.Stderr, TimeFormat: time.RFC3339}` for human-readable verbose output.
- **Levels:** `debug` in dev (default), `info` in prod; override with `LOG_LEVEL` env. `trace` level available for per-token debugging.
- **Per-run sub-logger:** `log.With().Str("run_id", id).Str("agent_id", id).Str("trace_id", id).Logger()` — every component gets a sub-logger with its context baked in (zerolog sub-loggers are the documented pattern).
- **No HTTP access logging:** there is no HTTP server (D-018). All events originate from bound Go method calls (frontend, CLI runner, or internal agents); correlation comes from explicit `run_id` / `trace_id` / `call_id` fields, not middleware.

## 13.2 Mandatory fields on every event
| Field | Source | Why |
|---|---|---|
| `ts` | `.Timestamp()` | ordering / replay |
| `level` | zerolog | severity |
| `event` | constant string | machine-readable event name |
| `run_id` | uuid | correlate whole run |
| `agent_id` | manager/research/planner/executor/verifier | attribute actions |
| `trace_id` / `span_id` | uuid | parent/child spans |
| `call_id` | uuid | correlates calls from frontend/CLI (no HTTP) |
| `tool_name` | tool contract | which tool |
| `retry_count` | retry loop | failure visibility |
| `input` / `output` | tool call | **full verbatim I/O** (redacted) |
| `tokens_in` / `tokens_out` / `cost_usd` | LLM wrapper | cost control |
| `duration_ms` | timer | latency |
| `status` | `ok` / `error` / `partial` / `blocked` | outcome |
| `verification` | verifier | `label` + `citation` |

## 13.3 Verbose event catalog (log every one)
**Run lifecycle:** `run.started`, `run.plan.completed`, `run.fanout.started`, `run.gather.completed`, `run.verification.completed`, `run.completed`, `run.cancelled`, `run.blocked`.
**Agents:** `agent.started`, `agent.input.received` (full shared-state keys), `agent.tool.requested`, `agent.handoff`, `agent.finished`, `agent.timeout`, `agent.failed`.
**Tools:** `tool.call.started`, `tool.schema.validation_failed`, `tool.call.completed`, `tool.call.retried`, `tool.call.failed`, `tool.rate_limited`, `tool.allowlist.denied`.
**Guardrails:** `budget.check`, `budget.exceeded`, `circuit_breaker.opened`, `hitl.requested`, `hitl.approved`, `hitl.rejected`, `sandbox.denied`, `redaction.applied`.
**LLM:** `llm.request.sent` (model, params, message count), `llm.response.received` (tokens, finish reason), `llm.tool_calls.parsed` (count + names), `llm.retry`.
**Eval:** `eval.task.started`, `eval.task.scored`, `eval.suite.completed`.
**Boundary:** `call.started`, `call.completed`, `call.failed` (bound-method calls from UI / CLI, logged with `call_id`).

## 13.4 Redaction (verbose ≠ leaking secrets)
Log **everything** except secrets. Before writing, the logging layer redacts:
- keys matching `password`, `token`, `secret`, `api_key`, `authorization`, `pat`, `private_key` (case-insensitive),
- values from `.env.local` / OS keychain (`go-keyring`),
- Git PATs, DB passwords, OAuth client secrets.
Log `redaction.applied` with the *field names* redacted (never the values). This keeps logs verbose and safe.

## 13.5 Example verbose lines

Dev console (ConsoleWriter):
```
18:49:15 INF tool.call.completed run_id=01J… agent_id=research tool_name=web_search retry_count=0 status=ok duration_ms=812 tokens_in=120 tokens_out=340 cost_usd=0.0012 input.query="postman dynamic variables" output.results=3 verification.label=verified verification.citation=tool:web_search#3
18:49:16 WRN budget.check run_id=01J… run_tokens=154200 run_limit=200000 percent=77.1 recommendation="reduce fan-out width"
```

Production JSON (one object per line, also appended to `runs/<id>/trace.jsonl`):
```json
{"level":"info","time":"2026-11-19T18:49:15Z","event":"agent.finished","run_id":"01J…","agent_id":"research","status":"ok","tokens_out":340,"verification":{"label":"verified","citation":"tool:web_search#3"},"message":"agent research finished with 3 verified claims"}
```

## 13.6 Run summary (verbose close-out)
At `run.completed`, emit one final structured object containing: total duration, agents used, tool calls by name, retries, tokens in/out, cost, verification label counts (verified/unverified/assumption), number of artifacts, merge decision, and HITL decisions. This is the human-readable "what happened" and is stored in `summary.json`.

## 13.7 Rules
- Never log a secret value. Redaction is mandatory, not optional.
- Never log a tool input/output *partially* — either full (redacted) or explicitly note why omitted.
- Every error log includes `retry_count` and a recovery hint message (verbose errors help agents recover — `docs/02`).
- Log in English, past tense event names, lowercase snake_case field names.
- `--quiet` flag reduces console verbosity but **never** reduces trace/JSONL detail.
