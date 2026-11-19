# 03 — Concurrency: Parallel Without Chaos

*Parallel agentic systems fail in boring ways: timeouts, rate limits, tool collisions, duplicate writes.* [4]

## The two different "parallel" knobs — do not conflate them

1. **Model emits multiple tool calls in one response** — provider-side *parallel tool calls*. [4]
2. **Your app/SDK executes multiple tool calls concurrently** — function/tool concurrency. [4]

You must handle both sides. A model that emits three tool calls in parallel gains nothing if your runtime executes them serially; a runtime that executes concurrently is useless if the model never batches.

## Fan-out → gather
Give the same input to multiple specialists in parallel, then merge. Common fan-out prompts: [7]
- "Generate 3 solution options"
- "Find risks & edge cases"
- "Estimate cost/time"
- "Write final draft"

```
        ┌──────────────┐
        │   MANAGER    │
        └──────┬───────┘
     fan-out (async)
  ┌──────┬──────┬──────┐
  ▼      ▼      ▼      ▼
 opt A  risks  cost  draft     (run concurrently)
  └──────┴──┬───┴──────┘
       gather / merge
           ▼
        final answer
```

If you build multi-agent yourself, implement parallel fan-out with an async/concurrency library. The chat's example is `asyncio.gather` in Python [5]; **our final choice is Go `errgroup` + `semaphore` (D-006)** — see `docs/12-go-agent-runtime.md`. The pattern (fan-out → gather) is the same; the library is the one that fits our Go, library-first runtime.

## Controls you must have
- **Per-agent timeouts** — one slow specialist must not hang the gather.
- **Rate limits** — provider and tool rate limits across *all* concurrent callers, not per agent in isolation.
- **Tool collision / duplicate-write prevention** — define which agent may write which shared key; ideally make writes idempotent.
- **Bounded fan-out** — cap the number of concurrent agents/tools (start with 2–4 agents [1]).
- **Budget guard** — parallelism multiplies spend; enforce per-run and per-agent caps.

## What to record in `DECISIONS.md`
- Max concurrent agents and max concurrent tool calls.
- Which shared-state keys each agent may write (prevents duplicate writes).
- The gather function's merge rule (see [`09-merge-and-evidence.md`](09-merge-and-evidence.md)).

Related reading: [parallel tool calls [4]], [async fan-out example [5]].
