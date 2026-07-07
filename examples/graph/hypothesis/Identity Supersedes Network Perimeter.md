---
"@id": Identity Supersedes Network Perimeter
"@type": hypothesis
class: novel
confidence: 0.58
---
*Per-request identity verification should replace network location as the primary unit of
access control.*

- derivedFrom:: [[gupta-2026-zerotrust]]

**Overview:** A novel reframing of access control in which trust derives from a verified
identity assertion on every request rather than from residence inside a network perimeter.

**Assumptions**
- Every request can be bound to a verifiable identity assertion.
- Policy is evaluable per request without prohibitive latency.

**Depends on**
- assumes:: [[Zero Trust Architecture]]
- assumes:: [[Identity Provider]]
- assumes:: [[Mutual TLS]]

**Addresses**
- addresses:: [[Policy Decision Latency at Scale]]

**Evidence**
- citesAsEvidence:: [[BeyondCorp]]
