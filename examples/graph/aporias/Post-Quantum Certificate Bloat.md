---
kind: aporia
title: Post-Quantum Certificate Bloat
source: [[chen-2026-pqkex]]
rank: 8.7
class: critical
---
*Post-quantum public keys and signatures inflate certificates beyond what current handshakes
comfortably carry.*

**Overview:** Lattice-based keys are an order of magnitude larger than classical ones,
pushing certificate chains past packet and buffer limits and degrading handshake performance
— a critical blocker for naive deployment.

## Issues
- Larger keys fragment the handshake across more packets.
- Middleboxes and embedded clients with fixed buffers reject oversized chains.

**Concerns**
- concerns:: [[CRYSTALS-Kyber]]
- concerns:: [[Certificate Authority]]

**Addressed by**
- addressedBy:: [[Hybrid Post-Quantum Key Exchange Is Deployable]]
