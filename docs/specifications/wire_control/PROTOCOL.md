# z-base Wire Control Protocol (WCP)

## Status

This document defines the **z-base Wire Control Protocol (WCP)**.

It specifies a **binary, versioned, role-aware wire protocol** governing how Actors and Base Stations exchange opaque byte sequences.

This protocol is **normative for wire behavior only**. It defines delivery obligations and structural validation rules and is **not authoritative over semantic meaning, state correctness, authorization, attribution, or lifecycle semantics**.

This document is the **single shared wire contract** referenced by:

[- Base Station Core](../base_station/components/STATION.md)

[- Actor Station Client](../actor/components/STATION_CLIENT.md)

RFC 2119 and RFC 8174 keywords apply.

---

## Design Goals

The Wire Control Protocol is designed to:

- support **fully asynchronous Actors** that may execute before, during, or after network connectivity,
- allow **baseline state injection** without coordination or blocking,
- preserve **semantic opacity** at the Base Station,
- scale across Resources with strict isolation guarantees,
- enable **mechanical protocol validation** without payload inspection,
- remain forward-compatible without invalidating existing implementations.

The protocol defines **how bytes move**, never **what they mean**.

---

## Core Invariants

- Payloads are opaque at all times.
- Delivery may be duplicated, reordered, or lossy.
- Verification gates byte exchange but does not define state lifecycle.
- Authority, merge semantics, attribution, and correctness exist strictly above the wire.
- Every connection is scoped to exactly one Resource.

---

## Transport Assumptions

- A reliable, message-framed transport (e.g. WebSocket).
- Message boundaries are preserved.
- Transport-level security (e.g. TLS) is assumed but not specified here.

---

## Frame Envelope

Every frame **MUST** use the following envelope:

```

[ 1 byte VERSION ][ 1 byte FRAME_CODE ][ N bytes PAYLOAD ]

```

### VERSION

- Indicates the Wire Control Protocol version.
- This specification defines **VERSION = 0x01**.
- Frames with unknown VERSION values **MUST** be rejected.

### FRAME_CODE

- Defines **wire-level obligation only**.
- Determines which party may emit the frame and how the Base Station must process it.
- Payload interpretation is undefined at the wire level.

---

## Frame Code Space Law *(Normative)*

Frame codes are allocated strictly by **emitter role** and **delivery obligation**.

This allocation rule is mandatory and mechanically enforceable.

| Range       | Class                     | Emitted By | Obligation |
|------------|---------------------------|------------|------------|
| 0x00–0x1F  | Station Control            | Station    | Verification and gating |
| 0x20–0x3F  | Client Submission          | Client     | Persistence or relay |
| 0x40–0x5F  | Station Relay / Offer      | Station    | Forwarding |
| 0x60–0x7F  | Reserved                   | —          | Undefined |
| 0x80–0xFF  | Forbidden                  | —          | Connection termination |

Any violation of emitter role or frame code range **MUST** result in connection termination.

---

## Defined Frame Codes (VERSION = 0x01)

### Station Control Frames (0x00–0x1F)

| Code | Name            | Direction             | Description |
|-----:|-----------------|-----------------------|-------------|
| 0x01 | AssertChallenge | Station → Client      | Verification challenge |

---

### Client Submission Frames (0x20–0x3F)

| Code | Name            | Direction             | Persisted | Description |
|-----:|-----------------|-----------------------|-----------|-------------|
| 0x21 | ProvePossession | Client → Station      | No        | Proof of Resource-bound key possession |
| 0x22 | SubmitSnapshot  | Client → Station      | Yes       | Persist opaque Snapshot |
| 0x23 | SubmitDelta     | Client → Station      | No        | Submit opaque Delta |
| 0x24 | RequestSnapshot | Client → Station      | No        | Request latest Snapshot |

---

### Station Relay Frames (0x40–0x5F)

| Code | Name           | Direction               | Description |
|-----:|----------------|--------------------------|-------------|
| 0x41 | OfferSnapshot  | Station → Client(s)     | Send persisted Snapshot |
| 0x42 | RelayDelta     | Station → Client(s)     | Relay Delta |

---

## Connection Phases *(Normative)*

### Phase 0 — Unverified

Immediately after connection establishment:

- The Station **MUST** send `AssertChallenge (0x01)`.
- The Client **MUST NOT** send any frame other than `ProvePossession (0x21)`.

Any other frame **MUST** cause immediate connection termination.

---

### Phase 1 — Verification

- The Client sends `ProvePossession (0x21)`.
- The Station verifies possession without semantic inspection.
- On failure, the connection **MUST** be closed.
- On success, the connection transitions to **Verified**.

---

### Phase 2 — Verified Session

Upon successful verification:

1. The connection is added to the verified peer set.
2. If a Snapshot is persisted, the Station **MUST immediately send** `OfferSnapshot (0x41)`.
3. Delta relay **MAY** begin concurrently.

No ordering guarantees exist between Snapshot offers and Delta relays.

---

## Snapshot Rules *(Wire-Level)*

### SubmitSnapshot (0x22)

- Payload is an opaque Snapshot.
- The Station **MUST** persist the payload verbatim.
- Any previously persisted Snapshot **MUST** be replaced.
- No inspection or validation is permitted.

---

### OfferSnapshot (0x41)

- Sent automatically after verification if a Snapshot exists.
- Sent in response to `RequestSnapshot (0x24)`.
- Payload is the latest persisted Snapshot.
- The Station **MUST NOT** wait for explicit request after verification.

---

### RequestSnapshot (0x24)

- The Client **MAY** request a Snapshot at any time after verification.
- The Station **MUST** respond with `OfferSnapshot (0x41)` if available.

---

## Delta Rules *(Wire-Level)*

### SubmitDelta (0x23)

- Payload is an opaque Delta.
- The Station **MUST NOT** persist Delta payloads.
- The Station **MUST** relay the payload to all verified peers except the sender.

---

### RelayDelta (0x42)

- Payload is forwarded verbatim.
- Delivery may be duplicated or reordered.
- No acknowledgment or ordering guarantees are provided.

---

## Resource Isolation

- Each connection is scoped to exactly one Resource.
- Frames **MUST NOT** cross Resource boundaries.
- Resource multiplexing within a session is forbidden.

---

## Failure Model

The protocol tolerates:

- connection churn,
- Station restarts,
- duplicate frames,
- reordering,
- partial delivery.

Recovery is entirely Actor-side.

---

## Conformance

An implementation conforms **iff** it:

- enforces VERSION and frame code space law,
- applies verification gating strictly,
- offers persisted Snapshots immediately after verification,
- preserves payload opacity and byte integrity,
- obeys all relay and persistence obligations.

---

## Closing Statement

This protocol defines **byte movement discipline only**.  
All semantic authority exists above the wire.
