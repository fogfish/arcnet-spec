# CORE — Markdown Knowledge Graph

**Status:** Draft · **Version:** 0.12 · **Date:** 2026-08-23

> Revision note (0.4 → 0.5): this revision makes predicates and types **first-class, schema nodes** rather than prose vocabulary (§9), reframes the node model explicitly in RDF terms (§3, §5), and renames the identity/classification front-matter fields to the JSON-LD `@id`/ `@type` pair (§10.1) — replacing the old `kind`/`id`/`title` fields. Merge is now declared **per predicate** (§9.3), not per node kind; the old kind-level merge table is retired. This is a **breaking change**: [`ARCNET-AST.md`](ARCNET-AST.md), [`SPEC.md`](SPEC.md), [`ARCNET-DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md), [`ARCNET-DOMAIN-CORE-THOUGHT.md`](ARCNET-DOMAIN-CORE-THOUGHT.md), and the example graphs under `examples/` still used the pre-0.5 `kind`/`id`/`title` fields and the kind-level merge table at the time this note was written. **Update:** `ARCNET-AST.md`, `ARCNET-DOMAIN-ARTICLE.md`, and `ARCNET-DOMAIN-CORE-THOUGHT.md` have since been brought current (see their own revision notes); `SPEC.md` and the example graphs under `examples/` still need a follow-up pass.
>
> Revision note (0.5 → 0.6): a `Class` node's `## Requires`/`## Recommended`/`## Optional` bullets were bare `[[predicate]]` mentions — not a clean fit for any of §5's five roles, and not themselves registered predicates as §9.1 requires everything else to be. Fixed by registering `required`/`recommended`/`optional` (role `link`, Class → Property) so those bullets read `required:: [[predicate]]` etc. like any other typed edge (§10.7). While closing that gap, also registered the remaining fields a `Property`/`Class` node's own front-matter/body already used without being registered themselves — `role`, `merge`, `label`, `aligned`, `description` (§10.7) — so the schema mechanism fully satisfies its own §16 checklist. `timeline`'s worked example was also corrected to type its `cites` bullets explicitly (`cites:: [[...]]`), matching `cites`'s own §10.6 registration as a role-`link` predicate rather than a bare mention.
>
> Revision note (0.6 → 0.7): a `Class` node's three-tier `## Requires`/`## Recommended`/`## Optional` membership was ambiguous — "recommended" gave a producer no checkable rule for when a predicate crosses from optional into expected. Simplified to two tiers: `## Requires` (MUST) and `## Optional` (MAY). The `recommended` predicate (§10.7) is retired; every predicate a type previously recommended is now either required or optional on that type, decided by whether its absence would leave the node incomplete for the graph's purpose (§11's own types show the reasoning per type).
>
> Revision note (0.7 → 0.8): adds `Reference` (§11.6), a fifth core type for an external document the graph is not ingesting. It reuses three existing predicates — `title`, `url`, `author` (§10.2) — rather than introducing new ones. This is additive, not breaking: no existing predicate, type, or merge behavior changes meaning. (§11.6's own boundary against `Resource` was corrected in 0.10, below.)
>
> Revision note (0.8 → 0.9): documents a naming-convention decision made previously but never written into CORE and never applied to CORE's own types: **type (class) names MUST be UpperCamelCase (PascalCase)**, distinct from predicate names, which MUST be camelCase (§8.3). The rule is now stated in §9.2. CORE's own five types are renamed accordingly: `source` → `Source`, `entity` → `Entity`, `resource` → `Resource`, `timeline` → `Timeline`, `reference` → `Reference`. Folder names (§6, e.g. `sources/`) and `@id`/citekey values are unaffected — only the `@type`/type-node `@id` string changes case. This is a **breaking change**: [`ARCNET-DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md) and the example graph under [`examples/graph/`](examples/graph/) still wrote lowercase `@type` values for CORE's types at the time this note was written and need a follow-up pass.
>
> Revision note (0.9 → 0.10): fixes a role mix-up between `Resource` and `Reference` introduced in 0.8. `Resource` (§11.4) was wrongly defined as "an external work the graph points to but has not ingested" — that is `Reference`'s role, not `Resource`'s. `Resource` is redefined as its long-standing implementation meaning: an anonymous fragment of *ingested* content that doesn't warrant its own dedicated type, classified by `tags` so a recurring pattern can later be promoted into a proper domain type (§15) — e.g. a core thought, modeled as a tagged `Resource` before a domain profile defines its own `Thought` type. `Resource`'s Requires/Optional change from `ref`/`relevance`/`url`/`authors`/`year`/`doi`/`status`/`isCitedBy`/`notes` to `text`/`tags`/`mentionedIn` (Requires) and `notes` (Optional); those displaced predicates move to `Reference`, which now correctly owns the full "external, not-yet-ingested work" behavior (required `title`/`ref`/`relevance`; optional `url`/`authors`/`year`/`doi`/`status`/`isCitedBy`/`notes`). `mentionedIn` (§10.4) is generalized from an `Entity`-only backlink to a `Resource` one too; `cites`/`isCitedBy` (§10.6) now target `Reference` rather than `Resource`. This is a **breaking change**, on top of the still-open follow-up from 0.9: any graph, domain profile, or the example graph under [`examples/graph/`](examples/graph/) using `Resource` for external, un-ingested works must move that content to `Reference`.
>
> Revision note (0.10 → 0.11): tightens §6 into a machine-checkable rule. Folder names were plural, lowercase, and sometimes unrelated to the type they held (`sources/` for `Source`, `_schema/types/` for `Class`), forcing every consumer to carry its own pluralization/case-folding table to get from a folder to a `@type` value. A type folder's name MUST now be **character-for-character identical to the type name** (§6), so folder ↔ type is string equality. CORE's folders are renamed accordingly: `sources/` → `Source/`, `entities/` → `Entity/`, `resources/` → `Resource/`, `references/` → `Reference/`, `_schema/predicates/` → `_schema/Property/`, `_schema/types/` → `_schema/Class/`. `_schema/` (a namespace prefix) and `timeline/` (a bucketed index, §11.5) are the two exempt non-type folders. Nothing about `@id`, `@type`, predicates, or merge behavior changes — only where nodes are filed — and because links resolve by basename (§4.2), no edge breaks. It is still a **breaking change** for any tool that hardcodes folder paths, and a re-filing pass for existing graphs. Domain profiles ([`ARCNET-DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md), [`ARCNET-DOMAIN-CORE-THOUGHT.md`](ARCNET-DOMAIN-CORE-THOUGHT.md), [`ARCNET-DOMAIN-INCIDENT.md`](ARCNET-DOMAIN-INCIDENT.md)) still name their type folders in the old style (e.g. `thoughts/`, `aporias/`, `hypothesis/`) and the example graph under [`examples/graph/`](examples/graph/) still uses the old layout; both need a follow-up pass.
>
> Revision note (0.11 → 0.12): fixes a predicate-registry drift accumulated since 0.5 — several predicates were required by a type or used in a worked example without ever being registered under §10, and five places cited a phantom `§10.7` that never existed (§10 ran 10.1–10.6, then jumped straight to 10.8). Fixed by renumbering the Schema predicates section from §10.8 to §10.7, closing the gap, and repointing every dangling `§10.7` citation to wherever the predicate is actually registered (mostly §10.2, one to §10.6). `Entity`'s required `definition` is retired — it duplicated the generic `text` predicate (§10.2), which `Entity` now requires instead. `authors` (plural, never registered) is corrected to the registered singular `author` everywhere it was used (`Source`, `Reference`). `year` on `Reference` is retired in favor of the existing `published` predicate. `notes` (`Entity`, `Resource`), `ref` and `status` (`Reference`'s worked example), `granularity` (`Timeline`'s worked example), and `relevance` (mentioned only in prose, never actually registered or required) are all retired outright — none were `_schema/Property/` nodes and none are reinstated. The §3 `.nt` example's stray `href` triple is corrected to `tags`: `href` names a rendering role (§5), not a predicate, so it must never appear as a triple's predicate position.

This document specifies the **domain-agnostic core** of a knowledge graph stored as plain Markdown: the RDF-aligned data model (§3), the node model built from it (§5), identity, folder layout, edges, the schema mechanism that makes predicates and types first-class graph nodes (§9), the core predicates and core types (`Source`, `Entity`, `Resource`, `Timeline`, `Reference`), citations, merge, version control, and the patch exchange format. It is **tool-agnostic** — it depends on no program, library, or language.

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
"@type": Entity
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
  a "Entity"
  category "independent" ; category "abstract" ; category "occurrent" ; category "script"
  tags "cryptography"
  text "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network."
  tags "cryptographic"
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
4. **Predicates are first class.** Every predicate used anywhere in the graph — front-matter field, body edge, or citation type — is itself a node under `_schema/Property/` (§9.1), declaring that predicate's serialization role (§5) and merge behavior (§9.3) once, for every type that uses it.
5. **Types are first class.** Every `@type` value in use is itself a node under `_schema/Class/` (§9.2), declaring which predicates it requires or permits (§11). CORE's own types (`Source`, `Entity`, `Resource`, `Timeline`, `Reference`) are ordinary instances of this mechanism, not special-cased.
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
  - one `## Predicate` block per `link`-role predicate in use, its bullets written exactly as `edge`-role bullets are (§8.2's list form, `predicate:: [[Target]]`) — the heading groups them for display, it does not change the bullet syntax. When a type's body consists of exactly one `link`-role predicate, the `## ` heading MAY be omitted, since the block is the entire body (e.g. `Timeline`'s `cites`, §11.5).

A node's **type** is named by its `@type` predicate and defined by CORE (§11) or a domain profile: the type's own schema node (§9.2) fixes which predicates it requires or permits.

## 6. Folder Structure

Each type has exactly one folder and each folder holds exactly one type. A type's folder name MUST be **character-for-character identical to the type name** (§9.2) same case, no pluralization, no abbreviation, no synonym (e.g. `Source`). Domain extensions are allowed to create a functional folder that contains heterogenous class combinations. 

Two functional folders are exempt, because neither is a type folder:

- **`_schema/`** — a namespace prefix, not a type. Its children are type folders and MUST follow the rule: `Property/` for predicate nodes (§9.1), `Class/` for type nodes (§9.2).
- **`timeline/`** — an index whose `Timeline` nodes (§11.5) are bucketed by granularity into `yearly/` and `monthly/` subfolders rather than filed flat. It is the one type whose nodes do not sit directly in a type folder, so the flat-folder rule does not apply.

Folders remain a filing convenience: because links resolve by basename (§4.2), a node may be re-filed without breaking edges, and its classification lives in predicates (§4.7), not in the folder. A consumer MUST read `@type` from the node, never infer it from the folder — the naming rule makes the folder a reliable *mirror* of the type, not a substitute for it (§4.7).

```
graph/
├── Source/               # Source nodes (one per ingested document)
├── Entity/               # Entity nodes (Sowa-typed)
├── Resource/             # Resource nodes (anonymous ingested content)
├── Reference/            # Reference nodes (external, not-yet-ingested works)
├── timeline/             # production-date index — Timeline nodes, bucketed
│   ├── yearly/           #   <YYYY>.md
│   └── monthly/          #   <YYYY-MM>.md
├── _schema/              # schema nodes + controlled vocabularies
│   ├── Property/         #   one Property node per predicate (§9.1)
│   ├── Class/            #   one Class node per type (§9.2)
│   └── aliases.md        #   entity alias table (§7.4) — not a node
└── …                     # domain-profile type folders, likewise type-named
```

`_schema/Property/` and `_schema/Class/` hold real nodes — `Property` and `Class` instances, governed by the same rules as any other node (§4.1–§4.2) — describing the graph's own vocabulary rather than its content. `_schema/aliases.md` is a plain aggregate file, not a node, and so carries no type folder.

## 7. Identity and Naming

### 7.1 General

A node's identity is its `@id`, equal to its basename, and MUST be unique across the whole graph. Basenames for title-identified types (entities, resources, and profile types) SHOULD be human-readable, title-cased, with spaces (e.g. `Forward Secrecy`). Colliding basenames MUST be disambiguated with a parenthetical qualifier (e.g. `Handshake Protocol (TLS)`).

The identity MUST NOT contains `/ \ : * ? " < > | .`

### 7.2 Source citekey

A `Source` node's `@id` is a citekey derived from the document's own metadata:

```
<first-author-surname>-<publication-year>-<slug-keyword>
```

lowercased, ASCII, hyphen-separated (e.g. `rescorla-2026-tls13`). `surname` is the first author's surname (`anon` if unknown); `year` is the publication year; `slug-keyword` is one or two salient title words. The same document always yields the same citekey. The document's own title is a distinct predicate (`title`, §10.2) — the citekey is not a substitute for it.

### 7.3 Title identity

For types identified by a name or claim (entities, resources, and domain types), the `@id` is the node's title — a concise human label (≤ ~6 words for claim-like types), phrased so the same subject or claim yields the same title. Unlike `Source`, these types carry no separate `title` predicate: `@id` already serves as both identity and display name.

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

Predicate names MUST be **camelCase**. Every predicate in use MUST be registered as a node under `_schema/Property/` (§9.1), aligned to a standard vocabulary term where one exists; otherwise it is graph-native (`arc:`). Producers MUST reuse a registered predicate before introducing a new one. Type (class) names follow the complementary rule, **UpperCamelCase** (§9.2).

## 9. Schema

Predicates and types are ordinary graph nodes: the mechanism CORE and every profile use to declare their own vocabulary is itself part of the graph, not external prose.

### 9.1 Predicate nodes

Every predicate in use — front-matter field, body edge, citation type — MUST be registered as a node at `_schema/Property/<name>.md`.

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

Every `@type` value in use MUST be registered as a node at `_schema/Class/<name>.md`, declaring the predicates it requires and permits.

Type names MUST be **UpperCamelCase** (PascalCase) — e.g. `Source`, `Entity`, `Reference` — the mirror image of the camelCase rule for predicate names (§8.3): predicates start lowercase, types start uppercase, so the two vocabularies are visually distinguishable wherever they appear, including bare in prose. The name chosen here is also the type's folder name verbatim (§6), so it is fixed in three places at once — the `@type` value, the schema node's basename, and the folder.

**Front-matter**
- `@id` (mandatory) — the type's PascalCase name, equal to the basename
- `@type` (mandatory) — the literal `Class`

**Body**
- `description` (mandatory) — a definition of the type (`description`, §10.7)
- `## Requires` (mandatory if the type has any type-specific mandatory predicate) — `required:: [[predicate]]` bullets a conforming instance MUST carry (`required`, §10.7)
- `## Optional` (optional) — `optional:: [[predicate]]` bullets a conforming instance MAY carry (`optional`, §10.7)

Every type implicitly requires `@id` and `@type` (§10.1); these are never repeated in a type's own `## Requires` list.

```markdown
---
"@id": Entity
"@type": Class
---
# Entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.2).

## Requires
- required:: [[category]]
- required:: [[text]]
- required:: [[mentionedIn]]

## Optional
- optional:: [[aliases]]
- optional:: [[tags]]
```

### 9.3 Merge vocabulary

When ingesting content contributes to a node whose `@id` already exists, each predicate on that contribution is merged per its own declared `merge` value (§9.1). **All merges MUST be commutative and idempotent**, so replay, reordering, and rollback are well-defined (§13). A predicate declares exactly one of:

- **`immutable`** — single-valued and permanently fixed by the first accepted write. Later writes cannot change it; divergent later contributions are ignored or rejected according to the conflict policy. Used by identity predicates (`@id`, `@type`) and facts fixed at production time (e.g. `published`).
- **`union`** — merge contributions by set union. Values are deduplicated according to the predicate's equality semantics. 
- **`firstWriteWin`** — single-valued. The first accepted write establishes the value. Later divergent contributions do not overwrite it and produce `needsReview`.
- **`fillIfEmpty`** — single-valued and initially absent. The first write fills the value; once present, later contributions have no effect and do not produce conflicts.
- **`lastWriteWin`** — single-valued mutable state. Each accepted write replaces the current value.
- **`append`** — accumulate contributions without removing existing content. For keyed lists, add only previously unseen keys while preserving order; for `text` roles, append new prose according to the predicate's concatenation rules.

A type-level "this node never changes after creation" behavior (e.g. `Source`, owned by a single producer) is not a separate merge value — it emerges from every one of that type's predicates independently being `immutable`, reinforced procedurally by the ingestion idempotency check (§13.2).

A domain profile MUST declare each of its own predicates' merge behavior from this menu (§14).

## 10. Core Predicates

The predicates below are CORE's own `_schema/Property/` nodes — every core type (§11) is built from this vocabulary. A domain profile MAY adds the predicates its own types require (§14), following the same node format (§9.1).

### 10.1 Identity (JSON-LD core)

CORE inherits two predicates from JSON-LD, mandatory on every node.

#### `@id`
**role:** `meta` · **merge:** `immutable`

Identifies the node; equal to the basename (§7).

#### `@type`
**role:** `meta` · **merge:** `immutable`

Names the node's class — CORE's (§11) or a profile's.

#### `aliases`
**role:** `meta` · **merge:** `union`

Alternative indentities (`skos:altLabel`, §7.4).


### 10.2 Content predicates

#### `tags`
**role:** `meta` · **merge:** `union`

Topical tags for discoverability.

#### `text`
**role:** `text` · **aligned:** `schema:text` · **merge:** `append`

Generic prose predicate. Each contribution appends to the existing prose rather than overwriting it, since separate documents may each add relevant text about the same subject over time. A type MAY instead declare its own, more specific text predicate (e.g. `abstract`, §10.2) when a precise name aids reading and a single, first-fixed value is wanted instead.

#### `title`
**role:** `meta` · **aligned:** `schema:title` · **merge:** `immutable`

The title of document of creative work as originally published. (e.g. full article title for `Source` or `Reference`).

#### `abstract`
**role:** `text` · **aligned:** `schema:abstract` · **merge:** `append`

An abstract is a short description that summarizes a creative work.

#### `author`
**role:** `meta` · **aligned:** `schema:author` · **merge:** `union`

The author of the content.

#### `url`
**role:** `meta` · **aligned:** `schema:url` · **merge:** `fillIfEmpty`

Canonical location of the document/work.

#### `doi`
**role:** `meta` · **aligned:** `schema:doi` · **merge:** `fillIfEmpty`

Digital object identifier.

#### `category`
**role:** `meta` · **merge:** `immutable`

Records John F. Sowa's top-level category:
- Level 1: independent · relative · mediating
- Level 2: physical · abstract
- Level 3: continuant · occurrent
- Level 4 (Leaf): object · process · schema · script · juncture · participation · description · history · structure · situation · reason · purpose

The `category` predicate MUST contain the four decoded words. Below is an allowed combinations that follows John F. Sowa taxonomy 
- `[independent, physical, continuant, object]`
- `[independent, physical, occurrent, process]`
- `[independent, abstract, continuant, schema]`
- `[independent, abstract, occurrent, script]`
- `[relative, physical, continuant, juncture]`
- `[relative, physical, occurrent, participation]`
- `[relative, abstract, continuant, description]`
- `[relative, abstract, occurrent, history]`
- `[mediating, physical, continuant, structure]`
- `[mediating, physical, occurrent, situation]`
- `[mediating, abstract, continuant, reason]`
- `[mediating, abstract, occurrent, purpose]`

#### `about`
**role:** `meta` · **merge:** `union`

The subject matter of an node: `technique`/`theory`/`platform`/`system`/`technology`/`language`/`framework`/`field`.

#### `genre`
**role:** `meta` · **merge:** `union`

Genre of the node: `paper`/`standard`/`tool`/`dataset`/`post`.


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
**role:** `link` · **merge:** `union` · **aligned:** `schema:mentions` · **from → to:** Source → Entity

Asserts that the source document mentions the entity; recorded under the source's own `## Mentions` block.

#### `mentionedIn`
**role:** `link` · **merge:** `union` · **aligned:** `schema:subjectOf` · **from → to:** Entity/Resource → Source

The inverse of `mentions` when carried by an `Entity`; more generally, any derived node's backlink to the source it was drawn from (§11.4's `Resource` uses it this way too) — recorded under the node's own `## Mentioned In` block.

### 10.5 Semantic predicates (entity ↔ entity / reference)

Semantic predicates relate one **entity** to another entity or reference. They are written as `edge`-role predicates in the entity body and assert how two subjects relate **in the world**, independent of any document. Choose the **most specific** predicate that holds; fall back to `related` only when none of the others fit. Inverses are optional backlinks — assert the direction natural to the node you are writing and let tooling derive the rest.

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

**Standard adherence.** `X conformsTo:: [[Y]]` asserts X complies with a named specification or schema Y (typically a reference). *e.g.* `Transport Layer Security` → `conformsTo:: [[RFC 8446]]`.

#### `related`
**role:** `edge` · **merge:** `union` · **aligned:** `skos:related`

**Associative link.** A non-hierarchical, non-compositional association between two connected subjects where none of the above applies. Last resort; prefer a specific predicate whenever one fits.

#### `referencedBy`
**role:** `edge` · **merge:** `union`

**Associative link.** A non-hierarchical, non-compositional asymmetric association when the object's own node doesn't explicitly link the subject back.

### 10.6 Citation predicates

A citation is a higher-order predicate: it does not assert a fact about the world, it asserts that a statement in the citing node is backed by an external work and qualifies how the work is used (§12). Used inline, at the point of the statement they support. Citation types SHOULD be drawn from the Citation Typing Ontology (`cito:`); a producer MUST select the most specific type that holds.

#### `cites`
**role:** `edge` · **merge:** `union` · **aligned:** `cito:cites` / `schema:citation` · **from → to:** Source → Reference

The general-purpose citation type; also the source's own structural link to a cited reference, recorded under its `## Cites` block.

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
**role:** `link` · **merge:** `union` · **aligned:** `cito:isCitedBy` · **from → to:** Reference → node

The inverse of any citation predicate — recorded as a backlink under the cited node's own `## isCitedBy` block.


### 10.7 Schema predicates

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
**role:** `text` · **merge:** `append`

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

### 11.2 `Source`

A node for one ingested document. The origin of all other nodes is derived from (**Identity:** citekey (§7.2).). The node exists becuase of the source. 

```markdown
---
"@id": Source
"@type": Class
---
# Source

A node for one ingested document.

## Requires
- required:: [[title]]
- required:: [[published]]
- required:: [[abstract]]
- required:: [[mentions]]

## Optional
- optional:: [[author]]
- optional:: [[url]]
- optional:: [[cites]]
- optional:: [[tags]]
- optional:: [[doi]]
```

A domain profile MAY add navigation blocks linking the document to its own derived types (e.g. `## Proposes`, `## Raises`).

```markdown
---
"@id": rescorla-2026-tls13
"@type": Source
title: "TLS 1.3: Design and Rationale"
author: [Eric Rescorla]
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
- cites:: [[rescorla-2018-rfc8446]]
```

### 11.3 `Entity`

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.2). **Identity:** `@id` + alias table (§7.3, §7.4).

```markdown
---
"@id": Entity
"@type": Class
---
# Entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.2).

## Requires
- required:: [[category]]
- required:: [[text]]
- required:: [[mentionedIn]]

## Optional
- optional:: [[aliases]]
- optional:: [[tags]]
- any §10.5 semantic predicate, as applicable
```

```markdown
---
"@id": Transport Layer Security
"@type": Entity
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

### 11.4 `Resource`

A node for a fragment of an ingested document's content that is relevant to the graph but does not warrant its own dedicated type. It is an anonymous, tag-classified catch-all that lets a domain's type vocabulary grow incrementally. For example, before a domain profile defines its own `Thought` type, an extracted core thought can be captured as a `Resource` tagged `tags: [thoughts]`; once that pattern stabilizes, the tag identifies exactly the set of nodes to promote into a proper type (§15). Distinct from an `Entity` (a named subject with identity and semantics of its own, §11.3) and a `Reference` (an external work the graph has not ingested, §11.6): a `Resource` is itself ingested content, filed and given provenance the same way an `Entity` is. **Identity:** `@id` (§7.3).

```markdown
---
"@id": Resource
"@type": Class
---
# Resource

A fragment of an ingested document's content that is relevant to the graph but does not warrant its own dedicated type.

## Requires
- required:: [[text]]
- required:: [[tags]]
- required:: [[mentionedIn]]
```

```markdown
---
"@id": Sessions Are Ingestable Documents
"@type": Resource
tags: [thoughts]
---
# Sessions Are Ingestable Documents

A work session and an ingested document share the same shape: both produce durable subjects and citable claims, so both can be captured by the same patch format.

## mentionedIn
- mentionedIn:: [[rescorla-2026-tls13]]
```

### 11.5 `Timeline`

A production-date index of ingested documents (not a change log — that is git, §13). **Identity:** the period code (§7.1), `YYYY` or `YYYY-MM`.

```markdown
---
"@id": Timeline
"@type": Class
---
# Timeline

A production-date index of ingested documents.

## Requires
- required:: [[cites]]
```

```markdown
---
"@id": 2026-04
"@type": Timeline
---
# April 2026

- cites:: [[rescorla-2026-tls13]] — *TLS 1.3: Design and Rationale* (Eric Rescorla) — 2026-04-12
- cites:: [[chen-2026-pqkex]] — *Post-Quantum Key Exchange in Practice* (Lin Chen) — 2026-04-28
```

### 11.6 `Reference`

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research — never ingested, unlike `Source` (§11.2), and unlike `Resource` (§11.4) never itself content drawn out of a document: it records only enough to identify, locate, and justify keeping the pointer. **Identity** is the document name established by authors.  


```markdown
---
"@id": Reference
"@type": Class
---
# Reference

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research.

## Requires
- required:: [[title]]

## Optional
- optional:: [[url]]
- optional:: [[author]]
- optional:: [[published]]
- optional:: [[doi]]
- optional:: [[isCitedBy]]
```

```markdown
---
"@id": The Transport Layer Security (TLS) Protocol Version 1.3
"@type": Reference
title: "The Transport Layer Security (TLS) Protocol Version 1.3"
author: [Eric Rescorla]
published: 2018
url: https://www.rfc-editor.org/rfc/rfc8446
---
# The Transport Layer Security (TLS) Protocol Version 1.3

The normative specification of TLS 1.3.

## isCitedBy
- isCitedBy:: [[rescorla-2026-tls13]]
```

## 12. Citations

A **citation is a higher-order predicate**: it does not assert a fact about the world, it asserts that a statement in this node is backed by an external work and qualifies how the work is used. Citations are defined as `Reference` nodes in the graph.

## 13. Version Control

Git **MUST** be used as the version-control system. No other system is used. Git is the authoritative record of how the graph changed; the format stores no change log.

### 13.1 One document, one commit

Ingesting a document is exactly one commit containing its entire contribution: the new `Source` node, any new nodes, merged predicates on existing nodes (§9.3), and the timeline entries for the document's `published` period.

### 13.2 Operations

- **Ingest** — `git add -A` then `git commit -F <msg>` (§13.3)
- **Locate commit** — `git log --grep=<id>`
- **Node history** — `git log --follow -- '<path-to-node>'`
- **Idempotency** — before ingesting, check `git ls-files --error-unmatch Source/<id>.md` — skip if present

The **retract** via `git revert <commit>` is only applicable to the latest commit. The retraction of the Source in the middle of change history requires analysis of node changes and it is allowed only if YAML metadata has not been created by this Source. If node has been merged from multiple sources git revert can remove YAML forntmatter that cause corruption of the node. 


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

A **patch** is a single Markdown file that serializes one document's entire contribution. A tool produces a patch from a document (an alternative output mode to writing the graph directly); a patch is shareable and is applied to any graph by a tool. A patch is a **parallel exchange serialization, not part of the graph**: never indexed as a node, never stored under `graph/`.

### 14.1 Properties

- A patch always carries **full nodes**, never deltas; reconciliation is performed at apply time by each predicate's merge behavior (§9.3).
- A patch renders as ordinary Markdown anywhere. Wiki-links are kept for fidelity; navigating them inside a patch requires patch-aware tooling and is not expected in general viewers.

### 14.2 Structure

A patch's Markdown consists of (i) a YAML front-matter **manifest**, carrying metadata about the patch itself, and (ii) a **graph body**, a Markdown document containing the nodes.

The exchange format does not carry index types (`Timeline`); the apply tool derives them from the source's metadata.

#### 14.2.1 Manifest

**Mandatory:** `@type: patch`; `document` (the source id); `published`.

**Recommended:** `title`; `stats`. These `title`/`published` are a convenience copy of the `Source` node's own predicates, for previewing the patch without opening its body. Per §14.1 every node section, including the `Source` node's, MUST still carry its own predicates in full.

#### 14.2.2 Graph body

All graph nodes are grouped by type under an H1, each node its own H2 subsection.

**H1 = node's `@type`** is `# <Type>` (case-insensitive); one per type present.
**H2 = node's `@id`** is `## <basename>`; one per node. `@type` comes from the H1 and `@id` from the H2; neither is repeated below.

**Under each H2:** a fenced ` ```yaml ` block with the node's remaining predicates with role `meta` (i.e. every `meta`-role predicate other than `@id`/`@type`, which the H1/H2 already supply — including any predicate also mirrored in the manifest); then the node body:

- The node's `text`-role predicate renders as a plain paragraph, unlabeled — a paragraph with no bold label prefix is always the `text`-role predicate (§5).
- Every `link`-role predicate in use renders as its own block: a bold label (the predicate's `label`, §10.7, or its name capitalized) followed by its `predicate:: [[Target]]` bullets — the bold-label equivalent of the graph's `## Predicate` block (§5), since headings are reserved for type and identity here.
- `edge`-role predicates render as `predicate:: [[Target]]` bullets, one heterogeneous block per node; a bold label MAY be added where it aids reading (§5), but is not required per bullet.

Markdown headings are reserved for type and identity; node bodies use bold labels, never headings.

```markdown
## SSL Protocol

\`\`\`yaml
category: [independent, abstract, occurrent, script]
aliases: [SSL, Secure Sockets Layer]
tags: [cryptography, protocols, legacy]
\`\`\`

The predecessor secure-channel protocol that TLS replaced.

- broader:: [[Handshake Protocol]]

**Mentioned in**
- mentionedIn:: [[rescorla-2026-tls13]]
```

### 14.3 Apply

A tool applies a patch to a target graph:

1. For each H2 node under each H1 type, reconstruct the node — `@type` from the H1, `meta`-role predicates from the ` ```yaml ` block, body (prose + `::` edges) as written.
2. If the node's `@id` does not exist in the target graph, create the file: the yaml block plus `@id`/`@type` become the node's front-matter, the body is written verbatim.
3. If it exists, apply each predicate's §9.3 merge behavior (union edges/aliases/sources; first-writer scalar fields, etc.).
4. Update the timeline (§10.4, `cites`) from the source's `published` date.
5. Commit per §13 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph (step 3's unions add nothing new on the second application).

## 15. Domain Extension

A conforming domain profile (`DOMAIN-<name>.md`) MUST define:

1. **Types** — which CORE types (§11) it adopts, and each domain-specific type's folder (named identically to the type, §6), schema node (§9.2, with an inline example), identity (§7), and the merge behavior of its own predicates (§9.3).
2. **Class vocabularies** — any class values its types carry.
3. **Predicates** — the registered, camelCase, standards-aligned predicates its types use (§8.3), each as its own `_schema/Property/` node (§9.1), beyond the core vocabulary (§10).
4. **Provenance model** — which types are derived and what they link back to (§4.6).
5. **A worked example** demonstrating every type, predicate, and the folder layout.

A profile MUST NOT redefine CORE mechanism (identity, edges, citations, the schema/merge framework, version control, patch format).

## 16. Core Conformance Checklist

- [ ] Every file is one node with valid YAML front-matter and mandatory `@id`/`@type` (§10.1).
- [ ] Every basename equals its node's `@id`, is unique and human-readable; every `[[link]]` resolves to a basename (§7).
- [ ] Every `Source`'s `@id` is a citekey equal to its basename (§7.2).
- [ ] Every `Entity` has a four-word decoded Sowa `category` (§10.2).
- [ ] Every derived node links to its source(s) (§4.6).
- [ ] Every predicate is camelCase and registered as a `_schema/Property/` node (§8.3, §9.1).
- [ ] Every `@type` in use is UpperCamelCase and registered as a `_schema/Class/` node declaring its Requires/Optional predicates (§9.2).
- [ ] Every type folder's name is character-for-character identical to its type name (§6).
- [ ] Every citation uses a registered citation-type predicate in the body (§12).
- [ ] Each predicate's merge behavior is one of the §9.3 menu and is commutative/idempotent.
- [ ] Each document was ingested as exactly one git commit (§13).
