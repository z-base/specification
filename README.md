# Abstract
z-base specifies a zero-knowledge state relay and persistence API for browser-based JavaScript clients. It defines how encrypted, opaque state blobs are stored, relayed, and synchronized by a non-authoritative service, while all state validity, conflict resolution, and cryptographic verification are performed client-side. The specification guarantees offline-first operation, eventual consistency, and unlinkability of stored data, such that the service cannot interpret, correlate, or decrypt application state. This document defines the normative requirements for clients and base stations to interoperate within this model.

## Status of This Document

This document is a Draft Specification for z-base, version 1.0.0, edited on 15 January 2026.
It is provided for review and experimentation and must not be considered stable or complete.
The contents may change at any time without notice, including normative requirements and conformance criteria.

This specification is AI-assisted and authored with contributions from Jori Lehtinen and GPT-5.2.
Feedback and discussion are encouraged, but implementations should expect breaking changes until this document reaches a finalized status.
