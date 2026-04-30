# Nemotron Review — Round 1

**Date:** 2026-04-30
**Reviewer:** Nemotron
**Docs reviewed:** VISION.md, CONCEPT.md, ROADMAP.md, MODULES.md, USE_CASES.md, RELATION_TO_IHN.md, SESSION_PORTABILITY_MANAGER_SPEC_2026-04-29.md

## Summary

Strong alignment with DeepSeek's assessment. The evidence-first thesis and iHomeNerd separation are particularly well-executed. A few architectural and implementation considerations warrant attention to ensure scalability and clarity.

---

## Strengths

- **Evidence-first framing** — The "what happened / what did we expect / what proof do we have / how reproducible" structure is actionable and distinct from vanilla test runners.
- **Clean substrate separation** — BugWitness owns testing opinions; iHomeNerd handles device access, OCR/speech, and coordination. Prevents scope creep in both projects.
- **Real-device-as-probe insight** — Recognizing phones/tablets as active testing agents (not just passive targets) leverages iHomeNerd's strengths and matches real bug environments.
- **Thoughtfully scoped anti-goals** — Explicitly rejecting Selenium-replacement, CI-platform, and load-testing ambitions focuses effort on evidence capture and reproducibility.
- **Session Portability Manager innovation** — Turning session-migration pain into a product feature shows lateral thinking and fits the evidence narrative.

---

## Concerns

### 1. **Module interfaces need early definition**
MODULES.md describes responsibilities but not contracts. For example:
- How does Scenario Runner trigger evidence capture in BugWitness Evidence?
- What data structure flows from TapTrace to Regression Compare?
- Without Phase 1 interface specs, implementation risks tight coupling or mismatched expectations.

### 2. **Evidence format lacks standardization**
Phase 1 focuses on scenario schema but omits evidence artifact structure. Need:
- Canonical JSON structure for screenshots, OCR results, timestamps, device metadata
- Versioning strategy for evidence schemas to support backward compatibility
- Clear relationships between evidence types (e.g., linking OCR text to UI coordinates)

### 3. **Partial success modeling is underspecified**
Current pass/fail binary doesn't capture nuanced outcomes:
- How to represent steps with expected vs actual timing variances?
- What about infrastructure errors (timeouts, network) vs application logic errors?
- How to represent "expected degradation" (e.g., slower performance but functional)?

### 4. **Local storage strategy undefined**
Given iHomeNerd as local substrate:
- Where are scenarios stored? Evidence bundles?
- What's the sync/backup story for evidence across devices?
- How do users export/share evidence bundles outside the iHomeNerd ecosystem?

---

## Expansion Ideas

### 5. **Evidence integrity mechanisms**
Add cryptographic hashing or signing to evidence bundles for:
- Audit trail compliance (for regulated industries)
- Tamper detection in shared environments
- Trusted handoffs to external agents or developers

### 6. **Adaptive evidence capture**
Not all steps need equal fidelity:
- Define evidence levels (minimal/standard/verbose) configurable per scenario
- Conditionally capture expensive evidence (full video, speech) only on failure
- Allow step-level overrides for OCR/speech analysis requirements

### 7. **Parameterized scenario execution**
Enable data-driven testing:
- Clear separation between scenario logic and test data fixtures
- Support for data sources (CSV, JSON fixtures)
- Built-in data generation for common patterns (dates, emails, IDs)

---

## Recommendations

1. **Define module interfaces** alongside scenario schema in Phase 1 (input/output contracts, data formats)
2. **Standardize evidence format** early—include JSON schema, versioning, and cross-artifact relationships
3. **Develop error taxonomy** distinguishing application failures, infrastructure issues, and environmental variables
4. **Clarify local persistence** strategy for scenarios/evidence using iHomeNerd's storage primitives
5. **Consider evidence validation** (hashing/signing) for compliance-sensitive use cases

These suggestions complement DeepSeek's excellent feedback and target technical foundations to support the ambitious vision.

(End of file)