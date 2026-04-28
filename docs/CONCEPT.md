# Concept

## What BugWitness is

BugWitness is a pluggable app concept that relies on `iHomeNerd` as its local
engine and evidence substrate.

BugWitness owns:

- scenario definitions
- flow execution logic
- assertions
- evidence packaging
- findings output

`iHomeNerd` provides:

- local node discovery
- secure runtime access
- browser-facing command surfaces
- OCR / speech / translation helpers
- storage and cross-device coordination

## First product shape

Start with:

- browser-based workflow testing
- mobile-browser workflow testing
- screenshot + OCR evidence capture
- fixture-based speech and upload flows
- reproducible markdown findings

## Example use cases

### 1. Scheduling app

Test a buggy booking flow such as a pet-walking scheduler:

- create recurring booking
- reschedule one event
- cancel one event
- verify timezone display
- capture screenshots and errors

### 2. Mobile UI trace

Use a real phone browser to:

- navigate a flow
- capture tap path
- record visible state
- correlate user action with observed failure

### 3. OCR-assisted validation

Use OCR to verify text-heavy flows:

- menu parsing
- labels and warnings
- confirmation pages
- PDF or image-driven interfaces

### 4. Speech-aware validation

Use speech fixtures or guided live checks to test:

- ASR upload paths
- TTS quality
- conversation loop regressions

## Feature modules

Possible modules inside BugWitness:

- `Scenario Runner`
- `BugWitness Evidence`
- `TapTrace`
- `Regression Compare`
- `Report Packager`

## Delivery shape

Likely progression:

1. docs and concept repo
2. shared test-case schema
3. top-level testing harness
4. integration with `iHomeNerd` Investigate-style evidence flows
