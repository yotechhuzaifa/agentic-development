# user_readme — for you

**Project name: AntHill** (D-017). Folder is now `/home/yotech70/AntHill/`.

Your personal sheet. Everything is **decided** — you no longer need to choose an architecture. Your remaining job is to verify a short factual list, then paste `HANDOVER.md` to an AI and approve its Phase 0 contracts.

## Your preferences — now locked in as decisions
| You said | Where it is locked |
|---|---|
| "use library rather raw coding" | D-005, D-006 + [`docs/14-library-policy.md`](docs/14-library-policy.md) — library registry in `docs/12.1` |
| "use chi library for routing" | D-018 — **removed as redundant**: Wails bindings handle UI↔Go; no HTTP router |
| "use zerolog for logging" | D-004 — verbose JSON, redaction, event catalog in `docs/13-verbose-logging.md` |
| "i like verbose" | D-013 — verbose logs, traces, errors, docs, CLI; `--quiet` only affects console |
| "randomGenerator for postman" | D-012 — gofakeit v7; full spec in [`PRODUCT/random-generator.md`](PRODUCT/random-generator.md) (RAND-01–RAND-08) |
| "choose the best options" | `DECISIONS.md` D-000–D-016 — **all accepted**, alternatives + reasons recorded |

## The product in one line
A lightweight desktop visual flow app for API + database workflows ("Postman Flows meets database workflows, in a lightweight desktop app"), built with Wails v3 + Go + Svelte 5, plus your added Postman-style random generator.

## The three parts of this kit
- **Method (how):** `docs/01`–`docs/14`, `checklist.md`, `CONTRIBUTING.md`.
- **Product (what):** `PRODUCT/product-brief.md`, `PRODUCT/tech-stack.md`, `PRODUCT/feature-list.md`, `PRODUCT/random-generator.md`, `BUILD-PLAN.md`.
- **Handoff (to AI):** `HANDOVER.md`.

## Things to keep in mind
1. **Nothing is left to decide architecturally.** Don't re-litigate `DECISIONS.md`; the AI is told not to either. To change something, write a new ADR that marks the old one superseded.
2. **Library-first, but only what is needed** (`docs/14`, D-019): use an approved library before writing custom code. zerolog, errgroup, gofakeit, LLM SDK, uuid, sqlite, pgx, go-git, jsonschema — UI↔Go via Wails bindings. No HTTP router, no custom logger, no hand-rolled random strings.
3. **Verbose is deliberate.** Big logs and big docs are the point, not a bug. Redaction keeps them safe (never log secret values).
4. **Verifier is mandatory.** Every claim needs a citation/tool-output ID; outputs labeled verified / unverified / assumption.
5. **Bound everything.** Concrete budgets/timeouts/HITL numbers are in D-014 / D-015 (e.g. 4 agents, 8 tools, 200k tokens/run, 60s agent timeout, 3 retries, breaker at 3 failures or 80% budget).
6. **Two conflicting stack threads in the chat were resolved** (D-011): use Wails + Go + go-git + go-keyring; ignore the chat's Tauri/Rust tools and the later Tauri stack table.
7. **The random generator is yours, not the chat's.** It is in Phase 1 because Postman import needs RAND-01/RAND-02 to resolve `{{$random...}}` variables.

## Remaining factual checks — tell the AI to verify, don't guess
These are in `HANDOVER.md` and `DECISIONS.md` §Factual checks:
- Official OpenAI Go SDK module path (`github.com/openai/openai-go` preferred).
- Goja module path (`github.com/dop251/goja` vs chat's `nicois/goja`).
- JSON-schema validator module, Wails v3 release/syntax, library versions.
- Exact Postman dynamic-variable names for RAND-01.
- **[CONFIRM]** "flow template system" and "API templa..." from your checklist (truncated in saved chat).

## Your next actions, in order
1. Skim `DECISIONS.md` so you know what was decided on your behalf.
2. Read `PRODUCT/random-generator.md` — the one feature that came from you.
3. Tick `checklist.md` (or let the AI do it in Phase 0).
4. When ready, paste `HANDOVER.md` into an AI session.
5. The AI will verify the **[VERIFY]** items, write Phase 0 contracts (documentation only), and **stop**. Review and approve.
6. Then start Phase 1 of `BUILD-PLAN.md`, one phase at a time.

## Hard rule for this folder right now
**No runnable code yet.** Documentation, contracts, ADRs, and placeholder README folders only — exactly as you asked.

## Source
Saved chat: `/mnt/c/Users/yotech70/Downloads/Directly Chat with Frontier AI Search Models (11_19_2026 6：49：15 PM).html` (original: https://arena.ai/c/01a1f3bc-7051-7394-90e3-20b1f1e092a7). Additions from you: random generator, Go runtime, verbosity (chi was requested then removed as redundant, D-018). Library facts verified with Context7: `/wailsapp/wails`, `/rs/zerolog`, `/brianvoe/gofakeit`, `/websites/pkg_go_dev_modernc_org_sqlite`, `/jackc/pgx`, `/go-git/go-git`.
