# 05 — State and Memory: Shared vs Private

*Define what is shared, what is private, and where artifacts live — before agents start writing to each other's state.* [8]

LangGraph-style stateful orchestration emphasizes **typed state, checkpoints, and long-running workflows** — this is the direction the industry is moving for reliability. [8]

## Three buckets

### 1. Shared state (visible to all agents that need it)
- Task goal and constraints.
- Key artifacts.
- Current final draft.
- Define **which keys each agent may read vs write** — this is your duplicate-write defense (see `03-concurrency.md`).

### 2. Per-agent private scratch
- Temporary reasoning.
- Intermediate notes.
- **Not** shared with other agents by default — sharing full context with every agent increases token usage and cross-contamination. [1]

### 3. Artifact store
- Documents, JSON outputs, code patches.
- Lives in `runs/<run-id>/artifacts/` initially; move to a DB when needed (record in `DECISIONS.md`).

## Rules of thumb
- **Share only what's needed.** Pass the minimal state slice each agent requires. [1]
- **Type the state.** A flat bag of untyped keys becomes unmaintainable fast; agree on a typed structure early. [8]
- **Checkpoint long runs.** Persist state so a long-running workflow can resume after a failure. [8][10]
- **Never let private scratch silently become shared** — make promotion to shared state an explicit, documented step.

## Contract requirement
Every agent contract in `agents/` must declare:
- shared keys it **reads**,
- shared keys it **writes**,
- what stays in its private scratch.

Related reading: [typed state / checkpoints direction [8]].
