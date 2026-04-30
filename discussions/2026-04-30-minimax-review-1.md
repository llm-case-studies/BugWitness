# Minimax Review — Round 1

**Date:** 2026-04-30
**Reviewer:** Minimax
**Docs reviewed:** VISION.md, CONCEPT.md, ROADMAP.md, MODULES.md, USE_CASES.md, RELATION_TO_IHN.md, SESSION_PORTABILITY_MANAGER_SPEC_2026-04-29.md, discussions/README.md, all prior DeepSeek/Grok/Nemotron/Qwen/Kimi-K2.6 reviews

## Summary

The prior five reviews have covered the architectural and philosophical terrain thoroughly. Rather than retread that ground, this review focuses on gaps that emerge at the ** seams between modules** — the handoff protocols, the trust propagation paths, and the narrative coherence problems that don't appear in any single module's spec but determine whether BugWitness produces insights or just artifacts.

---

## Strengths

- **Evidence narrative over evidence dump** — The framing around "what happened / what did we expect / what proof do we have / how reproducible" is not just a report template. It is a *reasoning structure* that forces every piece of evidence to serve a causal argument. Most testing tools produce data; BugWitness produces testimony.

- **Substrate discipline is genuinely rare** — The iHomeNerd boundary is not just a clean architectural choice; it is a strategic one. By keeping BugWitness opinionated about testing while deferring device access to the substrate, the project avoids the trap of becoming a "also does device access, also does OCR, also does..." blob.

- **Session Portability Manager as architectural proof-of-concept** — Kimi-K2.6 and Qwen both made this point well. The Session Manager is not just a utility; it is the first end-to-end test of the evidence pipeline. If BugWitness can faithfully migrate a session bundle with provenance intact, it has proven the thesis before it needs to prove it in production testing.

- **Anti-goals create a defensible identity** — "Not Selenium, not CI, not load testing" is a contract that future contributors can use to reject feature requests. The testing tool space is littered with projects that started focused and slowly became unrecognizable. The anti-goals are the load-bearing walls of this concept's identity.

---

## Concerns

### 1. Evidence narrative coherence is unspecified

BugWitness produces multiple evidence streams: screenshots, OCR text, TapTrace gestures, network logs, speech transcripts. Each stream is captured, processed, and stored independently. But at consumption time, these streams need to tell a *single coherent story* about what happened.

There is no discussion of:
- How to ensure the "narrative" of step 3's screenshot is consistent with step 3's TapTrace sequence and step 3's OCR result
- What happens when a screenshot shows one thing and the network log contradicts it (e.g., screenshot shows "loading" but the API returned 200)
- How the Report Packager resolves conflicts between evidence streams

**Risk:** BugWitness becomes a high-fidelity evidence *aggregator* rather than an evidence *interpreter*. Users get a pile of artifacts and must do the reasoning work themselves — which defeats the "clear findings" product goal.

**Recommendation:** Add a `EvidenceNarrative` layer to Phase 1. Define a `StepSummary` structure that contains the authoritative story of each step: what action was taken, what the DOM/state showed, what the OCR extracted, what the network logged, and how these agree or disagree. The Report Packager should render these summaries, not raw evidence bundles.

### 2. Cross-module handoff protocols are unmodeled

MODULES.md names six modules but says nothing about how they communicate. Prior reviews correctly flagged this as an interface problem, but the deeper issue is the *handoff protocol* — what guarantees does each module provide about the data it passes downstream?

Specifically:
- What does BugWitness Evidence guarantee about a screenshot before passing it to Regression Compare?
- What does Scenario Runner guarantee about step ordering before passing execution metadata to Report Packager?
- What does TapTrace produce that Regression Compare can reliably consume?

**Risk:** Module implementations diverge because each team makes different assumptions about what the upstream module "means to tell me." The resulting reports are inconsistent even when the underlying evidence is correct.

**Recommendation:** Define a minimal `HandoffContract` for each module pair in Phase 1. Not a full interface spec — just the minimum shared understanding of what data means and what guarantees attach to it.

### 3. Trust propagation through the evidence chain is invisible

Kimi-K2.6 raised attestation. Grok raised provenance. But neither addressed the specific problem of *trust propagation*: when evidence moves from capture (device) to processing (iHomeNerd OCR) to packaging (BugWitness Report Packager) to delivery (markdown report), how much does each step degrade or preserve the trust characteristics of the previous step?

The key question: if a screenshot is captured on a real device and processed by a local OCR model, what makes the final markdown report trustworthy enough to hand to a developer as evidence of a bug? Not cryptographic signing (Nemotron's concern) — but *semantic trust*: did the pipeline faithfully represent what the device observed?

**Risk:** Reports that look credible but misrepresent the actual device state are worse than no report — they mislead confidently. This is the "false witness" failure mode, and it is not addressed.

**Recommendation:** Introduce a `TrustLevel` annotation on every evidence artifact (captured_as_is / processed_faithfully / inferred / transformed). This lets consumers understand how far an artifact is from raw observation and weight it accordingly.

### 4. Evidence expiry and staleness is unaddressed

BugWitness captures evidence about an app at a point in time. Apps change. UI updates, API versions shift, new builds are deployed. A screenshot captured last week might be evidence of a bug that no longer exists in the current build.

No discussion of:
- How to annotate evidence with app version / build fingerprint
- How to detect when stored evidence is from a stale version
- Whether old evidence should be archived, flagged, or auto-pruned

**Risk:** BugWitness becomes a museum of old bugs. Over time, users accumulate evidence from many app versions and the finding relevance degrades. "We found this bug in version 2.1.4" is only meaningful if version 2.1.4 is the current version or if the bug still exists.

**Recommendation:** Define an `EvidenceMetadata` standard in Phase 1 that includes app version, build hash, and capture timestamp. The Report Packager should surface version context prominently and flag when evidence is older than the current deployed version.

### 5. The "handoff surface" problem: markdown works for humans, not for agents

Kimi-K2.6 suggested JSON-LD alongside markdown. Qwen's Scenario Linter is a machine-facing artifact. But the deeper problem is that BugWitness currently has no coherent strategy for serving both human consumers and agent consumers from the same evidence bundle.

A developer wants: a readable markdown report with screenshots
An agent wants: structured, machine-parseable findings it can act on

These are not the same artifact. If BugWitness produces one and converts it to the other, information is lost in both directions (markdown loses structure; converted JSON loses narrative intent).

**Recommendation:** Design evidence bundles as having two canonical views: the *narrative view* (markdown) and the *structural view* (JSON). Both are primary citizens, not derived from each other. This doubles the output surface but eliminates the conversion fidelity problem.

### 6. Scenario isolation and parallelism is undefined

All current docs assume sequential scenario execution: run scenario A, capture evidence, produce report. But a real QA workflow runs many scenarios: smoke tests, regression suites, exploratory sessions.

No discussion of:
- How scenarios share or isolate from each other (same browser instance? fresh browser per scenario?)
- Whether scenarios can run in parallel across devices
- How evidence from parallel runs is correlated or kept separate

**Risk:** Early implementation picks ad-hoc isolation defaults that become load-bearing as usage grows. If the wrong defaults are chosen (e.g., shared browser state between scenarios), baseline comparisons become unreliable.

**Recommendation:** Define a `ScenarioExecutionModel` in Phase 1 with clear isolation semantics. "Fresh browser per scenario" is the safest default for reproducibility; shared session might be needed for complex flows. Make the choice explicit.

---

## Expansion Ideas

### Finding Prioritization Framework

Not all findings are equal. A regression in the checkout flow is more urgent than a timezone label being off by one day in a calendar view. BugWitness should support severity classification:

- **Critical** — data loss, security issue, payment failure
- **High** — flow break, assertion failure in happy path
- **Medium** — visual regression, OCR mismatch
- **Low** — cosmetic, intermittent, environment-specific

The scenario schema should allow severity annotations on assertions. The Report Packager should sort findings by severity. This makes reports immediately actionable rather than requiring the consumer to triage everything equally.

### Evidence Diff Viewer

When Regression Compare identifies a change, showing "what changed" in a flat diff is less useful than showing "what changed in the context of the scenario narrative." An Evidence Diff Viewer would render:

- Side-by-side screenshot comparison with highlighted regions
- OCR text diff with character-level highlighting
- Network log diff with request/response delta

The viewer should be navigable by scenario step, so the user can step through changes in execution order, not in file-order.

### Cross-Scenario Correlation

When multiple scenarios run against the same app version and produce failures, those failures might share a root cause. A lightweight correlation engine could detect:

- Same API endpoint failing across different scenarios
- Same UI element being the point of divergence
- Time-correlated failures suggesting environmental cause

This is adjacent to Qwen's Cross-Session Correlation idea but scoped to scenario runs rather than user sessions.

### Scenario Health Scoring

Beyond pass/fail, a scenario's health could be measured over time:

- **Staleness score** — how long since last successful run
- **Flakiness score** — ratio of failures to total runs
- **Evidence cost** — storage consumed per run

This gives teams a dashboard for managing their scenario library, not just running it. A scenario that is always failing might be a broken scenario, not a broken app.

---

## Additional Perspectives

**On the witness metaphor's limits:** The "witness" framing is compelling but has an implicit limit — a witness testifies about what they observed, not about whether it matters. BugWitness needs to bridge to *assessment* (this is a real bug, not just a difference) and *recommendation* (here is how to reproduce it, here is the likely cause). The current docs stop at "capture evidence." The next layer is judgment, and it deserves at least a conceptual treatment.

**On the first milestone:** Qwen's recommendation — "one scenario, one evidence capture, one report" — is the right frame. But I would add: make that milestone produce evidence for a *real bug you already know about* in a real app. The spike should validate not just the technical wire but the emotional experience of receiving a BugWitness report and feeling like it captured truth accurately. If it doesn't feel credible in the first real use, no amount of schema rigor will recover it.

**On fixture versioning:** Fixture Lab is discussed as storing fixtures, but not versioning them. Fixtures are essentially test oracles — they define what "correct" looks like. When a fixture changes (e.g., a new speech sample replaces an old one), the Regression Compare baseline is invalidated without notice unless there is explicit versioning. This is a silent data integrity risk.

**On TapTrace specifically:** Qwen flagged the boundary ambiguity between TapTrace, Scenario Runner, and Regression Compare. My addition: TapTrace is most credible as a *passive recording layer* that the Scenario Runner queries. If TapTrace starts making its own assertions or producing its own findings, it becomes a second Scenario Runner and the modular decomposition breaks. Define TapTrace as strictly observational: it records what happened, it does not judge whether that was right.

---

## Summary of recommendations

1. Add an `EvidenceNarrative` layer that produces a coherent step-by-step story from heterogeneous evidence streams, resolving conflicts between streams explicitly
2. Define minimal `HandoffContract` specs for each module pair in Phase 1 — not full interfaces, just the shared understanding of what data means
3. Introduce a `TrustLevel` annotation on all evidence artifacts (captured_as_is / processed_faithfully / inferred / transformed) so consumers can weight evidence appropriately
4. Define `EvidenceMetadata` standard including app version and build hash; surface version context in reports and flag stale evidence
5. Design evidence bundles with two canonical views: narrative (markdown) and structural (JSON), both primary, neither derived
6. Define a `ScenarioExecutionModel` with explicit isolation semantics — fresh browser per scenario is the safe default
7. Add a finding prioritization framework (Critical / High / Medium / Low) with scenario-level severity annotations
8. Keep TapTrace strictly observational — a passive recording layer queried by Scenario Runner, not an active comparator
9. Add fixture versioning to Fixture Lab to prevent silent baseline invalidation
10. Run the first end-to-end spike against a *known real bug* and validate the emotional credibility of the output, not just its technical correctness

The prior reviews built the map. These suggestions aim to ensure the evidence that BugWitness produces is not just captured faithfully but interpreted coherently — because a pile of trustworthy evidence is still just a pile without a narrative to give it meaning.