# Agentic Development Readiness Checklist

Nothing here is optional. All architecture is **already decided** in `DECISIONS.md` — this checklist verifies the decisions were actually implemented in the Phase 0 contracts. Do not call the system "ready" until every box is checked.

## 1. Orchestration pattern — FINAL (D-001) [1][2][3]
- [x] **Manager + Specialists (agents-as-tools)** — manager owns final output, calls specialists like tools, merges
- [x] Handoffs / hosted multi-agent explicitly rejected (reasons in D-001)
- [ ] One owner is explicitly responsible for the final output (no "agents talking in circles")
- [ ] Division of labor documented in `agents/` (Research, Planner, Executor + Manager + Verifier)

## 2. Tool calling + execution layer — FINAL (D-005) [1]
- [ ] Agent loop defined
- [ ] Tool calling supported via **official OpenAI Go SDK** **[VERIFY path]** — no raw HTTP
- [ ] Orchestration wired in
- [ ] Runtime handles retries and multi-step execution
- [ ] Every tool has a strict JSON schema in `schemas/` + allow-list + cost tag

## 3. Concurrency controls — FINAL (D-006) [4][5]
- [ ] Model-side parallel tool calls accounted for
- [ ] App-side concurrent execution implemented with **`errgroup` + `semaphore`** (not raw WaitGroup)
- [ ] Fan-out bounded: `SetLimit(4)` agents, `semaphore(8)` tools (D-014)
- [ ] Rate limits and tool-collision handling in place
- [ ] Duplicate-write prevention defined (typed shared-state keys)

## 4. Observability — traces, logs, evaluation [1] + D-013
- [ ] Full transcript per agent
- [ ] Tool call inputs **and** outputs captured verbatim (redacted)
- [ ] Timestamps + retries captured
- [ ] Per-agent cost / tokens captured
- [ ] Success / failure labels recorded
- [ ] Runs replayable from `runs/` (SQLite index + JSONL)
- [ ] **Verbose logging standard implemented** (`docs/13`): mandatory fields, event catalog, redaction, verbose run summary
- [ ] Bound-method calls logged with `call_id` (no HTTP access logging — D-018)
- [ ] Evaluation set exists in `eval/` (20–100 tasks incl. RAND tests) + scoring

## 5. State & memory [8]
- [ ] Shared state defined (goal, constraints, key artifacts, final draft)
- [ ] Per-agent private scratch defined (temporary reasoning, notes)
- [ ] Artifact store defined (documents, JSON outputs, code patches)
- [ ] Typed state / checkpoints approach chosen for long-running reliability

## 6. Guardrails & security [9] + D-014/D-015
- [ ] Tool allow-list per agent (no agent can call "everything")
- [ ] Sandboxing for code execution / risky tools
- [ ] Budgets per run (200k tokens) and per agent (50k), timeouts (60s/30s), retries (3, backoff)
- [ ] Circuit breaker (3 consecutive failures or 80% budget)
- [ ] HITL on high-impact actions (spend >$1, prod writes, external sends, deletes, code exec, git push)

## 7. Failure handling [10]
- [ ] Per-agent timeout
- [ ] Per-tool retry policy with backoff
- [ ] Partial-result merging rule ("use what you got")
- [ ] Circuit breaker implemented
- [ ] Durable session / recovery via checkpoints

## 8. Merge strategy & evidence gating — FINAL (D-016)
- [x] Merge rule: **verifier gate → manager synthesis**
- [ ] Claims require citations or tool-output IDs
- [ ] Verifier agent checks "is this supported?" and blocks ungrounded claims
- [ ] Outputs labeled **verified / unverified / assumption**

## 9. Library-first + verbose (your preferences) — D-003/D-004/D-013/D-014
- [ ] **Wails bindings** used for UI↔Go transport; no HTTP router added (D-018)
- [ ] **zerolog** used for all logging, verbose by default, redaction enforced (no `log.Printf`)
- [ ] **errgroup/semaphore** for concurrency (no raw WaitGroup chains)
- [ ] **gofakeit v7** for all random data (no hand-rolled random strings)
- [ ] Library registry (`docs/12.1`) respected; new libraries require an ADR (`docs/14`)
- [ ] Verbose errors include recovery hints; `--quiet` never reduces trace detail

## 10. Postman Random Generator — added by you (D-012)
- [ ] RAND-01/RAND-02 dynamic variables resolve; unknown tokens warn + label unverified
- [ ] RAND-03–RAND-08 generator block, seed control, batch, export, verbose logs
- [ ] Schema files `schemas/random_generator.schema.json` + `random_output.schema.json`
- [ ] RAND eval tasks included in `eval/`

## 11. Factual checks before coding (not decisions)
- [ ] **[VERIFY]** LLM SDK path, Goja path, JSON-schema lib, Wails v3, library versions, Postman variable names
- [ ] **[CONFIRM]** "flow template system" and "API templa..." with the owner

## 12. Starter build (1–2 weeks) [6][7]
- [ ] Step 1: one manager + 2–3 specialists (Research, Planner, Executor) + Verifier
- [ ] Step 2: fan-out → gather (parallel specialists then merge)
- [ ] Step 3: evidence gating + verification (Verifier blocks ungrounded claims)

## 13. Anti-pattern guard — revisit these often [1]
- [ ] Not too many agents too early (start 2–4)
- [ ] Not sharing full context with every agent
- [ ] Merge strategy exists
- [ ] Observability exists
- [ ] Cost controls exist (budget per run and per agent)
- [ ] No raw coding where a library exists
