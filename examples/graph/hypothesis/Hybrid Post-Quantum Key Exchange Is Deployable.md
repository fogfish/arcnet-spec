---
"@id": Hybrid Post-Quantum Key Exchange Is Deployable
"@type": hypothesis
class: novel
confidence: 0.62
---
*Combining a classical and a lattice-based KEM in a single TLS key exchange is deployable on
today's infrastructure.*

- derivedFrom:: [[chen-2026-pqkex]]

**Overview:** A novel claim that hybrid key exchange provides quantum-resistant confidentiality
now, with the classical component preserving security even if the post-quantum component is
later broken — pending broader independent corroboration.

**Assumptions**
- The classical component stays secure if the post-quantum one is broken.
- Current TLS stacks tolerate the larger key-exchange messages.

**Depends on**
- assumes:: [[CRYSTALS-Kyber]]
- assumes:: [[Lattice Cryptography]]
- assumes:: [[Transport Layer Security]]

**Addresses**
- addresses:: [[Post-Quantum Certificate Bloat]]

**Evidence**
- citesAsEvidence:: [[CRYSTALS-Kyber Submission]]
