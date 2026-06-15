---
kind: aporia
title: Certificate Rotation Downtime
source: [[okonkwo-2026-mtls]]
rank: 6.5
class: solved
---
*Rotating service certificates historically required restarts that caused connection
downtime.*

**Overview:** Earlier mTLS deployments coupled certificate renewal to process restarts.
Sidecar-managed, hot-reloaded short-lived certificates resolve this, making the problem a
solved one in modern meshes.

## Issues
- Manual rotation windows caused dropped connections.
- Long-lived certificates widened the blast radius of a key compromise.

**Concerns**
- concerns:: [[Mutual TLS]]
- concerns:: [[Certificate Authority]]

**Addressed by**
- addressedBy:: [[Sidecar-Managed Rotation Is Viable]]

**Solved by**
- solvedBy:: [[SPIFFE]]
