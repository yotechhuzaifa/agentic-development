# `tools/` — tool contracts (placeholder, no code yet)

One contract per tool. Tools must be deterministic and boring. See `docs/02-execution-layer.md`.

## Planned files (example starter set)
| File | Purpose |
|---|---|
| `web_search.md` | Gather sources / evidence |
| `db_query.md` | Read from the database |
| `code_exec.md` | Run code in a sandbox |

## Required sections in every tool contract
- Strict JSON input/output schema (in `schemas/`).
- Idempotency notes.
- Input validation rules.
- Safe, recoverable, structured error messages (no secrets / raw stack traces).
- Rate-limit / cost tag.
- Sandboxing or HITL flag if risky.
- Which agents are allow-listed to call it.
- **Library used** for I/O (no raw HTTP/curl/random — `docs/14`); gofakeit v7 for any random data (D-012).
- **Log events** emitted from `docs/13` (`tool.call.*`, full verbose I/O, redacted).
- **Budget tag** and timeout/retry values from D-014.

Random-generator tools must also reference `PRODUCT/random-generator.md` (RAND-01–RAND-08) and the schemas in `schemas/`.

Add new tools only via the workflow in `CONTRIBUTING.md`.
