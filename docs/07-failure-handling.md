# 07 — Failure Handling

*Parallel systems fail more often — plan for it rather than being surprised by it.* [10]

## Required mechanisms
- [ ] **Per-agent timeout** — cap each specialist; gather must not wait forever on one branch.
- [ ] **Per-tool retry policy with backoff** — retries must be bounded and idempotent (see `02-execution-layer.md`).
- [ ] **Partial-result merging ("use what you got")** — if one specialist fails, decide whether the manager proceeds with the remaining results and labels the gap.
- [ ] **Circuit breaker** — stop spawning agents when confidence is low, the budget is exhausted, or failures cascade.
- [ ] **Durable sessions / recovery** — long-running runs must survive interruptions and resume from checkpoints. [10]

## Behavior matrix (fill in during implementation)
| Failure | Detection | Retry? | Fallback | Label on output |
|---|---|---|---|---|
| Agent timeout | timeout fires | No (or once) | use partial results | `unverified` + gap noted |
| Tool transient error | error class | Yes, backoff | alternate tool | note retry count |
| Tool validation error | schema reject | No | request corrected args | `unverified` |
| Low-confidence branch | verifier score | No | block claim | `unverified` / `assumption` |
| Budget exhausted | budget guard | No | stop spawning | `partial run` |

## Recovery design
Checkpoint shared state after each successful step (see `05-state-and-memory.md`). On resume, skip already-completed steps rather than re-running them — this avoids duplicate writes and wasted spend. Store enough in `runs/` to reconstruct where the run stopped. [10]

Related reading: [durable sessions and recovery [10]].
