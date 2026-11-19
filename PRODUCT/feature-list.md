# Feature List — the "what to build"

IDs here are used by [`BUILD-PLAN.md`](../BUILD-PLAN.md) and [`HANDOVER.md`](../HANDOVER.md). Priorities are taken directly from the chat. "Done-when" is a short acceptance criterion an AI agent can verify.

---

## 0. Core concept / differentiators (your original spec)
| ID | Feature | Done-when |
|---|---|---|
| CORE-01 | Visual canvas with blocks (Postman Flows-like) | Nodes, edges, minimap, controls, drag/drop, zoom/pan work |
| CORE-02 | REST API calling | A block sends an HTTP request and receives a response |
| CORE-03 | cURL parsing (import) | Pasting cURL creates a configured request block |
| CORE-04 | Database flows (Postgres + Mongo) | DB blocks can query both databases |
| CORE-05 | Global DB connection pool | One shared pool, not per-run connections; no connection leak under parallel runs |
| CORE-06 | Code blocks (JS/TS) via Goja sandboxed VM | User JS runs; `getSecret`, `showToast`, `setVariable`, `console.log` injected; no filesystem/network access |
| CORE-07 | If / condition / for / repeat blocks | Branching and loops execute correctly |
| CORE-08 | Secret/variable system (+ autocomplete for vars) | Secrets stored in `.env.local`, never synced; vars autocomplete in editors |
| CORE-09 | Template system (flow + API, clone) **[CONFIRM scope]** | A saved flow/API can be cloned into a new one |
| CORE-10 | Parallel block execution | Independent branches run concurrently; results gather correctly |
| CORE-11 | SQLite local storage (app state) | `~/.flowapp/app.db` holds history/cache/UI state |
| CORE-12 | Minimap | Canvas minimap present |
| CORE-13 | JS/TS expression syntax in blocks | Expressions evaluate with variable substitution |
| CORE-14 | GitHub sync (manual export/import minimum) | Flows export/import via Git-compatible flat files |
| CORE-15 | Lightweight desktop app | Single Wails binary, native WebView, no bundled Chromium |

---

## 1. 🔴 MUST HAVE — "ship or die"
| ID | Feature | Detail from chat | Done-when |
|---|---|---|---|
| MUST-01 | Response Viewer / Inspector | status code, headers, body (JSON/XML/HTML/raw), response time + size, pretty print / tree view, search within response, copy response / copy path (`response.data.users[0].name`) | Inspector shows all fields; clicking a JSON key copies its path |
| MUST-02 | Authentication System | Bearer, Basic, OAuth 2.0 (auth code + client credentials), API Key (header/query), auth inheritance (set once at flow level) | All 4 auth types work; child blocks inherit flow-level auth |
| MUST-03 | Environment System | dev/staging/prod switching, per-environment variable overrides, toolbar dropdown, `.env` import/export | Switching environment changes resolved variable values everywhere |
| MUST-04 | Request History / Log | auto-save every request+response with timestamp, searchable/filterable, re-run from history, diff two historical responses | History persists across restarts; diff view works |
| MUST-05 | Import / Export | Postman Collection v2.1 JSON, Insomnia YAML/JSON, OpenAPI/Swagger, HAR, export shareable JSON/YAML | Each import creates valid blocks; each export round-trips |
| MUST-06 | Keyboard Shortcuts + Command Palette | Cmd/Ctrl+K palette, Cmd+Enter execute, Cmd+S save, Cmd+D duplicate block, customizable | All bindings work |
| MUST-07 | Request Body Builders | JSON editor w/ highlight + validation, form-data (file upload), x-www-form-urlencoded, raw (text/XML/HTML), binary upload, GraphQL (future) | Each body type produces correct HTTP request |

## 2. 🟡 HIGH VALUE — "makes users stay + tell others"
| ID | Feature | Detail | Done-when |
|---|---|---|---|
| HIGH-01 | Request Chaining — visual variable mapping | Drag output field → input field, auto-generate extraction expression, show live data flowing during execution | Dragging creates a mapping; running shows values |
| HIGH-02 | Assertion / Test Block | `status == 200`, `response.body.id != null`, `response.time < 500ms`, visual pass/fail, aggregate results panel | Assertions evaluate; block turns green/red; panel aggregates |
| HIGH-03 | WebSocket Support Block | connect/listen/send, event-driven trigger (on message → continue) | A WS message can continue the flow |
| HIGH-04 | CLI Runner | `flowapp run my-flow.json --env production`, JSON/JUnit XML output | Headless run exits with proper code + output |
| HIGH-05 | Undo/Redo on Canvas | Cmd+Z / Cmd+Shift+Z, action history stack | Canvas edits undo/redo correctly |

## 3. 🟢 NICE TO HAVE — "after v1"
| ID | Feature |
|---|---|
| NICE-01 | Dark/Light theme |
| NICE-02 | Response diff viewer |
| NICE-03 | Mock server block |
| NICE-04 | Proxy support (HTTP/SOCKS) |
| NICE-05 | Certificate management (mTLS / client certs) |
| NICE-06 | Snippets / code generation (cURL, JS fetch, Python requests from any block) |
| NICE-07 | Flow execution timeline (Gantt-style parallel execution view) |
| NICE-08 | Block grouping / collapsing |
| NICE-09 | Commenting on blocks |
| NICE-10 | Search across all flows (endpoint URLs, variable names) |
| NICE-11 | Auto-save + crash recovery (SQLite WAL nearly free) |
| NICE-12 | Plugin/Extension system (v2+) |

## 4. Sync feature tiers (from the chat's sync roadmap)
| ID | Feature | Tier |
|---|---|---|
| SYNC-01 | Flat file format + manual export/import | Free / Phase 1 |
| SYNC-02 | Folder sync (Dropbox etc. — free, build nothing) | Free / Phase 1 |
| SYNC-03 | Git init + diff + pull | Free / Phase 2 |
| SYNC-04 | Git push + branch + merge UI | Pro |
| SYNC-05 | Auto-commit on save (debounced ~5s) | Pro |
| SYNC-06 | Self-hosted sync server | Pro / Phase 3 |
| SYNC-07 | Cloud sync via Cloudflare R2 | Cloud / Phase 4 |
| SYNC-08 | E2E encryption | Pro |
| SYNC-09 | Conflict resolution UI | Pro |
| SYNC-10 | Sync history / audit log | Cloud |
| SYNC-11 | Multi-device (3+) | Cloud |
| SYNC-12 | WebDAV + S3 adapters | Phase 5 |

## 5. Random Generator for Postman — added by you (not in the chat)
Decision **D-012**; full spec in [`random-generator.md`](random-generator.md). Library: `gofakeit/v7`.

| ID | Feature | Done-when |
|---|---|---|
| RAND-01 | Postman dynamic variables (`{{$randomInt}}`, `{{$randomUUID}}`, `{{$randomEmail}}`, …) resolved via gofakeit | Token resolves to correct type at run time; verbose log emitted |
| RAND-02 | Unknown-token behavior | Token left unchanged, warning logged, output labeled `unverified` |
| RAND-03 | Random Generator block with type picker | Category selected; data generated |
| RAND-04 | Schema-driven payload generation | Generated JSON validates against provided schema/example |
| RAND-05 | Seed control (fixed seed reproducible; seed 0 crypto-random) | Same seed → identical output; seed 0 → different |
| RAND-06 | Batch / count mode (N records as JSON array) | Exactly N records |
| RAND-07 | Export / reuse into request body, variable, or workspace file | One-click wiring works |
| RAND-08 | Verbose generation logging | `random.generated` events present in trace |

## Traceability
Everything in sections 0–4 maps to a statement in the saved chat; section 5 was requested by you and is marked as such. See [`docs/references.md`](../docs/references.md) for the footnote map. Items marked **[CONFIRM]** were truncated in the saved HTML ("API templa...", "flow template system") — confirm exact intent before treating them as finalized requirements.
