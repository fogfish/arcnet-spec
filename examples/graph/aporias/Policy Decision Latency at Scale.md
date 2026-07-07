---
"@id": Policy Decision Latency at Scale
"@type": aporia
class: unverified
---
*Evaluating access policy on every request may impose latency that undermines zero-trust at
scale.*

- derivedFrom:: [[gupta-2026-zerotrust]]

**Overview:** Per-request identity and policy checks add a round trip to a decision point.
Whether caching and local evaluation keep this within acceptable budgets under real load is
not yet established.

**Issues**
- Centralized policy engines become a latency and availability bottleneck.
- Aggressive caching weakens the freshness that zero trust depends on.

**Concerns**
- concerns:: [[Zero Trust Architecture]]
- concerns:: [[Identity Provider]]

**Addressed by**
- addressedBy:: [[Identity Supersedes Network Perimeter]]
