# References — where the numbers came from

The reference numbers in the docs (e.g. `[1]`, `[4]`) come from the footnotes in the original AI conversation you saved. They are kept as pointers so the team can recover the original source when needed.

**Primary source:** `Directly Chat with Frontier AI Search Models (11_19_2026 6：49：15 PM).html`
**Original chat:** https://arena.ai/c/01a1f3bc-7051-7394-90e3-20b1f1e092a7

## Mapping (from the guide you supplied)
| # | Topic in the guide | Notes |
|---|---|---|
| 1 | OpenAI Agents: orchestration + handoffs + tool calling + traces/guardrails | Used throughout as the core framing |
| 2 | Manager pattern / handoffs (agents-as-tools, triage routing) | See `01-orchestration-patterns.md` |
| 3 | OpenAI hosted "Multi-agent" capability | See `01`, `08` |
| 4 | Parallel tool calls (model-side) vs function/tool concurrency (app-side) | See `03` |
| 5 | Async fan-out (e.g. `asyncio.gather`) / SDK multi-agent patterns | See `02`, `03`, `08` |
| 6 | Manager pattern / research-planner-executor division | See `01`, `10` |
| 7 | Parallel specialists then merge (fan-out/gather pattern) | See `03` |
| 8 | LangGraph-style typed state, checkpoints, long-running workflows | See `05` |
| 9 | HITL + secure plan/execute patterns, secure agent systems | See `06` |
| 10 | Durable sessions and recovery for long-running runs | See `07` |
| 11 | Agents overview + orchestration + handoffs | See `08` |
| 12 | Cookbook multi-agent "portfolio collaboration" example | See `08` |
| 13 | AutoGen group chat (typically sequential turn-taking, not true parallel) | See `08` |

> **Note:** Some of the original footnotes pointed to third-party articles surfaced by the AI search models (e.g. desktop-framework articles embedded in the saved HTML). Treat those as illustrative context, not authoritative documentation. For implementation, verify against current official docs (OpenAI Agents SDK, LangGraph, AutoGen) via Context7 when the time comes.

This file is intentionally a pointer map, not a substitute for reading the official docs. When a decision in `DECISIONS.md` depends on a factual API claim, verify it against live docs before committing.
