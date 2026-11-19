# Tech Stack — full detail from the chat

`PRODUCT/product-brief.md` has the short version. This file holds the **whole stack discussion**: the verdict, the rejected alternatives, the per-component libraries, and the key architecture decisions. Everything below is extracted from the saved chat; items needing verification are flagged **[VERIFY]** and chat inconsistencies are flagged **[CONFLICT]**.

---

## 1. The verdict: Wails v3 + Go

> "Short answer: Yes, Wails (Go) is your best option. It's essentially Tauri but with Go instead of Rust." (chat)

Wails v3.0.0 was stated as released **2026-10-26, production-stable** **[VERIFY against official docs]**. Why it wins:
- Native WebView instead of bundled Chromium → ~15 MB binaries vs Electron's 150 MB+.
- Direct, typed Go↔JS method binding — no JSON IPC serialization overhead.
- Backend is ordinary Go: existing Go libraries/services/business logic can be reused; Wails only binds them to the window.
- You're learning Go — building this app *is* learning Go.
- Official Svelte template: `wails3 init -n myapp -t svelte`.

### Why NOT the alternatives (from the chat)
| Option | Why rejected |
|---|---|
| **Tauri (Rust)** | ~5 MB bundle, but Rust learning curve + borrow checker + JSON IPC overhead. "You'd be fighting Rust's borrow checker instead of building your product." |
| **Electron** | Fastest prototype, but 100 MB+ bundle, 200–500 MB RAM, Node backend — contradicts "lightweight." |
| **C# / .NET MAUI** | Enterprise desktop/mobile focus, buggy Linux story, weak canvas/flow ecosystem, C# second-class on macOS/Linux. |
| **Qt with C++** | Overkill; complex GPL/LGPL/commercial licensing; no React-Flow equivalent or npm ecosystem; build every UI component from scratch. |
| **Neutralinojs** | JS/TS only — can't write Go backend; stuck in JS for DB drivers, filesystem, heavy work. "Not viable for your DB pooling and sync requirements." |

---

## 2. Final stack summary (the chat's own "Final Stack Summary")

```
YOUR APP (~15-20 MB)
BACKEND: Go
  ├── Wails v3               (desktop framework, native WebView, typed bindings)
  ├── modernc.org/sqlite     (app DB — pure Go, cross-compile friendly)
  ├── jackc/pgx              (PostgreSQL, via pgxpool)
  ├── mongo-driver/v2        (MongoDB, internal connection pool)
  ├── goja                   (JS sandbox) [VERIFY import path — see §7]
  ├── go-git/go-git          (Git sync)
  ├── zalando/go-keyring     (secrets in OS keychain)
  ├── fsnotify               (file watcher)
  └── net/http               (REST client)
FRONTEND: Svelte 5
  ├── Svelte Flow            (canvas — official xyflow port)
  ├── CodeMirror 6           (code editor: JS/TS/SQL highlighting, autocomplete)
  ├── shadcn-svelte          (UI components)
  ├── Tailwind CSS v4        (styling)
  ├── Sonner                 (toasts — e.g. throw-error block, dialog())
  ├── Lucide                 (icons)
  ├── tinykeys               (keyboard shortcuts)
  ├── cmdk-sv                (command palette)
  └── marked                 (markdown rendering for text blocks)
STORAGE: Local filesystem
  ├── SQLite                 (app state, history, cache, UI prefs)
  └── .flow.json files       (flow data — flat, Git-diffable)
COST: $0.00 — all free, all open source, single binary, no server, no account, no subscription
```

### Component diagram (from the chat, condensed)
```
WAILS v3 APP
 GO BACKEND                         WEB FRONTEND (system WebView)
  Flow Engine                       Svelte Flow Canvas
   - execute flow / parallel / cond   - nodes, edges, minimap
  DB Manager                        CodeMirror 6
   - PG pool (pgx), Mongo pool        - code blocks, SQL editor, autocomplete
   - SQLite (app)                    UI (shadcn/Kobalte)
  HTTP Client                        - response viewer, variable panel,
   - net/http, cURL parser             console/terminal, environment mgr
  JS Sandbox (Goja VM)               Tailwind v4, Sonner, Lucide,
   - user code, console.log            tinykeys, cmdk
  Sync Engine                       Wails auto-generates TS bindings
   - go-git, file watcher, E2E enc    from Go structs + methods
  Keychain (go-keyring)

 LOCAL STORAGE: ~/.flowapp/app.db + workspaces/<project>/*.flow.json
```

---

## 3. Key architecture decisions (with the chat's reasoning)

### 3.1 JS code execution — Goja, not Node/V8/WebView
Users write JS in code blocks; it must be sandboxed and dependency-free.
| Option | Verdict | Reason |
|---|---|---|
| **Goja** (pure Go JS engine) | ✅ Best fit | Zero dependency, sandboxed, fast startup, runs in the Go process. ES2020 only (no latest TS features). |
| Embedded V8 (`v8go`) | ❌ Overkill | CGO required, large binary, complex build. |
| Spawn Node.js | ❌ Not portable | Requires Node installed, slow startup, security risk. |
| Run in WebView | ❌ Wrong context | Can't access the Go backend from user code. |

Injected into the VM: `getSecret(key)`, `showToast(msg)`, `setVariable(name, value)`, `console.log` (→ global terminal via Wails events). User code does variable manipulation, `console.log`, and data extraction — no filesystem or network access.

### 3.2 Database connection pooling — global pools in Go
One pool per connection ID, created once and reused across all flows:
```
DBManager {
  pgPools    map[string]*pgxpool.Pool   // connection ID -> pool
  mongoPools map[string]*mongo.Client   // connection ID -> client
  mu         sync.RWMutex
}
```
Use `sync.RWMutex` for concurrent map access. `pgx` has built-in pooling via `pgxpool` — no external pool library. The MongoDB Go driver manages its own internal pool. This is the reason the app needs a Go backend (CORE-05 global pool), and why CORE-05 must exist before CORE-10 parallel execution.

### 3.3 Flow execution engine — goroutines
"Go genuinely shines over Rust for your use case — goroutines are simpler than async Rust, and Go's scheduler handles thousands of concurrent flows effortlessly."
```
func (e *FlowEngine) executeParallel(blocks []Block, input any) []Result {
    results := make([]Result, len(blocks))
    var wg sync.WaitGroup
    for i, block := range blocks {
        wg.Add(1)
        go func(idx int, b Block) {
            defer wg.Done()
            results[idx] = e.executeBlock(b, input)
        }(i, block)
    }
    wg.Wait()
    return results
}
```
This is the implementation counterpart to the agentic fan-out/gather pattern in `docs/03-concurrency.md`.

### 3.4 Frontend — Svelte 5 recommended over SolidJS
| Factor | Svelte 5 | SolidJS |
|---|---|---|
| Learning curve | Easier (HTML-first syntax) | Medium (JSX, signals) |
| Flow library | ✅ Svelte Flow (official xyflow port) | ⚠️ needs React Flow + adapter |
| Bundle size | Tiny (compiler, no runtime) | Tiny (fine-grained reactivity) |
| Ecosystem | Growing fast, shadcn-svelte exists | Smaller, Kobalte for components |
| DX | Excellent (single-file components) | Excellent (true reactivity) |
| Wails templates | ✅ Official | ✅ Community |

Chat's recommendation: **Svelte 5** — first-class Svelte Flow canvas, shadcn-svelte, official Wails template, gentler learning curve while you're learning Go. **FINAL: Svelte 5 accepted (D-008)**; SolidJS rejected. Not an open decision.

---

## 4. Storage & sync technology

- **Flat `.flow.json` files** (JSON/YAML), one flow per file — Git-diffable, human-readable, merge-friendly. "The file format is your sync strategy."
- **One folder = one workspace** → maps to one Git repo.
- Secrets in `.env.local` — never synced; `.env.dev` / `.env.staging` optionally; never production secrets.
- SQLite = app state only (UI state, history, cache); never synced directly.
- Auto-generated `.gitignore` for `.env.local`, `app.db`, `node_modules`, `.cache`.
- Git engine: commit on save (debounced ~5s), push/pull manual or interval, side-by-side diff viewer on conflicts, PAT in OS keychain.

**[CONFLICT — RESOLVED by ADR D-011]** The chat's *sync* section was written from a Tauri/Rust perspective and names Tauri-specific tools that contradict the Wails + Go verdict:
- `libgit2` / `git2` crate "in Tauri backend" → use **go-git/go-git** instead.
- `tauri-plugin-stronghold` for keychain → use **zalando/go-keyring** instead.
- `tauri-plugin-sync`, `cloud_sync_tauri` (Rust/CRDT/LWW sync servers) → rejected; chosen path is flat files + go-git.
A later "Tech Stack (All Free)" table in the chat also lists **Tauri, SolidJS, better-sqlite3/sql.js** — it contradicts the detailed Wails + Go verdict and is likewise rejected. Do not mix any of these into the implementation. D-011 makes the final call: **Wails v3 + Go, go-git, go-keyring, fsnotify**.

---

## 5. Cost principle
Everything free, open source, single binary, no server, no account, no subscription. Desktop app = no cloud bills; SQLite = no DB bills. The only money flow is monetizing the software itself (Free / Pro $29 / Team $49 / Cloud $5 tiers — see `product-brief.md`).

---

## 6. Verify before build **[VERIFY]**
The chat's factual claims should be checked against live/official docs before implementation (use Context7 or official sites):
- Wails v3 release date / stability and `wails3 init -n myapp -t svelte` syntax.
- Goja import path (chat shows `github.com/nicois/goja` / `nicois/goja`; the well-known Goja module is `github.com/dop251/goja` — confirm).
- `mongo-driver/v2` version/path, `jackc/pgx` v5 `pgxpool` API, `cmdk-sv`, `shadcn-svelte`, `fsnotify`, `zalando/go-keyring` current versions.
- Svelte Flow / xyflow licensing and Wails Svelte template availability.

## 7. Decisions — ALL FINAL (see `DECISIONS.md`)
- **Svelte 5 + Svelte Flow** accepted (D-008); SolidJS rejected.
- **Tauri/Rust conflict resolved** in favor of Wails + Go + go-git + go-keyring (D-011).
- **Goja** accepted with ES2020 limitation; module path is a **[VERIFY]** check, not an open decision (D-009).
- All remaining **[VERIFY]** items are factual checks against official docs, not decisions.

---

## 8. Agentic runtime libraries (added by you — not in the chat)
The agentic layer is also Go and library-first (D-002–D-007, D-013). Verified via Context7 `/go-chi/docs`, `/rs/zerolog`, `/brianvoe/gofakeit`.

| Concern | Library | Decision |
|---|---|---|
| HTTP/JSON routing | **`github.com/go-chi/chi/v5`** + `/middleware` | D-003 |
| Logging (verbose) | **`github.com/rs/zerolog`** + `/hlog` | D-004, D-013 |
| Fan-out/gather | **`golang.org/x/sync/errgroup`** + `semaphore` | D-006 |
| LLM calls | Official OpenAI Go SDK **[VERIFY]** `github.com/openai/openai-go` | D-005 |
| Trace store | `modernc.org/sqlite` (WAL) + JSONL | D-007 |
| IDs | `github.com/google/uuid` | D-013 |
| Schema validation | JSON-schema validator **[VERIFY]** `github.com/santhosh-tekuri/jsonschema/v6` | D-005/D-006 |
| Random generator | **`github.com/brianvoe/gofakeit/v7`** | D-012, `PRODUCT/random-generator.md` |

Full registry, routes, components, and run lifecycle: `docs/12-go-agent-runtime.md`. Verbose logging standard: `docs/13-verbose-logging.md`. Library policy: `docs/14-library-policy.md`.
