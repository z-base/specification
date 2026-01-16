# z-base/docs/specifications/CONCEPT.md

# z-base — Concept Specification

## Abstract

z-base specifies a zero-knowledge state relay and persistence API for browser-based JavaScript Actors. It defines how encrypted, opaque state blobs are created, stored, relayed, and synchronized by a non-authoritative service, while all state validity, authorization, conflict resolution, attribution, revocation handling, and cryptographic verification are performed exclusively by Actors. The model guarantees offline-first operation, eventual consistency, unlinkability of stored data, cryptographic impostor resistance, and explicit post-revocation data erasure, such that infrastructure cannot interpret, correlate, decrypt, retain, or falsely attribute application state. This document defines the normative requirements for interoperable Actor and Base Station implementations.

---

## Status of This Document

This document is a **Draft Specification** for **z-base**, version **1.0.0**, edited on **16 January 2026**.  
It is provided for review and experimentation and **must not be considered stable or complete**.  
Normative requirements and conformance criteria may change without notice.

This specification is **AI-assisted** and authored with contributions from **Jori Lehtinen** and **GPT-5.2**.  
Implementations should expect breaking changes until this document reaches a finalized status.

---

## Conformance

This specification defines conformance requirements for two roles:

### Actor Implementation

Software executing in a user-controlled environment that creates, verifies, merges, encrypts, decrypts, attributes, revokes, garbage-collects, and authoritatively interprets application state.

### Base Station Implementation

A non-authoritative service that stores, forwards, and relays opaque binary data according to this specification.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

An implementation **conforms** if and only if it satisfies all normative requirements applicable to its declared role.  
Implementations may conform to one or both roles independently.

Sections marked *Non-Normative*, along with examples and walkthroughs, are informational and **do not affect conformance**.

---

## Terminology

**Actor**  
A user-controlled software agent that is authoritative over state creation, validation, authorization, attribution, conflict resolution, revocation handling, encryption, and decryption.

**Base Station**  
A non-authoritative service that stores, forwards, and relays opaque binary data without the ability to interpret, validate, correlate, decrypt, attribute, or retain semantic meaning.

**Resource**  
The sole unit of addressable state in z-base, identified by a high-entropy identifier and represented only in encrypted form.

**Resource Snapshot**  
An Actor-defined, pre-encryption representation of Resource state consisting of verifiable operations suitable for deterministic merge.

**Envelope**  
The encrypted form of a Resource Snapshot, safe for storage, transport, and forwarding by an untrusted Base Station.

**Opaque Blob**  
A serialized binary representation of an Envelope with no semantic meaning outside an Actor.

**Authoritative State**  
State whose validity, ordering, authorization, attribution, revocation effects, and merge semantics are determined exclusively by Actors.

**Offline-First Operation**  
A mode in which Actors can read and mutate state without network connectivity, synchronizing opportunistically.

**Eventual Consistency**  
A property whereby independent Actor replicas converge to equivalent authoritative state given sufficient time and communication.

**Actor Identity Key**  
A persistent cryptographic signing key pair uniquely identifying an Actor instance for attribution and impostor resistance, distinct from role keys.

**Acknowledgment (Ack)**  
A signed, non-mutating Actor statement attesting that a specific causal state has been locally merged and verified.

---

## Architecture Overview *(Non-Normative)*

z-base follows an **actor-authoritative architecture**. All meaningful interpretation, validation, authorization, attribution, revocation enforcement, and merge logic occurs within Actors. Base Stations act only as opaque persistence and relay nodes and never participate in state semantics.

Actors maintain local replicas of Resource state and interact with Base Stations using high-entropy identifiers and authenticated sessions to publish, retrieve, and relay encrypted Envelopes. All Base Station–handled data remains opaque and unlinkable.

Synchronization proceeds incrementally via encrypted snapshots and deltas. State progresses locally first; synchronization is opportunistic, unordered, and tolerant of duplication. Attribution, impostor resistance, garbage collection, and revocation effects are enforced entirely at the Actor layer.

---

## Resource Model *(Normative)*

A **Resource** is the sole unit of addressable state in z-base.

Each Resource **MUST** be identified by a **256-bit high-entropy identifier**, encoded as a **43-character Base64URL string**. The identifier **MUST** be unguessable and **MUST NOT** convey semantic meaning to a Base Station.

A Resource **MUST** be represented exclusively by an **Envelope**, produced by encrypting a Resource Snapshot.  
Base Stations **MUST** treat Envelopes as uninterpreted binary data and **MUST NOT** derive semantics, linkage, ordering, validity, authorship, or lifecycle state from identifiers or payloads.

---

## Resource Snapshot *(Normative)*

A **Resource Snapshot** is the canonical Actor-defined representation of Resource state prior to encryption.

A Snapshot **MUST** consist of an ordered collection of **verifiable operations**. Each operation **MUST**:

- be cryptographically authenticated for authorization by a role key,  
- be cryptographically attributable to an Actor identity key,  
- represent a CRDT operation suitable for deterministic merge, and  
- be authorized according to the Resource’s role model.

Interpretation, authorization, attribution, revocation detection, acknowledgment tracking, and merge semantics **MUST** be performed exclusively by Actors.  
Base Stations **MUST NOT** interpret operations, roles, signatures, identity bindings, acknowledgments, or revocation state.

A Snapshot **MUST** be encrypted by an Actor to produce an Envelope.

---

## Authority, Attribution, and Access *(Normative)*

Actors **MUST** be authoritative over Resource state. All validation, merge semantics, conflict resolution, attribution, acknowledgment evaluation, encryption, decryption, and revocation handling **MUST** occur at the Actor layer.

Base Stations **MUST NOT** enforce state rules, resolve conflicts, attribute authorship, track acknowledgments, or reject data based on content.

Access **MUST** be capability-based. Possession of a valid Resource identifier and corresponding cryptographic material is sufficient to attempt access.  
Base Stations **MUST** require protocol-defined authentication but **MUST NOT** gain semantic, attributional, or lifecycle insight as a result.

Resource Snapshots **MAY** contain application-defined references to other Resources. Such semantics **MUST** remain invisible to Base Stations.

---

## Actor Roles and Authorization Model *(Normative)*

z-base defines **role-based authorization** enforced exclusively by Actors.

### Roles

For each Resource, Actors **MUST** hold exactly one role:

- **Owner** — Full control, including assigning and revoking all roles and identity bindings.  
- **Manager** — May assign and revoke *Editor* and *Viewer* roles; **MUST NOT** modify Owner.  
- **Editor** — May produce state-modifying CRDT operations.  
- **Viewer** — Read-only access.  
- **Revoked** — No access.

### Role Keys and Verification

Each role **MUST** be associated with verifiable cryptographic key material.  
Operations **MUST** be signed using the key for the Actor’s current role.  
Actors **MUST NOT** accept operations whose role signatures do not verify against the role valid at the operation’s causal point.

### Role Transitions

Role assignments and revocations **MUST** be expressed as signed operations within the Snapshot.  
After merge, authoritative state reflects the most recent valid assignment.

---

## Actor Identity and Impostor Resistance *(Normative)*

### Actor Identity Keys

Each Actor **MUST** maintain a persistent **Actor Identity Key** pair distinct from all role keys.

The identity key:

- **MUST** uniquely identify a single Actor instance,  
- **MUST NOT** be shared between Actors,  
- **MAY** be rotated only via explicit signed operations recorded in the Resource Snapshot.

### Identity Binding

An Actor’s identity public key **MUST** be introduced and pinned into Resource state on first valid operation attributed to that Actor.

Once pinned, identity bindings **MUST** be used to verify future operations for attribution and impostor detection.

### Impostor Resistance Guarantees

An Actor **MUST** verify Actor identity signatures against pinned identity keys.  
Failure of identity verification **MUST** be surfaced as an impostor signal.

Identity verification failure **MUST NOT** alter CRDT merge correctness but **MUST** affect attribution, auditing, and acknowledgment validity.

---

## Operation Format and Signing Semantics *(Normative)*

An **operation** is a self-contained, verifiable CRDT mutation and **MUST** include:

- Resource Identifier  
- Actor Identifier  
- Role Signature  
- Actor Identity Signature  
- Operation Payload  
- Causal Metadata

### Dual-Signature Requirement

- The **Role Signature** authorizes *what* the operation is allowed to do.  
- The **Actor Identity Signature** proves *who* produced the operation.

Signatures **MUST** be detached and cover all fields except other signatures. Any modification **MUST** invalidate the affected signature.

Actors **MUST** verify role authorization, identity attribution, and causal consistency before merge.  
Invalid operations **MUST** be excluded without affecting valid ones.

---

## Acknowledgments and Verifiable Garbage Collection *(Normative)*

### Acknowledgment Emission

Actors **MAY** emit **acknowledgment operations** (“acks”) after successfully merging operations into local authoritative state.

An acknowledgment:

- **MUST** be signed using the Actor Identity Key,  
- **MUST** reference specific causal metadata being acknowledged,  
- **MUST NOT** mutate Resource state.

Actors **MUST NOT** emit acknowledgments for operations not yet merged locally.

### Ack Verification

Actors **MUST** verify acknowledgments against the pinned Actor Identity Public Key valid at the acknowledged causal point.  
Invalid acknowledgments **MUST** be ignored.

### Garbage Collection Constraint

Local garbage collection or compaction **MAY** be performed **only** when:

- all non-revoked Actors known in the authoritative ACL at the relevant causal point have emitted verifiable acknowledgments, and  
- compaction preserves deterministic replay, attribution, and future merge correctness.

If any required acknowledgment is missing, tombstones and historical operations **MUST** be retained.

Garbage collection **MUST** be local, unilateral, and never coordinated by Base Stations.

---

## Revocation Enforcement and Mandatory Erasure *(Normative)*

### Revocation Detection

Actors **MUST** monitor authoritative merged state for changes to their own role assignments.

### Mandatory Erasure

Upon detecting that its own role for a Resource has transitioned to **Revoked**, a compliant Actor **MUST**, without user intervention:

- irreversibly erase all decrypted state related to the Resource,  
- erase all cryptographic keys, derived secrets, and identity bindings for that Resource,  
- erase all cached Envelopes, operations, acknowledgments, and metadata enabling access,  
- remove all references to the Resource identifier from local APIs.

### Post-Revocation Behavior

After erasure, the Actor **MUST**:

- refuse all further interaction with the Resource,  
- ignore incoming frames referencing the Resource, and  
- permanently disable operation or acknowledgment emission for that Resource.

Failure to enforce mandatory erasure **MUST** be considered non-conformant.

---

## CRDT Merge Semantics *(Normative)*

Resource state **MUST** be derived from the validated operation set.

Given identical valid operation sets, all Actors **MUST** converge to identical authoritative state, independent of arrival order, duplication, or transport behavior.

Operations **MUST** be idempotent and replay-safe.  
Base Stations **MUST NOT** participate in merge, validation, acknowledgment handling, or revocation enforcement.

---

## Envelope Cryptographic Properties *(Normative)*

- **Confidentiality:** Only authorized Actors can decrypt.  
- **Integrity:** Modification or replay **MUST** be detectable.  
- **Determinism:** Decryption yields exactly the encrypted Snapshot.  
- **Non-Observability:** No semantic, attributional, or lifecycle leakage beyond routing necessity.  
- **Algorithm Agnosticism:** No specific algorithms are mandated.

---

## Synchronization and Wire Semantics *(Normative)*

This section defines how bytes move, not what they mean.

### Invariants

- Payloads are opaque to Base Stations.  
- Delivery may be lossy, duplicated, or reordered.  
- Authority, attribution, acknowledgment logic, and revocation remain with Actors.

### Framing

```

[ Frame Type ][ Payload Bytes ]

```

Frame Type indicates intent; Payload Bytes are forwarded losslessly.

### Access Gating

Before forwarding stateful frames, Base Stations **MUST** require proof of possession of cryptographic material associated with the Resource. The mechanism is implementation-defined and **MUST NOT** imply authority or identity.

### Snapshot and Delta Exchange

Actors **MAY** exchange full Snapshots or delta Envelopes.  
Ordering **MUST NOT** be assumed. Merge **MUST** be idempotent.

### Peer Relay and Offline Sync

Base Stations **MUST** act purely as relays.  
Identical semantics **MAY** be used offline (e.g. BroadcastChannel).

### Failure Handling

All failures **MUST** be recoverable via re-synchronization.  
Base Stations **MUST NOT** attempt repair.

---

## Security Considerations *(Non-Normative)*

z-base assumes untrusted infrastructure, hostile networks, and potentially malicious peers. Confidentiality, integrity, authorization, attribution, acknowledgment safety, and revocation enforcement are Actor-side responsibilities. Base Stations cannot forge, interpret, retain, or falsely attribute state.

Metadata leakage is minimized but not eliminated. Compromised user agents are out of scope; recovery is delegated to higher layers.

---

## Future Extensions *(Non-Normative)*

Potential extensions include standardized sharing and delegation, automation Actors, public Resources, interoperable schemas, identity interop, advanced key lifecycle management, transport profiles, and formal verification.

All extensions **MUST NOT** weaken actor authority, impostor resistance, acknowledgment safety, revocation guarantees, zero-knowledge properties, or offline-first operation.

---

## Closing Note

z-base is intentionally small, strict, and composable at its core.  
State evolves locally; trust does not.  
Innovation is expected to occur at the edges, not by expanding the trusted center.
