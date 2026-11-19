# 01 — Orchestration Patterns

*The hard part is coordination, not prompts. Pick one pattern and commit early — you can evolve later.* [1][2]

Without a crisp coordination scheme, teams end up with **agents talking in circles**, **duplicated work**, and **unpredictable costs**.

## Pattern A — Manager + Specialists ("agents as tools") — CHOSEN (D-001)
- One **main agent** stays responsible for the **final output**.
- It calls specialist agents **like tools**, then merges their results.
- Often the best default for parallel work. [2]

```
             ┌──────────────┐
  input ───► │   MANAGER    │  (owns final answer)
             └──────┬───────┘
        fan-out     │      fan-out
       ┌────────────┼────────────┐
       ▼            ▼            ▼
 ┌──────────┐  ┌──────────┐  ┌──────────┐
 │Research  │  │ Planner  │  │ Executor │  (specialists)
 └────┬─────┘  └────┬─────┘  └────┬─────┘
       └────────────┼────────────┘
              gather / merge
                    ▼
             final output
```

**Use when:** you want explicit control over who owns the answer and a clean merge point. Day-one starter roles: **Research** (gathers sources/evidence), **Planner** (decomposes task + subtask list), **Executor** (runs tools/DB/code/actions). [6]

## Pattern B — Handoffs / routing
- A **triage agent** decides "who should own this task" and hands off to a specialist.
- Best when tasks are **clearly separable into categories**. [2]

**Use when:** incoming work falls into distinct buckets and one agent shouldn't do everything. Note: classic handoff is a *routing* pattern; it is not automatically parallel compute.

## Pattern C — Hosted Multi-agent
- The platform's "Multi-agent" capability spins up subagents in parallel and synthesizes, so you implement less orchestration yourself. [3]

**Use when:** you want parallel subagents with less custom orchestration code. Trade-off: less control over internals and debugging detail.

## Choosing — decision criteria
| Criterion | Manager + Specialists | Handoffs | Hosted Multi-agent |
|---|---|---|---|
| Who owns final output | Explicit manager | Final specialist | Model synthesis |
| Control over merge | High | Medium | Lower |
| Custom code to write | More | Medium | Less |
| Best fit | General parallel work | Clean task categories | Speed to parallel with less code |

Record the choice in `DECISIONS.md`. Related reading: [orchestration + handoffs + guardrails [1]], [manager pattern [6]], [multi-agent patterns [5]].
