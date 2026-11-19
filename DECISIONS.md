# Decision Log (ADRs) — ALL DECISIONS FINAL

This is the binding record. Every architectural choice is **accepted**. There are no open product/architecture decisions left in this kit. Items marked **[VERIFY]** are *factual checks* against live docs, not decisions — verify them at implementation time, but do not re-open the decision. To change an accepted decision, add a new ADR that supersedes it (mark the old one `superseded by #N`); never silently deviate.

Template used: **Status / Date / Context / Decision / Alternatives / Consequences**.

---

## D-000 — Documentation-first, no code yet
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You asked for a documentation folder (originally `agentic-development`, now named **AntHill**) and explicitly said no codebase should be generated or written now.
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
- **Context:** You specified **zerolog** (logging) and library-first development with verbose output. You initially named chi for routing; D-018 removes it as redundant under Wails. The product stack (chat) is Go + Wails.
- **Decision:** The **agentic runtime is written in Go**, matching the product backend. One language across both layers.
- **Alternatives:** Python + OpenAI Agents SDK + `asyncio.gather` (the chat's guide default) — rejected because it abandons the Go/zerolog/library-first stack and splits the project into two languages.
- **Consequences:** All orchestration, tools, traces, and eval harness are Go. Fan-out uses Go concurrency libraries (D-006).

## D-003 — HTTP/JSON routing: chi v5 (SUPERSEDED by D-018)
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** The orchestration control plane needs an HTTP/JSON surface (start run, get trace, approve HITL, list tools, eval hooks).
- **Decision (original, now superseded by D-018):** chi v5 was selected for an HTTP control plane. Verified against `/go-chi/docs`.
- **Alternatives:** `net/http` ServeMux; gin/echo/fiber — all moot after the Wails binding decision.
- **Consequences:** No chi dependency in the default stack. Re-introduce only via ADR if a standalone daemon is added.

## D-004 — Logging: zerolog, verbose by default
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You asked for zerolog and for the project to be verbose.
- **Decision:** **`github.com/rs/zerolog`** only (no `hlog`, no `alice` — those existed only for HTTP access logging, which does not exist in a Wails-bound app). Structured JSON to stdout in production, `ConsoleWriter` (pretty) in dev. Default level **debug** in dev, **info** in prod, configurable via env. Every event carries `run_id`, `agent_id`, `tool_name`, `trace_id`/`span_id`, `call_id`, `duration_ms`, `tokens_in/out`, `cost_usd`, `status`, `retry_count`. Verified against `/rs/zerolog`. Full standard in `docs/13-verbose-logging.md`.
- **Alternatives:** `log/slog` (rejected — you asked for zerolog); `logrus` (rejected — slower, noisier API).
- **Consequences:** Large, highly readable logs; higher log volume/cost — accepted deliberately because you prefer verbose.

## D-005 — Agentic stack: in-house Go orchestration using libraries (not Python SDK / hosted / AutoGen)
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** The chat offered three agentic stacks. You want zerolog and library-first; chi was later dropped as redundant (D-018).
- **Decision:** **Option A adapted to Go** — build the orchestration in Go from well-chosen libraries; do not adopt the Python Agents SDK, hosted multi-agent, or AutoGen group chat. LLM calls go through a maintained Go SDK library. **[VERIFY]** official module path/version at implementation time (`github.com/openai/openai-go` preferred; community `github.com/sashabaranov/go-openai` as fallback).
- **Alternatives:** Hosted multi-agent (rejected — less control, no zerolog-level tracing); AutoGen (rejected — sequential group chat, Python).
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
- **Alternatives:** Tauri/Rust path (rejected — contradicts the primary, more detailed analysis and the Go decision).
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
- **Decision:** Require HITL for: any spend above **$1** per action, any write to production data, any external send (email/HTTP POST to third parties), any delete, any code execution with network/filesystem access, and any git push to a remote. Approvals are served through the bound `ApproveHITL` Go method and recorded in the trace.
- **Alternatives:** Fully autonomous (rejected — safety).
- **Consequences:** Runs may pause; timeout on approval → run marked `blocked`.

## D-016 — Verification is mandatory
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** Ungrounded claims are the main quality risk.
- **Decision:** Every claim must carry a citation or tool-output ID. The Verifier labels each output **verified / unverified / assumption** and blocks unverified material from manager synthesis. No exceptions.
- **Alternatives:** Optional verification (rejected — chat says this is the highest-value step).
- **Consequences:** Extra agent per run; accepted.

## D-017 — Project name: AntHill
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You wanted a name that "rings a bell." `ant hill` was the image you kept returning to, even before you could relate it to the product. Name searches with RivalSearchMCP showed the obvious alternatives were crowded: Flowforge (direct visual-API-workflow competitor), FlowFactory (n8n workflow studio), and every `Flow*` compound checked (Flowfoundry, Flowsmith, Flowweave, Flowcraft, Flowkit, Flowloom, Flowstudio, Flowbase, Flowmill, Flowlab, Flowdesk, Flowline, Flownode, Flowhouse, Flowworks) plus Conduit, Weave, and Queryflow. `PostThon` was clean but implies Python, while the runtime is Go + JS.
- **Decision:** Name the project **AntHill**. The metaphor maps exactly onto the architecture:
  - **Queen** = Manager/orchestrator (owns final output).
  - **Worker ants** = specialists (Research, Planner, Executor, Verifier) running in parallel.
  - **Tunnels** = flows; **chambers** = artifacts / shared state.
  - **Pheromone trails** = edges / data mappings between blocks.
  - **Mound entrance** = gather point.
  - **Colony survives lost ants** = fault tolerance, partial-result merging, retries.
  - **One workspace = one mound**, UI surface = Wails bindings, traces = verbose ant trails.
- **Alternatives:** Postflow (clean but less distinctive); PostThon (clean but Python-misleading); Flowforge/FlowFactory (taken); all other checked names (taken).
- **Consequences:** Distinct, memorable, thematically consistent with parallel agentic work. Local folder renamed from `agentic-development` to `AntHill` and all doc paths updated. **The git remote `github.com/yotechhuzaifa/agentic-development.git` is intentionally left unchanged** — renaming it is a separate, external-facing action for you to decide.

## D-018 — Transport: Wails bindings only; no HTTP router in the default stack
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** I re-checked Wails v3.1.0 docs via Context7 `/wailsapp/wails`. Wails generates TypeScript bindings from bound Go structs (`window["go"]["main"]["App"]["Greet"](...)`), with typed structs and Promise-based errors. There is no HTTP server or router between frontend and backend.
- **Decision:** **Remove chi and all HTTP middleware from the default stack.** For the Wails app, and for an agentic runtime embedded in the Wails process, expose operations (`StartRun`, `GetRun`, `GetTrace`, `ApproveHITL`, `ListTools`, `RunEval`) as **bound Go methods**. IDs come from `uuid`, not chi `RequestID`. No `hlog`, no `alice`. A chi HTTP daemon is **not** part of this design; adding one requires a new ADR showing a real need (e.g. a separate headless process).
- **Alternatives:** Keep chi as an "optional adapter" — rejected as unnecessary speculative infrastructure (you asked to remove redundancy).
- **Consequences:** Smaller dependency set, no HTTP attack surface, no extra serialization layer. D-003 is superseded by this ADR. zerolog stays — logging is orthogonal to transport.

## D-019 — Verified stack and removed redundancy
- **Status:** accepted
- **Date:** 2026-11-19
- **Context:** You asked to verify the tech stack and remove anything unnecessary, while keeping libraries over raw code.
- **Decision — KEPT (verified):**
  - `modernc.org/sqlite` — pure-Go `database/sql` driver, WAL supported (Context7 `/websites/pkg_go_dev_modernc_org_sqlite`). One DB for app state + run index.
  - `jackc/pgx` (`pgxpool`) — built-in pooling, configurable `MaxConns`/`MinIdleConns`/lifetime (Context7 `/jackc/pgx`).
  - `go-git/go-git` — pure-Go Git, porcelain APIs for clone/commit/push (Context7 `/go-git/go-git`).
  - `mongo-driver/v2` — internal connection pool.
  - `github.com/rs/zerolog` — verbose structured logging.
  - `golang.org/x/sync/errgroup` + `semaphore` — fan-out/gather and concurrency caps.
  - `github.com/openai/openai-go` **[VERIFY path]** — LLM calls via SDK.
  - `github.com/google/uuid` — correlation IDs (stdlib has no UUID).
  - JSON-schema validator **[VERIFY path]** — strict tool I/O validation.
  - `github.com/brianvoe/gofakeit/v7` — RAND feature + deterministic eval fixtures (Context7 `/brianvoe/gofakeit`).
  - Wails v3, Svelte 5 + Svelte Flow, CodeMirror 6, shadcn-svelte, Tailwind v4, Sonner, Lucide, tinykeys, cmdk-sv, marked, Goja, go-keyring, fsnotify — each owns a distinct concern.
- **Decision — REMOVED as unnecessary:**
  - **chi v5 + chi middleware** (D-018) — Wails bindings make it redundant.
  - **zerolog `hlog` and `alice`** — existed only for HTTP access logging; no HTTP server now.
  - **`github.com/joho/godotenv`** — unnecessary `.env` loader; use OS env / Wails dev environment.
  - **Separate trace database** — reuse the one SQLite DB; JSONL stays as append-only verbose trace.
  - **Tauri/Rust tools** (`git2`, `tauri-plugin-stronghold`, `tauri-plugin-sync`, `cloud_sync_tauri`, `better-sqlite3`, `sql.js`) — previously rejected in D-011.
  - **SolidJS / Kobalte / markdown-it / tauri stack table** — alternates already rejected.
- **Consequences:** Smaller, coherent library set. Every remaining dependency has a concrete owner and a verified purpose. No hand-rolled replacements.

---

## Factual checks — **[VERIFY]** (not decisions)
Do these against official docs at implementation time. They do not re-open any decision above:
1. `wails3 init -n myapp -t svelte` syntax (bindings themselves verified in D-018).
2. Goja module path (`github.com/dop251/goja` vs chat's `nicois/goja`).
3. Official OpenAI Go SDK module path/version (`github.com/openai/openai-go`).
4. `mongo-driver/v2`, `cmdk-sv`, `shadcn-svelte`, `fsnotify`, `zalando/go-keyring` current versions (`pgx` and `modernc.org/sqlite` verified).
5. JSON-schema validator module path.
6. Exact Postman dynamic-variable names for `RAND-01`.

**Library doc references verified via Context7:** `/wailsapp/wails` (v3.1.0), `/rs/zerolog`, `/brianvoe/gofakeit`, `/websites/pkg_go_dev_modernc_org_sqlite`, `/jackc/pgx`, `/go-git/go-git`, `/go-chi/docs` (chi verified then dropped in D-018).
