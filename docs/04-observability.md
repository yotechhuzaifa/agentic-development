# 04 — Observability: Traces, Logs, Evaluation

*"It seemed smart yesterday" is not enough. You cannot debug vibes.* [1]

OpenAI's docs treat "orchestration" as including multi-step tool use + handoffs + guardrails; debugging requires visibility into those steps. [1]

## Capture this for every run (non-negotiable)
- [ ] **Full transcript per agent** — every message in/out.
- [ ] **Tool call inputs AND outputs** — what was requested and what came back.
- [ ] **Timestamps + retries** — when each step happened, how many attempts.
- [ ] **Per-agent cost / tokens** — so parallelism doesn't silently blow the budget.
- [ ] **Success / failure labels** — so you can compute a quality signal over time.

## Storage layout (placeholder `runs/`)
```
runs/
  <run-id>/
    trace.jsonl        # ordered events: agent msgs, tool I/O, timestamps, retries
    metadata.json      # goal, constraints, model, budget, labels
    artifacts/         # documents, JSON outputs, code patches
    summary.json       # final output + merge decision + verification labels
```

Design goal: **any run must be replayable** from stored traces. If you can't replay runs and see tool calls, you can't improve the system. [1]

## Evaluation (`eval/`)
- Keep **20–100 realistic test tasks** with scoring scripts.
- Score not just final answers but process signals: correct tool use, adherence to allow-lists, evidence coverage, cost per task.
- Re-run the eval set after every agent/tool change (see `CONTRIBUTING.md`).

## Dashboards / reviews to plan for
- Cost per run and per agent (trend it; alert on spikes).
- Failure and retry rates by tool.
- Verified vs unverified claim ratio (evidence gating health).
- Latency of the slowest specialist (gather is only as fast as its slowest branch).

Related reading: [orchestration includes tool use + handoffs + guardrails [1]], [durable sessions [10]].
