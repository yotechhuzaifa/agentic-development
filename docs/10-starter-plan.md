# 10 — Starter Plan (1–2 Weeks)

*Start small, prove the pattern, then add the verifier. Do not begin with ten agents.*

## Step 1 — One manager + 2–3 specialists
Use a simple division of labor: [6]
- **Research agent** — gathers sources / evidence.
- **Planner agent** — decomposes the task and creates a subtask list.
- **Executor agent** — runs tools (DB, code, actions).
- **Manager** — merges and produces the final response.

This matches the manager pattern in OpenAI's guidance. Keep it to 2–4 agents total early on. [1]

## Step 2 — Add fan-out → gather (parallel specialists)
Give the same input to multiple specialists in parallel, then merge: [7]
- "Generate 3 solution options"
- "Find risks & edge cases"
- "Estimate cost/time"
- "Write final draft"

Implement the fan-out with **Go `errgroup` + `semaphore`** (D-006; the chat's Python equivalent is `asyncio.gather`) and make the merge rule explicit (verifier gate → manager synthesis, D-001/D-016). See `docs/12-go-agent-runtime.md` and `09-merge-and-evidence.md`. This "parallel specialists then merge" approach is a common multi-agent pattern.

## Step 3 — Add evidence gating and verification
Make it impossible for agents to make up facts silently:
- require citations / tool output IDs for claims,
- add a **verifier agent** that checks "Is this supported?",
- label outputs: **verified / unverified / assumption**.

This is the step that most improves production quality — do not skip it.

## High-value next step (start here)
> Use the **Manager + Specialists (agents-as-tools)** pattern, implement **parallel fan-out with async**, and add **one Verifier agent that blocks ungrounded claims**. That combination usually gets you ~80% of the value of parallel agentic development with manageable complexity.

## Definition of done for the starter phase
- [ ] Manager + 2–3 specialists running with the chosen pattern.
- [ ] Async fan-out with bounded concurrency and timeouts.
- [ ] Explicit merge rule implemented.
- [ ] Verifier blocking ungrounded claims; outputs labeled.
- [ ] Observability fields captured for every run (see `04-observability.md`).
- [ ] Per-run and per-agent budgets enforced.
- [ ] `eval/` has 20–100 tasks and a baseline score.
- [ ] All choices recorded in `DECISIONS.md`.
