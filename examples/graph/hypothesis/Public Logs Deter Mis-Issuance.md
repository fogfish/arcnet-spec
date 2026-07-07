---
"@id": Public Logs Deter Mis-Issuance
"@type": hypothesis
class: extended
confidence: 0.74
---
*Publishing every issued certificate to public, append-only logs deters and exposes
certificate mis-issuance.*

- derivedFrom:: [[laurie-2026-ctlog]]

**Overview:** Extends the established PKI trust model with an auditability layer: because
mis-issued certificates become publicly visible, authorities are held accountable after the
fact.

**Assumptions**
- Monitors actively inspect the logs for mis-issued certificates.
- Authorities face consequences once mis-issuance becomes public.

**Depends on**
- assumes:: [[Certificate Transparency]]
- assumes:: [[Audit Log]]
- assumes:: [[Certificate Authority]]

**Addresses**
- addresses:: [[Log Availability and Gossip]]

**Evidence**
- citesAsEvidence:: [[RFC 6962]]
