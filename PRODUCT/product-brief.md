# Product Brief — source of truth: "what we are building"

> Product facts below are extracted from the saved chat `Directly Chat with Frontier AI Search Models (11_19_2026 6：49：15 PM).html`, except the **Random Generator** (requested by you, in [`random-generator.md`](random-generator.md)) and the **Go agentic runtime libraries** (requested by you, chi + zerolog). All choices are final in [`DECISIONS.md`](../DECISIONS.md). Chat truncations are flagged **[CONFIRM]**; factual checks are **[VERIFY]**.

## One-line vision
A **lightweight desktop visual flow app for API + database workflows** — "Postman Flows meets database workflows, in a lightweight desktop app."

## Target user
Backend developers who daily hit **APIs and databases together** and have no unified lightweight tool. Solo-developer context: near-zero operational cost, free tools where possible, desktop app (no server = no cloud bills).

## Why it is unique (from the chat's competitor analysis)
No single tool does the full combo. **Your Tool** vs competitors (Postman Flows, ApiFlow, Flowter, n8n, Bruno):
- Visual canvas + blocks, lightweight **desktop**
- REST calling, **cURL parsing**
- **Database flows (Postgres + Mongo)** + **global DB connection pool**
- **Code blocks (JS/TS)**, **template system (clone)**
- **If / condition / for / repeat** blocks
- Secret/variable system, **parallel block execution**
- SQLite local storage, minimap, autocomplete for vars, JS/TS expression syntax, GitHub sync

ApiFlow in particular "lacks database integration, code blocks, template systems, and conditional logic blocks — all core to your idea."

**Switch triggers:** Auth system, environments, response inspector, keyboard shortcuts, import from Postman.
**Retention triggers:** Lightweight, fast, offline, Git-friendly, respects their data.

## Stack decisions (short version)
Full detail — framework verdict, rejected alternatives, every library, architecture decisions, and chat inconsistencies — is in **[`tech-stack.md`](tech-stack.md)**. Summary:

| Layer | Choice |
|---|---|
| Desktop | **Wails v3 + Go** (v3.0.0 released 2026-10-26, production-stable; native WebView; ~15 MB binaries; direct typed method binding instead of JSON IPC) |
| App DB | **modernc.org/sqlite** (pure Go, cross-compile friendly) — *app state only* |
| Frontend | **Svelte 5 + Svelte Flow** (ACCEPTED, D-008); **CodeMirror 6**; **shadcn-svelte**; **Tailwind CSS v4**; **Sonner**; Lucide; tinykeys; cmdk-sv; marked |
| JS sandbox | **Goja** (pure Go, sandboxed, ships in the binary). Inject `getSecret()`, `showToast()`, `setVariable()`, `console.log`. ES2020 only. Rejected: embedded V8, spawning Node, running in WebView |
| DB manager | Postgres pool via **pgx**, Mongo pool, SQLite app state |
| HTTP | **net/http** + cURL parser |
| Sync | **go-git**, file watcher, optional E2E encryption; **go-keyring** for OS keychain |

## Storage model (critical invariant)
- Flows are **flat files (`.flow.json`)**, one flow per file — Git-diffable, human-readable, merge-friendly.
- **One folder = one workspace** → maps to one Git repo.
- **Secrets in `.env.local` — NEVER synced** (git-ignored). `.env.dev` / `.env.staging` may sync optionally; **never production secrets**.
- **SQLite is for app state only** (history, cache, UI preferences) — do not sync it directly.
- Layout: `~/.flowapp/app.db` + `~/.flowapp/workspaces/<project>/*.flow.json`.

## Sync strategy (from the chat's phased plan)
- **Start:** flat files + folder sync (Dropbox / Syncthing / OneDrive = free, you build nothing).
- **Then:** Git sync as a Pro feature (**go-git/go-git** per D-011 — not the chat's `git2`/Tauri tool, commit on save debounced ~5s, push/pull, visual conflict diff).
- **Later:** self-hosted sync server (single binary ~5MB / Docker, port 8787), then cloud (Cloudflare R2 + Workers), then WebDAV + S3 adapters.
- **Don't:** use CRDTs, sync SQLite directly, build real-time collaboration, store secrets in sync, or build your own auth.

## Monetization (context only — not an engineering requirement)
Open Core + one-time purchase: Free (MIT, all block types, cURL import, single environment, request history, SQLite, minimap, manual export/import), **Pro $29 one-time**, **Team $49/seat**, **Cloud $5/mo**. Feature matrix in the chat lines ~422.

## Explicitly out of scope / later
Real-time collaboration, CRDTs, deep CI/CD platform, becoming n8n (scope-creep risk). GraphQL body editor marked "future". Plugin/extension system is v2+.

## Added by you (not in chat)
- **Postman-style Random Generator** — gofakeit v7, dynamic variables + generator block (`RAND-01`–`RAND-08`, D-012). Full spec: [`random-generator.md`](random-generator.md).
- **Go agentic runtime** — chi v5 routing + zerolog verbose logging + errgroup + official OpenAI Go SDK (D-002–D-007). Full spec: [`docs/12-go-agent-runtime.md`](../docs/12-go-agent-runtime.md).
- **Verbose everywhere** — verbose logs, traces, errors, docs, CLI (D-013, `docs/13-verbose-logging.md`).

## Remaining factual checks (not decisions)
- **[CONFIRM]** Exact meaning of the truncated items **"flow template system"** and **"API templa..."** from your checklist.
- **[VERIFY]** Wails v3 release/syntax, Goja module path, LLM SDK module path, library versions, Postman dynamic-variable names.
- Monetization/licensing timing is a business choice; recommended by chat to defer heavy work until after MVP (months 5–6).
