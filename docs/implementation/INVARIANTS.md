# Invariants — Implementation Rules

This document defines **how the specifications are implemented**, not how they are written or judged.  
These rules apply to **all code, tooling, tests, benchmarks, and packaging** in this repository.

Violation of any rule means the implementation is **incomplete**.

---

## 1. Test-Verified Implementation Only

- **All code MUST be test-verified**.
- Tests are mandatory at:
  - unit level,
  - integration level,
  - end-to-end (E2E) level.

- An implementation **DOES NOT EXIST** unless:
  - it is exercised by tests,
  - those tests pass deterministically.

Untested code is treated as dead code and **MUST be removed**.

---

## 2. End-to-End First Execution Model

- Every implemented behavior **MUST** be observable through an E2E execution path.
- Unit tests exist to *support* E2E correctness, not replace it.
- If behavior cannot be exercised end-to-end, it **MUST NOT be implemented**.

The running system is the proof.

---

## 3. Benchmarks as First-Class Artifacts

- Every meaningful logical unit **MUST** have:
  - a measurable benchmark,
  - a clearly defined scope,
  - recorded baseline results.

- Benchmarks **MUST**:
  - be tied to a specific logical unit,
  - be repeatable,
  - be versioned and tracked.

- Performance work **MUST**:
  - preserve test-defined behavior,
  - be validated by benchmark comparison,
  - allow recursive optimization without changing observable behavior.

---

## 4. Zero Boilerplate, Zero Duplication

- Code **MUST** be composed of reusable:
  - functions,
  - classes,
  - components.

- Boilerplate is **not allowed**.
- Repetition is **a defect**.

If two blocks of code look similar, they are wrong.

---

## 5. First Implementation Purity

- This is the **first implementation**.
- There is **no obligation** to preserve existing structure.
- Any unnecessary, unclear, or speculative code **MUST be deleted**.

Refactoring is not optional.  
Deletion is preferred.

---

## 6. Single-Way Usage Principle

- The implementation **MUST expose exactly ONE valid way to be used**.
- Optional configuration is **disallowed**.
- Variants, toggles, or alternative flows are **not permitted**.

Only configuration explicitly required by implementation needs (e.g. schema definitions) is allowed.

If something can be used in two ways, the implementation is wrong.

---

## 7. Strict Typing & Schema-Inferred SDK

- The implementation **MUST produce a strictly typed SDK**.
- Types **MUST** be inferred from schemas.
- Loose typing is forbidden.

The SDK **MUST**:
- guide correct usage through types alone,
- make invalid usage unrepresentable,
- be suitable for frontend application development.

Exact APIs will be defined later, but **type safety is non-negotiable**.

---

## 8. Portable, Reusable Infrastructure Code

The implementation **MUST yield reusable infrastructure artifacts**.

### API / Endpoint Layer
- Exportable as reusable classes or functions.
- Built with:
  - Hono
  - OpenAPI-compliant definitions
  - Zod for validation
  - Chafana where applicable

### Cloudflare Workers Support
- A portable **Durable Object class MUST be provided**.
- The Durable Object **MUST**:
  - integrate with the Hono endpoints,
  - be deployable on Cloudflare Workers,
  - contain no application-specific logic.

All infrastructure code **MUST be importable**, not copy-pasted.

---

## 9. No Partial Implementation

- Features are **all-or-nothing**.
- “Partially implemented” is invalid.
- “We’ll test it later” is invalid.

If something exists:
- it must be tested,
- it must be benchmarked,
- it must be typed,
- it must be reusable.

Otherwise it **MUST NOT exist**.

---

## 10. Implementation Completion Criteria

The implementation is complete **only when**:

- all code paths are test-covered,
- all tests pass deterministically,
- benchmarks exist and are recorded,
- no unused or duplicated code remains,
- there is exactly one correct usage path.

Anything less is an incomplete implementation.
