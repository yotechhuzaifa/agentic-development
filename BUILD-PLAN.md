# Build Plan / Backlog — what to hand an AI in order

This connects the **product feature list** ([`PRODUCT/feature-list.md`](PRODUCT/feature-list.md)) to the **agentic method** ([`docs/`](docs/)). All choices are final in [`DECISIONS.md`](DECISIONS.md) — do not re-decide. Each phase has a gate — do not start the next phase until the gate passes.

## Phase 0 — Agentic foundation (gate before any feature code)
Goal: the Go/chi/zerolog system for building is ready. See [`docs/10-starter-plan.md`](docs/10-starter-plan.md), [`docs/12-go-agent-runtime.md`](docs/12-go-agent-runtime.md), [`docs/13-verbose-logging.md`](docs/13-verbose-logging.md), [`docs/14-library-policy.md`](docs/14-library-policy.md).
- [x] Orchestration pattern FINAL: Manager + Specialists + Verifier (D-001)
- [x] Stack FINAL: in-house Go orchestration with chi/zerolog/errgroup (D-002–D-007)
- [x] Conflicts RESOLVED: Wails + Go + go-git + go-keyring (D-011)
- [ ] Run **[VERIFY]** factual checks (LLM SDK path, Goja path, Wails version, library versions, Postman tokens) and record results in `DECISIONS.md`
- [ ] Define 2–3 specialists (Research, Planner, Executor) + Manager + **Verifier** contracts in `agents/`
- [ ] Define tool allow-lists + strict JSON schemas in `schemas/`
- [ ] Observability: verbose transcripts, full tool I/O, timestamps, tokens/cost, labels; runs replayable
- [ ] Budgets per run/agent (D-014), timeouts, retries, circuit breaker; HITL triggers (D-015)
- [ ] `eval/` has 20–100 tasks (including RAND tests) and a baseline score
- **Gate:** a run can be replayed from verbose traces; budgets enforce; verifier blocks ungrounded claims; all logs follow `docs/13`.

## Phase 1 — MVP (chat's "Phase 1" + MUST HAVE)
- CORE-01 Visual canvas with blocks
- CORE-02 REST API calling
- CORE-03 cURL import
- CORE-11 SQLite app state
- CORE-12 Minimap
- CORE-15 Wails binary / lightweight desktop
- MUST-01 Response Viewer, MUST-02 Auth, MUST-03 Environments, MUST-04 History, MUST-05 Import/Export, MUST-06 Shortcuts, MUST-07 Body builders
- **RAND-01, RAND-02** (Postman dynamic variables — required for clean Postman import), **RAND-03–RAND-08** (generator block, seed control, batch, export, verbose logs) — see [`PRODUCT/random-generator.md`](PRODUCT/random-generator.md)
- SYNC-01 flat files + manual export/import, SYNC-02 folder sync
- **Gate:** a user can build a flow, call an API with auth + env, inspect the response, import a Postman collection with working random variables, generate fake payloads, and export/import it.

## Phase 2 — v1.0 (database + code + real flows)
- CORE-04 DB flows (PG + Mongo), CORE-05 global pool, CORE-06 Goja code blocks, CORE-07 condition/loop blocks, CORE-08 secrets/vars, CORE-13 expressions
- CORE-09 templates **[CONFIRM]**, CORE-10 parallel execution
- SYNC-03 Git init/diff/pull, SYNC-04 push/branch/merge UI, SYNC-05 auto-commit
- HIGH-01 visual chaining, HIGH-02 assertion block, HIGH-05 undo/redo
- **Gate:** a flow combines API + DB + code + branching, runs in parallel, and syncs via Git.

## Phase 3 — v1.5 (sync depth)
- SYNC-06 self-hosted sync server, SYNC-08 E2E encryption, SYNC-09 conflict UI
- HIGH-03 WebSocket block, HIGH-04 CLI runner
- **Gate:** headless CLI works; self-hosted sync works with encryption.

## Phase 4 — v2.0 (cloud) and v2.5
- SYNC-07 R2 cloud sync, SYNC-10 audit log, SYNC-11 multi-device, SYNC-12 WebDAV/S3
- NICE-* features as demand warrants
- Plugin system (v2+)

## Dependency notes
- CORE-08 (secrets/vars) must precede MUST-02 (auth), MUST-03 (environments), and RAND-01 (dynamic variable resolution).
- RAND-01/RAND-02 must precede MUST-05 (Postman import) so imported dynamic variables resolve correctly.
- CORE-05 (pool) must exist before CORE-10 (parallel execution) and CORE-04 (DB flows) to avoid connection leaks.
- Phase 0 gates every phase — no feature work without observability + verifier + budgets.

## How an AI agent should consume this
1. Read `HANDOVER.md` — the paste-ready prompt (all decisions already made).
2. Read `PRODUCT/product-brief.md`, `PRODUCT/tech-stack.md`, `PRODUCT/feature-list.md`, and `PRODUCT/random-generator.md` for the WHAT.
3. Read `docs/01`–`docs/11` (method), then `docs/12`–`docs/14` (Go runtime, verbose logging, library policy) for the HOW.
4. Respect `DECISIONS.md` — every choice is final; do not re-decide. Only **[VERIFY]** factual checks remain.
5. Be library-first (`docs/14`): chi for routing, zerolog for logging, errgroup for concurrency, gofakeit for random data, SDK for LLM calls — no raw coding.
6. Be verbose (`docs/13`) in logs, traces, errors, docs, and CLI.
7. Work one phase at a time and stop at each gate for review.
