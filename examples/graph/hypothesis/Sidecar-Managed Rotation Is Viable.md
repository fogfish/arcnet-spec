---
kind: hypothesis
title: Sidecar-Managed Rotation Is Viable
source: [[okonkwo-2026-mtls]]
rank: 7.8
class: extended
confidence: 0.78
---
*Delegating short-lived certificate issuance and rotation to sidecar proxies makes mTLS
operationally viable at scale.*

**Overview:** Extends mutual TLS from a manual, brittle configuration into an automated mesh
capability by moving certificate lifecycle management into the data-plane sidecars.

**Assumptions**
- Sidecars hot-reload certificates without dropping live connections.
- A mesh-internal authority issues short-lived certificates on demand.

**Depends on**
- assumes:: [[Mutual TLS]]
- assumes:: [[Sidecar Proxy]]
- assumes:: [[Certificate Authority]]

**Addresses**
- addresses:: [[Certificate Rotation Downtime]]

**Evidence**
- citesAsEvidence:: [[SPIFFE]]
