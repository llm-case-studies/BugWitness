# BugWitness Early Roadmap

This roadmap is intentionally narrow. The goal is to get to a useful first product shape fast.

## Phase 0. Concept framing

Status:

- done enough to start

Outputs:

- vision
- concept boundaries
- relation to iHomeNerd
- initial use cases
- initial modules

## Phase 1. Scenario and evidence contract

Goal:

- define how BugWitness describes a test scenario and a result

Deliverables:

- scenario schema
- result schema
- artifact bundle format
- markdown report template

Why first:

- this defines the product before implementation sprawl begins

## Phase 2. Browser-first workflow probe

Goal:

- test ordinary web and mobile-web flows through a browser-first harness

Deliverables:

- scenario runner prototype
- screenshot capture
- OCR extraction support
- simple pass/fail assertions
- evidence bundle export

Target outcomes:

- scheduling and booking app probe
- form and checkout flow probe

## Phase 3. Real-device probe mode

Goal:

- use phones and tablets as guided testing probes

Deliverables:

- semi-automated mobile browser flows
- manual checkpoint support
- device metadata capture
- environment and permission notes

Target outcomes:

- Safari and Android browser comparison
- microphone and camera path checks

## Phase 4. Speech and OCR fixture support

Goal:

- make speech and OCR features testable, not just demoable

Deliverables:

- fixture-based audio upload tests
- OCR fixture tests
- transcript comparison support
- backend and language routing visibility

## Phase 5. Regression compare and issue packaging

Goal:

- explain what changed between runs

Deliverables:

- run diff summaries
- screenshot and text comparison helpers
- issue-ready report packs
- agent-friendly findings output

## Deliberate non-goals for the first cycle

- full native-app automation platform
- giant CI-first framework
- generic load testing
- exhaustive cross-browser matrix on day one

The product should first become excellent at a narrower claim:
capture real workflow failures and package them into credible evidence.
