---
kind: thought
title: Latency Cost Justifies 1-RTT Risk
source: [[rescorla-2026-tls13]]
class: decision
maturity: developing
---
*Accepting the residual replay risk of zero-round-trip resumption is worth the latency win,
provided application-layer idempotency is enforced upstream.*

**About**
The 1-RTT handshake's latency savings are why TLS 1.3 standardized it despite a known replay
exposure; the design bet is that idempotency enforcement is cheaper than losing the round trip.

**Motivation**
Whether the performance gain of 0/1-RTT resumption is defensible once its replay weakness is
known.

**Concerns**
- concerns:: [[Forward Secrecy]]
- concerns:: [[Handshake Protocol]]

**Evidence**
- citesAsEvidence:: [[RFC 8446]]

**Next**
Quantify how much idempotency enforcement actually costs at the application layer before
treating this trade-off as settled.
