# z-base — Concept Specification

## Abstract

z-base specifies a zero-knowledge state relay and persistence system for browser-based JavaScript Actors. It defines how encrypted, opaque state blobs are produced by Actors, stored and relayed by non-authoritative infrastructure, and synchronized between peers. All state validity, authorization, attribution, conflict resolution, revocation enforcement, and cryptographic verification occur exclusively within Actors.

The system guarantees offline-first operation, eventual consistency, unlinkability of stored data, resistance to impostor attribution, and mandatory post-revocation erasure. Infrastructure is unable to interpret, correlate, decrypt, retain, or assign semantic meaning to application state.

---

## Status

This document is a **Draft Specification**, version **1.0.0**, edited **16 January 2026**.  
Normative requirements may change without notice.

---

## Conformance

This specification defines requirements for two roles:

- **Actor Implementation**
- **Base Station Implementation**

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are interpreted as described in RFC 2119 and RFC 8174.

An implementation conforms if and only if it satisfies all normative requirements applicable to its declared role.

---

## Terminology

(unchanged; definitions retained verbatim for Pass 1)

---

## Architecture Overview *(Non-Normative)*

z-base follows an actor-authoritative architecture. Actors are the sole locus of semantic interpretation and authority. Base Stations provide only opaque persistence and relay.

Synchronization is incremental and opportunistic. All correctness properties emerge from Actor-side validation and deterministic merge.

---

## Normative Subsystems

This specification is decomposed into the following normative documents:

- Actor model:  
  `actor/CONCEPT.md`

- Base Station model:  
  `base_station/CONCEPT.md`

- Wire-level protocol:  
  `wire_control/PROTOCOL.md`

Component-level behavior is defined only in component specifications and MUST conform to the above documents.

---

## Closing Note *(Non-Normative)*

z-base defines a minimal trusted core.  
Authority is local. Infrastructure is mechanical.
