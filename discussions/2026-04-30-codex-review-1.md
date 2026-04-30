# Codex Review -- Round 1

**Date:** 2026-04-30
**Reviewer:** Codex
**Docs reviewed:** VISION.md, CONCEPT.md, ROADMAP.md, MODULES.md, USE_CASES.md, RELATION_TO_IHN.md, SESSION_PORTABILITY_MANAGER_SPEC_2026-04-29.md, discussions/README.md, all prior reviews (DeepSeek, Grok, Nemotron, Qwen, Kimi-K2.6, Minimax, Claude Opus, GPT-OSS-120B, Gemini 3.1 Pro)

## Summary

The prior reviews have done unusually strong work. They have covered the obvious load-bearing decisions: scenario schema, browser harness, evidence bundle format, privacy, redaction, flakiness, topology, baseline stewardship, module handoffs, finding lifecycle, and the human/agent dual-view problem.

This review therefore focuses on a slightly different layer: **how BugWitness knows what it is capable of, how it knows what "expected" means, and how it avoids overclaiming when evidence is partial, model-derived, human-confirmed, or environment-dependent.** In short: BugWitness needs not only evidence, but epistemic humility. A strong witness does not merely say what it saw; it also says what it could not see, what tools it used to see, and why the expectation was valid in the first place.

---

## Strengths

- **The core thesis is still crisp after nine reviews** -- The phrase "what happened / what did we expect / what proof do we have / how reproducible is it" continues to hold up. It is a product primitive, not just copywriting.

- **The iHomeNerd boundary is strategically useful** -- BugWitness can stay opinionated about testing because it does not need to own device discovery, trust, OCR, speech, local model routing, and browser access from scratch. That is a rare leverage point.

- **The module names are memorable without being cute** -- Scenario Runner, BugWitness Evidence, TapTrace, Regression Compare, Report Packager, Fixture Lab. These names map to workstreams and user-facing concepts at the same time.

- **Session Portability Manager is the right kind of first weird thing** -- It stretches the product identity, but in a productive way. It forces the evidence pipeline to handle real artifacts, stale paths, integrity checks, migration reports, and recovery before the system is trusted with polished demo flows.

- **The discussion process is already becoming design memory** -- The repo is not only collecting opinions; it is building a vocabulary: FindingLifecycle, EvidenceCoverageReport, ExternalEvidencePort, Witness Timeline, Privacy Guardian, Baseline Steward, ScenarioBundle. That shared vocabulary is a real asset if it gets consolidated before implementation.

---

## Concerns

### 1. No capability negotiation contract with iHomeNerd

BugWitness depends on `iHomeNerd` for device access, browser control, local trust, OCR, speech, local model routing, and cross-device coordination. But the docs treat those capabilities as if they are simply present.

In practice, each host will have a different capability profile:

- OCR available locally, but speech unavailable
- browser control available on desktop, but not on a phone
- trusted TLS configured for one device, missing on another
- local vision model upgraded between two runs
- device probe reachable over LAN yesterday, unreachable today

**Risk:** Two BugWitness runs with the same scenario may produce different evidence quality because the substrate changed, but the report will look as if the scenario itself changed or the app regressed.

**Recommendation:** Add a `SubstrateCapabilityManifest` to Phase 1. At the start of every run, BugWitness should ask iHomeNerd: what capabilities are available, at what versions, with what permissions, and at what health level? The manifest should travel with the evidence bundle. Scenario steps should be able to declare whether a missing capability is `required`, `optional`, or `degrades_report`.

### 2. Accessibility is not yet a first-class evidence stream

The docs emphasize screenshots, OCR, speech, traces, and endpoint responses. That is strong for visual and multimodal flows, but it risks making BugWitness a sighted-user witness only.

Many real product failures live in the accessibility layer:

- a button is visible but has no accessible name
- the focus order jumps unpredictably after a modal opens
- a mobile menu works by touch but not by keyboard
- ARIA state says "expanded" while the UI is collapsed
- checkout errors are visible but not announced to assistive tech

**Risk:** BugWitness may produce beautiful evidence for flows that are still broken for keyboard and screen-reader users. Worse, OCR can say text is visible while the accessibility tree says it is unreachable.

**Recommendation:** Add an `AccessibilityWitness` evidence stream. Capture accessible names, roles, focus order, active element changes, landmark structure, ARIA state, and keyboard navigation outcomes. This stream should also inform scenario authoring: semantic selectors based on role/name are more stable and more humane than raw CSS paths.

### 3. Expected outcome provenance is underspecified

The central report question is "what did we expect?" But expectations are not all equal. An expected value may come from:

- an explicit scenario assertion
- a prior approved baseline
- a product requirement
- a human checkpoint
- an agent-generated hypothesis
- a fixture oracle
- a contract from an external system

If BugWitness says "expected total: $42.10, actual total: $43.10," the consumer needs to know why $42.10 was authoritative.

**Risk:** Reports may frame a mismatch as an application bug when the expectation was stale, inferred, low-confidence, or generated by an agent from incomplete context. This is the mirror image of the false-witness problem: not false observation, but false expectation.

**Recommendation:** Add an `ExpectationSource` or `OracleContract` field to assertions and findings. Include `source_type`, `source_ref`, `authority`, `version`, `tolerance`, and `confidence`. A finding should distinguish "the app violated a checked-in requirement" from "the app differed from an agent-inferred expectation."

### 4. OCR, speech, and vision model drift can invalidate comparisons

BugWitness treats OCR and speech as evidence helpers, but their outputs are model-derived. If the OCR model changes, a regression compare may show different text for the same screenshot. If a speech model changes, transcript diffs may reflect analyzer drift rather than app drift.

Prior reviews covered evidence provenance and model versions generally; the sharper issue is baseline validity. A baseline produced by analyzer version A may not be comparable to a new run processed by analyzer version B.

**Risk:** Regression Compare may report product changes that are really analyzer changes. Conversely, an improved analyzer may hide a real product regression by interpreting noisy evidence differently.

**Recommendation:** Add an `AnalyzerManifest` and an analyzer lockfile concept. Store raw artifacts and processed outputs separately. Every processed artifact should include analyzer ID, version, model hash where available, prompt/config, thresholds, and processing time. Regression Compare should warn when comparing processed outputs generated by different analyzers and offer re-analysis of old raw evidence under the new analyzer.

### 5. Human checkpoints need their own protocol

Phase 3 mentions guided mobile checks and manual checkpoint support, but "manual" cannot mean "unstructured note from the operator." In semi-automated testing, the human becomes part of the evidence chain.

Human confirmation is useful, but it is also noisy:

- two operators may interpret the same instruction differently
- one operator may skip a step under time pressure
- a human may confirm "looks good" without noticing a hidden accessibility failure
- a manual observation may need a screenshot, device photo, or audio note attached

**Risk:** Manual checkpoints become trusted evidence without the structure needed to audit them later.

**Recommendation:** Define a `HumanCheckpoint` schema. It should include the instruction shown to the operator, allowed responses, expected observation, timeout, required artifact capture, optional operator identity, and confidence. A human checkpoint should be rendered as evidence, not as an invisible pause in the run.

### 6. Evidence budgets are not modeled

Several reviews discuss storage size, overhead, and flakiness. A related but distinct issue is budgeting. Evidence capture has many costs: wall-clock time, CPU, battery, model calls, disk space, network transfer, and operator attention.

Without a budget, "capture more proof" will always sound correct until the run becomes too slow or expensive to use.

**Risk:** Users either over-capture everything and abandon the tool because it is heavy, or under-capture by habit and discover too late that the report lacks the one artifact needed to debug the bug.

**Recommendation:** Add a `RunBudget` to scenarios and runs. It can define max duration, max artifact size, evidence level, analyzer cost ceiling, retry limit, and failure escalation rules. The Report Packager should show budget consumption: "This run used 38 MB of 100 MB, 2 of 3 OCR calls, and 1 retry."

### 7. Runtime permissions and operator trust need a visible ledger

BugWitness will ask for sensitive capabilities: screenshots, audio fixtures, browser sessions, network logs, device access, local model routing, and possibly test account credentials. Privacy reviews have covered redaction after capture, but operator trust starts before capture.

Users need to know:

- what permission was used
- why it was needed
- whether evidence stayed local
- whether any artifact was exported
- whether a model call left the machine

**Risk:** A local-first evidence tool that feels opaque will lose trust even if it is technically private. The first-run and per-run permission story matters as much as the redaction story.

**Recommendation:** Add a `PermissionLedger` to every run. It should record capabilities requested, capabilities granted, evidence exported, model routes used, and redaction policies applied. This can double as an audit artifact for teams and as a user-facing reassurance mechanism.

---

## Expansion Ideas

### BugWitness Doctor

A `bugwitness doctor` command that checks substrate availability before a scenario run:

- iHomeNerd reachable
- browser adapter healthy
- OCR and speech analyzers available
- device trust configured
- storage writable
- redaction policy valid
- scenario schema version supported

This would turn capability negotiation into an operator-friendly first step instead of a hidden runtime failure.

### Accessibility Replay

An evidence viewer mode that replays the focus path and accessibility tree changes step by step. For example: "Tab 1 focused Search; Tab 2 skipped the date picker; Enter opened modal; screen reader announcement missing." This complements screenshot replay rather than competing with it.

### Oracle Workbench

A small authoring surface for reviewing expectations before they become authoritative. It would show each assertion, its source, tolerance, and confidence, then let a human or agent promote it to a checked-in oracle. This is especially useful for first-run baselines and agent-generated scenarios.

### Analyzer Lockfile

Similar in spirit to a dependency lockfile, this records the analyzers used to produce processed evidence:

- OCR model
- speech model
- vision model
- prompts or extraction templates
- threshold settings
- redaction processors

Teams can choose either "locked analyzers for stable comparisons" or "latest analyzers with explicit drift warnings."

### Synthetic Account Vault

State pollution reviews correctly focus on cleanup. A complementary concern is identity lifecycle. BugWitness should manage synthetic test users, payment tokens, phone numbers, email inboxes, and permission states as named fixtures with expiration and cleanup metadata. This prevents real customer data from leaking into evidence and makes destructive flows safer.

### Witness Receipts

For lightweight sharing, BugWitness could emit a tiny "receipt" separate from the full evidence bundle: finding ID, scenario ID, app build, artifact hashes, severity, privacy level, and storage pointer. This is useful for Slack, GitHub comments, and agent handoffs where the full bundle is too large or too sensitive to paste around.

---

## Additional Perspectives

**On the witness metaphor:** The strongest version of this metaphor includes limits. A credible witness says "I saw this," but also "I could not see behind the counter," "my view was blocked," or "I am relying on another witness for that part." BugWitness should make unknowns and blind spots first-class, not embarrassing. A report that says "network logs unavailable; OCR confidence low; expectation inferred from baseline candidate" is more trustworthy than a cleaner report that hides those caveats.

**On schema design:** The next schema draft should not start with only `steps`. It should start with the whole claim chain: `capabilities -> scenario intent -> expectations -> observations -> analysis -> findings -> handoff`. Steps are the execution shape, but the product value is the claim that follows from them.

**On first implementation:** Qwen and Minimax are right that the first milestone should be one real scenario, one evidence capture, one report. I would make that scenario include one accessibility assertion and one analyzer-derived artifact from day one. That forces the system to handle raw observation, processed observation, and semantic expectation before the architecture hardens around screenshots only.

**On iHomeNerd integration:** The boundary should be dynamic, not just documentary. "iHomeNerd provides OCR" is less useful than "this iHomeNerd host provides OCR model X with confidence scores, local-only routing, and average processing latency Y." A versioned capability handshake preserves separation while making the integration testable.

**On market trust:** Local-first is a major differentiator only if users can verify it. A PermissionLedger, analyzer manifest, and export manifest turn "trust us, it stays local" into something inspectable. That will matter for regulated teams, but also for small teams who simply do not want bug reports accidentally carrying secrets.

---

## Summary of recommendations

1. Add a `SubstrateCapabilityManifest` handshake with iHomeNerd and include it in every evidence bundle
2. Add `AccessibilityWitness` as a first-class evidence stream covering accessible names, roles, focus order, ARIA state, and keyboard navigation
3. Add `ExpectationSource` / `OracleContract` metadata so findings can explain why an expected result was authoritative
4. Add an `AnalyzerManifest` and analyzer lockfile so OCR/speech/vision drift does not masquerade as product regression
5. Define a structured `HumanCheckpoint` protocol for semi-automated mobile and real-device flows
6. Add `RunBudget` controls for time, artifact size, analyzer/model usage, retry limits, and evidence level
7. Add a per-run `PermissionLedger` showing capabilities used, model routes, exports, and redaction policies
8. Build `bugwitness doctor` early as the operator-facing expression of capability negotiation
9. Include accessibility and analyzer-derived evidence in the first end-to-end spike, not as later polish
10. Consolidate the growing review vocabulary into a Phase 1 architecture glossary before code starts landing

The previous reviews have made the product sharper. My additional push is simple: make BugWitness excellent not only at recording evidence, but at explaining the authority, limits, and operating conditions of that evidence. That is where "witness" becomes more than a metaphor.
