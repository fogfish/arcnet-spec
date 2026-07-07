# DOMAIN ARTICLE - Markdown Knowledge Graph extension with Articles

**Status:** Draft · **Version:** 0.7 · **Date:** 2026-07-07
**Extends:** [`CORE.md`](CORE.md)

> Revision note (0.4 → 0.5): follows CORE v0.6's predicate-first model. `kind`/`id`/`title` → `@type`/`@id`. `hypothesis`/`aporia`'s front-matter `source` field was a link value stored where CORE's `meta` role only ever holds scalars — it is now the `derivedFrom` predicate itself, asserted as a body edge like any other. The predicate registry table is retired: every predicate this profile introduces, `hypothesis`/`aporia` themselves, and the extension to CORE's own `source` type are now registered as individual `_schema/predicates/`/`_schema/types/` nodes (CORE §9), described one-per-heading (CORE §10's style) rather than in a table.
>
> Revision note (0.5 → 0.6): follows CORE v0.7's retirement of the `## Recommended` tier. Every predicate `hypothesis`/`aporia`/the `source` extension previously recommended is now optional — none of `overview`/`assumptions`/`assumes`/`addresses`/citation evidence (hypothesis), `overview`/`issues`/`concerns`/`addressedBy` (aporia), or `proposes`/`raises` (source extension) is essential to what the type minimally asserts (a claim/tension plus its provenance); each stays valid content a node commonly carries, just not one CORE can require of every instance.
>
> Revision note (0.6 → 0.7): `rank` was never a considered part of this design — it entered the worked examples without ever being justified against anything `hypothesis`/`aporia` actually need, and was mistakenly formalized into a registered predicate on top of that. Removed entirely: the predicate, its listing on both types, and every mention in §5. `class`/`confidence` are unaffected.

This profile ingests research articles and similar prose documents into a knowledge graph. It **adopts the four CORE types** (`source`, `entity`, `resource`, `timeline`, [CORE §11](CORE.md)) and adds two domain types — `hypothesis` and `aporia`. All CORE mechanism (identity §7, edges §8, schema/merge §9, citations §12, version control §13, patch §14) applies unchanged.

A complete worked example is in [`graph/`](graph/); its single-file patch serialization is in [`patches/`](patches/).

## 1. Folder Layout

```
graph/
├── sources/               # source nodes          (CORE §11.2)
├── entities/              # entity nodes, Sowa    (CORE §11.3)
├── hypothesis/            # hypothesis nodes      (§2.1)
├── aporias/               # aporia nodes          (§2.2)
├── resources/             # resource nodes        (CORE §11.4)
├── timeline/              # production index      (CORE §11.5)
│   ├── yearly/
│   └── monthly/
└── _schema/
    ├── predicates/         # this profile's own + reused CORE predicates (§4)
    ├── types/              # hypothesis.md, aporia.md, and source.md's extension (§2, §3.1)
    └── aliases.md          # entity alias table — not a node (CORE §7.4)
```

`hypothesis/` and `aporias/` are flat: a node's `class` lives in a predicate (§5), so it never determines file location. The profile also extends the core `source` type with domain navigation predicates (§3).

## 2. Domain Types

### 2.1 `hypothesis`

A conclusion distilled from sources. **Identity:** `@id` (CORE §7.3) — the claim in short form, equal to the basename; like `entity`/`resource`, it carries no separate title predicate.

```markdown
---
"@id": hypothesis
"@type": Class
---
# hypothesis

A conclusion distilled from sources.

## Requires
- required:: [[derivedFrom]]
- required:: [[claim]]

## Optional
- optional:: [[overview]]
- optional:: [[assumptions]]
- optional:: [[assumes]]
- optional:: [[addresses]]
- optional:: [[notes]]
- optional:: [[class]]
- optional:: [[confidence]]
- any CORE §10.6 citation predicate, as applicable
```

`class`/`confidence` are produced only by an optional validation pass (§5); an unvalidated `hypothesis` simply has none of them, a valid state.

```markdown
---
"@id": One-RTT Handshake Preserves Security
"@type": hypothesis
class: established
---
*A one round-trip handshake cuts connection-setup latency without weakening the protocol's security guarantees.*

- derivedFrom:: [[rescorla-2026-tls13]]

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

An open problem or unresolved tension. **Identity:** `@id` (CORE §7.3) — the tension in short form, equal to the basename.

```markdown
---
"@id": aporia
"@type": Class
---
# aporia

An open problem or unresolved tension.

## Requires
- required:: [[derivedFrom]]
- required:: [[tension]]

## Optional
- optional:: [[overview]]
- optional:: [[issues]]
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[solvedBy]]
- optional:: [[notes]]
- optional:: [[class]]
```

```markdown
---
"@id": Zero-RTT Replay Exposure
"@type": aporia
class: critical
---
*Zero round-trip resumption lets early application data be replayed by an attacker.*

- derivedFrom:: [[rescorla-2026-tls13]]

**Issues**
- Early data can be captured and re-sent to trigger duplicate side effects.
- Application-layer idempotency is required but not enforceable by the protocol.

**Concerns**
- concerns:: [[Transport Layer Security]]
- concerns:: [[Handshake Protocol]]

**Addressed by**
- addressedBy:: [[One-RTT Handshake Preserves Security]]
```

## 3. Extended Types

A profile MAY extend a CORE type's `_schema/types/` node with additional Optional predicates, without changing the type's identity or any existing predicate's own merge behavior — the contribution unions into the existing node like any other (CORE §9.3 `union`). This profile extends `source`; `entity`, `resource`, and `timeline` are used as CORE defines them.

### 3.1 `source` (extends CORE §11.2)

Beyond the core `## Mentions`/`## Cites` blocks, the article `source` adds two navigation predicates linking the document to the domain types it produces. Identity (citekey) and every other predicate's own merge behavior are inherited from CORE §11.2 unchanged.

```markdown
---
"@id": source
"@type": Class
---
## Optional
- optional:: [[proposes]]
- optional:: [[raises]]
```

This contribution's `## Optional` block unions into CORE's own `source` Class node (CORE §11.2) at ingestion time — the merged node ends up permitting `authors`/`url`/`cites`/`tags`/`doi` (from CORE) and `proposes`/`raises` (from this profile) alike, alongside CORE's own required `title`/`published`/`abstract`/`mentions`.

```markdown
---
"@id": rescorla-2026-tls13
"@type": source
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

## 4. Predicates

In addition to the core vocabulary (CORE §10), this profile registers the following as `_schema/predicates/` nodes (CORE §9.1), one per heading rather than in a table. Namespaces: `schema:`, `prov:`, `cito:`, `arc:` (graph-native).

### 4.1 Structural predicates (involving the domain types)

#### `proposes`
**role:** `link` · **merge:** `union` · **aligned:** `arc:proposes` · **from → to:** source → hypothesis

Asserts that the source document proposes the hypothesis; recorded under the source's own `## Proposes` block.

#### `raises`
**role:** `link` · **merge:** `union` · **aligned:** `arc:raises` · **from → to:** source → aporia

Asserts that the source document raises the aporia; recorded under the source's own `## Raises` block.

#### `derivedFrom`
**role:** `edge` · **merge:** `union` · **aligned:** `prov:wasDerivedFrom` · **from → to:** hypothesis/aporia → source

The provenance edge to the originating source(s) this node was distilled from — replaces the old front-matter `source` field, which never fit CORE's scalar-only `meta` role.

#### `assumes`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:assumes` · **from → to:** hypothesis → entity

The entities a hypothesis's premises depend on. Displayed under the bold label **Depends on**.

#### `concerns`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:about` · **from → to:** aporia → entity

The entities an aporia's open problem involves.

#### `addresses`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addresses` · **from → to:** hypothesis → aporia

The aporia a hypothesis's claim tackles.

#### `addressedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addressedBy` · **from → to:** aporia → hypothesis

The inverse of `addresses` — the hypotheses that tackle this aporia. Displayed under the bold label **Addressed by**.

#### `solvedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:solvedBy` · **from → to:** aporia → resource/hypothesis

The resource or hypothesis that resolves an aporia. Displayed under the bold label **Solved by**.

### 4.2 Type-specific predicates

#### `claim`
**Used by:** `hypothesis` · **role:** `text` · **merge:** `firstWriteWin`

A one-sentence statement of the conclusion, rendered emphasized (`*claim*`).

#### `tension`
**Used by:** `aporia` · **role:** `text` · **merge:** `firstWriteWin`

A one-sentence statement of the open problem, rendered emphasized (`*tension*`).

#### `overview`
**Used by:** `hypothesis`, `aporia` · **role:** `text` · **merge:** `firstWriteWin`

A short paragraph of context.

#### `assumptions`
**Used by:** `hypothesis` · **role:** `text` · **merge:** `append`

Literal premise statements the hypothesis depends on, one per bullet. Displayed under the bold label **Assumptions**.

#### `issues`
**Used by:** `aporia` · **role:** `text` · **merge:** `append`

Literal statements decomposing the open problem, one per bullet. Displayed under the bold label **Issues**.

#### `class`
**Used by:** `hypothesis`, `aporia` · **role:** `meta` · **merge:** `validatedOverwrite`

The node's validation class (§5) — enum values differ per type.

#### `confidence`
**Used by:** `hypothesis` · **role:** `meta` · **merge:** `validatedOverwrite`

A 0–1 numeric confidence score assigned by the validation pass.

### 4.3 Citation and reused predicates

Evidence and resolution use CORE's own citation vocabulary (CORE §10.6): `citesAsEvidence` and the other `cito:` types, displayed under the bold labels **Evidence** / **Solved by**. `notes` is CORE's own generic prose predicate (CORE §10.7), reused unchanged.

### 4.4 Domain extensions

#### `secures`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:secures` · **from → to:** entity → entity

#### `verifies`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:verifies` · **from → to:** entity → entity

## 5. Validation Classes

`class` on `hypothesis` and `aporia` is produced only by an **optional validation pass** against prior knowledge; an unvalidated node has no `class` predicate, which is a valid state. Per the `validatedOverwrite` merge (CORE §9.3), `class`/`confidence` are owned by that pass.

- **hypothesis's `class`** — `established` (well-supported by accepted knowledge); `extended` (an increment on accepted knowledge); `novel` (a new claim not yet corroborated).
- **aporia's `class`** — `critical` (open, material gap); `solved` (a known resolution exists, recorded via `solvedBy`); `unverified` (plausible, not yet confirmed).

## 6. Contradiction, Debate, and Question

Expressed with existing types and predicates; this profile adds no node types for them:

- **Question** → an `aporia` (class `unverified` until validated).
- **Contradiction** → a `disputes` edge between the two conflicting hypotheses; reify as an `aporia` when it warrants its own discussion.
- **Debate** → the subgraph of an `aporia`, the hypotheses that `address` it, and the `supports`/`disputes` edges among them.

## 7. Conformance

In addition to the CORE checklist ([CORE §16](CORE.md)):

- [ ] Every `hypothesis`/`aporia` has a `derivedFrom` edge and a claim/tension-based basename (CORE §7.3).
- [ ] `class` appears only where validation ran; node location never depends on it.
- [ ] Every document appears in the `timeline` files for its `published` period.
- [ ] Every predicate used is registered as a `_schema/predicates/` node (§4 or CORE §10).
- [ ] `hypothesis` and `aporia` are registered as `_schema/types/` nodes (§2); the extension to `source` (§3.1) unions cleanly into CORE's own.
