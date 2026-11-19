# 14 — Library-First Policy

You asked to prefer libraries over raw coding. This doc makes that a rule with a decision test, so it survives across developers and AI agents.

## 14.1 The rule
For any non-trivial concern, use a **maintained, well-documented library** before writing custom code. Write custom code only when no suitable library exists, or when the logic is the product's actual differentiator.

## 14.2 Decision test (apply in order)
1. **Required behavior already exists in a chosen library?** Use it. Change nothing.
2. **A library in the approved registry (`docs/12.1`) covers it?** Use that library.
3. **Standard library / platform covers it well** (e.g. `context`, `net/http` types, `encoding/json` through chi/zerolog)? Use it.
4. **No library fits and the logic is small?** Write it in the layer that owns the invariant.
5. **Existing structure blocks clean ownership?** Propose a coherent refactor via ADR.

Never add a single-implementation interface, speculative extension point, config for a fixed value, or premature service just to look "architected."

## 14.3 Approved registry (summary)
See `docs/12.1` for the full table. Headlines:
- **chi v5** — all HTTP/JSON routing and middleware.
- **zerolog** — all logging, verbose by default, redaction enforced (`docs/13`).
- **errgroup / semaphore** — all fan-out, gather, and concurrency caps.
- **official OpenAI Go SDK** — all LLM calls and tool-call parsing **[VERIFY module path]**.
- **modernc.org/sqlite** — run index and app state; JSONL for traces.
- **gofakeit v7** — random generator (D-012) and eval fixtures.
- **uuid** — all correlation IDs.
- **jsonschema validator** — strict tool I/O validation **[VERIFY module path]**.
- Product side: Wails v3, Svelte 5 + Svelte Flow, CodeMirror 6, Goja, pgx, mongo-driver, go-git, go-keyring, fsnotify, etc. (`PRODUCT/tech-stack.md`).

## 14.4 Adopting a new library
1. Add an ADR in `DECISIONS.md`: name, version, concern, alternatives, why it wins, license, maintenance signal.
2. Verify its API/docs with Context7 or official docs (record the source).
3. Add it to `docs/12.1` registry and to the relevant agent/tool contract.
4. Only then use it. Pin versions; update deliberately.

## 14.5 Anti-patterns (rejected)
- Hand-rolling HTTP routing when chi exists.
- Using `log.Printf` or building a custom logger when zerolog exists.
- Raw `sync.WaitGroup` chains when `errgroup` exists.
- Hand-rolled random strings when gofakeit exists.
- Writing raw HTTP calls to the LLM API when an SDK exists.
- Copy-pasting retry logic into every agent — put it in `internal/llm` / `internal/tool` once.

## 14.6 What verbosity changes here
Library choices must be **documented verbosely**: each registry entry states why it won and what it replaced. Dependency updates are logged as `dependency.updated` events with version and rationale. No silent swaps.

## 14.7 Standard library still wins when…
The concern is trivial, stable, and has no meaningful library (e.g. `errors`, `context`, `time`, `strings`). But HTTP, logging, concurrency orchestration, schema validation, IDs, and fake data always go through the registry.
