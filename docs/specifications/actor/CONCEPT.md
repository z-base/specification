# z-base/docs/specifications/actor/CONCEPT.md

# Actor — Concept Specification

## Abstract

An **Actor** is a user-controlled, authoritative execution unit responsible for creating, validating, mutating, attributing, synchronizing, revoking, erasing, and cryptographically protecting application state. All semantic meaning, trust decisions, authorization, conflict resolution, and lifecycle enforcement occur exclusively inside the Actor. External systems—storage, transport, peers, and infrastructure—are treated as untrusted, opaque, and non-authoritative.

An Actor is **fully asynchronous by design**. It exposes state immediately using a developer-defined schema with default values and incrementally becomes consistent as operations are discovered, received, validated, and merged. Applications interact with evolving state through an event-driven model; no Actor operation blocks on I/O or remote data.

This specification defines the **normative requirements** for an Actor implementation: its authority boundary, identity and key model, internal components, asynchronous state realization model, event system, merge semantics, synchronization behavior, revocation enforcement, and failure tolerance. The Actor is the sole locus of truth. Infrastructure merely moves bytes.

---

## Status of This Document

This document is a **Draft Specification** for the **Actor Concept**, version **1.0.0**, edited on **16 January 2026**.  
It is provided for review and experimentation and **must not be considered stable or complete**.  
Normative requirements and component boundaries may change without notice.

---

## Conformance

This specification defines requirements for **Actor Implementations**.

An implementation **conforms** if and only if it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

---

## Actor Definition *(Normative)*

An **Actor** is a software agent executing in a user-controlled environment that is **authoritative over state**.

An Actor **MUST**:

- create and mutate state locally,
- expose state immediately using schema-defined defaults,
- validate and authorize all operations it accepts,
- attribute operations to cryptographic identities,
- merge state deterministically and asynchronously,
- emit events as state advances,
- enforce revocation and mandatory erasure,
- encrypt, decrypt, and authenticate all persisted or transmitted state,
- operate correctly without network connectivity.

An Actor **MUST NOT** delegate semantic interpretation, authorization, attribution, merge logic, revocation enforcement, or state realization timing to any external system.

---

## Authority Boundary *(Normative)*

The Actor defines a strict authority boundary.

### Inside the Boundary (Trusted)

- schema interpretation and default state construction  
- state validation and authorization  
- cryptographic verification  
- role and capability enforcement  
- causal ordering and merge semantics  
- asynchronous event emission  
- acknowledgment logic  
- revocation detection and enforcement  
- garbage collection decisions  

### Outside the Boundary (Untrusted)

- storage  
- transport  
- ordering  
- availability  
- durability  
- peers  

An Actor **MUST** treat all externally supplied data as hostile until fully verified.

---

## Actor Identity *(Normative)*

### Actor Identity Key

Each Actor **MUST** possess a persistent **Actor Identity Key** pair.

The identity key:

- **MUST** uniquely identify a single Actor instance,  
- **MUST NOT** be shared across Actors,  
- **MUST** be used for attribution and acknowledgment signing,  
- **MUST** be independent from authorization or role keys.

Loss or compromise of the identity key **MUST** be treated as loss of Actor continuity.

### Identity Persistence

An Actor **MUST** persist its identity key securely across restarts.  
Identity persistence **MUST NOT** depend on network availability.

---

## Asynchronous State Model *(Normative)*

### Schema-Defined Default State

For each Resource, an Actor **MUST** be initialized with a **developer-defined schema** that declares:

- the shape of state,
- default values for all fields,
- invariants enforced by the Actor.

Upon access, the Actor **MUST** expose an **immediate default state view** derived solely from the schema, without waiting for storage, synchronization, or merge completion.

### Incremental State Realization

Authoritative state **MUST** advance only through validated merge of operations.  
State **MUST** become progressively more complete as operations are discovered, received, validated, and merged.

No consumer **MUST** be required to wait for “full state” availability.

---

## Asynchronous Event Engine *(Normative)*

An Actor **MUST** include an **Asynchronous Event Engine** governing how state changes are surfaced.

### Event Semantics

- State changes **MUST** be communicated exclusively via asynchronous events.
- Events **MUST** be emitted only after local validation and merge.
- Events **MUST NOT** be emitted speculatively for unmerged operations.

### Event Categories

An Actor **MUST** be able to emit at least:

- **State Advancement Events** — authoritative state changes after merge  
- **Operation Rejection Events** — invalid or unauthorized operations  
- **Conflict or Correction Events** — causal reconciliation signals  
- **Revocation Events** — role loss and enforced erasure  
- **Error Events** — structural or cryptographic failures  

### Non-Blocking Guarantees

- Event emission **MUST** be non-blocking.
- State access **MUST** never block on I/O, synchronization, or merge.
- UI or application layers **MAY** subscribe to events and update incrementally.

---

## Internal Component Model *(Normative)*

An Actor implementation **MUST** be decomposable into the following **distinguishable components**, each specifiable independently.

### Required Components

- **Identity & Key Manager**
- **Credential & Capability Manager (Discoverable Credentials)**
- **Offline Storage Engine**
- **Resource Manager**
- **Operation Engine**
- **Merge Engine**
- **Asynchronous Event Engine**
- **Revocation Monitor & Enforcer**
- **Acknowledgment Engine**
- **Garbage Collector / Compactor**
- **Base Station Client**
- **Peer & Local Broadcast Channel**
- **Synchronization Controller**
- **Cryptographic Envelope Engine**
- **Policy & Validation Layer**

Additional components **MAY** exist but **MUST NOT** weaken Actor authority, asynchrony, offline-first behavior, revocation guarantees, or deterministic merge semantics.

---

## State Ownership *(Normative)*

An Actor **MUST** maintain its own local replica of all state it participates in.

Local merged state is **authoritative for that Actor**.  
Default schema state is **authoritative until superseded by merged operations**.

Remote data **MAY** influence state only after validation and merge.

---

## Operation Production *(Normative)*

An Actor **MAY** produce an operation only if:

- authorization is valid at the causal point,
- the operation satisfies schema and policy rules,
- causal dependencies are satisfied.

Operation production **MUST NOT** directly mutate authoritative state.

---

## Operation Acceptance *(Normative)*

An Actor **MUST** validate every incoming operation prior to merge.

Invalid operations **MUST** be rejected silently or via explicit events, without side effects.

---

## Deterministic Merge *(Normative)*

Given identical valid operation sets, all conforming Actors **MUST** converge to identical authoritative state.

Merge **MUST** be associative, commutative, idempotent, and independent of arrival order.

---

## Offline-First Operation *(Normative)*

An Actor **MUST** be fully functional offline.

Asynchrony **MUST NOT** be disabled by lack of connectivity.

---

## Synchronization Behavior *(Normative)*

Synchronization is opportunistic, unordered, lossy, and untrusted.

Actors **MUST** recover solely through replay and merge.

---

## Revocation Enforcement *(Normative)*

Upon detecting revocation, an Actor **MUST** immediately and irreversibly erase all access and state for the affected Resource and emit a revocation event.

---

## Acknowledgment Semantics *(Normative)*

Acknowledgments **MUST** be non-mutating, identity-signed, and emitted only after merge.

---

## Garbage Collection *(Normative)*

Garbage collection **MUST** preserve deterministic replay, attribution, and revocation guarantees and **MUST** be local.

---

## Failure Model *(Normative)*

Actors **MUST** tolerate crashes, restarts, partial state, duplication, malicious peers, and hostile infrastructure without authority escalation or silent corruption.

---

## Security Posture *(Non-Normative)*

Security emerges from local authority, explicit verification, deterministic merge, and asynchronous state realization—not from trusted infrastructure.

---

## Closing Principle *(Non-Normative)*

An Actor is not a client.  
An Actor is not a node.  
An Actor is an asynchronous, sovereign executor of truth, bounded only by the keys it holds and the rules it enforces.
