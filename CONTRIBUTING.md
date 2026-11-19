# Contributing — multi-developer rules

This project is documentation-first. These rules keep a growing team from producing conflicting agents, duplicate tools, and unreviewed behavior changes.

## Golden rule
**No agent or tool gets implemented until it has a contract file in `agents/` or `tools/`.** The contract is the spec; code is downstream of it.

## Workflow
1. Propose a change as a short note in `DECISIONS.md` (what, why, alternatives, decision).
2. Once agreed, add/update the contract file in `agents/` or `tools/` (and its JSON schema in `schemas/`).
3. Only then implement. Keep the implementation's behavior identical to the contract.
4. Update [`checklist.md`](checklist.md) if the change affects readiness.
5. Add or update at least one eval task in `eval/` that exercises the change.

## Naming conventions
- Agent contract files: `agents/<role>_agent.md`, e.g. `agents/research_agent.md`. The manager is `agents/manager.md`.
- Tool contract files: `tools/<verb>_<noun>.md`, e.g. `tools/web_search.md`, `tools/db_query.md`.
- Schema files: `schemas/<same-name>.schema.json` matching the agent/tool or final output name.
- One role per agent. Resist "utility" agents that do everything.

## What every agent contract must state
- **Role** — one sentence, what this agent owns.
- **Inputs** — the typed fields it receives (link the schema).
- **Tools allowed** — explicit allow-list (never "all tools").
- **Output** — exact shape (link the schema) and how it signals done/failed.
- **Shared state written** — which keys it may write (and which it may only read).
- **Failure behavior** — timeout, retry expectation, what a partial result looks like.
- **Escalation** — when it must hand back to the manager or request HITL.

## What every tool contract must state
- Strict JSON input/output schema (in `schemas/`).
- Idempotency notes.
- Validation rules.
- Safe, recoverable error messages (never raw secrets or stack traces into the model).
- Rate limit / cost tag (used by the budget guard).
- Whether it needs sandboxing or HITL.

## Review checklist (for whoever reviews a PR)
- [ ] Change matches an entry in `DECISIONS.md`.
- [ ] Contract file updated before/alongside behavior.
- [ ] Tool allow-list is as narrow as possible.
- [ ] Observability fields (transcript, tool I/O, tokens, timestamps, labels) still captured.
- [ ] Budget/rate-limit impact considered.
- [ ] Merge behavior still deterministic and documented.
- [ ] Eval task added/updated.

## Keeping it scalable
- Prefer **adding a specialist** over making one agent do more, but only up to 2–4 agents early on. [1]
- Share only the state an agent needs — not the entire context. [1]
- Keep tools deterministic and boring; fix flakiness in the tool layer before tweaking prompts.
- If a decision is expensive to reverse (security, provider, data model), it must be an explicit ADR in `DECISIONS.md`.
