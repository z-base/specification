# Gateway — Component Specification

## Role

The Gateway is a public ingress component responsible for accepting inbound connection requests and routing them deterministically to Base Station instances.

The Gateway is non-authoritative and does not participate in protocol semantics. Its role is strictly limited to connection handling and deterministic routing based on a Resource Identifier.

## Scope

This specification defines the behavior, constraints, and authority boundaries of the Gateway component.

This document does not define:
– protocol semantics  
– verification rules  
– authorization models  
– state handling  
– synchronization behavior  

Those concerns are defined exclusively by Actor, Base Station, and Wire Control specifications.

## Definition

The Gateway accepts inbound connection requests conforming to a transport-level handshake compatible with the z-base Wire Control Protocol.

Each inbound connection request MUST encode a Resource Identifier in the request path. The Gateway extracts this identifier and routes the connection to the corresponding Base Station instance.

The Gateway has no knowledge of:
– Actors  
– Envelopes  
– Snapshots  
– protocol codes  
– session state  
– verification status  

## Responsibilities

The Gateway MUST:
– accept inbound connection requests  
– extract a Resource Identifier from the initial request path  
– route each connection deterministically to a Base Station instance associated with that Resource Identifier  
– operate without maintaining persistent state  

The Gateway MUST ensure that routing behavior is deterministic: identical Resource Identifiers MUST result in routing to the same Base Station instance, subject only to availability constraints.

## Authority Boundaries

The Gateway has no authority over protocol semantics.

The Gateway MUST NOT:
– interpret, validate, or modify payload contents  
– inspect message frames beyond what is required for connection routing  
– establish or upgrade connections to Verified Sessions  
– attribute identity  
– authorize actions  
– validate state  
– enforce capabilities  
– participate in synchronization or merge decisions  

All semantic interpretation and authority reside exclusively with Base Station and Actor components.

## Routing Semantics

Routing decisions MUST be derived solely from the Resource Identifier encoded in the inbound request path.

The Gateway MUST NOT route based on:
– message contents  
– headers beyond the request path  
– protocol-level frames  
– client identity or metadata  

The mapping between Resource Identifiers and Base Station instances MUST be deterministic and externally consistent.

## Failure Model

Gateway failure MUST NOT compromise protocol correctness or state integrity.

Failure of a Gateway instance MAY result in connection termination or transient unavailability, but MUST NOT result in:
– state loss  
– state corruption  
– semantic divergence  

Gateway instances MUST be replaceable without requiring changes to Actor or Base Station behavior.

## Security Considerations

The Gateway is treated as untrusted.

All data passing through the Gateway MUST be assumed observable and mutable by the Gateway, without affecting protocol safety due to end-to-end encryption and Actor-side validation guarantees defined elsewhere in the specification.

The Gateway MUST NOT be relied upon for confidentiality, integrity, or authenticity guarantees.

## Relationship to Other Components

The Gateway precedes Base Station components in the connection lifecycle.

The Gateway does not form part of the z-base protocol graph and is not a protocol participant. It exists solely as an infrastructural routing layer enabling scalable ingress to Base Station instances.

Removal or replacement of the Gateway MUST NOT alter protocol semantics.
