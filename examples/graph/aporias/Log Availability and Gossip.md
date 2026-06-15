---
kind: aporia
title: Log Availability and Gossip
source: [[laurie-2026-ctlog]]
rank: 6.8
class: unverified
---
*Without effective client gossip, a log can present split views and undermine the
transparency guarantee.*

**Overview:** Certificate Transparency's guarantee assumes clients can detect when a log
shows different histories to different parties. Whether deployed gossip protocols actually
achieve this at scale is unverified.

## Issues
- Clients rarely run gossip, so split-view attacks may go undetected.
- Log availability failures degrade the auditability the system promises.

**Concerns**
- concerns:: [[Certificate Transparency]]

**Addressed by**
- addressedBy:: [[Public Logs Deter Mis-Issuance]]
