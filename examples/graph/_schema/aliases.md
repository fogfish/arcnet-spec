# Alias Table

Canonical node ↔ alternative names encountered across documents. This is the `entity` identity
registry (`ARCNET-CORE.md` §7.4). The canonical basename is the preferred label
(`skos:prefLabel`); each alias is an `skos:altLabel` also recorded in the node's `aliases`
predicate. Producers consult this table before creating a new entity so synonyms collapse onto
the canonical node. A plain aggregate file, not a node.

| Canonical node                | Aliases (`skos:altLabel`)                |
| ------------------------------ | ----------------------------------------- |
| [[Transport Layer Security]]   | TLS, TLS 1.3                              |
| [[Mutual TLS]]                 | mTLS, mutual authentication TLS           |
| [[CRYSTALS-Kyber]]             | Kyber, ML-KEM                             |
| [[Certificate Authority]]      | CA, issuing CA                            |
| [[Certificate Transparency]]   | CT, CT logs                               |
| [[Identity Provider]]          | IdP                                       |
| [[Forward Secrecy]]            | Perfect Forward Secrecy, PFS              |
| [[Formal Verification]]        | mechanized verification, formal methods   |
| [[Audit Log]]                  | append-only log                           |
| [[Lattice Cryptography]]       | lattice-based cryptography                |
| [[Handshake Protocol]]         | handshake                                 |
| [[SSL Protocol]]               | SSL, Secure Sockets Layer                 |
| [[Merkle Tree]]                | hash tree                                 |
| [[Zero Trust Architecture]]    | zero trust, ZTA                           |
| [[Sidecar Proxy]]              | sidecar                                   |
