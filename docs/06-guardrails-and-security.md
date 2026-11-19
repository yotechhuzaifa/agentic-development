# 06 — Guardrails and Security (especially with parallel execution)

*Parallelism multiplies the surface area: more agents calling more tools at the same time means more ways to do the wrong thing, faster.* [9]

## Must-haves
- [ ] **Tool allow-lists per agent** — a specialist can only call the tools it needs. No agent gets "all tools."
- [ ] **Sandboxing** for code execution / risky tools.
- [ ] **Rate limits + budgets per run** — enforce at the orchestrator level across all concurrent agents.
- [ ] **Human-in-the-loop (HITL)** for high-impact actions (spend, writes to production, external sends, deletions).
- [ ] **Secure plan/execute separation** — planning agents should not directly perform privileged actions; an executor with a narrow allow-list performs them, ideally behind HITL. [9]

## Why allow-lists matter more in parallel
When several agents run concurrently, a broad tool grant means any one of them can trigger a side effect while others are mid-flight — making failures hard to attribute and roll back. Narrow grants make behavior attributable to a single agent and a single trace entry.

## Budget control
- Set a **per-run** cap and a **per-agent** cap.
- Tag every tool with a cost/rate profile (see `tools/` contracts).
- Stop spawning new agents when the budget threshold is hit (pairs with the circuit breaker in `07-failure-handling.md`).
- Parallelism multiplies spend — without caps this is the fastest way to a surprise bill.

## HITL trigger examples (customize for your domain)
- Any tool that spends money or calls a paid external API above a threshold.
- Any write to a production datastore.
- Any outward-facing communication.
- Any code execution with file/network access.

Record the final HITL list in `DECISIONS.md`. Research literature and engineering guides repeatedly highlight HITL and secure plan/execute patterns as crucial for resilient agent systems. [9]
