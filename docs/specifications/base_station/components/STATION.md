# Base Station Core — Component Specification

## Status

This document defines the **per-Resource Base Station Core** component.

It specifies the **normative responsibilities, constraints, and prohibitions** of a Base Station instance serving **exactly one Resource**.

This component **MUST** implement the wire-level behavior defined in:

- [Wire Control Protocol](../../wire_control/PROTOCOL.md)

and **MUST** conform to:

- [Base Station — Concept Specification](../CONCEPT.md)

RFC 2119 and RFC 8174 keywords apply.

---

## Role Definition

The **Base Station Core** is a per-Resource, stateful infrastructure component that:

- performs possession-based verification,
- maintains a verified peer connection set,
- persists the latest opaque Resource Snapshot,
- relays opaque Snapshot and Delta frames.

The Base Station Core:

- has **no semantic awareness**,
- has **no authority over state correctness**,
- is **not an Actor**.

---

## Resource Scope *(Normative)*

- The Base Station Core **MUST** serve **exactly one Resource identifier**.
- All connections, frames, persistence, and in-memory state **MUST** be scoped to that Resource.
- Cross-Resource routing, persistence, or peer awareness is **non-conformant**.

---

## Protocol Compliance *(Normative)*

The Base Station Core **MUST** implement the Wire Control Protocol exactly as defined in:

- [Wire Control Protocol](../../wire_control/PROTOCOL.md)

Including, but not limited to:

- frame envelope and version enforcement,
- frame code space law,
- connection phases and verification gating,
- relay and persistence obligations.

Any deviation is **non-conformant**.

---

## Connection Handling *(Normative)*

### Unverified Phase

Upon connection establishment, the Base Station Core **MUST**:

1. Immediately emit `AssertChallenge (0x01)`.
2. Reject or terminate the connection if any frame other than `ProvePossession (0x21)` is received.

No Snapshot or Delta frames **MAY** be persisted or relayed during this phase.

---

### Verification Phase

The Base Station Core **MUST**:

- verify proof of possession of Resource-bound cryptographic material,
- perform verification without inspecting payload meaning,
- treat verification strictly as a **gating mechanism**, not identity attribution.

On verification failure, the connection **MUST** be terminated.

---

### Verified Session

Upon successful verification, the Base Station Core **MUST**:

1. Add the connection to the verified peer set.
2. **Immediately offer the persisted Snapshot**, if one exists, via `OfferSnapshot (0x41)`.
3. Begin concurrent relay of Delta frames without ordering guarantees.

Verification **MUST NOT** be treated as a synchronization barrier.

---

## Snapshot Responsibilities *(Normative)*

### Snapshot Persistence

- The Base Station Core **MUST** persist only frames received via `SubmitSnapshot (0x22)`.
- Persistence **MUST** be byte-for-byte opaque.
- Only the **latest Snapshot** **MUST** be retained.
- Persisted Snapshots **MUST** survive restart, eviction, or hibernation.

The Base Station Core **MUST NOT**:

- merge Snapshots,
- validate Snapshot contents,
- infer causality, authorship, or lifecycle state.

---

### Snapshot Offering

The Base Station Core **MUST**:

- automatically send `OfferSnapshot (0x41)` after verification if a Snapshot exists,
- respond to `RequestSnapshot (0x24)` with `OfferSnapshot (0x41)` when possible.

The Base Station Core **MUST NOT** require explicit readiness signals or coordination.

---

## Delta Responsibilities *(Normative)*

- Delta payloads **MUST NOT** be persisted.
- Upon receiving `SubmitDelta (0x23)`, the Base Station Core **MUST** relay the payload to all verified peers except the sender.
- Delivery **MAY** be duplicated, reordered, or dropped.

The Base Station Core **MUST NOT** attempt replay protection, acknowledgment tracking, or ordering enforcement.

---

## Relay Semantics *(Normative)*

For all relayed frames, the Base Station Core **MUST**:

- preserve payload bytes exactly,
- avoid reflection to the sender,
- avoid semantic inspection,
- avoid blocking on persistence or peer state.

---

## Failure and Restart Model *(Normative)*

The Base Station Core **MUST** tolerate:

- process restarts,
- connection churn,
- duplicate frames,
- partial relay and reordering.

On restart, the Base Station Core **MUST**:

- restore the latest persisted Snapshot into memory,
- resume normal protocol behavior.

Correctness is preserved because the Base Station Core is non-authoritative.

---

## Security and Isolation *(Normative)*

The Base Station Core **MUST** ensure:

- isolation between Resources,
- isolation between verified and unverified connections,
- that overload or failure of one Resource **MUST NOT** cascade to others.

The Base Station Core **MUST NOT**:

- log Snapshot or Delta payloads,
- derive semantic meaning from identifiers or payloads,
- attribute Actor identity beyond possession gating.

---

## Explicit Non-Goals *(Normative)*

The Base Station Core **MUST NOT**:

- validate or merge state,
- enforce authorization or roles,
- track acknowledgments,
- detect or enforce revocation,
- act as a source of truth.

Any such behavior is **non-conformant**.

---

## Conformance

A Base Station Core implementation conforms **iff** it:

- fully implements the Wire Control Protocol,
- enforces strict per-Resource isolation,
- persists only opaque Snapshots,
- relays frames without semantic interpretation,
- injects persisted Snapshots immediately after verification.

---

## Closing Statement *(Non-Normative)*

The Base Station Core enforces wire discipline and byte persistence only.  
All semantic authority exists above this component.
