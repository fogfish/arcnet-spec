---
"@id": One-RTT Handshake Preserves Security
"@type": hypothesis
class: established
confidence: 0.86
---
*A one round-trip handshake cuts connection-setup latency without weakening the protocol's
security guarantees.*

- derivedFrom:: [[rescorla-2026-tls13]]

**Overview:** By fixing the key-exchange parameters earlier in the negotiation, TLS 1.3
reaches an authenticated, forward-secret state in a single round trip — a property now
widely corroborated by deployment experience and analysis.

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
