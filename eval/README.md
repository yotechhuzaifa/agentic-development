# `eval/` — evaluation tasks and scoring (placeholder, no code yet)

Keep **20–100 realistic test tasks** with scoring scripts.

## What to score
- Final answer quality.
- Correct tool use and adherence to allow-lists.
- Evidence coverage (verified vs unverified claims).
- Cost per task and latency.
- Failure/retry behavior.

## Rules
- Add or update at least one eval task for every agent/tool change (`CONTRIBUTING.md`).
- Re-run the whole set after changes to get a comparable baseline.
- Store per-task labels so trends are visible over time.

- **RAND tasks required:** verification of all Postman dynamic-variable tokens (RAND-01), unknown-token warning (RAND-02), schema-driven output (RAND-04), seed reproducibility vs seed 0 (RAND-05), batch count (RAND-06), export wiring (RAND-07), and verbose log events (RAND-08). Use **gofakeit v7** with a fixed seed for deterministic fixtures (D-012).

Purpose: observability and evaluation are what let you improve the system instead of guessing. See `docs/04-observability.md` and `docs/13-verbose-logging.md`.
