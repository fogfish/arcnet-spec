# DOMAIN ARTICLE - Markdown Knowledge Graph extension with Articles

**Status:** Draft · **Version:** 0.8 · **Date:** 2026-08-27
**Extends:** [`ARCNET-CORE.md`](ARCNET-CORE.md)

> Revision note (0.4 → 0.5): follows CORE v0.6's predicate-first model. `kind`/`id`/`title` → `@type`/`@id`. `hypothesis`/`aporia`'s front-matter `source` field was a link value stored where CORE's `meta` role only ever holds scalars — it is now the `derivedFrom` predicate itself, asserted as a body edge like any other. The predicate registry table is retired: every predicate this profile introduces, `hypothesis`/`aporia` themselves, and the extension to CORE's own `source` type are now registered as individual `_schema/predicates/`/`_schema/types/` nodes (CORE §9), described one-per-heading (CORE §10's style) rather than in a table.
>
> Revision note (0.5 → 0.6): follows CORE v0.7's retirement of the `## Recommended` tier. Every predicate `hypothesis`/`aporia`/the `source` extension previously recommended is now optional — none of `overview`/`assumptions`/`assumes`/`addresses`/citation evidence (hypothesis), `overview`/`issues`/`concerns`/`addressedBy` (aporia), or `proposes`/`raises` (source extension) is essential to what the type minimally asserts (a claim/tension plus its provenance); each stays valid content a node commonly carries, just not one CORE can require of every instance.
>
> Revision note (0.6 → 0.7): `rank` was never a considered part of this design — it entered the worked examples without ever being justified against anything `hypothesis`/`aporia` actually need, and was mistakenly formalized into a registered predicate on top of that. Removed entirely: the predicate, its listing on both types, and every mention in §5. `class`/`confidence` are unaffected.
>
> Revision note (0.7 → 0.8): follows CORE v0.9's type-naming rule (types MUST be UpperCamelCase) and v0.11's folder rule (a type's folder name MUST be character-for-character identical to the type name): `source`/`entity`/`resource`/`timeline`/`hypothesis`/`aporia` are renamed `Source`/`Entity`/`Resource`/`Timeline`/`Hypothesis`/`Aporia` everywhere — `@type` values, `_schema/Class/` basenames, and folder names (`sources/`→`Source/`, `entities/`→`Entity/`, `hypothesis/`→`Hypothesis/`, `aporias/`→`Aporia/`, `resources/`→`Resource/`); `_schema/predicates/`→`_schema/Property/`, `_schema/types/`→`_schema/Class/` per CORE v0.11 §6. `authors` is corrected to the registered singular `author` (CORE v0.12) in the `Source` extension's worked example. `notes`, used optionally by `Hypothesis`/`Aporia`, was retired from CORE in v0.12 without being reinstated — it is now registered as this profile's own `_schema/Property/` node (§4.2) instead of cited as CORE's. `class`/`confidence`'s declared merge behavior, `validatedOverwrite`, was never a member of CORE's closed merge menu (§9.3); corrected to `lastWriteWin`, the closest fit since each validation-pass run replaces the prior value. The stale `[CORE.md](CORE.md)` link is corrected to this repository's actual filename, `ARCNET-CORE.md`.

This profile ingests research articles and similar prose documents into a knowledge graph. It **adopts the four CORE types** (`Source`, `Entity`, `Resource`, `Timeline`, [CORE §11](ARCNET-CORE.md)) and adds two domain types — `Hypothesis` and `Aporia`. All CORE mechanism (identity §7, edges §8, schema/merge §9, citations §12, version control §13, patch §14) applies unchanged.

A complete worked example is in [`graph/`](graph/); its single-file patch serialization is in [`patches/`](patches/).

## 1. Folder Layout

```
graph/
├── Source/                # source nodes          (CORE §11.2)
├── Entity/                # entity nodes, Sowa    (CORE §11.3)
├── Hypothesis/             # hypothesis nodes      (§2.1)
├── Aporia/                 # aporia nodes          (§2.2)
├── Resource/               # resource nodes        (CORE §11.4)
├── timeline/               # production index      (CORE §11.5)
│   ├── yearly/
│   └── monthly/
└── _schema/
    ├── Property/            # this profile's own + reused CORE predicates (§4)
    ├── Class/               # Hypothesis.md, Aporia.md, and Source.md's extension (§2, §3.1)
    └── aliases.md           # entity alias table — not a node (CORE §7.4)
```

`Hypothesis/` and `Aporia/` are flat: a node's `class` lives in a predicate (§5), so it never determines file location. The profile also extends the core `Source` type with domain navigation predicates (§3).

## 2. Domain Types

### 2.1 `Hypothesis`

A conclusion distilled from sources. **Identity:** `@id` (CORE §7.3) — the claim in short form, equal to the basename; like `Entity`/`Resource`, it carries no separate title predicate.

```markdown
---
"@id": Hypothesis
"@type": Class
---
# Hypothesis

A conclusion distilled from sources.

## Requires
- required:: [[derivedFrom]]
- required:: [[claim]]

## Optional
- optional:: [[overview]]
- optional:: [[assumptions]]
- optional:: [[assumes]]
- optional:: [[addresses]]
- optional:: [[class]]
- optional:: [[confidence]]
- any CORE §10.6 citation predicate, as applicable
```

`class`/`confidence` are produced only by an optional validation pass (§5); an unvalidated `Hypothesis` simply has none of them, a valid state.

```markdown
---
"@id": One-RTT Handshake Preserves Security
"@type": Hypothesis
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

### 2.2 `Aporia`

An open problem or unresolved tension. **Identity:** `@id` (CORE §7.3) — the tension in short form, equal to the basename.

```markdown
---
"@id": Aporia
"@type": Class
---
# Aporia

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
- optional:: [[class]]
```

```markdown
---
"@id": Zero-RTT Replay Exposure
"@type": Aporia
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

A profile MAY extend a CORE type's `_schema/Class/` node with additional Optional predicates, without changing the type's identity or any existing predicate's own merge behavior — the contribution unions into the existing node like any other (CORE §9.3 `union`). This profile extends `Source`; `Entity`, `Resource`, and `Timeline` are used as CORE defines them.

### 3.1 `Source` (extends CORE §11.2)

Beyond the core `## Mentions`/`## Cites` blocks, the article `Source` adds two navigation predicates linking the document to the domain types it produces. Identity (citekey) and every other predicate's own merge behavior are inherited from CORE §11.2 unchanged.

```markdown
---
"@id": Source
"@type": Class
---
## Optional
- optional:: [[proposes]]
- optional:: [[raises]]
```


```markdown
---
"@id": rescorla-2026-tls13
"@type": Source
title: "TLS 1.3: Design and Rationale"
author: [Eric Rescorla]
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

In addition to the core vocabulary (CORE §10), this profile registers the following as `_schema/Property/` nodes (CORE §9.1), one per heading rather than in a table. Namespaces: `schema:`, `prov:`, `cito:`, `arc:` (graph-native).

### 4.1 Structural predicates (involving the domain types)

#### `proposes`
**role:** `link` · **merge:** `union` · **aligned:** `arc:proposes` · **from → to:** Source → Hypothesis

Asserts that the source document proposes the hypothesis; recorded under the source's own `## Proposes` block.

#### `raises`
**role:** `link` · **merge:** `union` · **aligned:** `arc:raises` · **from → to:** Source → Aporia

Asserts that the source document raises the aporia; recorded under the source's own `## Raises` block.

#### `derivedFrom`
**role:** `edge` · **merge:** `union` · **aligned:** `prov:wasDerivedFrom` · **from → to:** Hypothesis/Aporia → Source

The provenance edge to the originating source(s) this node was distilled from — replaces the old front-matter `source` field, which never fit CORE's scalar-only `meta` role.

#### `assumes`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:assumes` · **from → to:** Hypothesis → Entity

The entities a hypothesis's premises depend on. Displayed under the bold label **Depends on**.

#### `concerns`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:about` · **from → to:** Aporia → Entity

The entities an aporia's open problem involves.

#### `addresses`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addresses` · **from → to:** Hypothesis → Aporia

The aporia a hypothesis's claim tackles.

#### `addressedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addressedBy` · **from → to:** Aporia → Hypothesis

The inverse of `addresses` — the hypotheses that tackle this aporia. Displayed under the bold label **Addressed by**.

#### `solvedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:solvedBy` · **from → to:** Aporia → Resource/Hypothesis

The resource or hypothesis that resolves an aporia. Displayed under the bold label **Solved by**.

### 4.2 Type-specific predicates

#### `claim`
**Used by:** `Hypothesis` · **role:** `text` · **merge:** `firstWriteWin`

A one-sentence statement of the conclusion, rendered emphasized (`*claim*`).

#### `tension`
**Used by:** `Aporia` · **role:** `text` · **merge:** `firstWriteWin`

A one-sentence statement of the open problem, rendered emphasized (`*tension*`).

#### `overview`
**Used by:** `Hypothesis`, `Aporia` · **role:** `text` · **merge:** `firstWriteWin`

A short paragraph of context.

#### `assumptions`
**Used by:** `Hypothesis` · **role:** `text` · **merge:** `append`

Literal premise statements the hypothesis depends on, one per bullet. Displayed under the bold label **Assumptions**.

#### `issues`
**Used by:** `Aporia` · **role:** `text` · **merge:** `append`

Literal statements decomposing the open problem, one per bullet. Displayed under the bold label **Issues**.

#### `class`
**Used by:** `Hypothesis`, `Aporia` · **role:** `meta` · **merge:** `lastWriteWin`

The node's validation class (§5) — enum values differ per type. Each validation-pass run replaces the prior value.

#### `confidence`
**Used by:** `Hypothesis` · **role:** `meta` · **merge:** `lastWriteWin`

A 0–1 numeric confidence score assigned by the validation pass.

### 4.3 Citation and reused predicates

Evidence and resolution use CORE's own citation vocabulary (CORE §10.6): `citesAsEvidence` and the other `cito:` types, displayed under the bold labels **Evidence** / **Solved by**.

### 4.4 Domain extensions

#### `secures`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:secures` · **from → to:** Entity → Entity

#### `verifies`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:verifies` · **from → to:** Entity → Entity

## 5. Validation Classes

`class` on `Hypothesis` and `Aporia` is produced only by an **optional validation pass** against prior knowledge; an unvalidated node has no `class` predicate, which is a valid state. Per the `lastWriteWin` merge (CORE §9.3), `class`/`confidence` are owned by that pass — each run replaces whatever value it previously assigned.

- **`Hypothesis`'s `class`** — `established` (well-supported by accepted knowledge); `extended` (an increment on accepted knowledge); `novel` (a new claim not yet corroborated).
- **`Aporia`'s `class`** — `critical` (open, material gap); `solved` (a known resolution exists, recorded via `solvedBy`); `unverified` (plausible, not yet confirmed).

## 6. Contradiction, Debate, and Question

Expressed with existing types and predicates; this profile adds no node types for them:

- **Question** → an `Aporia` (class `unverified` until validated).
- **Contradiction** → a `disputes` edge between the two conflicting hypotheses; reify as an `Aporia` when it warrants its own discussion.
- **Debate** → the subgraph of an `Aporia`, the hypotheses that `address` it, and the `supports`/`disputes` edges among them.

## 7. Conformance

In addition to the CORE checklist ([CORE §16](ARCNET-CORE.md)):

- [ ] Every `Hypothesis`/`Aporia` has a `derivedFrom` edge and a claim/tension-based basename (CORE §7.3).
- [ ] `class` appears only where validation ran; node location never depends on it.
- [ ] Every document appears in the `timeline` files for its `published` period.
- [ ] Every predicate used is registered as a `_schema/Property/` node (§4 or CORE §10).
- [ ] `Hypothesis` and `Aporia` are registered as `_schema/Class/` nodes (§2); the extension to `Source` (§3.1) unions cleanly into CORE's own.
