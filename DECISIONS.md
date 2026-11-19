# Decision Log (ADRs) — ALL DECISIONS FINAL

This is the binding record. Every architectural choice is **accepted**. There are no open product/architecture decisions left in this kit. Items marked **[VERIFY]** are *factual checks* against live docs, not decisions — verify them at implementation time, but do not re-open the decision. To change an accepted decision, add a new ADR that supersedes it (mark the old one `superseded by #N`); never silently deviate.

Template used: **Status / Date / Context / Decision / Alternatives / Consequences**.

---

## D-000 — Documentation-first, no code yet
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You asked for an agentic-development folder and explicitly said no codebase should be generated or written now.
- **Decision:** This folder contains documentation, contracts, ADRs, and placeholder README folders only. No runnable code.
- **Alternatives:** Generate a working manager/specialist implementation now — rejected per your instruction.
- **Consequences:** The contracts here become the spec for a later implementation step.

## D-001 — Orchestration pattern: Manager + Specialists with a Verifier gate
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Parallel agents need one owner of the final answer and an explicit merge point. Handoffs/routing is for clean categories; hosted multi-agent gives less control and trace detail.
- **Decision:** **Manager + Specialists ("agents as tools")**. Manager owns the final output, fans out to specialists, gathers results, and a dedicated **Verifier** blocks ungrounded claims before synthesis. Merge rule: **verifier gate → manager synthesis**.
- **Alternatives:** Handoffs/routing (rejected — not a parallel fan-out pattern by itself); hosted multi-agent (rejected — less control over traces, budgets, and internal behavior).
- **Consequences:** High control, explicit ownership, deterministic merge; we own the orchestration code (mitigated by library-first policy, D-005/D-006).

## D-002 — Implementation language: Go (for the agentic runtime too)
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You specified **chi** (routing) and **zerolog** (logging) — both Go libraries — and asked for library-first development with verbose output. The product stack (chat) is also Go + Wails.
- **Decision:** The **agentic runtime is written in Go**, matching the product backend. One language across both layers.
- **Alternatives:** Python + OpenAI Agents SDK + `asyncio.gather` (the chat's guide default) — rejected because it abandons chi/zerolog and splits the project into two languages.
- **Consequences:** All orchestration, tools, traces, and eval harness are Go. Fan-out uses Go concurrency libraries (D-006).

## D-003 — HTTP/JSON routing: chi v5
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** The orchestration control plane needs an HTTP/JSON surface (start run, get trace, approve HITL, list tools, eval hooks).
- **Decision:** **`github.com/go-chi/chi/v5`** with `github.com/go-chi/chi/v5/middleware`. Use route groups (`r.Route`), `r.Mount`, `chi.URLParam`, and middleware `RequestID`, `Recoverer`, `CleanPath`, `Timeout`, `RealIP`/`ClientIP*`. Verified against `/go-chi/docs`.
- **Alternatives:** `net/http` ServeMux (rejected — too raw); gin/echo/fiber (rejected — heavier, less composable; chi is idiomatic and middleware-friendly).
- **Consequences:** Clean, modular API; request IDs flow into zerolog fields (D-004/D-013).

## D-004 — Logging: zerolog, verbose by default
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You asked for zerolog and for the project to be verbose.
- **Decision:** **`github.com/rs/zerolog`** (plus `github.com/rs/zerolog/hlog` for HTTP request logging). Structured JSON to stdout in production, `ConsoleWriter` (pretty) in dev. Default level **debug** in dev, **info** in prod, configurable via env. Every event carries `run_id`, `agent_id`, `tool_name`, `trace_id`/`span_id`, `request_id`, `duration_ms`, `tokens_in/out`, `cost_usd`, `status`, `retry_count`. Verified against `/rs/zerolog`. Full standard in `docs/13-verbose-logging.md`.
- **Alternatives:** `log/slog` (rejected — you asked for zerolog); `logrus` (rejected — slower, noisier API).
- **Consequences:** Large, highly readable logs; higher log volume/cost — accepted deliberately because you prefer verbose.

## D-005 — Agentic stack: in-house Go orchestration using libraries (not Python SDK / hosted / AutoGen)
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** The chat offered three agentic stacks. You want chi/zerolog and library-first.
- **Decision:** **Option A adapted to Go** — build the orchestration in Go from well-chosen libraries; do not adopt the Python Agents SDK, hosted multi-agent, or AutoGen group chat. LLM calls go through a maintained Go SDK library. **[VERIFY]** official module path/version at implementation time (`github.com/openai/openai-go` preferred; community `github.com/sashabaranov/go-openai` as fallback).
- **Alternatives:** Hosted multi-agent (rejected — less control, no chi/zerolog); AutoGen (rejected — group chat is sequential by default; Python).
- **Consequences:** We own orchestration logic, but every cross-cutting concern is delegated to a library (D-003–D-006).

## D-006 — Concurrency: `errgroup` library, not raw WaitGroup
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You prefer libraries over raw coding. The chat's sample used raw `sync.WaitGroup`.
- **Decision:** Use **`golang.org/x/sync/errgroup`** for fan-out/gather (cancellation + error propagation + bounded concurrency via `SetLimit`). `semaphore` from the same package for tool-level concurrency caps.
- **Alternatives:** Raw `sync.WaitGroup` + goroutines (rejected — more hand-written coordination, weaker error propagation).
- **Consequences:** First error cancels siblings cleanly; concurrency limits are declarative.

## D-007 — Trace/artifact storage: SQLite + JSONL
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Runs must be replayable; artifacts need durable storage.
- **Decision:** `modernc.org/sqlite` for run metadata/index; append-only JSONL for full traces; files under `runs/<run-id>/` as designed. SQLite WAL mode.
- **Alternatives:** Pure files only (rejected — no query index); external DB (rejected — adds server cost, violates $0 principle).
- **Consequences:** Local, free, replayable, easy to back up.

## D-008 — Product frontend: Svelte 5 + Svelte Flow
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Chat compares Svelte 5 vs SolidJS and recommends Svelte 5.
- **Decision:** **Svelte 5 + Svelte Flow (xyflow)** + CodeMirror 6 + shadcn-svelte + Tailwind v4 + Sonner + Lucide + tinykeys + cmdk-sv + marked.
- **Alternatives:** SolidJS (rejected — needs React Flow adapter, smaller ecosystem).
- **Consequences:** First-class canvas, official Wails template.

## D-009 — JS code blocks: Goja sandbox
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Chat's detailed comparison of Goja vs V8/Node/WebView.
- **Decision:** **Goja** (pure Go, sandboxed, ships in binary), ES2020. Inject `getSecret`, `showToast`, `setVariable`, `console.log`. **[VERIFY]** module path — chat shows `github.com/nicois/goja`, known module is `github.com/dop251/goja`.
- **Alternatives:** V8 (CGO), Node (not portable), WebView (wrong context) — all rejected.
- **Consequences:** Code blocks are limited to ES2020, acceptable for variable manipulation/data extraction.

## D-010 — Databases: pgx + mongo-driver; global pools
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Product requires PG + Mongo flows with a global connection pool.
- **Decision:** `jackc/pgx` (`pgxpool`) + `mongo-driver/v2`, one pool/client per connection ID in a `DBManager` guarded by `sync.RWMutex`. No external pool library.
- **Alternatives:** Per-run connections (rejected — leaks under parallel execution).
- **Consequences:** CORE-05 must be built before CORE-10 parallel execution (recorded in `BUILD-PLAN.md`).

## D-011 — Sync: flat files + go-git + go-keyring; discard Tauri/Rust tools
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** The chat's sync section mixed Tauri/Rust tools (`git2` crate, `tauri-plugin-stronghold`, `tauri-plugin-sync`, `cloud_sync_tauri`) with the Wails + Go verdict; a later "Tech Stack (All Free)" table also listed Tauri/SolidJS/better-sqlite3. These contradict the detailed verdict.
- **Decision:** Resolve in favor of the main verdict: **Wails v3 + Go**, git via **`go-git/go-git`**, secrets via **`zalando/go-keyring`**, file watching via **`fsnotify`**. Discard all Tauri/Rust sync tools and the later Tauri stack table. Flat `.flow.json` files, one folder = one workspace, `.env.local` never synced, SQLite app-state-only.
- **Alternatives:** Tauri/Rust path (rejected — contradicts the primary, more detailed analysis and your chi/zerolog Go decision).
- **Consequences:** One coherent Go codebase; no CGO/Rust toolchain; the [CONFLICT] flags in `PRODUCT/tech-stack.md` are resolved by this ADR.

## D-012 — Postman-style Random Generator: gofakeit v7
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** A feature not in the chat but requested by you: a random generator for Postman-style workflows.
- **Decision:** Use **`github.com/brianvoe/gofakeit/v7`** (310+ generators, seeded reproducible mode via `gofakeit.New(seed)` / `gofakeit.Seed(n)`, seed 0 = crypto-random). Implement two surfaces: (1) **Postman-compatible dynamic variables** in any variable/expression context (`{{$randomInt}}`, `{{$randomUUID}}`, `{{$randomEmail}}`, … mapped to gofakeit functions), and (2) a **Random Generator block/panel** that produces structured fake payloads (JSON/schema-driven) for request bodies and test data. Full spec in `PRODUCT/random-generator.md`. **[VERIFY]** exact Postman dynamic-variable names against current Postman docs.
- **Alternatives:** Hand-rolled random strings (rejected — library-first); Faker.js via JS sandbox (rejected — slower, less integrated).
- **Consequences:** New feature IDs `RAND-01`–`RAND-08`; added to MVP Phase 1 because import-from-Postman compatibility depends on it.

## D-013 — Verbosity: verbose everywhere
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You said you like verbose and want the project verbose.
- **Decision:** Verbose is a first-class project property: verbose structured logs (D-004), verbose traces with full tool I/O, verbose error messages that tell the agent how to recover, verbose documentation (every doc has rationale + alternatives + examples), verbose run summaries, and verbose CLI output with a `--quiet` opt-out. See `docs/13-verbose-logging.md`.
- **Alternatives:** Terse defaults (rejected — explicit user preference).
- **Consequences:** Higher log/storage volume and larger docs — accepted deliberately.

## D-014 — Safety defaults (concrete numbers)
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Budgets, timeouts, and breakers need concrete starting values rather than "configure later."
- **Decision:** Starting limits (tune later via ADR): max concurrent agents **4**, max concurrent tool calls **8**, per-run token budget **200k**, per-agent token budget **50k**, agent timeout **60s**, tool timeout **30s**, retries **3** with exponential backoff, circuit breaker opens after **3** consecutive failures or **80%** budget use. All enforced in the Go runtime and logged verbosely.
- **Alternatives:** "Configure later" (rejected — parallelism multiplies spend immediately).
- **Consequences:** Predictable spend and bounded failures from day one.

## D-015 — Human-in-the-loop triggers
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** High-impact actions need approval.
- **Decision:** Require HITL for: any spend above **$1** per action, any write to production data, any external send (email/HTTP POST to third parties), any delete, any code execution with network/filesystem access, and any git push to a remote. Approvals are served over the chi API and recorded in the trace.
- **Alternatives:** Fully autonomous (rejected — safety).
- **Consequences:** Runs may pause; timeout on approval → run marked `blocked`.

## D-016 — Verification is mandatory
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Ungrounded claims are the main quality risk.
- **Decision:** Every claim must carry a citation or tool-output ID. The Verifier labels each output **verified / unverified / assumption** and blocks unverified material from manager synthesis. No exceptions.
- **Alternatives:** Optional verification (rejected — chat says this is the highest-value step).
- **Consequences:** Extra agent per run; accepted.

---

## Factual checks — **[VERIFY]** (not decisions)
Do these against official docs at implementation time. They do not re-open any decision above:
1. Wails v3 release date / stability and `wails3 init -n myapp -t svelte` syntax.
2. Goja module path (`github.com/dop251/goja` vs chat's `nicois/goja`).
3. Official OpenAI Go SDK module path/version (`github.com/openai/openai-go`).
4. `mongo-driver/v2`, `jackc/pgx`, `cmdk-sv`, `shadcn-svelte`, `fsnotify`, `zalando/go-keyring` current versions.
5. Exact Postman dynamic-variable names for `RAND-01`.

**Library doc references used to make these decisions (Context7):** `/go-chi/docs`, `/rs/zerolog`, `/brianvoe/gofakeit`.
