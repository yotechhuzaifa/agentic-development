# 12 — Go Agent Runtime (library-first, verbose)

This is the concrete runtime design that implements `docs/01`–`docs/11`. Choices are final in `DECISIONS.md` (D-002–D-019) — notably **D-018: no HTTP router in the default stack**. **No runnable code exists yet** — this is the contract for future implementation. Language: **Go**. Policy: use a library for every cross-cutting concern; never hand-roll what a maintained library already does well — but do not add libraries that nothing needs.

## 12.1 Approved library registry (verified 2026-11-19)

| Concern | Library | Why / verification |
|---|---|---|
| UI ↔ Go transport | **Wails v3 bindings** (not a library — framework feature) | Generated TS methods on bound Go structs, typed structs, Promise errors. Verified `/wailsapp/wails` v3.1.0. No HTTP/router needed. |
| Logging | **`github.com/rs/zerolog`** | Zero-allocation structured JSON, verbose catalog in `docs/13`. Verified `/rs/zerolog`. No `hlog`/`alice`. |
| LLM calls | **Official OpenAI Go SDK** **[VERIFY]** `github.com/openai/openai-go` | Maintained SDK, tool calling + parallel tool calls. No raw HTTP. |
| Fan-out/gather | **`golang.org/x/sync/errgroup`** + `semaphore` | Cancellation, error propagation, `SetLimit(4)` agents / `semaphore(8)` tools (D-006, D-014). |
| Run/trace store | **`modernc.org/sqlite`** (WAL) + append-only JSONL | Pure Go `database/sql`, WAL supported — verified `/websites/pkg_go_dev_modernc_org_sqlite`. One DB for app state + run index. |
| IDs | **`github.com/google/uuid`** | `run_id`, `agent_id`, `span_id`, `trace_id` (stdlib has no UUID). |
| Strict JSON schemas | JSON-schema validator **[VERIFY]** `github.com/santhosh-tekuri/jsonschema/v6` | Validates tool I/O against `schemas/`. |
| Random/fake data | **`github.com/brianvoe/gofakeit/v7`** | RAND feature + deterministic eval fixtures. Verified `/brianvoe/gofakeit`. |
| Postgres | **`jackc/pgx`** (`pgxpool`) | Built-in pooling, configurable limits/lifetime. Verified `/jackc/pgx`. |
| MongoDB | **`mongo-driver/v2`** | Internal connection pool; no external pool library. |
| Git sync | **`go-git/go-git`** | Pure-Go clone/commit/push porcelain. Verified `/go-git/go-git`. |
| Secrets | **`zalando/go-keyring`** | OS keychain for tokens/PATs; never in `.env.local` sync. |
| File watching | **`fsnotify`** | Workspace file changes. |
| JS sandbox | **Goja** **[VERIFY path]** | Pure-Go sandboxed JS for code blocks. |
| Frontend | Svelte 5 + Svelte Flow, CodeMirror 6, shadcn-svelte, Tailwind v4, Sonner, Lucide, tinykeys, cmdk-sv, marked | Each owns a distinct UI concern. See `PRODUCT/tech-stack.md`. |

**Removed as unnecessary (D-018/D-019):** chi v5 + chi middleware, zerolog `hlog`, `alice`, `godotenv`, separate trace DB, all Tauri/Rust tools, SolidJS/Kobalte/markdown-it alternates.

> Anything not in this registry requires an ADR. Prefer the standard library only where no suitable library exists (`context`, `encoding/json`, `time`).

## 12.2 Control surface — bound Go methods (no HTTP)

The Wails-bound `Orchestrator` struct exposes these methods to the Svelte frontend (and to the headless CLI runner as ordinary Go calls):

```
StartRun(ctx, goal, constraints, budget) (runID, error)
GetRun(ctx, runID) (RunStatus, error)
GetTrace(ctx, runID) ([]TraceEvent, error)
CancelRun(ctx, runID) error
ApproveHITL(ctx, runID, decision) error
ListArtifacts(ctx, runID) ([]Artifact, error)
ListAgents(ctx) ([]AgentInfo, error)
ListTools(ctx) ([]ToolInfo, error)
ValidateToolArgs(ctx, toolName, argsJSON) error
RunEval(ctx) (EvalReport, error)
```

- Wails generates the TS declarations automatically; structs need valid `json` tags.
- Timeouts come from `context.WithTimeout` (60s agent / 30s tool), not HTTP middleware.
- Correlation IDs come from `uuid`, and are passed as explicit log fields (D-013).
- If a future standalone daemon is truly needed, re-introduce an HTTP adapter via ADR — it is **not** in this design.

## 12.3 Components (one responsibility each)

```
cmd/ant-hill           Wails entrypoint: config, bind Orchestrator, graceful shutdown
internal/orchestrator  Manager: fan-out, gather, merge, budget, circuit breaker
internal/agent         Agent interface; manager + research/planner/executor/verifier
internal/tool          Tool interface; registry; strict schema validation; allow-lists
internal/llm           SDK wrapper: retries, token accounting, tool-call parsing
internal/state         typed shared state, per-agent scratch, artifact store
internal/trace         JSONL writer + SQLite index + replay reader
internal/guard         budgets, timeouts, HITL, sandbox policy, redaction
internal/eval          eval task runner + scoring (gofakeit fixtures)
internal/logging       zerolog setup + event catalog (docs/13)
```

## 12.4 Run lifecycle (verbose, traceable)

1. **Start** — `StartRun` → generate `run_id` (uuid), create `runs/<run-id>/`, write `metadata.json` with goal/constraints/budget/model, log `run.started`.
2. **Plan** — Manager calls Planner specialist; typed subtask list; log `agent.plan.completed`.
3. **Fan-out** — `errgroup.WithContext(ctx)`, `group.SetLimit(4)`; launch Research/Planner/Executor concurrently. Each agent receives a copy of only the shared-state keys it may read (`docs/05`).
4. **Tool calls** — model emits multiple tool calls; `semaphore(8)` caps concurrency; each call validated against `schemas/`; full I/O logged verbatim (redacted per D-013).
5. **Gather** — `group.Wait()`; first error cancels siblings; partial results labeled and kept (`docs/07`).
6. **Verify** — Verifier checks every claim against citations/tool-output IDs; labels `verified / unverified / assumption`; unverified material removed.
7. **Synthesize** — Manager writes final answer from verified material only; `summary.json` records merge decision + labels.
8. **Finish** — log `run.completed` with tokens in/out, cost, duration, retries, agents, verification counts. Run is replayable.

## 12.5 Concurrency in practice (library-first)

- Agents: `errgroup` with `SetLimit(4)` and `context.WithTimeout(60s)`.
- Tools: `semaphore.NewWeighted(8)`, released with `defer`.
- Retries: bounded loop in `internal/llm` / `internal/tool` (max 3, exponential backoff) — written once, not per agent.
- Circuit breaker: counter in `internal/orchestrator`; opens at 3 consecutive failures or 80% budget; logs `circuit_breaker.opened`.

## 12.6 Verbose trace event shape (`docs/13`)

```json
{
  "ts": "2026-11-19T18:49:15Z",
  "level": "info",
  "event": "tool.call.completed",
  "run_id": "…", "agent_id": "research", "span_id": "…", "trace_id": "…", "call_id": "…",
  "tool_name": "web_search", "retry_count": 0,
  "input": {"query": "…"}, "output": {"results": 3},
  "tokens_in": 120, "tokens_out": 340, "cost_usd": 0.0012,
  "duration_ms": 812, "status": "ok",
  "verification": {"label": "verified", "citation": "tool:web_search#3"}
}
```

## 12.7 Contract rule
Every agent (`agents/`) and tool (`tools/`) contract must name its library, log events (`docs/13`), budget tag, and allow-list membership.

## 12.8 Verify before implementation
Items in `DECISIONS.md` §Factual checks (SDK path, JSON-schema path, Goja path, Postman variables). Verified libs already checked via Context7 are listed in D-019.
