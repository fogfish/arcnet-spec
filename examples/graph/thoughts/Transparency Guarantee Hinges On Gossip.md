---
"@id": Transparency Guarantee Hinges On Gossip
"@type": thought
cognition: question
maturity: emerging
---
*Certificate Transparency's split-view detection guarantee only holds if clients actually run
an effective gossip protocol — an assumption the ecosystem has not yet validated at scale.*

- derivedFrom:: [[laurie-2026-ctlog]]

**About**
The Merkle-tree audit structure proves a log is internally consistent, but says nothing about
whether two clients were shown the *same* log. That second guarantee depends entirely on
gossip, which is rarely deployed in practice. This thought reframes Certificate Transparency's
headline guarantee as conditional rather than absolute.

**Motivation**
Whether the transparency property the ecosystem advertises actually holds in deployments where
gossip is absent or weak.

**Concerns**
- concerns:: [[Certificate Transparency]]
- concerns:: [[Audit Log]]

**Evidence**
- citesAsEvidence:: [[RFC 6962]]

**Next**
Survey deployed clients for gossip support and measure how often split-view conditions would
go undetected in practice.
