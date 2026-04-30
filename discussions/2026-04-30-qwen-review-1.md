# Qwen Review — Round 1

**Date:** 2026-04-30
**Reviewer:** Qwen
**Docs reviewed:** VISION.md, CONCEPT.md, ROADMAP.md, MODULES.md, USE_CASES.md, RELATION_TO_IHN.md, SESSION_PORTABILITY_MANAGER_SPEC_2026-04-29.md, discussions/README.md, prior DeepSeek/Grok/Nemotron reviews

## Summary

The concept framing is unusually disciplined for a Phase-0 repo. The evidence-first thesis, clean iHomeNerd boundary, and anti-goal clarity are all well-executed. Prior reviews covered the major architectural gaps thoroughly. This review focuses on execution strategy, developer ergonomics, and systemic risks that emerge once code starts landing — areas that received lighter treatment in earlier rounds.

---

## Strengths

- **Phase discipline is rare** — Most concept repos over-scope. This one explicitly names what it will *not* be, and the boundaries are defensible. The "evidence, not execution" identity is sticky and differentiating.

- **Session Portability Manager as dogfooding** — Building a real tool for your own pain point (OpenCode session migration) while simultaneously using it as a BugWitness prototype is excellent product strategy. It validates the "witness" thesis in practice, not just in docs.

- **Use cases are failure-mode-driven, not feature-driven** — "Date pickers break," "timezone handling is error-prone," "some UIs aren't testable through DOM queries." These are the kinds of statements that come from actual debugging scars, not competitive analysis.

- **Modular decomposition is well-named** — Scenario Runner, Evidence, TapTrace, Regression Compare, Report Packager, Fixture Lab. Each name communicates responsibility without overlap. This will make it easier to staff parallel workstreams later.

---

## Concerns

### 1. No implementation scaffolding or spike timeline

The repo is pure docs with no `src/`, no `package.json`, no `pyproject.toml`, no Makefile, no CI config. While Phase 0 is "concept framing," the jump from docs to Phase 1 (schema definition) needs a concrete spike plan. Without even a skeleton repo structure, the first implementation session will spend cycles on project setup rather than schema design.

**Risk:** The "done enough to start" status on Phase 0 may be optimistic. A minimal scaffold (language choice, build tool, test runner, linting) should be part of Phase 0 completion, not deferred to Phase 1.

### 2. Scenario schema ambiguity is the highest-leverage unknown

All three prior reviews flagged this, and it bears repeating with more specificity: the schema choice determines the entire developer experience. A YAML schema attracts ops-minded users. A TypeScript fluent API attracts developers. A Markdown DSL attracts PMs and non-technical testers. These are different audiences with different willingness to pay.

**Recommendation:** Prototype *three* scenario formats for the *same* use case (e.g., the pet-walking booking flow) and evaluate which feels most natural to write, read, and debug. The winner becomes Phase 1's deliverable.

### 3. Evidence bundle size and storage growth is unaddressed

A single scenario run with screenshots, OCR text, speech transcripts, request/response logs, and TapTrace sequences could easily produce 50–200 MB of artifacts. At 10 runs/day, that's 1.5–6 GB/month per user. There is no discussion of:

- Compression strategy for evidence bundles
- Retention policies (how long do you keep old runs?)
- Deduplication (same screenshot captured at multiple steps)
- Storage backend abstraction (local disk now, S3 later?)

**Risk:** Storage bloat becomes a user complaint within weeks of first real usage. The storage strategy should be part of Phase 1's artifact bundle format design.

### 4. Error taxonomy and failure classification is missing

Nemotron touched on partial success, but the deeper issue is classification. When a scenario fails, is it because:

- The application under test has a bug (the happy path for BugWitness)
- The browser harness timed out (infrastructure failure)
- The OCR model returned garbage (dependency failure)
- The scenario itself is wrong (user error)
- The device was unreachable (environment failure)

Each category requires different handling, different evidence emphasis, and different remediation guidance. Without a taxonomy, every failure looks the same in the report.

### 5. TapTrace module scope is ambiguous relative to Scenario Runner

MODULES.md says TapTrace owns "mobile tap sequences" and "mismatch reports between expected and actual flow path." But Scenario Runner owns "step execution" and "expected outcomes." The boundary between "what the runner expected" and "what TapTrace recorded" is unclear.

**Question:** Is TapTrace a passive recorder that the Scenario Runner queries, or an active comparator that produces its own findings? If the latter, TapTrace and Regression Compare have overlapping responsibilities.

### 6. No discussion of scenario versioning or migration

Scenarios will evolve. When the scenario schema changes (and it will), existing scenarios need migration. When a user's app changes, their scenarios need updating. There is no mention of:

- Schema versioning strategy
- Backward compatibility guarantees
- Scenario migration tooling
- "Scenario drift" detection (when a scenario no longer matches the app it tests)

---

## Expansion Ideas

### Evidence Compression Pipeline

Implement a post-run compression step that:
- Deduplicates identical screenshots (common in static pages)
- Compresses images with lossless or near-lossless algorithms
- Stores OCR text as plain text (already small)
- Produces a single `.tar.gz` or `.zip` evidence bundle with an index file

This keeps storage manageable and makes evidence bundles portable (email, Slack, issue attachments).

### Scenario Health Dashboard

A lightweight view that shows:
- Which scenarios pass consistently (healthy)
- Which scenarios fail intermittently (flaky)
- Which scenarios haven't run recently (stale)
- Which scenarios produce the most evidence (expensive)

This turns BugWitness from a "run and report" tool into a "monitor and maintain" system.

### Evidence Replay Mode

The ability to replay an evidence bundle step-by-step in a browser-like viewer: screenshot → overlay OCR text → show expected vs actual → show assertion result. This is the "Witness Playground" concept from Grok's review, but focused on *consumption* rather than *authoring*.

### Cross-Session Correlation

When multiple BugWitness runs hit the same app simultaneously (e.g., a booking flow test and a checkout flow test), correlated failures might indicate a shared root cause (database down, API change). A lightweight correlation engine could surface these patterns.

### "Minimum Viable Evidence" Mode

Not every run needs full fidelity. Define evidence levels:
- **Smoke:** screenshot + pass/fail only
- **Standard:** screenshot + OCR + request log
- **Forensic:** full video + speech + TapTrace + all headers

Configurable per scenario or per run. Reduces storage and execution time for routine checks.

### Scenario Linter

A static analysis tool that validates scenarios before execution:
- Detects unreachable steps
- Flags missing preconditions
- Warns about overly broad assertions
- Suggests evidence capture points

This catches scenario authoring errors before they waste execution time.

---

## Execution Strategy Perspective

### Language choice urgency

The `.gitignore` hedges between Python and Node.js. This is fine for Phase 0 but becomes a blocker the moment Phase 1 starts. The decision should be driven by:

1. **iHomeNerd's language** — If iHomeNerd is Python, BugWitness should probably be Python too (shared libraries, easier debugging across the boundary).
2. **Browser automation ecosystem** — Playwright has excellent Python and Node support. Puppeteer is Node-only. CDP libraries exist for both.
3. **Team expertise** — Who is writing the first code?

**Recommendation:** Decide before Phase 1. The schema format (YAML vs TypeScript DSL) may depend on the language choice.

### Parallel workstream planning

The six modules suggest natural parallelism:
- **Stream A:** Scenario Runner + Scenario Schema (Phase 1–2)
- **Stream B:** Evidence Capture + Report Packager (Phase 1–2)
- **Stream C:** Fixture Lab (Phase 1–4, elevated)
- **Stream D:** Session Portability Manager (spike, independent)
- **Stream E:** TapTrace + Regression Compare (Phase 3–5)

Streams A and B must coordinate on the scenario/result schema. Stream C can work independently once the fixture format is defined. Stream D is already spec'd and can proceed in parallel.

### First code milestone definition

The most important Phase 1 milestone is not "define schemas" — it's "run one scenario, capture one piece of evidence, produce one report." This end-to-end wireframe, even with hardcoded data, validates that the module contracts are coherent before any real implementation begins.

---

## On the Session Portability Manager

DeepSeek questioned whether this belongs as a core feature or sibling utility. I disagree with the framing — it belongs as **both**. It is:

- A **utility** in the sense that it solves a developer workflow problem, not a testing problem
- A **core feature** in the sense that it validates BugWitness's thesis: "something happened, it was recorded imperfectly, we need a trustworthy way to inspect and move it"

The Session Portability Manager is BugWitness's first real product, even if it's not a testing tool. It proves the evidence-handling architecture works on a concrete, immediately useful problem. This is a strength, not a scope creep.

---

## Summary of recommendations

1. Create a minimal project scaffold (language, build tool, test runner) as Phase 0 completion criteria
2. Prototype three scenario formats for the same use case and pick the winner
3. Define evidence bundle storage strategy (compression, retention, deduplication) in Phase 1
4. Develop an error/failure taxonomy with classification categories and handling strategies
5. Clarify the TapTrace vs Scenario Runner vs Regression Compare boundary with a data-flow diagram
6. Add scenario versioning and migration strategy to Phase 1 scope
7. Decide on Python vs Node.js before Phase 1 begins
8. Define the first end-to-end wireframe milestone: one scenario, one evidence capture, one report
9. Keep Session Portability Manager as both utility and core feature — it's the first proof of the evidence thesis
10. Consider a "Minimum Viable Evidence" mode to manage storage growth from day one
