# `runs/` — trace and artifact storage (placeholder, no code yet)

Intended layout for a single run:

```
runs/
  <run-id>/
    trace.jsonl        # ordered events: agent msgs, tool I/O, timestamps, retries
    metadata.json      # goal, constraints, model, budget, labels
    artifacts/         # documents, JSON outputs, code patches
    summary.json       # final output + merge decision + verification labels
```

## Purpose
- Make every run **replayable** — if you can't replay and see tool calls, you can't improve the system.
- Capture the observability fields from `docs/04-observability.md`: transcript per agent, tool I/O, timestamps + retries, per-agent cost/tokens, success/failure labels.
- Be **verbose** (D-013): follow the mandatory field set and event catalog in `docs/13-verbose-logging.md`. `trace.jsonl` is one JSON object per event with `run_id`, `agent_id`, `span_id`, `request_id`, full redacted `input`/`output`, tokens/cost, duration, status, retry count, and verification label.
- Store `summary.json` with the verbose run close-out: totals, verification label counts, merge decision, HITL decisions, and random-generation seed/type info when RAND features ran.

## Decision (final)
Local files + SQLite index (D-007), WAL mode; no external DB. Privacy/retention: redact secrets before writing (`docs/13`), and define a retention policy for transcripts in a future ADR.
