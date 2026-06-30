# Example Knowledge Graph

This directory is a **reference example** conforming to [`../CORE.md`](../CORE.md), the
[`../DOMAIN-ARTICLE.md`](../DOMAIN-ARTICLE.md) profile, and the
[`../ARCNET-CORE-THOUGHT.md`](../ARCNET-CORE-THOUGHT.md) extension. The data
is **artificial** — six fictional-but-plausible documents on secure-protocol engineering —
chosen so that subjects recur across documents and the graph is genuinely interconnected.
Use it to validate the format's usability (open it in Obsidian, traverse the links, query
the front-matter).

## The six source documents

| Citekey                  | Title                                       | Published  |
| ------------------------ | ------------------------------------------- | ---------- |
| `rescorla-2026-tls13`    | TLS 1.3: Design and Rationale               | 2026-04-12 |
| `chen-2026-pqkex`        | Post-Quantum Key Exchange in Practice       | 2026-04-28 |
| `laurie-2026-ctlog`      | The Certificate Transparency Ecosystem      | 2026-05-09 |
| `gupta-2026-zerotrust`   | Zero Trust Beyond the Perimeter             | 2026-05-22 |
| `bhargavan-2026-fverify` | Formal Verification of Handshake Protocols  | 2026-06-03 |
| `okonkwo-2026-mtls`      | Deploying mTLS in Service Meshes            | 2026-06-15 |

## Layout

```
graph/
├── sources/       6 source nodes (the provenance origins)
├── entities/      17 Sowa-typed subjects (category as a decoded word-bag)
├── hypothesis/    6 conclusions (flat; class in front-matter, set by validation)
├── aporias/       6 open problems (flat; class in front-matter)
├── thoughts/      2 Core Thoughts (flat; relate only to source/entity/resource)
├── resources/     6 citations / backlog items
├── timeline/      production-date index (yearly + monthly)
└── _meta/         predicate registry + alias table
```

## Things to look for

- **Shared entities knit the graph:** `Transport Layer Security`, `Certificate Authority`,
  `Mutual TLS`, `Identity Provider`, `Handshake Protocol`, `Forward Secrecy` each appear in
  two or more sources (see their `mentionedIn::` lists).
- **Bidirectional provenance:** a source `mentions::`/`proposes::`/`raises::` its outputs;
  every hypothesis/aporia points back via `source:`.
- **Decoded categories:** entities record `category: [independent, abstract, occurrent, script]`
  rather than the `iao:script` code (see [`../CORE.md`](../CORE.md) §9.2.1).
- **Standards-aligned predicates:** edges use camelCase names mapped to schema.org, DCMI
  Terms, CiTO, SKOS, and PROV-O (see [`_meta/predicates.md`](_meta/predicates.md)).
- **Citations are inline and typed:** evidence is `citesAsEvidence::` placed next to the
  claim it supports, not in a global bibliography (see [`../CORE.md`](../CORE.md) §8).
- **Class is optional:** hypothesis/aporia `class` is set only by the validation pass; the
  folder is flat, so an unvalidated node (no class) still has a home.
- **Core Thoughts are domain-agnostic:** `thoughts/` links only to `source`/`entity`/`resource`
  (`wasDerivedFrom`, `concerns`, `citesAsEvidence` — all already-registered predicates) and never
  to a `hypothesis`/`aporia`, so the extension composes with the article profile without
  depending on it (see [`../ARCNET-CORE-THOUGHT.md`](../ARCNET-CORE-THOUGHT.md)).
