# DOMAIN-DOC — Article Extension

**Status:** Accepted · **Version:** 0.4 · **Date:** 2026-06-15
**Extends:** [`CORE.md`](CORE.md)

This profile ingests research articles and similar prose documents into a knowledge graph. It
**adopts the four CORE kinds** (`source`, `entity`, `resource`, `timeline`, [CORE §9](CORE.md))
and adds two domain kinds — `hypothesis` and `aporia`. All CORE mechanism (identity §6, edges
§7, citations §8, merge §10, version control §11, patch §12) applies unchanged.

A complete worked example is in [`graph/`](graph/); its single-file patch serialization is in
[`patches/`](patches/).

## 1. Folder Layout

```
graph/
├── sources/             # source nodes        (CORE §9.1)
├── entities/            # entity nodes, Sowa   (CORE §9.2)
├── hypothesis/          # hypothesis nodes     (§2.1)
├── aporias/             # aporia nodes         (§2.2)
├── resources/           # resource nodes       (CORE §9.3)
├── timeline/            # production index     (CORE §9.4)
│   ├── yearly/
│   └── monthly/
└── _meta/
    ├── predicates.md    # predicate registry
    └── aliases.md       # entity alias table
```

`hypothesis/` and `aporias/` are flat: a node's `class` lives in front-matter (§5), so it never
determines file location. The profile also extends the core `source` kind with domain navigation
blocks (§3).

## 2. Domain Kinds

### 2.1 `hypothesis`

A conclusion distilled from sources. **Identity:** title (CORE §6.3). **Merge:**
validated-overwrite (CORE §10).

**Front-matter**
- `kind` (mandatory) — the literal `hypothesis`
- `title` (mandatory) — the claim in short form, equal to the basename
- `source` (mandatory) — `wasDerivedFrom` link(s) to the originating source node(s)
- `rank` (recommended) — salience, 0–10
- `class` (optional) — `established` | `extended` | `novel`; only after validation (§5)
- `confidence` (optional) — 0–1 strength of the classification; only after validation

**Body**
- `claim` (mandatory) — a one-sentence statement of the conclusion (rendered emphasized)
- `overview` (recommended) — a short paragraph of context
- `**Assumptions**` (recommended) — literal premise statement (one per bullet)
- `**Depends on**` (recommended) — `assumes::` edges to the entities those premises depend on
- `**Addresses**` (recommended) — `addresses::` edges to the aporias the claim tackles
- `**Evidence**` (recommended) — citation edges (`citesAsEvidence::`, …; CORE §8)
- `notes` (optional) — additional prose

```markdown
---
kind: hypothesis
title: One-RTT Handshake Preserves Security
source: [[rescorla-2026-tls13]]
rank: 8.5
class: established
confidence: 0.86
---
*A one round-trip handshake cuts connection-setup latency without weakening the protocol's
security guarantees.*

**Assumptions**
- Forward secrecy is preserved across the 1-RTT key schedule.
- Both peers authenticate during the handshake before application data flows.

**Depends on**
- assumes:: [[Forward Secrecy]]
- assumes:: [[Handshake Protocol]]

**Addresses**
- addresses:: [[Zero-RTT Replay Exposure]]

**Evidence**
- citesAsEvidence:: [[RFC 8446]]
```

### 2.2 `aporia`

An open problem or unresolved tension. **Identity:** title (CORE §6.3). **Merge:**
validated-overwrite (CORE §10).

**Front-matter**
- `kind` (mandatory) — the literal `aporia`
- `title` (mandatory) — the tension in short form, equal to the basename
- `source` (mandatory) — `wasDerivedFrom` link(s) to the originating source node(s)
- `rank` (recommended) — severity, 0–10
- `class` (optional) — `critical` | `solved` | `unverified`; only after validation (§5)

**Body**
- `tension` (mandatory) — a one-sentence statement of the open problem (emphasized)
- `overview` (recommended) — a short paragraph of context
- `## Issues` (recommended) — literal statement decomposing the problem
- `**Concerns**` (recommended) — `concerns::` edges to the entities the problem involves
- `**Addressed by**` (recommended) — `addressedBy::` edges to the hypotheses that tackle it
- `**Solved by**` (optional) — `solvedBy::` edges to the resolving resources/hypotheses
- `notes` (optional) — additional prose

```markdown
---
kind: aporia
title: Zero-RTT Replay Exposure
source: [[rescorla-2026-tls13]]
rank: 9.0
class: critical
---
*Zero round-trip resumption lets early application data be replayed by an attacker.*

## Issues
- Early data can be captured and re-sent to trigger duplicate side effects.
- Application-layer idempotency is required but not enforceable by the protocol.

**Concerns**
- concerns:: [[Transport Layer Security]]
- concerns:: [[Handshake Protocol]]

**Addressed by**
- addressedBy:: [[One-RTT Handshake Preserves Security]]
```

## 3. Extended Kinds

A profile MAY extend a CORE kind with additional Recommended/Optional elements, without changing
the kind's identity or merge operation. This profile extends `source`; `entity`, `resource`, and
`timeline` are used as CORE defines them.

### 3.1 `source` (extends CORE §9.1)

Beyond the core `## Mentions` (entities) and `## Cites` (resources) body blocks, the article
`source` adds two navigation blocks linking the document to the domain kinds it produces.
Identity (citekey), front-matter, and merge (none) are inherited from CORE §9.1 unchanged.

**Body (added)**
- `## Proposes` (recommended) — `proposes::` edges to the hypotheses derived from the document
- `## Raises` (recommended) — `raises::` edges to the aporias derived from the document

```markdown
---
kind: source
id: rescorla-2026-tls13
title: "TLS 1.3: Design and Rationale"
authors: [Eric Rescorla]
published: 2026-04-12
url: https://example.org/tls13-design
tags: [tls, protocols, handshake]
---
# TLS 1.3: Design and Rationale

A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip
resumption.

## Mentions
- mentions:: [[Transport Layer Security]]
- mentions:: [[Forward Secrecy]]

## Proposes
- proposes:: [[One-RTT Handshake Preserves Security]]

## Raises
- raises:: [[Zero-RTT Replay Exposure]]

## Cites
- cites:: [[RFC 8446]]
```

## 4. Predicate Vocabulary

In addition to the core vocabulary ([CORE §7.4](CORE.md)), this profile registers the following
in [`graph/_meta/predicates.md`](graph/_meta/predicates.md). Namespaces: `schema:`, `prov:`,
`cito:`, `arc:` (graph-native).

**Structural** (involving the domain kinds):

| Predicate        | From → To                    | Aligned term        |
| ---------------- | ---------------------------- | ------------------- |
| `proposes`       | source → hypothesis          | arc:proposes        |
| `raises`         | source → aporia              | arc:raises          |
| `wasDerivedFrom` | hypothesis/aporia → source   | prov:wasDerivedFrom |
| `assumes`        | hypothesis → entity          | arc:assumes         |
| `concerns`       | aporia → entity              | schema:about        |
| `addresses`      | hypothesis → aporia          | arc:addresses       |
| `addressedBy`    | aporia → hypothesis          | arc:addressedBy     |
| `solvedBy`       | aporia → resource/hypothesis | arc:solvedBy        |

The `source` front-matter field on `hypothesis`/`aporia` carries the `wasDerivedFrom`
(`dcterms:source`) provenance edge.

**Citation** (CORE §8): `citesAsEvidence` and the other `cito:` types, used in `**Evidence**`
and `**Solved by**`.

**Domain extensions** (`arc:`): `secures`, `verifies` (entity → entity).

## 5. Validation Classes

`class` on `hypothesis` and `aporia` is produced only by an **optional validation pass** against
prior knowledge; an unvalidated node has no `class`, which is a valid state. Per the
`validated-overwrite` merge (CORE §10), `class`/`confidence`/`rank` are owned by that pass.

- **hypothesis.class** — `established` (well-supported by accepted knowledge); `extended` (an
  increment on accepted knowledge); `novel` (a new claim not yet corroborated).
- **aporia.class** — `critical` (open, material gap); `solved` (a known resolution exists,
  recorded via `solvedBy`); `unverified` (plausible, not yet confirmed).

## 6. Contradiction, Debate, and Question

Expressed with existing kinds and predicates; this profile adds no node types for them:

- **Question** → an `aporia` (class `unverified` until validated).
- **Contradiction** → a `disputes` edge between the two conflicting hypotheses; reify as an
  `aporia` when it warrants its own discussion.
- **Debate** → the subgraph of an `aporia`, the hypotheses that `address` it, and the
  `supports`/`disputes` edges among them.

## 7. Conformance

In addition to the CORE checklist ([CORE §14](CORE.md)):

- [ ] Every `hypothesis`/`aporia` has a `source` link and a title-based basename (CORE §6.3).
- [ ] `class` appears only where validation ran; node location never depends on class.
- [ ] Every document appears in the `timeline` files for its `published` period.
- [ ] Every predicate used is in §4 or the core vocabulary, and registered in
      [`graph/_meta/predicates.md`](graph/_meta/predicates.md).
