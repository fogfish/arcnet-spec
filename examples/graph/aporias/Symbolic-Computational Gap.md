---
kind: aporia
title: Symbolic-Computational Gap
source: [[bhargavan-2026-fverify]]
rank: 8.3
class: critical
---
*A proof in the symbolic model does not, by itself, guarantee security in the computational
model.*

**Overview:** Symbolic verifiers treat cryptographic primitives as perfect black boxes;
real adversaries exploit probabilistic and algebraic structure the symbolic abstraction
hides, leaving a gap between "verified" and "secure".

## Issues
- Symbolic soundness theorems hold only under strong, often unmet, assumptions.
- Computational proofs remain largely manual and do not scale to full protocols.

**Concerns**
- concerns:: [[Formal Verification]]

**Addressed by**
- addressedBy:: [[Mechanized Proofs Catch Flaws Early]]
