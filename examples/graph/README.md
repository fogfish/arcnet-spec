# Example Knowledge Graph

This directory is a **reference example** conforming to [`../ARCNET-CORE.md`](../ARCNET-CORE.md),
the [`../ARCNET-DOMAIN-ARTICLE.md`](../ARCNET-DOMAIN-ARTICLE.md) profile, and the
[`../ARCNET-DOMAIN-CORE-THOUGHT.md`](../ARCNET-DOMAIN-CORE-THOUGHT.md) extension. The data
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
├── hypothesis/    6 conclusions (flat; class in a predicate, set by validation)
├── aporias/       6 open problems (flat; class in a predicate)
├── thoughts/      2 Core Thoughts (flat; relate only to source/entity/resource)
├── resources/     6 citations / backlog items
├── timeline/      production-date index (yearly + monthly)
└── _schema/
    ├── predicates/  one node per predicate in use (61 files)
    ├── types/       one node per @type in use: source, entity, resource, timeline,
    │                hypothesis, aporia, thought (7 files)
    └── aliases.md   entity alias table — a plain aggregate file, not a node
```

## Things to look for

- **Shared entities knit the graph:** `Transport Layer Security`, `Certificate Authority`,
  `Mutual TLS`, `Identity Provider`, `Handshake Protocol`, `Forward Secrecy` each appear in
  two or more sources (see their `mentionedIn::` lists).
- **Bidirectional provenance:** a source `mentions::`/`proposes::`/`raises::` its outputs;
  every hypothesis/aporia/thought points back via `derivedFrom::` (CORE's scalar-only `meta`
  role can't hold a link, so this is a body edge, not a front-matter field).
- **Decoded categories:** entities record `category: [independent, abstract, occurrent, script]`
  rather than the `iao:script` code (see [`../ARCNET-CORE.md`](../ARCNET-CORE.md) §10.7's
  `category` predicate).
- **Predicates and types are graph nodes, not documentation:** every predicate and every
  `@type` in use here is a real node under `_schema/predicates/`/`_schema/types/`
  (see [`../ARCNET-CORE.md`](../ARCNET-CORE.md) §9) — there is no separate registry file to
  keep in sync by hand.
- **Citations are inline and typed:** evidence is `citesAsEvidence::` placed next to the
  claim it supports, not in a global bibliography (see [`../ARCNET-CORE.md`](../ARCNET-CORE.md)
  §12).
- **Class is optional:** `hypothesis`/`aporia`'s `class` is set only by the validation pass;
  the folder is flat, so an unvalidated node (no class) still has a home.
- **`thought`'s cognitive role isn't called `class`:** it's `cognition`, a deliberately
  different predicate from `hypothesis`/`aporia`'s `class` — same name, different meaning and
  merge behavior, would collide under CORE's one-node-per-predicate-name model otherwise (see
  [`../ARCNET-DOMAIN-CORE-THOUGHT.md`](../ARCNET-DOMAIN-CORE-THOUGHT.md)'s revision note).
- **Core Thoughts are domain-agnostic:** `thoughts/` links only to `source`/`entity`/`resource`
  (`derivedFrom`, `concerns`, `citesAsEvidence` — all reused predicates) and never
  to a `hypothesis`/`aporia`, so the extension composes with the article profile without
  depending on it (see [`../ARCNET-DOMAIN-CORE-THOUGHT.md`](../ARCNET-DOMAIN-CORE-THOUGHT.md)).
  Each such source also carries the inverse `## generatedThought` backlink.
- **Aliases are recorded twice, deliberately.** An entity's own `aliases:` field (e.g.
  `Transport Layer Security`'s `[TLS, TLS 1.3]`) is what a consumer reads; `_schema/aliases.md`
  is the producer-facing lookup table consulted *before* creating a new entity, so a synonym
  collapses onto the canonical node instead of forking a duplicate (CORE §7.4).
