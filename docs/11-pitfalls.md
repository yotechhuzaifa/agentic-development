# 11 — Common Mistakes That Kill Parallel Agent Projects

Revisit this list at every design review. [1]

## 1. Making too many agents too early
Start with **2–4**. More agents usually increase confusion and cost faster than they increase quality.

## 2. Letting all agents share the full context
This increases token usage and cross-contamination. Share only what each agent needs (see `05-state-and-memory.md`).

## 3. No merge strategy
You need explicit rules — **voting, ranking, verifier, or manager synthesis**. Otherwise outputs conflict. See `09-merge-and-evidence.md`.

## 4. No observability
If you can't replay runs and see tool calls, you can't improve the system. See `04-observability.md`.

## 5. No cost controls
Parallelism multiplies spend. Add budgets per run and per agent. See `06-guardrails-and-security.md`.

## Bonus anti-patterns worth watching
- **Assuming "multi-agent" means parallel compute** — AutoGen group chat is often sequential by default. [13]
- **Confusing the two parallel knobs** — model-side batched tool calls vs app-side concurrent execution. [4]
- **Fixing flakiness with prompt tuning** instead of making tools deterministic (schemas, validation, idempotency).
- **Skipping evidence gating** — then hallucinated claims flow straight into the final answer.
- **Building without typed state / checkpoints** — long runs become unrecoverable. [8]
