# z-base — Concept Specification

## Abstract

z-base specifies a zero-knowledge state relay and persistence system for browser-based JavaScript Actors. It defines how encrypted, opaque state blobs are created by Actors, stored and relayed by non-authoritative infrastructure, and synchronized between peers. All state validity, authorization, attribution, conflict resolution, revocation handling, and cryptographic verification are performed exclusively by Actors.

The model guarantees offline-first operation, eventual consistency, unlinkability of stored data, cryptographic impostor resistance, and mandatory post-revocation data erasure. Infrastructure cannot interpret, correlate, decrypt, retain, or falsely attribute application state.

This document defines the normative requirements for interoperable Actor and Base Station implementations.

---

## Status

This document is a **Draft Specification**, version **1.0.0**, edited **16 January 2026**.  
It is provided for review and experimentation and **MUST NOT** be considered stable or complete.

Normative requirements and conformance criteria may change without notice.

---

## Conformance

This specification defines conformance requirements for two roles:

### Actor Implementation

Software executing in a user-controlled environment that creates, validates, merges, encrypts, decrypts, attributes, revokes, garbage-collects, and authoritatively interprets application state.

### Base Station Implementation

A non-authoritative service that stores, forwards, and relays opaque binary data according to this specification.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

An implementation conforms **iff** it satisfies all normative requirements applicable to its declared role.  
Implementations may conform to one or both roles independently.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

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

z-base follows an **actor-authoritative architecture**.

All meaningful interpretation, validation, authorization, attribution, revocation enforcement, and merge logic occur within Actors. Base Stations act only as opaque persistence and relay infrastructure and never participate in state semantics.

Actors maintain local replicas of Resource state and interact with Base Stations using authenticated sessions to publish, retrieve, and relay encrypted Envelopes. All data handled by Base Stations remains opaque and unlinkable.

Synchronization is incremental, unordered, and tolerant of duplication. Correctness, attribution, impostor resistance, garbage collection, and revocation effects are enforced entirely at the Actor layer.

---

## Normative Subdocuments

This specification is composed of the following normative documents:

- [Actor Concept Specification](actor/CONCEPT.md)
- [Base Station Concept Specification](base_station/CONCEPT.md)
- [Wire Control Protocol](wire_control/PROTOCOL.md)

Component-level behavior is defined in component specifications referenced by the above documents and **MUST** conform to them.

---

## Closing Note *(Non-Normative)*

z-base intentionally minimizes trusted infrastructure.

State authority is local to Actors.  
Infrastructure is limited to byte transport and opaque persistence.
