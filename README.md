# Agentic Development — Complete Readiness Kit

> Parallel agentic development = building multiple AI "agents" (specialists) that work at the same time on different parts of a task (**fan-out**), then merge their results (**gather**) under one **manager / orchestrator** agent. The hard part is not "more prompts" — it is **orchestration, safety, state, debugging, and cost control**. [1]

This folder is **not a codebase and contains no runnable code** (by design). It is the complete specification you hand to an AI (or a developer) later. **Every decision is final** in [`DECISIONS.md`](DECISIONS.md); only a short list of factual **[VERIFY]** checks remains before implementation.

## The three guiding preferences (from you)
1. **Library-first** — use maintained libraries for every cross-cutting concern; no raw coding. Policy in [`docs/14-library-policy.md`](docs/14-library-policy.md).
2. **chi for routing, zerolog for logging** — final decisions D-003 / D-004.
3. **Verbose everywhere** — verbose logs, traces, errors, docs, and CLI; D-013 / [`docs/13-verbose-logging.md`](docs/13-verbose-logging.md).

## What is in here
- **The method (how to build):** `docs/01`–`docs/14`, `checklist.md`, `CONTRIBUTING.md`.
- **The product (what to build):** `PRODUCT/`, `BUILD-PLAN.md`. Includes the **Postman-style Random Generator** you requested (not in the chat).
- **The runtime (what to build with):** Go + chi + zerolog + errgroup + gofakeit + official OpenAI Go SDK — `docs/12-go-agent-runtime.md`.
- **The handoff:** `HANDOVER.md` — a paste-ready prompt; the AI verifies facts, writes Phase 0 contracts, and stops for your approval.

## How to use this folder
1. Read [`user_readme.md`](user_readme.md) — your personal sheet.
2. Read [`DECISIONS.md`](DECISIONS.md) — everything is decided; only **[VERIFY]** items remain.
3. Read the method: `docs/01`–`docs/14` in order.
4. Read the product: `PRODUCT/product-brief.md` → `PRODUCT/tech-stack.md` → `PRODUCT/feature-list.md` → `PRODUCT/random-generator.md` → `BUILD-PLAN.md`.
5. Tick [`checklist.md`](checklist.md).
6. When ready, paste [`HANDOVER.md`](HANDOVER.md) into an AI session.
7. Only after approval: implement against the contracts in `agents/`, `tools/`, `schemas/`, `runs/`, `eval/`.

## Table of contents

| Path | Purpose |
|---|---|
| [`user_readme.md`](user_readme.md) | Personal cheat sheet — preferences, what to remember, next actions |
| [`HANDOVER.md`](HANDOVER.md) | **Paste-ready prompt** — all decisions made; AI verifies facts + writes contracts, then stops |
| [`DECISIONS.md`](DECISIONS.md) | **All ADRs FINAL** (D-000–D-016) + remaining [VERIFY] factual checks |
| [`PRODUCT/product-brief.md`](PRODUCT/product-brief.md) | What we are building — vision, stack, storage invariants, out of scope |
| [`PRODUCT/tech-stack.md`](PRODUCT/tech-stack.md) | Full stack discussion — verdict, rejected alternatives, every library, resolved conflicts, agentic runtime libs |
| [`PRODUCT/feature-list.md`](PRODUCT/feature-list.md) | Complete feature list with IDs (CORE / MUST / HIGH / NICE / SYNC / **RAND**) |
| [`PRODUCT/random-generator.md`](PRODUCT/random-generator.md) | **Postman-style random generator** (RAND-01–RAND-08, gofakeit v7) — your addition |
| [`BUILD-PLAN.md`](BUILD-PLAN.md) | Phased backlog with feature IDs and gates |
| [`checklist.md`](checklist.md) | Non-negotiable readiness checklist |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Multi-developer workflow, naming, review rules |
| [`docs/01-orchestration-patterns.md`](docs/01-orchestration-patterns.md) | Manager + specialists, handoffs, hosted multi-agent |
| [`docs/02-execution-layer.md`](docs/02-execution-layer.md) | Agent harness, tool calling, agent loops |
| [`docs/03-concurrency.md`](docs/03-concurrency.md) | Fan-out/gather, the two "parallel" knobs |
| [`docs/04-observability.md`](docs/04-observability.md) | Traces, logs, evaluation |
| [`docs/05-state-and-memory.md`](docs/05-state-and-memory.md) | Shared vs private state, artifacts |
| [`docs/06-guardrails-and-security.md`](docs/06-guardrails-and-security.md) | Allow-lists, sandboxing, budgets, HITL |
| [`docs/07-failure-handling.md`](docs/07-failure-handling.md) | Timeouts, retries, partial merges, breakers |
| [`docs/08-stack-options.md`](docs/08-stack-options.md) | Stack choice — **DECIDED: Go + chi/zerolog** (A adapted) |
| [`docs/09-merge-and-evidence.md`](docs/09-merge-and-evidence.md) | Merge strategies and evidence gating |
| [`docs/10-starter-plan.md`](docs/10-starter-plan.md) | 1–2 week starter plan |
| [`docs/11-pitfalls.md`](docs/11-pitfalls.md) | Common mistakes that kill these projects |
| [`docs/12-go-agent-runtime.md`](docs/12-go-agent-runtime.md) | **Go runtime** — chi routes, components, lifecycle, library registry |
| [`docs/13-verbose-logging.md`](docs/13-verbose-logging.md) | **Verbose zerolog standard** — fields, event catalog, redaction |
| [`docs/14-library-policy.md`](docs/14-library-policy.md) | **Library-first policy** — decision test, approved registry |
| [`docs/references.md`](docs/references.md) | Where the reference numbers came from |
| `agents/` | (placeholder) manager + specialist contracts — must name library + log events |
| `tools/` | (placeholder) tool contracts — schemas, allow-lists, cost tags |
| `schemas/` | (placeholder) strict JSON I/O contracts |
| `runs/` | (placeholder) verbose trace / artifact storage layout |
| `eval/` | (placeholder) test tasks + scoring (incl. RAND tests) |

## The one-sentence north star

**One manager + 2–3 specialists, parallel fan-out with Go `errgroup`, one Verifier that blocks ungrounded claims, chi for the API, zerolog for verbose traces, libraries for everything, plus a gofakeit-powered Postman random generator** — that is the complete, decided plan. [1]
