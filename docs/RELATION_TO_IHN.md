# Relation To iHomeNerd

BugWitness should be designed as an ecosystem app, not a fork of `iHomeNerd`.

## Separation of concerns

### iHomeNerd

Acts as:

- local AI and device substrate
- secure runtime host
- capability router
- evidence collection helper
- local report generator

Relevant `iHomeNerd` capabilities:

- browser-facing Command Center
- `Investigate`-style evidence collection
- OCR / image analysis
- speech tooling
- LAN discovery and trust

### BugWitness

Acts as:

- testing app
- scenario library
- flow runner
- bug evidence organizer
- regression and findings surface

## Why keep them separate

This separation keeps `iHomeNerd` from turning into:

- a giant product blob
- a test-only tool
- a niche QA brand

And it lets BugWitness evolve independently for:

- QA teams
- solo devs
- local-first debugging
- client-facing repro capture

## Shared future opportunities

BugWitness can later share subsystems with:

- `iScamHunter`
- `Investigate`
- OCR-driven drill generation
- cross-device real-world probing

The common theme is:

- collect evidence
- normalize it
- compare it
- explain it
