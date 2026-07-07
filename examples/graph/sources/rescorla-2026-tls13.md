---
"@id": rescorla-2026-tls13
"@type": source
title: "TLS 1.3: Design and Rationale"
authors: [Eric Rescorla]
published: 2026-04-12
url: https://example.org/tls13-design
created: 2026-06-13
tags: [tls, protocols, handshake]
---
# TLS 1.3: Design and Rationale

A design retrospective on the TLS 1.3 handshake, explaining how the protocol collapses the
round trips of earlier versions while strengthening its forward-secrecy guarantees, and the
residual risk introduced by zero round-trip resumption.

## Mentions
- mentions:: [[Transport Layer Security]]
- mentions:: [[SSL Protocol]]
- mentions:: [[Handshake Protocol]]
- mentions:: [[Forward Secrecy]]
- mentions:: [[Certificate Authority]]

## Proposes
- proposes:: [[One-RTT Handshake Preserves Security]]

## Raises
- raises:: [[Zero-RTT Replay Exposure]]

## Cites
- cites:: [[RFC 8446]]

## generatedThought
- generatedThought:: [[Latency Cost Justifies 1-RTT Risk]]
