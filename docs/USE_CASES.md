# BugWitness Use Cases

BugWitness should start from real failure-prone workflows, not abstract test-framework ambitions.

## Primary use cases

### 1. Scheduling and booking flows

Examples:

- dog walking or pet-care booking
- salon or clinic appointment scheduling
- tutoring or lesson booking
- event registration with time-slot selection

Why this matters:

- date and time pickers break often
- timezone handling is error-prone
- mobile form flows are fragile
- confirmation and reminder states are easy to regress

### 2. Mobile web checkout and submission flows

Examples:

- add to cart and checkout
- multi-step lead forms
- upload and submit workflows
- opt-in and confirmation flows

Why this matters:

- these flows fail in small layout details, validation edges, and hidden state changes
- they are painful to debug from a single screenshot

### 3. Evidence-heavy bug reporting

Examples:

- capture the failing state
- record the exact sequence of steps
- preserve OCR text from dialogs, canvases, or images
- bundle logs, screenshots, and notes into a usable report

Why this matters:

- many teams know something is broken but lack clean proof
- BugWitness should reduce "cannot reproduce" loops

### 4. Real-device regression checks

Examples:

- Safari on iPhone vs Chromium on desktop
- small Android screen vs foldable external display
- microphone, camera, or permission paths on real devices

Why this matters:

- desktop-only testing misses real mobile failures
- a device-as-probe model fits well with iHomeNerd

### 5. OCR-assisted UI validation

Examples:

- verify rendered invoice text
- validate menu text or order summaries
- detect broken empty states or mis-rendered call-to-action text
- inspect image-heavy or canvas-heavy UIs

Why this matters:

- some important UIs are not meaningfully testable through DOM queries alone

### 6. Speech-aware workflow testing

Examples:

- upload a known audio fixture and validate transcript quality
- test voice reply availability and behavior
- verify language routing and backend selection

Why this matters:

- speech features are expanding, but test coverage around them is usually poor

## First target market shape

BugWitness should first target:

- small product teams
- founders with mobile-web products
- QA-light teams that still need evidence-rich bug reports
- internal agent-assisted development workflows

It should not try to compete on day one with giant generic automation suites.

## First anti-goals

BugWitness should not start as:

- a giant Selenium replacement
- a full CI platform
- a native-app reverse-engineering product
- a generic load-testing tool

The first value is clear: run meaningful user scenarios, preserve evidence, and explain what failed.
