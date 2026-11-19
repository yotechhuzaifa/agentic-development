# HANDOVER.md — paste this to an AI when you are ready to build

Copy the block below verbatim into a new AI session. All decisions are already made — the AI's job is to read, confirm, verify factual items, and write Phase 0 contracts, then stop for your approval.

---

## PROMPT START

You are taking over a project that has been **fully specified but not yet implemented**. Do NOT write application code yet. First read the specification folder, confirm your understanding, run the listed factual checks, then produce only the Phase 0 contracts described below and STOP for approval.

### Where the spec lives
Root: `/home/yotech70/AntHill/`

Read first, in order:
1. `README.md` — map of the kit
2. `user_readme.md` — the owner's preferences (library-first, verbose, Wails bindings, zerolog)
3. `DECISIONS.md` — **every decision is FINAL and binding**. Do not re-open decisions. Only the **[VERIFY]** factual checks remain.
4. `PRODUCT/product-brief.md` — the product vision, storage invariants, out of scope
5. `PRODUCT/tech-stack.md` — Wails v3 + Go product stack, rejected alternatives, resolved conflicts
6. `PRODUCT/feature-list.md` — complete feature list with IDs and acceptance criteria
7. `PRODUCT/random-generator.md` — the Postman-style random generator (RAND-01–RAND-08) added by the owner, not in the original chat
8. `BUILD-PLAN.md` — phased backlog and phase gates
9. `docs/01`–`docs/11` — the agentic orchestration method
10. `docs/12-go-agent-runtime.md` — the concrete Go runtime (Wails-bound methods, components, run lifecycle)
11. `docs/13-verbose-logging.md` — the verbose zerolog standard
12. `docs/14-library-policy.md` — library-first policy and approved registry
13. `CONTRIBUTING.md` — contract-before-code rule and naming conventions

### What the product is (short version)
A lightweight desktop visual flow app for API + database workflows — "Postman Flows meets database workflows, in a lightweight desktop app." Stack: **Wails v3 + Go**, **Svelte 5 + Svelte Flow**, SQLite (`modernc.org/sqlite`) for **app state only**, Goja for sandboxed JS, pgx/Mongo pools, go-git sync. Flows are flat `.flow.json` files (one folder = one workspace); secrets in `.env.local` NEVER sync; SQLite is never synced directly. Plus the owner's added **Postman-style Random Generator** (gofakeit v7).

### What is FINAL (do not re-decide)
- Orchestration: **Manager + Specialists + Verifier gate**; merge = verifier gate → manager synthesis.
- Runtime language: **Go**, library-first.
- Transport: **Wails bound Go methods** — no HTTP router (chi removed as redundant, D-018). Logging: **zerolog**, verbose by default, redaction enforced.
- Concurrency: **errgroup + semaphore** (not raw WaitGroup). LLM calls: official Go SDK **[VERIFY path]**.
- Storage: SQLite WAL + JSONL traces. Frontend: Svelte 5. JS sandbox: Goja **[VERIFY path]**.
- Sync: Wails + go-git + go-keyring; **discard all Tauri/Rust tools** mentioned in the chat (conflict resolved in D-011).
- Safety: budgets/timeouts/HITL/circuit-breaker values in D-014/D-015. Verification mandatory (D-016).
- Verbosity is a first-class requirement (D-013, `docs/13`).

### The only remaining work before coding — factual checks **[VERIFY]**
Run these against official docs/Context7 and record results in `DECISIONS.md` (do not change any decision):
1. Official OpenAI Go SDK module path/version (`github.com/openai/openai-go` preferred).
2. Goja module path (`github.com/dop251/goja` vs chat's `nicois/goja`).
3. JSON-schema validator module (`github.com/santhosh-tekuri/jsonschema/v6`).
4. Wails v3 release/stability and `wails3 init -n myapp -t svelte`.
5. Library versions: `mongo-driver/v2`, `jackc/pgx`, `cmdk-sv`, `shadcn-svelte`, `fsnotify`, `zalando/go-keyring`.
6. Exact Postman dynamic-variable names for `RAND-01`/`RAND-02`.
7. Truncated product items `[CONFIRM]`: "flow template system" and "API templa..." — ask the owner, do not guess.

### How you must work (non-negotiable)
1. **Contract before code.** No agent/tool implemented until it has a contract in `agents/` or `tools/` and a strict schema in `schemas/`.
2. **Library-first, but only necessary libraries** (`docs/14`, D-019): use zerolog, errgroup, gofakeit, the LLM SDK, and the Go libraries in `docs/12.1`. UI↔Go goes through Wails bindings. No HTTP router, no custom logger, no hand-rolled random strings, no raw LLM HTTP calls.
3. **Verbose** (`docs/13`): every event with the full mandatory field set, full tool I/O, verbose errors with recovery hints, verbose run summary. `--quiet` reduces console only, never traces.
4. **Manager + Specialists** with parallel fan-out/gather and a **Verifier** blocking ungrounded claims; label verified / unverified / assumption.
5. **Observability from day one:** full transcript per agent, tool inputs AND outputs, timestamps + retries, per-agent tokens/cost, labels; every run replayable from `runs/`.
6. **Bound everything:** allow-lists, sandboxing, per-run/agent budgets (D-014), timeouts, retries with backoff, circuit breaker, HITL (D-015).
7. **Never invent product requirements.** If something is missing/ambiguous, mark it and ask. Respect the "don't" list (no CRDTs, no SQLite sync, no real-time collab, no secrets in sync, no custom auth) and the D-011 conflict resolution.

### Your first deliverable (in this session)
Do not implement features. Produce only:
1. A short written confirmation of your understanding of the product, the final decisions, and the library-first/verbose rules.
2. The results of the **[VERIFY]** checks above appended to `DECISIONS.md` (as factual notes, not new decisions), plus any `[CONFIRM]` questions for me.
3. The Phase 0 **contracts** (still documentation, not runnable code): `agents/` (manager + 2–3 specialists + verifier, each naming its library and log events), `tools/` (first tool set with schemas + allow-lists + cost tags), `schemas/` (strict I/O), `runs/` (verbose trace format), `eval/` (20–100 tasks including RAND tests), and a `cmd/orchestrator` route map per `docs/12.2`.

STOP after that and wait for my approval. Then we begin Phase 1 from `BUILD-PLAN.md`, one phase at a time, stopping at each gate.

## PROMPT END

---

## Notes for you (not part of the prompt)
- The AI will not re-decide anything — it only verifies facts and writes contracts. You stay in control.
- Everything is from your saved chat except the Random Generator, the Go runtime (zerolog + verified libraries), and verbosity — those are your explicit additions, marked as such in the docs. chi was requested initially but removed in D-018 as redundant under Wails.
- Saved chat: `/mnt/c/Users/yotech70/Downloads/Directly Chat with Frontier AI Search Models (11_19_2026 6：49：15 PM).html`.
