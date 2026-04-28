# BugWitness

BugWitness is a testing and evidence app built to sit on top of `iHomeNerd`.

The goal is simple:

- exercise real app flows
- capture proof when something breaks
- turn that proof into clear findings

BugWitness is not meant to replace `iHomeNerd`.
`iHomeNerd` is the local substrate:

- discovery
- trust / TLS
- browser and device access
- local AI routing
- OCR / speech / evidence capture
- report synthesis

BugWitness is the specialist app on top:

- test scenarios
- replay and trace
- bug evidence collection
- regression comparison
- findings and handoff notes

## Initial docs

- [docs/VISION.md](docs/VISION.md)
- [docs/CONCEPT.md](docs/CONCEPT.md)
- [docs/RELATION_TO_IHN.md](docs/RELATION_TO_IHN.md)

## Working direction

Near-term focus:

- web and mobile-web workflow testing
- evidence capture from real browsers and devices
- guided real-device checks with semi-automated harnesses
- OCR- and speech-aware testing where useful

Longer-term:

- scenario execution
- tap / gesture tracing
- regression clustering
- bug report generation
- overlap with marketplace / investigation style evidence collection
