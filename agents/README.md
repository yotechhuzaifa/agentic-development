# `agents/` — agent contracts (placeholder, no code yet)

This folder will hold **one contract per agent**. Nothing here is implemented yet. Each contract is the spec that a future implementation must follow.

## Planned files
| File | Role |
|---|---|
| `manager.md` | Owns the final output; fans out to specialists and merges (with verifier gate) |
| `research_agent.md` | Gathers sources / evidence |
| `planner_agent.md` | Decomposes task into subtasks |
| `executor_agent.md` | Runs tools (DB, code, actions) |
| `verifier_agent.md` | Blocks ungrounded claims; labels verified / unverified / assumption |

## Required sections in every contract
- **Role** — one sentence.
- **Inputs** — typed fields (link `schemas/`).
- **Tools allowed** — explicit allow-list, never "all tools".
- **Output** — exact shape + done/failed signal (link `schemas/`).
- **Shared state read / written** — exact keys.
- **Private scratch** — what stays local.
- **Failure behavior** — timeout, retries, partial-result shape.
- **Escalation** — when to hand back to manager or request HITL.
- **Library used** — which approved library (`docs/12.1`) owns its I/O; no raw coding (`docs/14`).
- **Log events emitted** — from the verbose catalog in `docs/13`, with mandatory fields.
- **Budget tag** — token/cost class from D-014.

All choices are final in `DECISIONS.md`; contracts must reflect Manager + Specialists + Verifier (D-001), Go runtime (D-002), chi/zerolog/errgroup (D-003–D-006), and verbosity (D-013).

See `CONTRIBUTING.md` for the full rules and `docs/01`, `docs/05`, `docs/09` for the reasoning.
