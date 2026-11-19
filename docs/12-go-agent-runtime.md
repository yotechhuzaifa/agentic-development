# 12 — Go Agent Runtime (library-first, verbose)

This is the concrete runtime design that implements `docs/01`–`docs/11`. All choices are final in `DECISIONS.md` (D-002–D-007, D-013–D-016). **No runnable code exists yet** — this is the contract the future implementation must satisfy. Language: **Go**. Policy: **use a library for every cross-cutting concern; never hand-roll what a maintained library already does well.**

## 12.1 Approved library registry (agentic runtime)

| Concern | Library | Why / notes |
|---|---|---|
| HTTP routing | **`github.com/go-chi/chi/v5`** (+ `/middleware`) | Composable router, route groups, `Mount`, `chi.URLParam`, middleware chain. Verified `/go-chi/docs`. |
| Logging | **`github.com/rs/zerolog`** (+ `/hlog`) | Zero-allocation structured JSON logging; verbose event catalog in `docs/13`. Verified `/rs/zerolog`. |
| LLM calls | **Official OpenAI Go SDK** **[VERIFY]** `github.com/openai/openai-go`; fallback `github.com/sashabaranov/go-openai` | Maintained SDK instead of raw HTTP calls. Tool calling + parallel tool calls via SDK types. |
| Fan-out/gather | **`golang.org/x/sync/errgroup`** (+ `semaphore`) | Cancellation, error propagation, `SetLimit(4)` for agents and `semaphore(8)` for tools. Replaces raw WaitGroup (D-006). |
| Config/env | **`github.com/joho/godotenv`** + `os.Getenv` | Load `.env` locally; secrets never logged (D-013 redaction). |
| Run/trace store | **`modernc.org/sqlite`** (WAL) + append-only JSONL | Pure Go, free, replayable (D-007). |
| IDs | **`github.com/google/uuid`** | `run_id`, `agent_id`, `span_id`, `trace_id`, `request_id`. |
| Strict JSON schemas | **`github.com/santhosh-tekuri/jsonschema/v6`** **[VERIFY]** | Validate every tool input/output against `schemas/` contracts. |
| Random/fake data | **`github.com/brianvoe/gofakeit/v7`** | Product Random Generator (D-012) **and** deterministic fixtures for `eval/`. Verified `/brianvoe/gofakeit`. |
| HTTP middleware extras | chi `RequestID`, `Recoverer`, `CleanPath`, `Timeout`, `ClientIP*`; zerolog `hlog` | Correlation IDs and verbose request logs flow together. |
| Product libraries (from chat) | Wails v3, Svelte 5 + Svelte Flow, CodeMirror 6, shadcn-svelte, Tailwind v4, Sonner, Lucide, tinykeys, cmdk-sv, marked, Goja, pgx, mongo-driver, go-git, go-keyring, fsnotify | See `PRODUCT/tech-stack.md`. |

> Anything not in this registry requires an ADR before adoption. Prefer the standard library only where no suitable maintained library exists (e.g. `net/http` types, `context`, `encoding/json` used *through* chi/zerolog).

## 12.2 Control plane — chi routes (verbose API)

```
POST   /v1/runs                       start a run (goal, constraints, budget)
GET    /v1/runs/{runID}               run status + summary + verification labels
GET    /v1/runs/{runID}/trace         full JSONL trace (replay source)
POST   /v1/runs/{runID}/cancel        cancel a running run
POST   /v1/runs/{runID}/approve       HITL decision for a blocked action
GET    /v1/runs/{runID}/artifacts     list artifacts
GET    /v1/agents                     registered agents + tool allow-lists
GET    /v1/tools                      registered tools + schemas + cost tags
POST   /v1/tools/{toolName}/validate  validate args against schema before execution
POST   /v1/eval/runs                  run the 20–100 eval tasks
GET    /v1/health                     liveness + version + config summary
```

Middleware chain (order matters): `RequestID` → `ClientIP*` → `hlog.NewHandler` → `hlog.AccessHandler` → `hlog.RequestIDHandler` → `Recoverer` → `CleanPath` → `Timeout(60s)` → budget guard → allow-list guard. Every request gets a `request_id` that is copied into the run's log fields.

## 12.3 Components (one responsibility each)

```
cmd/orchestrator        entrypoint: config, chi router, graceful shutdown
internal/api            chi handlers + middleware (no business logic)
internal/orchestrator   Manager: fan-out, gather, merge, budget, circuit breaker
internal/agent          Agent interface; manager + research/planner/executor/verifier
internal/tool           Tool interface; registry; strict schema validation; allow-lists
internal/llm            LLM SDK wrapper: retries, token accounting, tool-call parsing
internal/state          typed shared state, per-agent scratch, artifact store
internal/trace          JSONL writer + SQLite index + replay reader
internal/guard          budgets, timeouts, HITL, sandbox policy, redaction
internal/eval           eval task runner + scoring (uses gofakeit fixtures)
internal/logging        zerolog setup + event catalog (docs/13)
```

## 12.4 Run lifecycle (verbose, traceable)

1. **Start** — `POST /v1/runs` → generate `run_id` (uuid), create `runs/<run-id>/`, write `metadata.json` with goal/constraints/budget/model, log `run.started` with all fields.
2. **Plan** — Manager calls Planner specialist; produce typed subtask list; log `agent.plan.completed` with subtask count.
3. **Fan-out** — `errgroup.WithContext(ctx)`, `group.SetLimit(4)`; launch Research/Planner/Executor (and any needed specialists) concurrently. Each agent gets a **copy** of only the shared-state keys it may read (D-... state rules in `docs/05`).
4. **Tool calls** — model emits multiple tool calls; tool concurrency capped by `semaphore(8)`; each call validated against `schemas/` before execution; every input/output logged verbatim to trace (redacted per D-013).
5. **Gather** — `group.Wait()`; first error cancels siblings; partial results are labeled and kept ("use what you got", `docs/07`).
6. **Verify** — Verifier checks every claim against citations/tool-output IDs; labels `verified / unverified / assumption`; unverified claims are removed before synthesis.
7. **Synthesize** — Manager writes final answer from verified material only; writes `summary.json` with merge decision + labels.
8. **Finish** — log `run.completed` with totals: tokens in/out, cost, duration, retries, agents used, verification counts. Persist trace, artifacts, and summary. Run is now replayable.

## 12.5 Concurrency in practice (library-first)

- Agents: `errgroup` with `SetLimit(4)` and a cancellable context from `middleware.Timeout` / `context.WithTimeout`.
- Tools: `semaphore.NewWeighted(8)` acquired per tool call; release via `defer`.
- Retries: `errgroup` worker wraps tool call in a retry loop (max 3, exponential backoff) — implemented in `internal/llm`/`internal/tool`, not repeated in each agent.
- Circuit breaker: counter in `internal/orchestrator`; opens at 3 consecutive failures or 80% budget; logs `circuit_breaker.opened`.

## 12.6 Verbose trace event shape (see `docs/13` for full catalog)

```json
{
  "ts": "2026-11-19T18:49:15Z",
  "level": "info",
  "event": "tool.call.completed",
  "run_id": "…", "agent_id": "research", "span_id": "…", "request_id": "…",
  "tool_name": "web_search", "retry_count": 0,
  "input": {"query": "…"}, "output": {"results": 3},
  "tokens_in": 120, "tokens_out": 340, "cost_usd": 0.0012,
  "duration_ms": 812, "status": "ok",
  "verification": {"label": "verified", "citation": "tool:web_search#3"}
}
```

## 12.7 Contract rule (unchanged, now library-aware)
Every agent (`agents/`) and tool (`tools/`) contract must additionally name:
- the **library** it uses for I/O (no raw socket/curl/HTTP hand-rolling),
- the **log events** it emits (from `docs/13`),
- its **budget tag** and **allow-list membership**.

## 12.8 Verify before implementation
Items in `DECISIONS.md` §Factual checks — especially the LLM SDK module path, JSON-schema library, and Goja import path. Verify with Context7/official docs, then start Phase 0 of `BUILD-PLAN.md`.
