# 08 — Choosing a Stack — DECIDED

The chat offered three options. The decision is **final** in D-005. This doc keeps the comparison as the record of *why*, then states what was chosen.

## The chat's options (context)
| | Control | Orchestration code | True parallel by default | Fit |
|---|---|---|---|---|
| A — OpenAI Agents SDK + async | High | More | You implement it | Product / control |
| B — Hosted multi-agent | Lower | Less | Yes | Speed to parallel |
| C — AutoGen group chat | Medium | Medium | **No** (sequential turn-taking) [13] | Conversation patterns |

## Decision — Option A, adapted to Go
- **Rejected B** (hosted multi-agent): less control over traces, budgets, internal behavior; no zerolog-level observability.
- **Rejected C** (AutoGen): group chat is sequential by default, and Python.
- **Rejected A-as-written** (Python + Agents SDK + `asyncio.gather`): splits the project into two languages and discards the Go/zerolog/library-first stack.
- **Chosen: in-house Go orchestration with libraries** — same control as Option A, but implemented in Go with the approved registry (`docs/12.1`, D-002–D-007).

## What that means concretely
| Concern | Chosen | Decision |
|---|---|---|
| UI ↔ Go transport | **Wails bound methods**, no HTTP router | D-018 |
| Logging | **zerolog**, verbose, redacted | D-004, D-013 |
| LLM calls | Official **OpenAI Go SDK** **[VERIFY path]** | D-005 |
| Fan-out/gather | **errgroup + semaphore** (Go, not `asyncio.gather`) | D-006 |
| Traces | **modernc.org/sqlite** + JSONL | D-007 |
| Random data | **gofakeit v7** (product RAND + eval fixtures) | D-012 |
| Product stack | Wails v3 + Go + Svelte 5 + Svelte Flow | D-008–D-011 |

## Verification note
Provider APIs change fast. Before implementation, verify current docs (Context7 / official sites) for: handoffs/parallel tool calls/tracing/durable execution in the chosen LLM SDK, and the remaining **[VERIFY]** items in `DECISIONS.md`. Wails bindings, zerolog, gofakeit, sqlite, pgx and go-git were already verified. The reference numbers here point to the original conversation's sources, not necessarily current documentation. **[VERIFY]** items are factual checks — they do not re-open this decision.

Full runtime design: [`12-go-agent-runtime.md`](12-go-agent-runtime.md). Library policy: [`14-library-policy.md`](14-library-policy.md).
