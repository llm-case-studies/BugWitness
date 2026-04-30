# BugWitness Session Portability Manager Spec

**Date:** 2026-04-29  
**Audience:** Alex, Codex, Claude, DeepSeek, free hosted models, future BugWitness contributors  
**Status:** Spike-spec / assignment seed

## 1. Why this exists

During OpenCode host migration from the Mac mini to the Debian iMac, we proved:

- session history is durable and portable in principle
- current browser UI is not a reliable session-management surface
- direct per-session export/import is flaky for larger sessions
- full-state migration works, but is too manual and too fragile

That makes session portability a good BugWitness problem.

This is not a generic "agent platform" idea. It is a narrow evidence and operations tool:

- inspect session inventories
- validate session integrity
- migrate sessions between hosts
- rewrite stale project-path metadata safely
- reopen or deep-link specific sessions later
- compare session histories and provider/model usage

## 2. Product framing

BugWitness should treat agent sessions the same way it treats bug evidence:

- something happened
- it was recorded imperfectly
- we need a trustworthy way to inspect, preserve, package, and move it

So this feature belongs in BugWitness as a **Session Portability Manager** or **Session Lab** rather than in iHomeNerd core.

## 3. Problem statement

Current OpenCode pain points observed in the field:

1. The visible browser session list may not match the sessions actually present in the backend.
2. Migrated sessions can remain hidden if their stored `directory` paths still point to the source machine.
3. `opencode export` on `1.14.29` appears to truncate larger sessions at `65536` bytes.
4. There is no obvious built-in tool to:
   - list sessions across hosts
   - validate session completeness
   - export them safely in bulk
   - rebind them to a new local checkout
   - analyze provider/model/session relationships

## 4. Target user stories

### User story A — Recover yesterday's work

As a developer using OpenCode on multiple machines,
I want to move my session history from one host to another,
so I can continue work after sleep, hardware changes, or service relocation.

### User story B — Audit what happened

As a human supervising free or paid models,
I want to see which sessions used which models and providers,
so I can understand cost, quality, and reliability.

### User story C — Repair broken visibility

As a user who knows sessions exist in the backend but not in the UI,
I want to detect and fix stale path or project metadata,
so the sessions become visible and resumable again.

### User story D — Package handoff

As a tester or operator,
I want to hand a bounded session set to another machine or service,
so work can continue in a controlled environment.

## 5. Scope of the first spike

### In scope

- inspect OpenCode session inventory from local storage
- read session metadata from SQLite
- map:
  - session ID
  - title
  - project/worktree
  - stored directory
  - provider/model
  - timestamps
- detect stale path mismatches
- backup and migrate OpenCode durable state between hosts
- rewrite stale directory metadata safely
- generate deep links to specific sessions
- produce human-readable migration reports

### Out of scope

- building a full OpenCode replacement
- changing upstream OpenCode internals directly
- generic multi-agent orchestration
- arbitrary cross-provider model brokerage
- broad support for every agent app on day one

## 6. Recommended architecture

### Option 1 — Standalone sidecar service

A small local service that runs next to OpenCode and exposes a narrow API/UI for:

- listing sessions
- analyzing metadata
- exporting/importing validated bundles
- rebinding sessions to a new worktree
- generating deep links

Pros:

- does not require forking OpenCode first
- easier to prototype quickly
- can work against known local storage layout

Cons:

- may need updating if OpenCode internals change

### Option 2 — BugWitness web tool only

A browser-first tool that reads local files through a small helper script.

Pros:

- lighter

Cons:

- weaker operational control
- more awkward for multi-host migration

### Option 3 — Fork or patch OpenCode UI

Pros:

- best integrated UX if successful

Cons:

- higher maintenance burden
- slower path to a useful result

### Recommended first implementation

Start with **Option 1: standalone sidecar service**.

Keep it small and practical.

## 7. Proposed MVP capabilities

### 7.1 Inventory endpoint

Input:

- local OpenCode data directory

Output per session:

- session ID
- title
- project ID
- project worktree
- stored session directory
- provider ID
- model ID
- created/updated times
- likely visibility health
- direct URL path if host/port is known

### 7.2 Integrity check

Checks:

- missing DB
- missing `storage/session_diff` files
- missing snapshots
- path mismatch between `session.directory` and active project worktree
- likely truncated export bundles

### 7.3 Bulk migration

Capabilities:

- SQLite online backup
- copy `storage/`, `snapshot/`, `tool-output/`
- preserve or separately inject auth/config
- optional target-path rewrite rules
- dry-run report before mutate

### 7.4 Path rebinding

Given:

- source path `/Users/alex/Projects/iHomeNerd`
- target path `/home/alex/Projects/iHomeNerd-testing`

Do:

- rewrite relevant session directory metadata
- back up DB first
- emit before/after summary

### 7.5 Deep-link generator

Given:

- host URL
- project worktree path
- session ID

Return:

- OpenCode-compatible direct session URL

Example pattern observed:

```text
http://host:4096/<base64url(project-path)>/session/<session-id>
```

### 7.6 Report generation

Produce:

- markdown migration report
- JSON inventory report
- list of recoverable sessions
- list of suspect or broken sessions

## 8. Data sources to support initially

Known OpenCode local state:

- `~/.local/share/opencode/opencode.db`
- `~/.local/share/opencode/storage/`
- `~/.local/share/opencode/snapshot/`
- `~/.local/share/opencode/tool-output/`
- optional auth/config:
  - `~/.local/share/opencode/auth.json`
  - `~/.config/opencode/opencode.env`

Important note:

- credentials should be copied only intentionally
- reports should redact secrets by default

## 9. Suggested implementation shape

A free-model-friendly assignment should stay bounded.

### Suggested stack

Either:

- Python + `sqlite3` + small FastAPI app

or:

- TypeScript + Bun/Node + SQLite library + small HTTP server

Python is probably the fastest safe path for the first spike because:

- SQLite handling is built in
- migration logic is straightforward
- no frontend framework is required on day one

### Suggested deliverables

1. `sessionlab inspect <opencode-dir>`
2. `sessionlab migrate --src <dir> --dst-host <host> --dst-dir <path>`
3. `sessionlab rebind-path --db <db> --from <old> --to <new>`
4. `sessionlab link --host <url> --project <path> --session <id>`
5. optional tiny web UI:
   - session table
   - health badges
   - deep-link buttons
   - migration dry-run view

## 10. Acceptance criteria for the spike

The spike is successful if it can:

1. inventory sessions on one OpenCode host
2. identify provider/model usage correctly
3. detect path mismatches after migration
4. migrate at least one real OpenCode workspace from host A to host B
5. make previously hidden sessions visible or directly reopenable on host B
6. generate a readable findings report without leaking credentials

## 11. Known field evidence from iHomeNerd

Observed in real migration work:

- `opencode export` for larger sessions was unreliable on `1.14.29`
- full SQLite + storage migration worked
- browser list initially hid migrated sessions
- rewriting `session.directory` to the new host path fixed backend/session alignment
- direct session URLs worked once project path and session ID were known

This is enough evidence to justify the spike.

## 12. Assignment prompt for a free model

Build a narrow BugWitness prototype for OpenCode session portability.

Start with a local-only tool that can:

- inspect an OpenCode data directory
- list sessions with IDs, titles, provider/model, stored path, and timestamps
- detect stale path metadata
- back up the SQLite DB
- rewrite old session directory paths to a new worktree path
- generate direct session URLs from host + project path + session ID
- emit both JSON and markdown reports

Do not try to redesign all of OpenCode.
Focus on the smallest useful tool that reproduces and solves the migration problem we just observed in the field.

## 13. Next product step after the spike

If the spike works, BugWitness can grow this into a broader agent-evidence surface:

- cross-host session diffing
- model/provider cost analysis
- session bundles for handoff
- recovery of interrupted agent work
- session-to-issue packaging

But the first deliverable should stay disciplined:

**make OpenCode sessions inspectable, portable, and recoverable.**
