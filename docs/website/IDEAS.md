# `/www` — Website, Platform, and Specification Scope

The `/www` directory contains the **entire public-facing z-base surface**, including **the React SPA, tooling, documentation assets, and the normative specification site**. It is intentionally **large and multi-purpose**.

---

## What `/www` Contains

### 1. React Single Page Application (SPA)

- Primary interactive UI for z-base
- Built with React
- Uses z-base directly (no mocks)
- Communicates with Base Stations hosted on Cloudflare Workers (Hono, OpenAPI)

---

### 2. Live z-base Demonstration Platform

The SPA is a **real, working system**.

It **MUST** demonstrate:
- Actor and Resource creation
- Snapshot persistence and recovery
- Delta relay and realtime updates
- Async state convergence
- Multi-device synchronization
- Collaboration scenarios

---

### 3. Guided Onboarding & Interactive Tutorials

The SPA includes progress-tracked learning paths:

- **No-code deployment**
  - Cloudflare Dashboard steps
- **Low-code deployment**
  - Wrangler CLI steps
- **Application building**
  - Schema → Resource → State
- **Credential management**
  - Adding discoverable credentials
  - State recovery across credentials
- **Failure & recovery**
  - Local state deletion
  - Snapshot-based restoration
- **Realtime & collaboration**
  - Cross-device delta observation

All progress **MUST** be driven by z-base state itself.

---

### 4. Reference Implementation UI

- Canonical UI implementation of z-base concepts
- Executable documentation
- Shows how Actors, Resources, Snapshots, Deltas, and Base Stations are used in practice

---

### 5. Protocol & Debug Tooling

Built-in tooling for:
- observing delta streams
- snapshot load events
- merge events
- connection and verification status
- Base Station interaction traces (without payload inspection)

---

### 6. Source Code Hosting & Inspection

- `/www` **MUST** expose and reference the actual implementation source code
- Embedded code viewers and links
- No illustrative or fake snippets — only real code

---

### 7. ReSpec Normative Specification Site

- `/www/spec/` **MUST** host a static **ReSpec `index.html`**
- This is the **canonical general + normative specification** of z-base
- Constructed from all Concept Specifications (Actor, Base Station, Wire, Security, Host Environment)
- No application logic
- No SPA behavior
- Pure specification content

This ReSpec site is also published on a **separate URL** (e.g. `spec.z-base.dev`), but **lives in this repository**.

---

### 8. End-to-End Test Harness

- `/www` is the **primary Playwright E2E test target**
- Tests cover:
  - UI flows
  - protocol behavior
  - recovery scenarios
  - multi-device sync
  - collaboration

The SPA **MUST remain testable and deterministic**.

---

## What `/www` Is Not

- Not a marketing site
- Not a minimal demo
- Not optional

---

## Design Principle

`/www` is where:

- specifications become executable,
- infrastructure behavior becomes observable,
- failures become undeniable.

If it cannot be demonstrated, specified, and tested here, it does not belong in z-base.
