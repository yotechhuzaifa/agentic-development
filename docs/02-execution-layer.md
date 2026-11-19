# 02 — Tool Calling and the Execution Layer (Agent Harness)

*Parallel agents are only useful if they can reliably **do things** — search, code, DB queries, ticket creation, etc.* [1]

OpenAI's Agents platform and SDK documentation emphasizes four things your harness must provide: [1]
1. **Agent loops** — the observe → think → act → observe cycle.
2. **Tool calling** — a stable way for the model to request a tool with typed arguments.
3. **Orchestration & handoffs** — who runs when, and who takes ownership.
4. **A runtime that handles retries / multi-step execution** — durable enough to survive partial failure.

## Making tools something agents can trust
Tools are where "agent flakiness" is born or killed. Specify each one in `tools/` + `schemas/` with:

- **Strict JSON schemas** — no loose/ambiguous inputs or outputs.
- **Idempotent where possible** — a retried call should not double-apply.
- **Strong input validation** — reject bad arguments before the tool does work.
- **Safe error messages** — structured, actionable, and safe for the model to read and recover from; never leak secrets or raw stack traces.

> Making tools boring reduces flakiness more than prompt tuning does.

## Tool contract checklist
- [ ] Strict JSON input/output schema in `schemas/`
- [ ] Idempotency documented
- [ ] Input validation documented
- [ ] Recoverable, safe error format
- [ ] Rate-limit / cost tag attached
- [ ] Sandboxing or HITL flag if risky
- [ ] Allow-listed to only the agents that need it

## Failure-aware execution
The runtime must support: retries with backoff, multi-step execution state, and recovery of long-running runs. See [`07-failure-handling.md`](07-failure-handling.md) and the "durable sessions and recovery" guidance. [10]

Related reading: [running agents + concurrency knobs [4]], [multi-agent patterns in the SDK repo [5]].
