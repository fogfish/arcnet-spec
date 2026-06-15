---
kind: aporia
title: Zero-RTT Replay Exposure
source: [[rescorla-2026-tls13]]
rank: 9.0
class: critical
---
*Zero round-trip resumption lets early application data be replayed by an attacker.*

**Overview:** The latency win of sending data in the first flight comes at the cost of
replayability: there is no server-side round trip to guarantee freshness of 0-RTT data, an
open and material risk for non-idempotent requests.

## Issues
- Early data can be captured and re-sent to trigger duplicate side effects.
- Application-layer idempotency is required but not enforceable by the protocol.

**Concerns**
- concerns:: [[Transport Layer Security]]
- concerns:: [[Handshake Protocol]]

**Addressed by**
- addressedBy:: [[One-RTT Handshake Preserves Security]]
