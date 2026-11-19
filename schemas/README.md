# `schemas/` — strict JSON contracts (placeholder, no code yet)

Every agent input/output and every tool input/output gets a strict JSON schema here. Naming: `<agent-or-tool-name>.schema.json`, plus a final-output schema (e.g. `final_output.schema.json`).

## Why strict schemas matter
- Reduce agent flakiness more than prompt tuning.
- Enable strong input validation before a tool does work.
- Provide a stable recovery contract for failed agents.
- Make outputs machine-checkable for observability and evaluation.

## Rules
- No loose/ambiguous fields; define types, required fields, and enums.
- Include explicit status/label fields (e.g. `verified | unverified | assumption`).
- Version breaking changes (record in `DECISIONS.md`).
- Include `random_generator.schema.json` and `random_output.schema.json` for the RAND feature (`PRODUCT/random-generator.md`).
- Validate with the approved JSON-schema library **[VERIFY module path]** (`docs/12.1`); never hand-roll validation (`docs/14`).
- Schemas are verbose: describe every field, its purpose, allowed values, and an example — not just types.

See `docs/02-execution-layer.md` and `docs/09-merge-and-evidence.md`.
