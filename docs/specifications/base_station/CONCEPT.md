# Base Station — Concept Specification

## Abstract

A **Base Station** is a non-authoritative infrastructure service responsible for authenticated connectivity, persistence of opaque Resource Snapshots, and relay of opaque Snapshot and Delta frames for exactly one Resource.

A Base Station has no semantic awareness and no authority over state correctness, authorization, attribution, merge logic, or lifecycle enforcement. It operates strictly as an opaque byte relay and persistence mechanism.

This document defines the **normative requirements** for Base Station implementations.

---

## Status

This document is a **Draft Specification**, version **1.0.0**, edited **16 January 2026**.  
It is provided for review and experimentation and **MUST NOT** be considered stable or complete.

Normative requirements and component boundaries may change without notice.

---

## Conformance

This specification defines requirements for **Base Station implementations**.

An implementation conforms **iff** it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

---

## Base Station Definition *(Normative)*

A Base Station is a per-Resource infrastructure service that:

- serves **exactly one Resource identifier**,  
- verifies **possession** of Resource-bound cryptographic material,  
- persists **opaque Resource Snapshots**, and  
- relays **opaque Snapshot and Delta frames** between verified peers.

A Base Station **MUST NOT**:

- interpret payload contents,  
- validate or merge state,  
- enforce authorization or role semantics,  
- attribute identity,  
- track acknowledgments, or  
- enforce revocation or lifecycle rules.

---

## Resource Scope *(Normative)*

- Each Resource **MUST** be served by exactly one logically isolated Base Station.
- All connections, frames, persistence, and peer sets **MUST** be scoped to that Resource.
- Cross-Resource routing, persistence, or peer visibility is **non-conformant**.

---

## Architectural Components *(Normative)*

A conforming Base Station implementation **MUST** include a per-Resource stateful core component defined in:

- [Base Station Core — Component Specification](components/STATION.md)

Additional infrastructure components (e.g. edge routing, traffic filtering) **MAY** exist but **MUST NOT** introduce application semantics or authority.

---

## Protocol Compliance *(Normative)*

A Base Station **MUST** implement all wire-level behavior exactly as defined in:

- [Wire Control Protocol](../wire_control/PROTOCOL.md)

The Base Station **MUST NOT** extend, reinterpret, or weaken the protocol.

---

## Snapshot Handling *(Normative)*

- Snapshots received from verified clients **MUST** be treated as opaque byte sequences.
- Only the **latest Snapshot** **MUST** be persisted.
- Persisted Snapshots **MUST** survive restart, eviction, or hibernation.
- Snapshot payloads **MUST NOT** be inspected, modified, or validated.

---

## Delta Handling *(Normative)*

- Delta payloads **MUST NOT** be persisted.
- Delta frames **MUST** be relayed to all verified peers except the sender.
- Delivery **MAY** be duplicated, reordered, or lossy.

---

## Failure Model *(Normative)*

A Base Station **MUST** tolerate:

- restarts and hibernation,  
- connection churn,  
- duplicate frames,  
- partial relay and reordering.

Correctness is preserved because the Base Station is non-authoritative.

---

## Security and Isolation *(Normative)*

A Base Station **MUST** ensure:

- isolation between Resources,  
- isolation between verified and unverified connections, and  
- that overload or failure of one Resource **MUST NOT** cascade to others.

A Base Station **MUST NOT** log Snapshot or Delta payloads or derive semantic meaning from identifiers or payloads.

---

## Explicit Non-Goals *(Normative)*

A Base Station **MUST NOT**:

- merge CRDTs,  
- enforce ACLs or roles,  
- implement acknowledgment tracking,  
- implement revocation logic, or  
- act as a source of truth.

Any such behavior is **non-conformant**.

---

## Closing Principle *(Non-Normative)*

The Base Station provides persistence and relay only.  
All semantic authority resides with Actors.
