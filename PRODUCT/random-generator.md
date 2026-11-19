# Random Generator for Postman — product + runtime spec

This feature is **not in the saved chat**. You requested it, so it is added here as a first-class feature with IDs `RAND-01`–`RAND-08`. Decision: **D-012** (library: `github.com/brianvoe/gofakeit/v7`, verified via Context7 `/brianvoe/gofakeit`). **[VERIFY]** exact Postman dynamic-variable names against current Postman docs.

## Why it matters
Postman users rely on dynamic variables like `{{$randomInt}}` and `{{$randomUUID}}` to generate test data on every run. Importing a Postman collection with those variables must not produce broken flows. A built-in generator also removes the need for ad-hoc JS code blocks just to make fake payloads.

## Two surfaces

### Surface A — Postman-compatible dynamic variables (RAND-01, RAND-02)
Any variable/expression context resolves `{{$random...}}` tokens at run time.

| Postman-style token | gofakeit function (v7) | Notes |
|---|---|---|
| `{{$randomInt}}` | `faker.Int()` / `IntRange` | integer |
| `{{$randomFloat}}` | `faker.Float64()` | float |
| `{{$randomBoolean}}` | `faker.Bool()` | boolean |
| `{{$randomUUID}}` / `{{$guid}}` | `faker.UUID()` | UUID v4 |
| `{{$randomFirstName}}` | `faker.FirstName()` | name |
| `{{$randomLastName}}` | `faker.LastName()` | name |
| `{{$randomFullName}}` | `faker.Name()` | full name |
| `{{$randomUserName}}` | `faker.Username()` | username |
| `{{$randomEmail}}` | `faker.Email()` | email |
| `{{$randomPassword}}` | `faker.Password()` | password |
| `{{$randomPhoneNumber}}` | `faker.Phone()` | phone |
| `{{$randomCity}}` / `{{$randomCountry}}` | `faker.City()` / `faker.Country()` | address |
| `{{$randomCompanyName}}` | `faker.Company()` | company |
| `{{$randomWord}}` / `{{$randomSentence}}` | `faker.Word()` / `faker.Sentence()` | text |
| `{{$randomDate}}` | `faker.Date()` | date |
| `{{$randomIP}}` | `faker.IPv4()` | network |
| `{{$timestamp}}` | `time.Now().Unix()` (standard library) | not a gofakeit function |

- **RAND-01** — resolver in the variable system: detects `{{$random...}}`, calls gofakeit, substitutes value, logs `random.variable.resolved` verbosely (token, resolved field name, seed mode — never the secret-like value unless safe).
- **RAND-02** — unknown-token behavior: leave token unchanged and emit a verbose warning + mark output `unverified` (never silently invent a value).

### Surface B — Random Generator block / panel (RAND-03 … RAND-08)
A dedicated block on the canvas (and a side panel) that generates structured fake data.

| ID | Feature | Done-when |
|---|---|---|
| RAND-03 | Generator block with **type picker** (person, company, address, internet, phone, date, finance, lorem, etc.) | User picks a category; output appears |
| RAND-04 | **Schema-driven payload generation** — paste a JSON schema or example JSON; generate a matching fake object | Generated JSON validates against the schema |
| RAND-05 | **Seed control** — fixed seed for reproducible runs (`gofakeit.New(seed)` / `gofakeit.Seed(n)`), seed 0 = crypto-random | Same seed → identical output; seed 0 → different each run |
| RAND-06 | **Count / batch mode** — generate N records as a JSON array | `count` produces exactly N items |
| RAND-07 | **Export / reuse** — send generated data into a request body block, variable, or `.json` file in the workspace | One click wires generator output into a downstream block |
| RAND-08 | **Verbose generation log** — every generation event logged with type, count, seed mode, duration, field names | `runs/<id>/trace.jsonl` contains `random.generated` events |

## Runtime contract
- Package: `internal/random` in the Go agent runtime; also exposed to product code blocks.
- Library: `gofakeit/v7`. Use a **per-run Faker instance** with the run's seed (`gofakeit.New(seed)`) so results are reproducible and isolated between concurrent runs. `seed 0` → crypto-random per run.
- Concurrency: gofakeit `New` uses a locked PCG generator (thread-safe) — safe under fan-out.
- Schemas: add `schemas/random_generator.schema.json` (input: `type`, `count`, `seed`, `schema`/`example`, `locale`) and `schemas/random_output.schema.json` (output: `records`, `seed_used`, `duration_ms`).
- Logs: emit `random.variable.resolved` and `random.generated` events per `docs/13`, with full verbose detail and redaction rules applied.
- Budget tag: `random` tools are `low` cost; still counted and logged.

## Example flow (Postman import → generator → request)
```
[cURL import] → [Random Generator: 10 users, seed=42] → [Request body builder] → [REST call] → [Response inspector]
```
A collection containing `{{$randomEmail}}` imports cleanly: the token becomes a dynamic variable resolved by RAND-01 using the run seed.

## Testing (`eval/`)
- 10+ tasks: each Postman token resolves to the correct type; unknown token warns; same seed reproducible; seed 0 non-reproducible; schema-driven output validates; batch count exact; export wires into body builder; verbose log events present; redaction works; concurrent runs isolated.

## Placement in the plan
Added to **Phase 1 (MVP)** in `BUILD-PLAN.md` because Postman import (MUST-05) depends on RAND-01/RAND-02. RAND-03–RAND-08 land in Phase 1/2 per the backlog.

## Open factual check (not a decision)
- **[VERIFY]** Exact Postman dynamic-variable names/semantics from current Postman docs; map any extras to gofakeit functions or add a small, documented adapter (library-first: prefer gofakeit's built-ins).
