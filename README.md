## Abstract

z-base specifies a zero-knowledge state relay and persistence API for browser-based JavaScript actors. It defines how encrypted, opaque state blobs are stored, relayed, and synchronized by a non-authoritative service, while all state validity, conflict resolution, and cryptographic verification are performed by actors. The specification guarantees offline-first operation, eventual consistency, and unlinkability of stored data, such that the service cannot interpret, correlate, or decrypt application state. This document defines the normative requirements for actors and base stations to interoperate within this model.


## Status of This Document

This document is a **Draft Specification** for **z-base**, version **1.0.0**, edited on **15 January 2026**.
It is provided for review and experimentation and **must not be considered stable or complete**.
The contents may change at any time without notice, including normative requirements and conformance criteria.

This specification is **AI-assisted** and authored with contributions from **Jori Lehtinen** and **GPT-5.2**.
Feedback and discussion are encouraged, but implementations should expect breaking changes until this document reaches a finalized status.


## Conformance

This specification defines conformance requirements for the following roles:

**Actor Implementation**
Software executing in a user-controlled environment that creates, verifies, merges, encrypts, decrypts, and authoritatively interprets application state.

**Base Station Implementation**
A non-authoritative service that stores, relays, and forwards opaque data according to this specification.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

An implementation **conforms** to this specification if and only if it satisfies all normative requirements applicable to its declared role.
An implementation may conform to one or more roles independently.

Sections explicitly marked as *non-normative*, as well as examples and notes, are provided for informational purposes only and **do not affect conformance**.


## Terminology

**Actor**
A user-controlled software agent that is authoritative over state creation, validation, conflict resolution, encryption, and decryption.

**Base Station**
A non-authoritative service that stores, forwards, and relays opaque binary data without the ability to interpret, validate, correlate, or decrypt its contents.

**Resource**
The sole unit of addressable state in z-base, identified by a high-entropy identifier and represented only in encrypted form.

**Resource Snapshot**
An Actor-defined, pre-encryption representation of Resource state consisting of verifiable operations suitable for deterministic merge.

**Envelope**
The encrypted form of a Resource Snapshot, safe for storage, transport, and forwarding by an untrusted Base Station.

**Opaque Blob**
A serialized binary representation of an Envelope that has no semantic meaning outside an Actor.

**Authoritative State**
State whose validity, ordering, and merge semantics are determined exclusively by Actors.

**Offline-First Operation**
A mode of operation in which Actors can read and mutate state without network connectivity, with synchronization occurring opportunistically.

**Eventual Consistency**
A property whereby independent Actor replicas converge to equivalent authoritative state given sufficient time and communication.

---

## Architecture Overview *(Non-Normative)*

z-base follows an **actor-authoritative architecture** in which all meaningful state interpretation, validation, authorization, and conflict resolution occur within Actors. Base Stations function solely as opaque relay and persistence nodes and never participate in state semantics.

Actors maintain local replicas of Resource state and interact with Base Stations using high-entropy identifiers and authenticated sessions to publish, retrieve, and relay encrypted Envelopes. All data handled by Base Stations remains opaque and unlinkable.

Synchronization proceeds incrementally, advancing state through successive encrypted snapshots. Each step reveals only what is necessary to discover subsequent state, preserving unlinkability while enabling offline-first operation and eventual consistency.


## Resource Model *(Normative)*

A **Resource** is the sole unit of addressable state in z-base.

Each Resource **MUST** be identified by a **256-bit high-entropy identifier**, encoded as a **43-character Base64URL string**. The identifier **MUST** be unguessable and **MUST NOT** convey semantic meaning to a Base Station.

A Resource **MUST** be represented exclusively by an **Envelope**.
An Envelope is produced by an Actor by encrypting a **Resource Snapshot**.
Base Stations **MUST** treat Envelopes as uninterpreted binary data and **MUST NOT** derive semantic meaning, linkage, ordering, or validity from Resource identifiers or Envelope contents.

### Resource Snapshot *(Normative)*

A **Resource Snapshot** is the canonical, Actor-defined representation of Resource state prior to encryption.

A Resource Snapshot **MUST** consist of an ordered collection of **verifiable operations**. Each operation **MUST**:

* be cryptographically authenticated by an Actor using verifiable key material,
* represent a CRDT operation suitable for deterministic merge by Actors, and
* be authorized according to a static role model governing permitted operations.

Interpretation of operations, enforcement of roles, and merge semantics **MUST** be performed exclusively by Actors. Base Stations **MUST NOT** interpret operations, roles, or signatures.

A Resource Snapshot **MUST** be encrypted by an Actor to produce an Envelope.

### Authority and Access *(Normative)*

Actors **MUST** be authoritative over Resource state. All validation, merge semantics, conflict resolution, encryption, and decryption **MUST** be performed by Actors. Base Stations **MUST NOT** enforce state rules, resolve conflicts, or reject data based on content.

Resource access **MUST** be capability-based. Possession of a valid Resource identifier and corresponding cryptographic material is sufficient to attempt access. Base Stations **MUST** require protocol-defined authentication steps but **MUST NOT** gain knowledge of Resource semantics as a result.

Resources **MAY** reference other Resources only by identifier. Any such relationships **MUST** be interpreted solely by Actors and remain opaque to Base Stations.

## Actor Roles and Authorization Model (Normative)

z-base defines role-based authorization for Resources. Roles determine the actions a given Actor is permitted to perform on a Resource. Authorization MUST be enforced by Actors at the time operations are applied and merged; Base Stations MUST NOT enforce role permissions.

Defined Roles

Actors MUST be assigned one of the following roles for each Resource:

Owner — Full control over the Resource, including transferring ownership and granting or revoking other roles.

Manager — Permission to grant and revoke Editor and Viewer roles, but MUST NOT change Owner.

Editor — Permission to produce CRDT operations that modify state fields of the Resource.

Viewer — Read-only; MUST NOT produce state-modifying operations.

Revoked — No read or write access; reads are masked to an initial or null state, and write operations are rejected.

These roles and their permissions MUST be enforced by any Actor that merges operations into a Resource Snapshot. Roles MUST be verifiable via cryptographic evidence attached to each operation.

Role Keys and Verification

Each role MUST be associated with verifiable key material. Actors MUST sign operations with the key corresponding to their assigned role for the Resource.
Actors MUST NOT sign operations with key material for roles they do not hold.

Role keys MUST be included in the Resource Snapshot in a way that Actors can verify signatures against the ACL. Actors MUST NOT accept operations whose signatures cannot be verified against the stored role key for that Actor’s identity at the operation’s timestamp.

Authorization Enforcement

When an Actor produces a CRDT operation:

The operation MUST include a detached cryptographic signature using the Actor’s current role key.

The operation MUST include sufficient metadata to determine the targeted Resource and intended CRDT change.

When an Actor merges operations into a local replica:

The Actor MUST verify the signature of each operation against the role key recorded in the Resource Snapshot at the operation’s causal timestamp.

If verification fails or the role does not permit the attempted operation type, the Actor MUST reject that operation from the merge.

Rejection due to unauthorized role MUST NOT prevent merging of other valid operations.

Actors MAY update the ACL by producing CRDT operations that change role assignments, subject to role permissions: Owners MUST be able to assign or revoke any role; Managers MUST be able to assign or revoke Editor and Viewer roles but not Owner.

Role Transition Rules

Role transitions MUST be expressible as signed operations within the Resource Snapshot. Authoritative state after merge MUST reflect the most recent valid role assignment.

An Actor’s effective role for a Resource is defined by the state of the ACL as determined by merged, verifiably signed operations up to the current snapshot.
