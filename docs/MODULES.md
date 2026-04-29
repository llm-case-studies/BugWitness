# BugWitness Modules

BugWitness should stay modular so the product can grow without turning into an undifferentiated test blob.

## Scenario Runner

Owns:

- scenario definitions
- step execution
- preconditions
- expected outcomes
- guided manual checkpoints

This is the orchestration layer for a test flow.

## BugWitness Evidence

Owns:

- screenshots
- OCR text
- visible error extraction
- request and response attachments where available
- timestamps and environment notes

This is the proof layer.

## TapTrace

Owns:

- mobile tap sequences
- gesture and control-path recording
- replay-oriented action logs
- mismatch reports between expected and actual flow path

This is a likely feature or sub-brand, not the whole product.

## Regression Compare

Owns:

- run-to-run diffing
- screenshot comparison support
- transcript and OCR text comparison
- environment-aware summaries

This answers:

- what changed
- what regressed
- what improved

## Report Packager

Owns:

- markdown findings
- JSON artifacts
- developer handoff notes
- human-readable summaries

This is the delivery layer.

## Fixture Lab

Owns:

- prerecorded speech fixtures
- image fixtures for OCR
- canonical scenario inputs
- stable comparison baselines

This keeps testing deterministic where possible.

## Relation to iHomeNerd

These modules should rely on iHomeNerd for:

- device and node discovery
- local trust and TLS handling
- browser and runtime access
- OCR and speech helpers
- local model routing
- cross-device coordination

BugWitness should own the testing opinion.
iHomeNerd should remain the substrate.
