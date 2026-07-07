# CORE — Markdown Knowledge Graph

**Status:** Draft · **Version:** 0.7 · **Date:** 2026-07-07

> Revision note (0.4 → 0.5): this revision makes predicates and types **first-class, schema nodes** rather than prose vocabulary (§9), reframes the node model explicitly in RDF terms (§3, §5), and renames the identity/classification front-matter fields to the JSON-LD `@id`/ `@type` pair (§10.1) — replacing the old `kind`/`id`/`title` fields. Merge is now declared **per predicate** (§9.3), not per node kind; the old kind-level merge table is retired. This is a **breaking change**: [`ARCNET-AST.md`](ARCNET-AST.md), [`SPEC.md`](SPEC.md), [`ARCNET-DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md), [`ARCNET-DOMAIN-CORE-THOUGHT.md`](ARCNET-DOMAIN-CORE-THOUGHT.md), and the example graphs under `examples/` still used the pre-0.5 `kind`/`id`/`title` fields and the kind-level merge table at the time this note was written. **Update:** `ARCNET-AST.md`, `ARCNET-DOMAIN-ARTICLE.md`, and `ARCNET-DOMAIN-CORE-THOUGHT.md` have since been brought current (see their own revision notes); `SPEC.md` and the example graphs under `examples/` still need a follow-up pass.
>
> Revision note (0.5 → 0.6): a `Class` node's `## Requires`/`## Recommended`/`## Optional` bullets were bare `[[predicate]]` mentions — not a clean fit for any of §5's five roles, and not themselves registered predicates as §9.1 requires everything else to be. Fixed by registering `required`/`recommended`/`optional` (role `link`, Class → Property) so those bullets read `required:: [[predicate]]` etc. like any other typed edge (§10.8). While closing that gap, also registered the remaining fields a `Property`/`Class` node's own front-matter/body already used without being registered themselves — `role`, `merge`, `label`, `aligned`, `description` (§10.8) — so the schema mechanism fully satisfies its own §16 checklist. `timeline`'s worked example was also corrected to type its `entries` bullets explicitly (`entries:: [[...]]`), matching `entries`'s own §10.7 registration as a role-`link` predicate rather than a bare mention.
>
> Revision note (0.6 → 0.7): a `Class` node's three-tier `## Requires`/`## Recommended`/`## Optional` membership was ambiguous — "recommended" gave a producer no checkable rule for when a predicate crosses from optional into expected. Simplified to two tiers: `## Requires` (MUST) and `## Optional` (MAY). The `recommended` predicate (§10.8) is retired; every predicate a type previously recommended is now either required or optional on that type, decided by whether its absence would leave the node incomplete for the graph's purpose (§11's own types show the reasoning per type).

This document specifies the **domain-agnostic core** of a knowledge graph stored as plain Markdown: the RDF-aligned data model (§3), the node model built from it (§5), identity, folder layout, edges, the schema mechanism that makes predicates and types first-class graph nodes (§9), the core predicates and core types (`source`, `entity`, `resource`, `timeline`), citations, merge, version control, and the patch exchange format. It is **tool-agnostic** — it depends on no program, library, or language.

A **domain profile** (`DOMAIN-<name>.md`) extends this core with additional types and their predicate vocabulary, registered the same way CORE's own are (§9). Profiles depend on CORE; CORE never references a profile. The reference profile is [`DOMAIN-ARTICLE.md`](DOMAIN-ARTICLE.md), with a worked example under [`graph/`](graph/).

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in RFC 2119. Each type's schema node (§9.2) declares its predicates at two levels, each a body section:

- **`## Requires`** — a conforming instance MUST carry this predicate.
- **`## Optional`** — a conforming instance MAY carry it.

There is no third, "recommended" tier: a predicate a type merely tends to carry, without a checkable rule for when its absence makes a node incomplete, is optional; a predicate the type's purpose depends on is required.

## 2. Purpose and Scope

A knowledge graph is a set of linked Markdown files. Each file is a node; `[[wiki-links]]` between files are edges. Underneath the Markdown, every node is a **bag of predicates about one subject** — the same statement an RDF triple makes — and Markdown is a lossless, human-readable serialization of that bag (§3, §5). CORE defines the substrate shared by all domains and the core predicates/types every document-ingestion domain reuses; a domain profile adds domain-specific ones. Because predicates and types are themselves graph nodes (§9), the vocabulary is part of the graph, not external prose. CORE does not define how nodes are produced.

## 3. RDF Model

A node names one **subject**. Its `@id` (§10.1) is the subject's identifier; its `@type` names the `rdfs:Class` the subject is an instance of (§11 lists CORE's own classes; a profile adds its own). Every other predicate on the node is a statement **about** that subject — one RDF triple `(@id, predicate, value)` per value; a multi-valued predicate decomposes into one triple per element.

```markdown
---
"@id": Transport Layer Security
"@type": entity
category: [independent, abstract, occurrent, script]
tags: [cryptography]
---
# Transport Layer Security

A [[cryptographic]] protocol that establishes an authenticated, confidential channel over an untrusted network.

- replaces:: [[SSL Protocol]]
- conformsTo:: [[RFC 8446]]

## mentionedIn
- mentionedIn:: [[rescorla-2026-tls13]]
```

```nt
"Transport Layer Security"
  a "entity"
  category "independent" ; category "abstract" ; category "occurrent" ; category "script"
  tags "cryptography"
  text "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network."
  href "cryptographic"
  replaces "SSL Protocol"
  conformsTo "RFC 8446"
  mentionedIn "rescorla-2026-tls13"
  .
```

The conversion is lossless and reversible in both directions — Markdown → triples → Markdown reproduces the original file, and triples → Markdown → triples reproduces the original bag — because every predicate's rendering position is fixed once, on the predicate's own schema node (§9.1), not decided ad hoc per document. §5 gives the five positions a predicate can render to.

This inverts the classical RDFS reading for a practical reason. `rdfs:domain` states which classes a property's subjects belong to; for a document-ingestion graph the checkable direction runs the other way — a **class declares which predicates its instances require** (§9.2, §11). CORE's `Class`/`Property` nodes are exactly `rdfs:Class`/`rdf:Property`, just authored for that direction.

## 4. Design Invariants

A conforming graph MUST uphold these invariants:

1. **One node, one file.** Each node is a single `.md` file with a YAML front-matter header and a Markdown body.
2. **Identity is the `@id`.** A node's identity is its mandatory `@id` predicate, equal to the file's basename (without `.md`), unique across the whole graph (§7). Links resolve by basename, independent of folder; a node MAY be moved between folders without affecting any edge.
3. **Edges are wiki-links.** An edge is `[[Target]]`, where `Target` is another node's basename. An edge MAY be typed by a predicate (§8).
4. **Predicates are first class.** Every predicate used anywhere in the graph — front-matter field, body edge, or citation type — is itself a node under `_schema/predicates/` (§9.1), declaring that predicate's serialization role (§5) and merge behavior (§9.3) once, for every type that uses it.
5. **Types are first class.** Every `@type` value in use is itself a node under `_schema/types/` (§9.2), declaring which predicates it requires or permits (§11). CORE's own types (`source`, `entity`, `resource`, `timeline`) are ordinary instances of this mechanism, not special-cased.
6. **Derived nodes carry provenance.** A node distilled from a document MUST link to the document node(s) it was derived from.
7. **Classification is data, not location.** A node's `@type` and any other classification lives in predicates; folders MAY mirror it but MUST NOT be its sole carrier.
8. **Append-only growth, defined merges.** Adding content creates new files or merges into existing ones per each predicate's declared merge behavior (§9.3). Merges are commutative and idempotent at the predicate level, so replay, reordering, and rollback are well-defined.
9. **Git is the history.** Version, provenance-over-time, and rollback are recorded in git (§13). The format stores no change log.

## 5. Node Model

Every node is a YAML front-matter block (delimited by `---`) followed by a Markdown body. Both are constructed from the same underlying bag of predicates (§3) about the node's subject; the split between front-matter and body, and the exact placement within the body, is fixed by each predicate's declared **role** — an attribute of the predicate's own schema node (§9.1), not a convention applied per document:

- **`meta`** — a front-matter scalar or array, in the YAML block.
- **`text`** — prose, as a paragraph in the body.
- **`href`** — an inline, **untyped** wiki-link, inline inside a `text` predicate's prose.
- **`edge`** — a typed edge, a bullet in a heterogeneous block: `predicate:: [[Target]]` (§8.2).
- **`link`** — a typed, homogeneous edge list, its own `## Predicate` block, one bullet per target, predicate implicit.

Because every predicate's role is declared exactly once, reconstructing a node's Markdown from its triples — or recovering its triples by reading the Markdown — needs no per-type special-casing: a consumer resolves each predicate's role and the rendering (or parsing) follows mechanically. Consumers MUST preserve unknown predicates, so the format tolerates a schema that grows over time.

Concretely:

- **Front-matter** carries every predicate with role `meta`, including the mandatory `@id` and `@type` (§10.1). A predicate name beginning with `@` MUST be written as a quoted YAML string key — `"@id"`, `"@type"` — never bare (`@id:`). YAML reserves the unquoted `@` character as an indicator for future use, so a bare `@id:` key is invalid or inconsistently accepted across parsers; quoting is the only form every conforming YAML parser accepts.
- **Body** carries:
  - prose, from `text`-role predicates;
  - inline `[[link]]`s, from `href`-role predicates — untyped mentions embedded in a `text` predicate's prose. A `text` predicate's own statement is never replaced by an embedded link, only annotated by it.
  - one heterogeneous block of `predicate:: [[Target]]` bullets, holding every `edge`-role predicate in use on this node, grouped by purpose under a bold label where that aids reading;
  - one `## Predicate` block per `link`-role predicate in use, its bullets written exactly as `edge`-role bullets are (§8.2's list form, `predicate:: [[Target]]`) — the heading groups them for display, it does not change the bullet syntax. When a type's body consists of exactly one `link`-role predicate, the `## ` heading MAY be omitted, since the block is the entire body (e.g. `timeline`'s `entries`, §11.5).

A node's **type** is named by its `@type` predicate and defined by CORE (§11) or a domain profile: the type's own schema node (§9.2) fixes which predicates it requires or permits.

## 6. Folder Structure

Each type has one folder, named after the type; nodes are filed flat within it. Folders are a filing convenience: because links resolve by basename (§4.2), a node may be re-filed without breaking edges, and its classification lives in predicates (§4.7), not in the folder.

```
graph/
├── sources/              # source nodes (one per ingested document)
├── entities/             # entity nodes (Sowa-typed)
├── resources/            # resource nodes (citations, topics)
├── timeline/             # production-date index
│   ├── yearly/           #   <YYYY>.md
│   └── monthly/          #   <YYYY-MM>.md
├── _schema/              # schema nodes + controlled vocabularies
│   ├── predicates/       #   one node per predicate (§9.1)
│   ├── types/            #   one node per type/class (§9.2)
│   └── aliases.md        #   entity alias table (§7.4) — not a node
└── …                     # domain-profile type folders
```

`_schema/predicates/` and `_schema/types/` hold real nodes — `Property` and `Class` instances, governed by the same rules as any other node (§4.1–§4.2) — describing the graph's own vocabulary rather than its content. `_schema/aliases.md` remains a plain aggregate file, not a node, exactly as `_meta/aliases.md` did before this revision (§6 old numbering).

## 7. Identity and Naming

### 7.1 General

A node's identity is its `@id`, equal to its basename, and MUST be unique across the whole graph. Basenames for title-identified types (entities, resources, and profile types) SHOULD be human-readable, title-cased, with spaces (e.g. `Forward Secrecy`). Colliding basenames MUST be disambiguated with a parenthetical qualifier (e.g. `Handshake Protocol (TLS)`).

### 7.2 Source citekey

A `source` node's `@id` is a citekey derived from the document's own metadata:

```
<first-author-surname>-<publication-year>-<slug-keyword>
```

lowercased, ASCII, hyphen-separated (e.g. `rescorla-2026-tls13`). `surname` is the first author's surname (`anon` if unknown); `year` is the publication year; `slug-keyword` is one or two salient title words. The same document always yields the same citekey. The document's own title is a distinct predicate (`title`, §10.7) — the citekey is not a substitute for it.

### 7.3 Title identity

For types identified by a name or claim (entities, resources, and domain types), the `@id` is the node's title — a concise human label (≤ ~6 words for claim-like types), phrased so the same subject or claim yields the same title. Unlike `source`, these types carry no separate `title` predicate: `@id` already serves as both identity and display name.

### 7.4 Aliases

A subject named differently across documents has exactly one **canonical** node. The canonical `@id` is the preferred label (`skos:prefLabel`); alternatives are recorded in the node's `aliases` predicate (`skos:altLabel`) and in `_schema/aliases.md`. Producers MUST consult the alias table before creating an entity, so synonyms collapse onto the canonical node.

## 8. Edges and Predicates

### 8.1 Link syntax

An edge is `[[Target]]`, a basename reference; it MAY carry a display alias `[[Target|text]]`. A bare `[[Target]]` with no predicate is an `href`-role, untyped mention (§5).

### 8.2 Predicate forms

A predicate types an edge. Three forms are permitted; producers SHOULD use the list form for `edge`-role predicates and MUST use the heading form for `link`-role predicates (§5):

- **List form:** `- replaces:: [[SSL Protocol]]`
- **Body form:** `conformsTo:: [[RFC 8446]]`
- **Inline form:** `... was [citesAsEvidence:: [[RFC 8446]]] standardized.`

The `::` token separates predicate from target.

### 8.3 Predicate naming and registration

Predicate names MUST be **camelCase**. Every predicate in use MUST be registered as a node under `_schema/predicates/` (§9.1), aligned to a standard vocabulary term where one exists; otherwise it is graph-native (`arc:`). Producers MUST reuse a registered predicate before introducing a new one.

## 9. Schema

Predicates and types are ordinary graph nodes: the mechanism CORE and every profile use to declare their own vocabulary is itself part of the graph, not external prose.

### 9.1 Predicate nodes

Every predicate in use — front-matter field, body edge, citation type — MUST be registered as a node at `_schema/predicates/<name>.md`.

**Front-matter**
- `@id` (mandatory) — the predicate's camelCase name, equal to the basename
- `@type` (mandatory) — the literal `Property`
- `role` (mandatory) — one of `meta`, `text`, `href`, `edge`, `link` (§5)
- `merge` (mandatory) — one of the merge behaviors (§9.3)
- `label` (recommended) — human-readable title shown as a `link`-role predicate's `## ` heading
- `aligned` (recommended) — the standard-vocabulary term this predicate maps to (e.g. `dcterms:isPartOf`), or `arc:<name>` if graph-native

**Body**
- `description` (mandatory) — one to a few sentences of prose describing the predicate's meaning

```markdown
---
"@id": isPartOf
"@type": Property
role: edge
merge: union
aligned: "dcterms:isPartOf"
---
# isPartOf

Asserts that the subject is a component or member of the whole named by the target — composition (part–whole), not generalization. See [[broader]] for is-a.
```

### 9.2 Type nodes

Every `@type` value in use MUST be registered as a node at `_schema/types/<name>.md`, declaring the predicates it requires and permits.

**Front-matter**
- `@id` (mandatory) — the type's name, equal to the basename
- `@type` (mandatory) — the literal `Class`

**Body**
- `description` (mandatory) — a definition of the type (`description`, §10.8)
- `## Requires` (mandatory if the type has any type-specific mandatory predicate) — `required:: [[predicate]]` bullets a conforming instance MUST carry (`required`, §10.8)
- `## Optional` (optional) — `optional:: [[predicate]]` bullets a conforming instance MAY carry (`optional`, §10.8)

Every type implicitly requires `@id` and `@type` (§10.1); these are never repeated in a type's own `## Requires` list.

```markdown
---
"@id": entity
"@type": Class
---
# entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.7).

## Requires
- required:: [[category]]
- required:: [[definition]]
- required:: [[mentionedIn]]

## Optional
- optional:: [[aliases]]
- optional:: [[tags]]
```

### 9.3 Merge vocabulary

When ingesting content contributes to a node whose `@id` already exists, each predicate on that contribution is merged per its own declared `merge` value (§9.1). **All merges MUST be commutative and idempotent**, so replay, reordering, and rollback are well-defined (§13). A predicate declares exactly one of:

- **`immutable`** — the first write is permanent; a later contribution to the same predicate on the same subject is a no-op. Used by identity predicates (`@id`, `@type`) and facts fixed at production time (e.g. `published`).
- **`union`** — set-union the predicate's values across contributions, deduplicated.
- **`firstWriteWin`** — a single-valued predicate: the first writer's value wins; a later divergent value is flagged `needsReview` rather than silently overwritten.
- **`fillIfEmpty`** — a single-valued, optional predicate: stays absent until first written, then behaves as `firstWriteWin`.
- **`lastWriteWin`** — a single-valued predicate whose latest write always wins — bookkeeping state that legitimately changes over time (e.g. `updated`).
- **`append`** — grow the predicate's content without discarding what is already there: an ordered, uniquely-keyed list gets a new entry (re-insertion of an already-present entry is a no-op); a `text`-role predicate gets the new prose concatenated after the existing prose.
- **`validatedOverwrite`** — overwritten only by a designated validation pass (last-validation-wins); used by profile predicates whose value is computed after the fact.

A type-level "this node never changes after creation" behavior (e.g. `source`, owned by a single producer) is not a separate merge value — it emerges from every one of that type's predicates independently being `immutable`, reinforced procedurally by the ingestion idempotency check (§13.2).

A domain profile MUST declare each of its own predicates' merge behavior from this menu (§14).

## 10. Core Predicates

The predicates below are CORE's own `_schema/predicates/` nodes — every core type (§11) is built from this vocabulary. Following the RDFS convention of specifying each property under its own heading, one node per predicate, rather than in a shared table. A domain profile adds the predicates its own types require (§14), following the same node format (§9.1).

### 10.1 Identity (JSON-LD core)

CORE inherits two predicates from JSON-LD, mandatory on every node.

#### `@id`
**role:** `meta` · **merge:** `immutable`

Identifies the node; equal to the basename (§7).

#### `@type`
**role:** `meta` · **merge:** `immutable`

Names the node's class — CORE's (§11) or a profile's.

### 10.2 Content predicates

#### `tags`
**role:** `meta` · **merge:** `union`

Topical tags for discoverability.

#### `text`
**role:** `text` · **aligned:** `schema:text` · **merge:** `append`

Generic prose predicate. Each contribution appends to the existing prose rather than overwriting it, since separate documents may each add relevant text about the same subject over time. A type MAY instead declare its own, more specific text predicate (e.g. `abstract`, `definition`, `relevance`, §10.7) when a precise name aids reading and a single, first-fixed value is wanted instead.

### 10.3 Metadata and control predicates

#### `published`
**role:** `meta` · **merge:** `immutable`

ISO-8601 production date of the document a node derives from; drives the timeline (§11.5).

#### `created`
**role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp the node was created in the graph.

#### `updated`
**role:** `meta` · **merge:** `lastWriteWin`

ISO-8601 timestamp of the node's last modification.

### 10.4 Structural predicates (core types)

#### `mentions`
**role:** `link` · **merge:** `union` · **aligned:** `schema:mentions` · **from → to:** source → entity

Asserts that the source document mentions the entity; recorded under the source's own `## Mentions` block.

#### `mentionedIn`
**role:** `link` · **merge:** `union` · **aligned:** `schema:subjectOf` · **from → to:** entity → source

The inverse of `mentions` — recorded as a backlink under the entity's own `## mentionedIn` block.

### 10.5 Semantic predicates (entity ↔ entity / resource)

Semantic predicates relate one **entity** to another entity or resource. They are written as `edge`-role predicates in the entity body and assert how two subjects relate **in the world**, independent of any document. Choose the **most specific** predicate that holds; fall back to `related` only when none of the others fit. Inverses are optional backlinks — assert the direction natural to the node you are writing and let tooling derive the rest.

Choosing: ask in order — is it a *kind of* (`broader`), a *part of* (`isPartOf`), a *dependency* (`requires`), a *successor of* (`replaces`), *conformance to a standard* (`conformsTo`)? Only if none hold, use `related`.

#### `broader`
**role:** `edge` · **merge:** `union` · **aligned:** `skos:broader`

**Generalization.** `X broader:: [[Y]]` asserts Y is the more general concept, X a kind or specialization of it. Concept hierarchy, not composition. *e.g.* `Mutual TLS` → `broader:: [[Transport Layer Security]]`.

#### `narrower`
**role:** `edge` · **merge:** `union` · **aligned:** `skos:narrower`

The inverse of `broader` — an optional backlink from the more general concept to the specialization.

#### `isPartOf`
**role:** `edge` · **merge:** `union` · **aligned:** `dcterms:isPartOf`

**Composition (part–whole).** `X isPartOf:: [[Y]]` asserts X is a component or member of the whole Y. Mereology, not "is a kind of" (that is `broader`). *e.g.* `Certificate Transparency` → `isPartOf:: [[Audit Log]]`.

#### `hasPart`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:hasPart`

The inverse of `isPartOf` — an optional backlink from the whole to a component.

#### `requires`
**role:** `edge` · **merge:** `union` · **aligned:** `dcterms:requires`

**Functional dependency.** `X requires:: [[Y]]` asserts X needs Y to function, hold, or be delivered. Use for prerequisites, not membership or kinds. *e.g.* `Forward Secrecy` → `requires:: [[Handshake Protocol]]`.

#### `replaces`
**role:** `edge` · **merge:** `union` · **aligned:** `dcterms:replaces`

**Supersession over time.** `X replaces:: [[Y]]` asserts X supplants an older Y (Y obsolete in favour of X). Use for versions and standards that succeed one another. *e.g.* `Transport Layer Security` → `replaces:: [[SSL Protocol]]`.

#### `isReplacedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `dcterms:isReplacedBy`

The inverse of `replaces` — an optional backlink from the superseded subject to its successor.

#### `conformsTo`
**role:** `edge` · **merge:** `union` · **aligned:** `dcterms:conformsTo`

**Standard adherence.** `X conformsTo:: [[Y]]` asserts X complies with a named specification or schema Y (typically a resource). *e.g.* `Transport Layer Security` → `conformsTo:: [[RFC 8446]]`.

#### `related`
**role:** `edge` · **merge:** `union` · **aligned:** `skos:related`

**Associative link.** A non-hierarchical, non-compositional association between two connected subjects where none of the above applies. Last resort; prefer a specific predicate whenever one fits.

### 10.6 Citation predicates

A citation is a higher-order predicate: it does not assert a fact about the world, it asserts that a statement in the citing node is backed by an external work and qualifies how the work is used (§12). Used inline, at the point of the statement they support. Citation types SHOULD be drawn from the Citation Typing Ontology (`cito:`); a producer MUST select the most specific type that holds.

#### `cites`
**role:** `link` · **merge:** `union` · **aligned:** `cito:cites` / `schema:citation` · **from → to:** source → resource

The general-purpose citation type; also the source's own structural link to a cited resource, recorded under its `## Cites` block.

#### `citesAsEvidence`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:citesAsEvidence`

Cites the target as evidence for the citing statement.

#### `citesAsAuthority`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:citesAsAuthority`

Cites the target as an authoritative source for the citing statement.

#### `supports`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:supports`

The citing statement is supported by the target.

#### `confirms`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:confirms`

The citing statement confirms findings in the target.

#### `extends`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:extends`

The citing statement extends work in the target.

#### `critiques`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:critiques`

The citing statement critiques the target.

#### `disputes`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:disputes`

The citing statement disputes claims in the target.

#### `refutes`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:refutes`

The citing statement refutes claims in the target.

#### `isCitedBy`
**role:** `link` · **merge:** `union` · **aligned:** `cito:isCitedBy` · **from → to:** resource → node

The inverse of any citation predicate — recorded as a backlink under the cited node's own `## isCitedBy` block.

### 10.7 Type-specific predicates

Predicates used by exactly the core types named below, rather than shared across every type the way §10.1–§10.3 are.

#### `title`
**Used by:** `source` · **role:** `meta` · **merge:** `immutable`

The document title as published — distinct from `@id` when `@id` is a derived citekey (§7.2).

#### `abstract`
**Used by:** `source` · **role:** `text` · **merge:** `firstWriteWin`

A short prose summary of the document.

#### `authors`
**Used by:** `source`, `resource` · **role:** `meta` · **merge:** `union`

Ordered list of author names.

#### `url`
**Used by:** `source`, `resource` · **role:** `meta` · **merge:** `fillIfEmpty`

Canonical location of the document/work.

#### `doi`
**Used by:** `resource` · **role:** `meta` · **merge:** `fillIfEmpty`

Digital object identifier.

#### `category`
**Used by:** `entity` · **role:** `meta` · **merge:** `firstWriteWin`

Records John F. Sowa's top-level category, identified by a three-letter code and a leaf name (e.g. `ipc:object`), **decoded into a bag of words**. The three letters decode by position; the leaf name is appended:

- Position 1 — `i` independent · `r` relative · `m` mediating
- Position 2 — `p` physical · `a` abstract
- Position 3 — `c` continuant · `o` occurrent
- Leaf — object · process · schema · script · juncture · participation · description · history · structure · situation · reason · purpose

The full code-to-word mapping:

- `ipc:object` → `[independent, physical, continuant, object]`
- `ipo:process` → `[independent, physical, occurrent, process]`
- `iac:schema` → `[independent, abstract, continuant, schema]`
- `iao:script` → `[independent, abstract, occurrent, script]`
- `rpc:juncture` → `[relative, physical, continuant, juncture]`
- `rpo:participation` → `[relative, physical, occurrent, participation]`
- `rac:description` → `[relative, abstract, continuant, description]`
- `rao:history` → `[relative, abstract, occurrent, history]`
- `mpc:structure` → `[mediating, physical, continuant, structure]`
- `mpo:situation` → `[mediating, physical, occurrent, situation]`
- `mac:reason` → `[mediating, abstract, continuant, reason]`
- `mao:purpose` → `[mediating, abstract, occurrent, purpose]`

The `category` predicate MUST contain the four decoded words. A consumer MAY recompose the code.

#### `aliases`
**Used by:** `entity` · **role:** `meta` · **merge:** `union`

Alternative names (`skos:altLabel`, §7.4).

#### `definition`
**Used by:** `entity` · **role:** `text` · **merge:** `firstWriteWin`

A 1–3 sentence definition of the subject.

#### `notes`
**Used by:** `entity`, `resource` · **role:** `text` · **merge:** `firstWriteWin`

Additional prose.

#### `ref`
**Used by:** `resource` · **role:** `meta` · **merge:** `immutable`

Resource type: a citable work (`paper`/`standard`/`tool`/`dataset`/`post`) or a topic/area (`technique`/`theory`/`platform`/`system`/`technology`/`language`/`framework`/`field`); a profile MAY extend the set.

#### `year`
**Used by:** `resource` · **role:** `meta` · **merge:** `fillIfEmpty`

Year of publication.

#### `status`
**Used by:** `resource` · **role:** `meta` · **merge:** `lastWriteWin`

`read` or `backlog` — a `backlog` resource is a research target.

#### `relevance`
**Used by:** `resource` · **role:** `text` · **merge:** `firstWriteWin`

A 1–2 sentence note on why the resource matters.

#### `granularity`
**Used by:** `timeline` · **role:** `meta` · **merge:** `immutable`

`yearly` or `monthly`.

#### `entries`
**Used by:** `timeline` · **role:** `link` · **merge:** `append`

The `source` nodes whose `published` date falls in this period, ordered by date.

#### `heading`
**Used by:** `timeline` · **role:** `meta` · **merge:** `firstWriteWin`

A human-readable title for the period, shown as H1 in place of the bare `@id` (period code).

### 10.8 Schema predicates

Predicates used by exactly `Property`/`Class` nodes (§9.1, §9.2) — the schema mechanism's own vocabulary, registered like any other predicate rather than left as unregistered structure.

#### `role`
**Used by:** `Property` · **role:** `meta` · **merge:** `immutable`

One of `meta`/`text`/`href`/`edge`/`link` (§5): the predicate's serialization position.

#### `merge`
**Used by:** `Property` · **role:** `meta` · **merge:** `immutable`

One of the merge behaviors (§9.3): how contributions to this predicate combine.

#### `label`
**Used by:** `Property` · **role:** `meta` · **merge:** `firstWriteWin`

Human-readable title shown as a `link`-role predicate's `## ` heading; defaults to the predicate name, capitalized.

#### `aligned`
**Used by:** `Property` · **role:** `meta` · **merge:** `firstWriteWin`

The standard-vocabulary term this predicate maps to (e.g. `dcterms:isPartOf`), or `arc:<name>` if graph-native.

#### `description`
**Used by:** `Property`, `Class` · **role:** `text` · **merge:** `firstWriteWin`

Prose describing the predicate's or type's meaning — the body text of a `Property`/`Class` node.

#### `required`
**Used by:** `Class` · **role:** `link` · **merge:** `union` · **label:** `Requires`

Asserts that the class requires the target predicate on every conforming instance. Recorded under the class's own `## Requires` block.

#### `optional`
**Used by:** `Class` · **role:** `link` · **merge:** `union`

Asserts that the class permits the target predicate. Recorded under the class's own `## Optional` block.

## 11. Core Types

### 11.1 Every node

Every node, regardless of type, carries the two JSON-LD predicates (§10.1): `@id` and `@type`. A type's own `## Requires`/`## Optional` lists (§9.2) never repeat these two — they are universal.

### 11.2 `source`

A node for one ingested document — the provenance origin other nodes derive from. **Identity:** citekey (§7.2).

```markdown
---
"@id": source
"@type": Class
---
# source

A node for one ingested document — the provenance origin other nodes derive from.

## Requires
- required:: [[title]]
- required:: [[published]]
- required:: [[abstract]]
- required:: [[mentions]]

## Optional
- optional:: [[authors]]
- optional:: [[url]]
- optional:: [[cites]]
- optional:: [[tags]]
- optional:: [[doi]]
```

`mentions` is required because ingesting a document that names no entity contributes nothing a subject-graph can use; `authors`/`url` are optional since a document's authorship or canonical location is not always known (§7.2's citekey rule already allows `anon`), and `cites` is optional since not every document cites an external work.

A domain profile MAY add navigation blocks linking the document to its own derived types (e.g. `## Proposes`, `## Raises`).

```markdown
---
"@id": rescorla-2026-tls13
"@type": source
title: "TLS 1.3: Design and Rationale"
authors: [Eric Rescorla]
published: 2026-04-12
url: https://example.org/tls13-design
tags: [tls, protocols]
---
# TLS 1.3: Design and Rationale

A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption.

## Mentions
- mentions:: [[Transport Layer Security]]
- mentions:: [[Forward Secrecy]]

## Cites
- cites:: [[RFC 8446]]
```

### 11.3 `entity`

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.7). **Identity:** `@id` + alias table (§7.3, §7.4).

```markdown
---
"@id": entity
"@type": Class
---
# entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.7).

## Requires
- required:: [[category]]
- required:: [[definition]]
- required:: [[mentionedIn]]

## Optional
- optional:: [[aliases]]
- optional:: [[tags]]
- optional:: [[notes]]
- any §10.5 semantic predicate, as applicable
```

`mentionedIn` is required per invariant 6 (§4) — an entity exists only because some source mentions it, so the backlink to at least that source MUST already hold.

```markdown
---
"@id": Transport Layer Security
"@type": entity
category: [independent, abstract, occurrent, script]
aliases: [TLS, TLS 1.3]
tags: [cryptography]
---
# Transport Layer Security

A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network.

- replaces:: [[SSL Protocol]]
- conformsTo:: [[RFC 8446]]

## mentionedIn
- mentionedIn:: [[rescorla-2026-tls13]]
```

### 11.4 `resource`

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research. Distinct from a `source` (ingested, derived *from*) and an `entity` (a subject the content is *about*). **Identity:** `@id` (§7.3).

```markdown
---
"@id": resource
"@type": Class
---
# resource

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research.

## Requires
- required:: [[ref]]
- required:: [[relevance]]

## Optional
- optional:: [[url]]
- optional:: [[isCitedBy]]
- optional:: [[authors]]
- optional:: [[year]]
- optional:: [[doi]]
- optional:: [[status]]
- optional:: [[notes]]
```

`relevance` is required for the same reason `source.abstract`/`entity.definition` are: a resource with no stated reason to matter isn't yet worth having filed. `isCitedBy` is optional — a `backlog` resource (`status`, §10.7) legitimately exists before anything cites it.

```markdown
---
"@id": RFC 8446
"@type": resource
ref: standard
authors: [Eric Rescorla]
year: 2018
url: https://www.rfc-editor.org/rfc/rfc8446
status: read
---
# RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3

The normative specification of TLS 1.3.

## isCitedBy
- isCitedBy:: [[rescorla-2026-tls13]]
```

### 11.5 `timeline`

A production-date index of ingested documents (not a change log — that is git, §13). **Identity:** the period code (§7.1), `YYYY` or `YYYY-MM`.

```markdown
---
"@id": timeline
"@type": Class
---
# timeline

A production-date index of ingested documents.

## Requires
- required:: [[granularity]]
- required:: [[entries]]

## Optional
- optional:: [[heading]]
```

```markdown
---
"@id": 2026-04
"@type": timeline
granularity: monthly
---
# April 2026

- entries:: [[rescorla-2026-tls13]] — *TLS 1.3: Design and Rationale* (Eric Rescorla) — 2026-04-12
- entries:: [[chen-2026-pqkex]] — *Post-Quantum Key Exchange in Practice* (Lin Chen) — 2026-04-28
```

## 12. Citations

A **citation is a higher-order predicate**: it does not assert a fact about the world, it asserts that a statement in this node is backed by an external work and qualifies how the work is used. Citations are recorded **inline, at the point of the statement they support** (§10.6); the graph defines no global bibliography.

- The subject is the citing node (or a specific assertion in its body); the target is a node representing an external work; the predicate is a citation type from §10.6.
- A producer MUST select the most specific citation type that holds.
- **Mandatory:** a registered citation-type predicate and a `[[wiki-link]]` target, in the body of the citing node. **Recommended:** placement adjacent to the supported statement and recording the inverse `isCitedBy` on the target.

## 13. Version Control

Git **MUST** be used as the version-control system. No other system is used. Git is the authoritative record of how the graph changed; the format stores no change log.

### 13.1 One document, one commit

Ingesting a document is exactly one commit containing its entire contribution: the new `source` node, any new nodes, merged predicates on existing nodes (§9.3), and the timeline entries for the document's `published` period.

### 13.2 Operations

- **Ingest** — `git add -A` then `git commit -F <msg>` (§13.3)
- **Retract** — `git revert <commit>`
- **Locate commit** — `git log --grep=<id>`
- **Node history** — `git log --follow -- '<path-to-node>'`
- **Idempotency** — before ingesting, check `git ls-files --error-unmatch sources/<id>.md` — skip if present

Because each contribution is additive (§9.3), `git revert` resolves shared-node edits with a standard three-way merge.

### 13.3 Commit message format

```
graph(ingest): <id> — <title>

<one-line summary>

Nodes: +<counts by type>
Edges: +<n>
Timeline: <YYYY-MM>
Source-Id: <id>
```

- **Mandatory:** subject `graph(ingest): <id> — <title>` (retraction uses `graph(retract): <id>`); the `Nodes:` stats line; the `Source-Id:` trailer.
- **Recommended:** `Edges:`, `Timeline:`, and any source URL/date trailers.
- **Optional:** the summary paragraph.

## 14. Document Patch (Exchange Format)

A **patch** is a single Markdown file that serializes one document's entire contribution. A tool produces a patch from a document (an alternative output mode to writing the graph directly); a patch is shareable and is applied to any graph by a tool. A patch is a **parallel exchange serialization, not part of the graph**: never indexed as a node, never stored under `graph/`, never extracted from an existing graph.

### 14.1 Properties

- A patch always carries **full nodes**, never deltas; reconciliation is performed at apply time by each predicate's merge behavior (§9.3).
- A patch renders as ordinary Markdown anywhere. Wiki-links are kept for fidelity; navigating them inside a patch requires patch-aware tooling and is not expected in general viewers.

### 14.2 Structure

- **Front-matter (manifest)** — the single real front-matter block. **Mandatory:** `@type: patch`; `document` (the source id); `published`. **Recommended:** `title`; `stats`.
- **Body — node sections grouped by type:**
  - **H1 = node's `@type`** — `# <Type>` (case-insensitive); one per type present.
  - **H2 = node's `@id`** — `## <basename>`; one per node. `@type` comes from the H1 and `@id` from the H2; neither is repeated below.
  - **Under each H2:** a fenced ` ```yaml ` block with the node's remaining predicates with role `meta`; then the node body (prose + `::` edges with canonical `[[Node]]` targets).
  - Markdown headings are reserved for type and identity; node bodies use bold labels, never headings.
- Index types (`timeline`) are not carried; the apply tool derives them from the source's metadata.

### 14.3 Apply

A tool applies a patch to a target graph:

1. For each H2 node under each H1 type, reconstruct the node — `@type` from the H1, `meta`-role predicates from the ` ```yaml ` block, body (prose + `::` edges) as written.
2. If the node's `@id` does not exist in the target graph, create the file: the yaml block plus `@id`/`@type` become the node's front-matter, the body is written verbatim.
3. If it exists, apply each predicate's §9.3 merge behavior (union edges/aliases/sources; first-writer scalar fields, etc.).
4. Update the timeline (§10.4, `entries`) from the source's `published` date.
5. Commit per §13 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph (step 3's unions add nothing new on the second application).

## 15. Domain Extension

A conforming domain profile (`DOMAIN-<name>.md`) MUST define:

1. **Types** — which CORE types (§11) it adopts, and each domain-specific type's folder, schema node (§9.2, with an inline example), identity (§7), and the merge behavior of its own predicates (§9.3).
2. **Class vocabularies** — any class values its types carry.
3. **Predicates** — the registered, camelCase, standards-aligned predicates its types use (§8.3), each as its own `_schema/predicates/` node (§9.1), beyond the core vocabulary (§10).
4. **Provenance model** — which types are derived and what they link back to (§4.6).
5. **A worked example** demonstrating every type, predicate, and the folder layout.

A profile MUST NOT redefine CORE mechanism (identity, edges, citations, the schema/merge framework, version control, patch format).

## 16. Core Conformance Checklist

- [ ] Every file is one node with valid YAML front-matter and mandatory `@id`/`@type` (§10.1).
- [ ] Every basename equals its node's `@id`, is unique and human-readable; every `[[link]]` resolves to a basename (§7).
- [ ] Every `source`'s `@id` is a citekey equal to its basename (§7.2).
- [ ] Every `entity` has a four-word decoded Sowa `category` (§10.7).
- [ ] Every derived node links to its source(s) (§4.6).
- [ ] Every predicate is camelCase and registered as a `_schema/predicates/` node (§8.3, §9.1).
- [ ] Every `@type` in use is registered as a `_schema/types/` node declaring its Requires/Optional predicates (§9.2).
- [ ] Every citation uses a registered citation-type predicate in the body (§12).
- [ ] Each predicate's merge behavior is one of the §9.3 menu and is commutative/idempotent.
- [ ] Each document was ingested as exactly one git commit (§13).
