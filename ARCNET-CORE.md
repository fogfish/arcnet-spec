# CORE — Markdown Knowledge Graph

**Status:** Accepted · **Version:** 0.4 · **Date:** 2026-06-15

This document specifies the **domain-agnostic core** of a knowledge graph stored as plain
Markdown: the node model, identity, folder layout, edges, citations, the core node kinds
(`source`, `entity`, `resource`, `timeline`), the merge framework, version control, and the
patch exchange format. It is **tool-agnostic** — it depends on no program, library, or language.

A **domain profile** (`DOMAIN-<name>.md`) extends this core with additional node kinds and their
predicate vocabulary. Profiles depend on CORE; CORE never references a profile. The reference
profile is [`DOMAIN-ARTICLE.md`](DOMAIN-ARTICLE.md), with a worked example under [`graph/`](graph/).

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in
RFC 2119. Each schema declares its elements at three levels:

- **Mandatory** — a conforming node MUST contain this element.
- **Recommended** — a conforming node SHOULD contain it when the information is available.
- **Optional** — a conforming node MAY contain it.

## 2. Purpose and Scope

A knowledge graph is a set of linked Markdown files. Each file is a node; `[[wiki-links]]`
between files are edges. CORE defines the substrate shared by all domains and the core node
kinds every document-ingestion domain reuses; a domain profile adds domain-specific kinds. CORE
does not define how nodes are produced.

## 3. Design Invariants

A conforming graph MUST uphold these invariants:

1. **One node, one file.** Each node is a single `.md` file with a YAML front-matter header
   and a Markdown body.
2. **Identity is the basename.** A node's identity is its file basename (without `.md`), unique
   across the whole graph (§6). Links resolve by basename, independent of folder; a node MAY be
   moved between folders without affecting any edge.
3. **Edges are wiki-links.** An edge is `[[Target]]`, where `Target` is another node's
   basename. An edge MAY be typed by a predicate (§7).
4. **Derived nodes carry provenance.** A node distilled from a document MUST link to the
   document node(s) it was derived from.
5. **Classification is data, not location.** Any class a node carries lives in front-matter;
   folders MAY mirror it but MUST NOT be its sole carrier.
6. **Append-only growth, defined merges.** Adding content creates new files or merges into
   existing ones via the per-kind merge operations (§10). Merges are commutative and idempotent
   at field level.
7. **Git is the history.** Version, provenance-over-time, and rollback are recorded in git
   (§11). The format stores no change log.

## 4. Node Model

Every node is a YAML front-matter block (delimited by `---`) followed by a Markdown body.
Consumers MUST preserve unknown fields.

- **Front-matter** carries the node's **scalar attributes**, including the mandatory `kind`.
- **Body** carries prose plus two kinds of bullet:
  - an **edge** — `predicate:: [[Target]]` — a relation to another **node** (something with
    independent identity and reuse across the graph);
  - a **literal** — a plain bullet — an atomic statement belonging to exactly **one** node
    and having no independent identity.
- A literal MAY embed an inline `[[link]]` where it names a node, but the statement itself is
  never replaced by the link. Edges and literals serving the same purpose are grouped into
  separate, bold-labelled body blocks.

A **node kind** is named by the `kind` field and defined by CORE (§9) or a domain profile. Each
kind definition fixes the kind's folder, identity (§6), schema (front-matter + body, at the
three levels of §1), and merge operation (§10).

## 5. Folder Structure

Each node kind has one folder, named after the kind; nodes are filed flat within it. Folders are
a filing convenience: because links resolve by basename (§3.2), a node may be re-filed without
breaking edges, and any class a node carries lives in front-matter (§3.5), not in the folder.
Controlled vocabularies live under `_meta/` and are not nodes.

```
graph/
├── sources/             # source nodes (one per ingested document)
├── entities/            # entity nodes (Sowa-typed)
├── resources/           # resource nodes (citations, topics)
├── timeline/            # production-date index
│   ├── yearly/          #   <YYYY>.md
│   └── monthly/         #   <YYYY-MM>.md
├── _meta/               # controlled vocabularies (not nodes)
│   ├── predicates.md    #   predicate registry (§7)
│   └── aliases.md       #   entity alias table (§6.4)
└── …                    # domain-profile kind folders
```

## 6. Identity and Naming

### 6.1 General

A node's identity is its basename and MUST be unique across the whole graph. Basenames for
title-identified kinds (entities, resources, and profile kinds) SHOULD be human-readable,
title-cased, with spaces (e.g. `Forward Secrecy`). Colliding basenames MUST be disambiguated
with a parenthetical qualifier (e.g. `Handshake Protocol (TLS)`).

### 6.2 Source citekey

A `source` node's identity is a citekey derived from the document's own metadata:

```
<first-author-surname>-<publication-year>-<slug-keyword>
```

lowercased, ASCII, hyphen-separated (e.g. `rescorla-2026-tls13`). `surname` is the first
author's surname (`anon` if unknown); `year` is the publication year; `slug-keyword` is one or
two salient title words. The same document always yields the same citekey.

### 6.3 Title identity

For kinds identified by a name or claim (entities, resources, and domain kinds), the basename is the node's title — a concise human label (≤ ~6 words for
claim-like kinds), phrased so the same subject or claim yields the same title.

### 6.4 Aliases

A subject named differently across documents has exactly one **canonical** node. The canonical
basename is the preferred label (`skos:prefLabel`); alternatives are recorded in the node's
`aliases` field (`skos:altLabel`) and in `_meta/aliases.md`. Producers MUST consult the alias
table before creating an entity, so synonyms collapse onto the canonical node.

## 7. Edges and Predicates

### 7.1 Link syntax

An edge is `[[Target]]`, a basename reference; it MAY carry a display alias `[[Target|text]]`. A
bare `[[Target]]` with no predicate is an untyped mention.

### 7.2 Predicate forms

A predicate types an edge. Three forms are permitted; producers SHOULD use the list form for
structural edges:

- **List form:** `- replaces:: [[SSL Protocol]]`
- **Body form:** `conformsTo:: [[RFC 8446]]`
- **Inline form:** `... was [citesAsEvidence:: [[RFC 8446]]] standardized.`

The `::` token separates predicate from target.

### 7.3 Predicate naming and registry

Predicate names MUST be **camelCase**. Every predicate in use MUST be registered in
`_meta/predicates.md`, aligned to a standard vocabulary term where one exists; otherwise it is
graph-native (`arc:`). Producers MUST reuse a registered predicate before introducing a new one.

### 7.4 Core predicate vocabulary

Predicates used by the core kinds. Namespaces: `schema:`, `dcterms:`, `cito:`, `skos:`.

**Structural** (core kinds):

| Predicate     | From → To         | Aligned term     |
| ------------- | ----------------- | ---------------- |
| `mentions`    | source → entity   | schema:mentions  |
| `mentionedIn` | entity → source   | schema:subjectOf |
| `cites`       | source → resource | cito:cites       |
| `isCitedBy`   | resource → node   | cito:isCitedBy   |

**Semantic** predicates relate one **entity** to another entity or resource. They are written as
edges in the entity body (CORE §4) and assert how two subjects relate **in the world**,
independent of any document. Choose the **most specific** predicate that holds; fall back to
`related` only when none of the others fit. Inverses (`narrower`, `hasPart`, `isReplacedBy`) are
optional backlinks — assert the direction natural to the node you are writing and let tooling
derive the rest.

- `broader` / `narrower` (skos:broader / skos:narrower) — **generalization.** `X broader:: [[Y]]`
  asserts that Y is the more general concept and X a kind or specialization of it (is-a,
  type-of). Use for concept hierarchy, not composition.
  *e.g.* `Mutual TLS` → `broader:: [[Transport Layer Security]]`.
- `isPartOf` / `hasPart` (dcterms:isPartOf / schema:hasPart) — **composition (part–whole).**
  `X isPartOf:: [[Y]]` asserts X is a component or member of the whole Y. Use for mereology, not
  for "is a kind of" (that is `broader`).
  *e.g.* `Certificate Transparency` → `isPartOf:: [[Audit Log]]`.
- `requires` (dcterms:requires) — **functional dependency.** `X requires:: [[Y]]` asserts X needs
  Y to function, hold, or be delivered. Use for prerequisites, not membership or kinds.
  *e.g.* `Forward Secrecy` → `requires:: [[Handshake Protocol]]`.
- `replaces` / `isReplacedBy` (dcterms:replaces / dcterms:isReplacedBy) — **supersession over
  time.** `X replaces:: [[Y]]` asserts X supplants an older Y (Y obsolete in favour of X). Use for
  versions and standards that succeed one another.
  *e.g.* `Transport Layer Security` → `replaces:: [[SSL Protocol]]`.
- `conformsTo` (dcterms:conformsTo) — **standard adherence.** `X conformsTo:: [[Y]]` asserts X
  complies with a named specification or schema Y (typically a resource).
  *e.g.* `Transport Layer Security` → `conformsTo:: [[RFC 8446]]`.
- `related` (skos:related) — **associative link.** A non-hierarchical, non-compositional
  association between two connected subjects where none of the above applies. Last resort; prefer
  a specific predicate whenever one fits.

Choosing: ask in order — is it a *kind of* (`broader`), a *part of* (`isPartOf`), a *dependency*
(`requires`), a *successor of* (`replaces`), *conformance to a standard* (`conformsTo`)? Only if
none hold, use `related`.

A domain profile adds the predicates its own kinds require (§13).

## 8. Citations

A **citation is a higher-order predicate**: it does not assert a fact about the world, it
asserts that a statement in this node is backed by an external work and qualifies how the work
is used. Citations are recorded **inline, at the point of the statement they support**; the
graph defines no global bibliography.

- The subject is the citing node (or a specific assertion in its body); the target is a node
  representing an external work; the predicate is a **citation type**.
- Citation types SHOULD be drawn from the Citation Typing Ontology (`cito:`): `cites`,
  `citesAsEvidence`, `citesAsAuthority`, `supports`, `confirms`, `extends`, `critiques`,
  `disputes`, `refutes`, and the inverse `isCitedBy`. A producer MUST select the most specific
  type that holds.
- **Mandatory:** a registered citation-type predicate and a `[[wiki-link]]` target, in the body
  of the citing node. **Recommended:** placement adjacent to the supported statement and
  recording the inverse `isCitedBy` on the target.

## 9. Core Kinds

### 9.1 `source`

A node for one ingested document — the provenance origin other nodes derive from.
**Identity:** citekey (§6.2). **Merge:** none (§10).

**Front-matter**
- `kind` (mandatory) — the literal `source`
- `id` (mandatory) — the citekey, equal to the basename (§6.2)
- `title` (mandatory) — the document title as published
- `published` (mandatory) — production date, ISO-8601; drives the timeline (§9.4)
- `authors` (recommended) — ordered list of author names
- `url` (recommended) — canonical location of the document
- `tags` (optional) — topical tags
- `doi` (optional) — digital object identifier

**Body**
- `abstract` (mandatory) — a short prose summary of the document
- `## Mentions` (recommended) — `mentions::` edges to the entities derived from the document
- `## Cites` (recommended) — `cites::` edges to the resources referenced

A domain profile MAY add navigation blocks linking the document to its own derived kinds (e.g.
`## Proposes`, `## Raises`).

```markdown
---
kind: source
id: rescorla-2026-tls13
title: "TLS 1.3: Design and Rationale"
authors: [Eric Rescorla]
published: 2026-04-12
url: https://example.org/tls13-design
tags: [tls, protocols]
---
# TLS 1.3: Design and Rationale

A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip
resumption.

## Mentions
- mentions:: [[Transport Layer Security]]
- mentions:: [[Forward Secrecy]]

## Cites
- cites:: [[RFC 8446]]
```

### 9.2 `entity`

A node for a subject occurring in sources, typed by Sowa category (§9.2.1).
**Identity:** title + alias table (§6.3, §6.4). **Merge:** union (§10).

**Front-matter**
- `kind` (mandatory) — the literal `entity`
- `title` (mandatory) — the canonical name, equal to the basename (`skos:prefLabel`)
- `category` (mandatory) — the Sowa category as a four-word bag (§9.2.1)
- `aliases` (recommended) — alternative names (`skos:altLabel`)
- `tags` (optional) — topical tags

**Body**
- `definition` (mandatory) — a 1–3 sentence definition of the subject
- `semantic edges` (recommended) — domain/semantic predicate edges (`broader`, `isPartOf`, …)
- `## mentionedIn` (recommended) — `mentionedIn::` links to each source the entity occurs in
- `notes` (optional) — additional prose

```markdown
---
kind: entity
title: Transport Layer Security
category: [independent, abstract, occurrent, script]
aliases: [TLS, TLS 1.3]
tags: [cryptography]
---
# Transport Layer Security

A cryptographic protocol that establishes an authenticated, confidential channel over an
untrusted network.

- replaces:: [[SSL Protocol]]
- conformsTo:: [[RFC 8446]]

## mentionedIn
- mentionedIn:: [[rescorla-2026-tls13]]
```

#### 9.2.1 Category (Sowa)

The `category` records John F. Sowa's top-level category, identified by a three-letter code and
a leaf name (e.g. `ipc:object`), **decoded into a bag of words**. The three letters decode by
position; the leaf name is appended:

| Position | Letter      | Word                                                                                                                             |
| -------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 1        | `i` `r` `m` | independent · relative · mediating                                                                                               |
| 2        | `p` `a`     | physical · abstract                                                                                                              |
| 3        | `c` `o`     | continuant · occurrent                                                                                                           |
| leaf     | —           | object · process · schema · script · juncture · participation · description · history · structure · situation · reason · purpose |

| Code                | `category` bag of words                          |
| ------------------- | ------------------------------------------------ |
| `ipc:object`        | `[independent, physical, continuant, object]`    |
| `ipo:process`       | `[independent, physical, occurrent, process]`    |
| `iac:schema`        | `[independent, abstract, continuant, schema]`    |
| `iao:script`        | `[independent, abstract, occurrent, script]`     |
| `rpc:juncture`      | `[relative, physical, continuant, juncture]`     |
| `rpo:participation` | `[relative, physical, occurrent, participation]` |
| `rac:description`   | `[relative, abstract, continuant, description]`  |
| `rao:history`       | `[relative, abstract, occurrent, history]`       |
| `mpc:structure`     | `[mediating, physical, continuant, structure]`   |
| `mpo:situation`     | `[mediating, physical, occurrent, situation]`    |
| `mac:reason`        | `[mediating, abstract, continuant, reason]`      |
| `mao:purpose`       | `[mediating, abstract, occurrent, purpose]`      |

The `category` field MUST contain the four decoded words. A consumer MAY recompose the code.

### 9.3 `resource`

A node for an external work the graph points to but has not ingested, or a topic/area tracked
for reading or research. Distinct from a `source` (ingested, derived *from*) and an `entity` (a
subject the content is *about*). **Identity:** title (§6.3). **Merge:** union, first-writer (§10).

**Front-matter**
- `kind` (mandatory) — the literal `resource`
- `title` (mandatory) — the work or topic name, equal to the basename
- `ref` (mandatory) — resource type — a citable work (`paper` / `standard` / `tool` / `dataset` / `post`) or a topic/area (`technique` / `theory` / `platform` / `system` / `tool` /`technology` / `language` / `framework` / `field`); a profile MAY extend the set
- `url` (recommended) — canonical location
- `authors` (optional) — ordered list of author names
- `year` (optional) — year of publication
- `doi` (optional) — digital object identifier
- `status` (optional) — `read` or `backlog` (a `backlog` resource is a research target)

**Body**
- `relevance` (recommended) — a 1–2 sentence note on why the resource matters
- `## isCitedBy` (recommended) — `isCitedBy::` backlinks to the citing nodes
- `notes` (optional) — additional prose

```markdown
---
kind: resource
title: RFC 8446
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

### 9.4 `timeline`

A production-date index of ingested documents (not a change log — that is git, §11).
**Identity:** the period code (§6.3). **Merge:** append (§10).

**Front-matter**
- `kind` (mandatory) — the literal `timeline`
- `period` (mandatory) — the period code, equal to the basename: `YYYY` or `YYYY-MM`
- `granularity` (mandatory) — `yearly` or `monthly`

**Body**
- `entries` (mandatory) — the `source` nodes whose `published` date falls in `period`, ordered by date
- `heading` (recommended) — a human-readable title for the period

```markdown
---
kind: timeline
period: 2026-04
granularity: monthly
---
# April 2026

- [[rescorla-2026-tls13]] — *TLS 1.3: Design and Rationale* (Eric Rescorla) — 2026-04-12
- [[chen-2026-pqkex]] — *Post-Quantum Key Exchange in Practice* (Lin Chen) — 2026-04-28
```

## 10. Merge Operations

When ingesting content yields a node whose basename already exists, the contribution is merged
per the node's declared merge operation. **All merges MUST be commutative and idempotent at
field level**, so replay, reordering, and rollback are well-defined. A kind selects one:

- **none** — the node is owned by a single producer; a second production of the same basename is
  a no-op (`source`).
- **union** — set-union the node's edges and multi-valued fields; for scalar fields, the first
  writer wins and a later divergent value is flagged `needsReview` (`entity`). A variant,
  **union, first-writer**, additionally permits filling a previously-empty optional scalar
  (`resource`).
- **append** — insert the contribution into an ordered list, keyed for uniqueness so
  re-insertion is a no-op (`timeline`).
- **validated-overwrite** — multi-valued fields union; designated scalar fields are owned by an
  optional validation pass and overwritten only by it (last-validation-wins). Used by profile
  kinds whose classification is computed after the fact.

A domain profile MUST declare each of its kinds' merge operation from this menu.

## 11. Version Control

Git **MUST** be used as the version-control system. No other system is used. Git is the
authoritative record of how the graph changed; the format stores no change log.

### 11.1 One document, one commit

Ingesting a document is exactly one commit containing its entire contribution: the new `source`
node, any new nodes, merged edges on existing nodes (§10), and the timeline entries for the
document's `published` period.

### 11.2 Operations

| Operation         | Git commands                                                                             |
| ----------------- | ---------------------------------------------------------------------------------------- |
| **Ingest**        | `git add -A` then `git commit -F <msg>` (§11.3)                                          |
| **Retract**       | `git revert <commit>`                                                                    |
| **Locate commit** | `git log --grep=<id>`                                                                    |
| **Node history**  | `git log --follow -- '<path-to-node>'`                                                   |
| **Idempotency**   | before ingesting, check `git ls-files --error-unmatch sources/<id>.md` — skip if present |

Because each contribution is additive (§10), `git revert` resolves shared-node edits with a
standard three-way merge.

### 11.3 Commit message format

```
graph(ingest): <id> — <title>

<one-line summary>

Nodes: +<counts by kind>
Edges: +<n>
Timeline: <YYYY-MM>
Source-Id: <id>
```

- **Mandatory:** subject `graph(ingest): <id> — <title>` (retraction uses `graph(retract):
  <id>`); the `Nodes:` stats line; the `Source-Id:` trailer.
- **Recommended:** `Edges:`, `Timeline:`, and any source URL/date trailers.
- **Optional:** the summary paragraph.

## 12. Document Patch (Exchange Format)

A **patch** is a single Markdown file that serializes one document's entire contribution. A tool
produces a patch from a document (an alternative output mode to writing the graph directly); a
patch is shareable and is applied to any graph by a tool. A patch is a **parallel exchange
serialization, not part of the graph**: never indexed as a node, never stored under `graph/`,
never extracted from an existing graph.

### 12.1 Properties

- A patch always carries **full nodes**, never deltas; reconciliation is performed at apply time
  by the merge operations (§10).
- A patch renders as ordinary Markdown anywhere. Wiki-links are kept for fidelity; navigating
  them inside a patch requires patch-aware tooling and is not expected in general viewers.

### 12.2 Structure

- **Front-matter (manifest)** — the single real front-matter block. **Mandatory:** `kind:
  patch`; `document` (the source id); `published`. **Recommended:** `title`; `stats`.
- **Body — node sections grouped by kind:**
  - **H1 = node kind** — `# <Kind>` (case-insensitive); one per kind present.
  - **H2 = node identity** — `## <basename>`; one per node. `kind` comes from the H1 and
    `title`/`id` from the H2; neither is repeated below.
  - **Under each H2:** a fenced ` ```yaml ` block with the node's remaining scalar attributes;
    then the node body (prose + `::` edges with canonical `[[Node]]` targets).
  - Markdown headings are reserved for kind and identity; node bodies use bold labels, never
    headings.
- Index kinds (`timeline`) are not carried; the apply tool derives them from the source's
  metadata.

### 12.3 Apply


A tool applies a patch to a target graph:

1. For each H2 node under each H1 kind, reconstruct the node — `kind` from the H1, scalar
   fields from the ` ```yaml ` block, body (prose + `::` edges) as written.
2. If the node basename does not exist in the target graph, create the file: the yaml block
   becomes the node's front-matter, the body is written verbatim.
3. If it exists, apply the §10 merge operation for the node's kind (union edges / aliases /
   sources; first-writer scalar fields).
4. Update the timeline ((§10)) from the source's `published` date.
5. Commit per §11 — applying one patch is exactly one commit.

A patch is **idempotent**: applying it twice yields the same graph (step 3 unions add nothing
new on the second application).


## 13. Domain Extension

A conforming domain profile (`DOMAIN-<name>.md`) MUST define:

1. **Node kinds** — which CORE core kinds (§9) it adopts, and each domain-specific kind's folder,
   schema (front-matter + body tables at the three levels of §1, with an inline example),
   identity (§6), and merge operation (§10).
2. **Class vocabularies** — any class values its kinds carry.
3. **Predicate vocabulary** — the registered, camelCase, standards-aligned predicates its kinds
   use (§7), beyond the core vocabulary (§7.4).
4. **Provenance model** — which kinds are derived and what they link back to (§3.4).
5. **A worked example** demonstrating every kind, predicate, and the folder layout.

A profile MUST NOT redefine CORE mechanism (identity, edges, citations, merge framework, version
control, patch format).

## 14. Core Conformance Checklist

- [ ] Every file is one node with valid YAML front-matter and a `kind`.
- [ ] Every basename is unique and human-readable; every `[[link]]` resolves to a basename.
- [ ] Every `source` has a citekey `id` equal to its basename (§6.2).
- [ ] Every `entity` has a four-word decoded Sowa `category` (§9.2.1).
- [ ] Every derived node links to its source(s) (§3.4).
- [ ] Every predicate is camelCase and registered in `_meta/predicates.md` (§7).
- [ ] Every citation uses a registered citation-type predicate in the body (§8).
- [ ] Each kind's merge operation is one of the §10 menu and is commutative/idempotent.
- [ ] Each document was ingested as exactly one git commit (§11).
