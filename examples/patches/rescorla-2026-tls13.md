---
"@type": patch
document: rescorla-2026-tls13
title: "TLS 1.3: Design and Rationale"
published: 2026-04-12
stats: { nodes: 9, edges: 30 }
---

# Source

## rescorla-2026-tls13

```yaml
published: 2026-04-12
authors: [Eric Rescorla]
url: https://example.org/tls13-design
tags: [tls, protocols, handshake]
```

A design retrospective on the TLS 1.3 handshake, explaining how the protocol collapses the
round trips of earlier versions while strengthening its forward-secrecy guarantees, and the
residual risk introduced by zero round-trip resumption.

**Mentions**
- mentions:: [[Transport Layer Security]]
- mentions:: [[SSL Protocol]]
- mentions:: [[Handshake Protocol]]
- mentions:: [[Forward Secrecy]]
- mentions:: [[Certificate Authority]]

**Proposes**
- proposes:: [[One-RTT Handshake Preserves Security]]

**Raises**
- raises:: [[Zero-RTT Replay Exposure]]

**Cites**
- cites:: [[RFC 8446]]

# Entity

## Transport Layer Security

```yaml
category: [independent, abstract, occurrent, script]
aliases: [TLS, TLS 1.3]
tags: [cryptography, protocols]
```

A cryptographic protocol that establishes an authenticated, confidential channel between two
parties over an untrusted network.

- replaces:: [[SSL Protocol]]
- isPartOf:: [[Handshake Protocol]]
- conformsTo:: [[RFC 8446]]

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]

## SSL Protocol

```yaml
category: [independent, abstract, occurrent, script]
aliases: [SSL, Secure Sockets Layer]
tags: [cryptography, protocols, legacy]
```

The predecessor secure-channel protocol that TLS replaced.

- broader:: [[Handshake Protocol]]

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]

## Handshake Protocol

```yaml
category: [independent, abstract, occurrent, script]
aliases: [handshake]
tags: [cryptography, protocols]
```

The negotiation phase in which two parties authenticate and agree on keys before exchanging
application data.

- secures:: [[Forward Secrecy]]

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]

## Forward Secrecy

```yaml
category: [independent, abstract, continuant, schema]
aliases: [Perfect Forward Secrecy, PFS]
tags: [cryptography, property]
```

The security property that compromise of a long-term key does not reveal past session keys.

- requires:: [[Handshake Protocol]]

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]

## Certificate Authority

```yaml
category: [independent, physical, continuant, object]
aliases: [CA, issuing CA]
tags: [pki, identity]
```

An organization that issues and vouches for digital certificates binding public keys to
identities.

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]

# Hypothesis

## One-RTT Handshake Preserves Security

```yaml
class: established
confidence: 0.86
```

*A one round-trip handshake cuts connection-setup latency without weakening the protocol's
security guarantees.*

- derivedFrom:: [[rescorla-2026-tls13]]

**Assumptions**
- Forward secrecy is preserved across the 1-RTT key schedule.
- Both peers authenticate during the handshake before application data flows.

**Depends on**
- assumes:: [[Forward Secrecy]]
- assumes:: [[Handshake Protocol]]

**Addresses**
- addresses:: [[Zero-RTT Replay Exposure]]

**Evidence**
- citesAsEvidence:: [[RFC 8446]]

# Aporia

## Zero-RTT Replay Exposure

```yaml
class: critical
```

*Zero round-trip resumption lets early application data be replayed by an attacker.*

- derivedFrom:: [[rescorla-2026-tls13]]

**Issues**
- Early data can be captured and re-sent to trigger duplicate side effects.
- Application-layer idempotency is required but not enforceable by the protocol.

**Concerns**
- concerns:: [[Transport Layer Security]]
- concerns:: [[Handshake Protocol]]

**Addressed by**
- addressedBy:: [[One-RTT Handshake Preserves Security]]

# Resource

## RFC 8446

```yaml
ref: standard
authors: [Eric Rescorla]
year: 2018
url: https://www.rfc-editor.org/rfc/rfc8446
status: read
```

The normative specification of TLS 1.3, including the one round-trip and zero round-trip
handshake modes.

**Cited by**
- isCitedBy:: [[rescorla-2026-tls13]]
- isCitedBy:: [[One-RTT Handshake Preserves Security]]
