# 09 — Merge Strategies and Evidence Gating

*"No merge strategy" and "no observability" are two of the classic project-killers. This doc covers the first.*

## Why this matters
Fan-out produces several candidate outputs. Without explicit rules you get conflicting answers, and a verifier becomes the thing that decides what survives. A good default is **manager synthesis**, optionally backed by a verifier.

## Merge strategies (pick one, document it)
| Strategy | How it works | When to use |
|---|---|---|
| **Manager synthesis** | Manager reads all specialist outputs and writes the final answer | Chosen (D-001/D-016); pairs with Manager + Specialists |
| **Voting** | Multiple outputs; majority wins | Objective, checkable answers |
| **Ranking** | A judge ranks candidates; top wins | When quality is comparable but subjective |
| **Verifier gate** | A verifier blocks unsupported claims before merge | Any domain where hallucination is costly |

You can combine these: **verifier gate → manager synthesis** is the **chosen** merge rule (D-001/D-016), not merely a recommendation.

## Evidence gating — make it impossible to make up facts silently
- Require **citations / tool output IDs** for claims.
- Add a **verifier agent** that checks: "Is this supported?"
- Label outputs: **verified / unverified / assumption**.
- This has a *huge* practical difference in production quality.

```
specialist outputs
        │
        ▼
  ┌───────────────┐
  │  VERIFIER     │  checks each claim: supported? cited?
  └───────┬───────┘
          ▼
   verified / unverified / assumption
          ▼
   ┌──────────────┐
   │   MANAGER    │  synthesizes only from verified material
   └──────────────┘
          ▼
     final answer
```

## Contract requirement
The verifier's contract in `agents/` must define:
- what counts as "supported" (citation present, tool-output ID present, etc.),
- what it does with unsupported claims (block / label / downgrade),
- how it reports back to the manager.

Related reading: [parallel specialists then merge [7]], [evidence gating step 3 of starter plan].
