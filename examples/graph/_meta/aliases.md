---
kind: meta
title: Alias Table
---
# Alias Table

Canonical node ↔ alternative names encountered across documents. This is the `entity` identity
registry (`CORE.md` §6.3). The
canonical basename is the preferred label (`skos:prefLabel`); each alias is an `skos:altLabel`
also recorded in the node's `aliases` front-matter. Producers consult this table before
creating a new entity so synonyms collapse onto the canonical node.

| Canonical node                | Aliases (`skos:altLabel`)                |
| ----------------------------- | ---------------------------------------- |
| [[Transport Layer Security]]  | TLS, TLS 1.3                             |
| [[Mutual TLS]]                | mTLS, mutual authentication TLS         |
| [[CRYSTALS-Kyber]]            | Kyber, ML-KEM                           |
| [[Certificate Authority]]     | CA, issuing CA                          |
| [[Certificate Transparency]]  | CT, CT logs                             |
| [[Identity Provider]]         | IdP                                     |
| [[Forward Secrecy]]           | Perfect Forward Secrecy, PFS            |
| [[Formal Verification]]       | mechanized verification, formal methods |
