---
"@type": patch
document: ARCNET-CORE.md
title: "CORE Schema Bootstrap"
published: 2026-07-07
stats: { nodes: 53, edges: 27 }
---

# Property

## tags

```yaml
role: meta
merge: union
```

Topical tags for discoverability.

## text

```yaml
role: text
merge: append
aligned: "schema:text"
```

Generic prose predicate. Each contribution appends to the existing prose rather than overwriting it, since separate documents may each add relevant text about the same subject over time. A type MAY instead declare its own, more specific text predicate (e.g. `abstract`, `definition`, `relevance`, §10.7) when a precise name aids reading and a single, first-fixed value is wanted instead.

## published

```yaml
role: meta
merge: immutable
```

ISO-8601 production date of the document a node derives from; drives the timeline (§11.5).

## created

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp the node was created in the graph.

## updated

```yaml
role: meta
merge: lastWriteWin
```

ISO-8601 timestamp of the node's last modification.

## mentions

```yaml
role: link
merge: union
aligned: "schema:mentions"
```

Asserts that the source document mentions the entity; recorded under the source's own `## Mentions` block.

## mentionedIn

```yaml
role: link
merge: union
aligned: "schema:subjectOf"
```

The inverse of `mentions` — recorded as a backlink under the entity's own `## mentionedIn` block.

## broader

```yaml
role: edge
merge: union
aligned: "skos:broader"
```

**Generalization.** `X broader:: [[Y]]` asserts Y is the more general concept, X a kind or specialization of it. Concept hierarchy, not composition. *e.g.* `Mutual TLS` → `broader:: [[Transport Layer Security]]`.

## narrower

```yaml
role: edge
merge: union
aligned: "skos:narrower"
```

The inverse of `broader` — an optional backlink from the more general concept to the specialization.

## isPartOf

```yaml
role: edge
merge: union
aligned: "dcterms:isPartOf"
```

**Composition (part–whole).** `X isPartOf:: [[Y]]` asserts X is a component or member of the whole Y. Mereology, not "is a kind of" (that is `broader`). *e.g.* `Certificate Transparency` → `isPartOf:: [[Audit Log]]`.

## hasPart

```yaml
role: edge
merge: union
aligned: "schema:hasPart"
```

The inverse of `isPartOf` — an optional backlink from the whole to a component.

## requires

```yaml
role: edge
merge: union
aligned: "dcterms:requires"
```

**Functional dependency.** `X requires:: [[Y]]` asserts X needs Y to function, hold, or be delivered. Use for prerequisites, not membership or kinds. *e.g.* `Forward Secrecy` → `requires:: [[Handshake Protocol]]`.

## replaces

```yaml
role: edge
merge: union
aligned: "dcterms:replaces"
```

**Supersession over time.** `X replaces:: [[Y]]` asserts X supplants an older Y (Y obsolete in favour of X). Use for versions and standards that succeed one another. *e.g.* `Transport Layer Security` → `replaces:: [[SSL Protocol]]`.

## isReplacedBy

```yaml
role: edge
merge: union
aligned: "dcterms:isReplacedBy"
```

The inverse of `replaces` — an optional backlink from the superseded subject to its successor.

## conformsTo

```yaml
role: edge
merge: union
aligned: "dcterms:conformsTo"
```

**Standard adherence.** `X conformsTo:: [[Y]]` asserts X complies with a named specification or schema Y (typically a resource). *e.g.* `Transport Layer Security` → `conformsTo:: [[RFC 8446]]`.

## related

```yaml
role: edge
merge: union
aligned: "skos:related"
```

**Associative link.** A non-hierarchical, non-compositional association between two connected subjects where none of the above applies. Last resort; prefer a specific predicate whenever one fits.

## cites

```yaml
role: link
merge: union
aligned: "cito:cites / schema:citation"
```

The general-purpose citation type; also the source's own structural link to a cited resource, recorded under its `## Cites` block.

## citesAsEvidence

```yaml
role: edge
merge: union
aligned: "cito:citesAsEvidence"
```

Cites the target as evidence for the citing statement.

## citesAsAuthority

```yaml
role: edge
merge: union
aligned: "cito:citesAsAuthority"
```

Cites the target as an authoritative source for the citing statement.

## supports

```yaml
role: edge
merge: union
aligned: "cito:supports"
```

The citing statement is supported by the target.

## confirms

```yaml
role: edge
merge: union
aligned: "cito:confirms"
```

The citing statement confirms findings in the target.

## extends

```yaml
role: edge
merge: union
aligned: "cito:extends"
```

The citing statement extends work in the target.

## critiques

```yaml
role: edge
merge: union
aligned: "cito:critiques"
```

The citing statement critiques the target.

## disputes

```yaml
role: edge
merge: union
aligned: "cito:disputes"
```

The citing statement disputes claims in the target.

## refutes

```yaml
role: edge
merge: union
aligned: "cito:refutes"
```

The citing statement refutes claims in the target.

## isCitedBy

```yaml
role: link
merge: union
aligned: "cito:isCitedBy"
```

The inverse of any citation predicate — recorded as a backlink under the cited node's own `## isCitedBy` block.

## title

```yaml
role: meta
merge: immutable
```

The document title as published — distinct from `@id` when `@id` is a derived citekey (§7.2).

## abstract

```yaml
role: text
merge: firstWriteWin
```

A short prose summary of the document.

## authors

```yaml
role: meta
merge: union
```

Ordered list of author names.

## url

```yaml
role: meta
merge: fillIfEmpty
```

Canonical location of the document/work.

## doi

```yaml
role: meta
merge: fillIfEmpty
```

Digital object identifier.

## category

```yaml
role: meta
merge: firstWriteWin
```

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

## aliases

```yaml
role: meta
merge: union
```

Alternative names (`skos:altLabel`, §7.4).

## definition

```yaml
role: text
merge: firstWriteWin
```

A 1–3 sentence definition of the subject.

## notes

```yaml
role: text
merge: firstWriteWin
```

Additional prose.

## ref

```yaml
role: meta
merge: immutable
```

Resource type: a citable work (`paper`/`standard`/`tool`/`dataset`/`post`) or a topic/area (`technique`/`theory`/`platform`/`system`/`technology`/`language`/`framework`/`field`); a profile MAY extend the set.

## year

```yaml
role: meta
merge: fillIfEmpty
```

Year of publication.

## status

```yaml
role: meta
merge: lastWriteWin
```

`read` or `backlog` — a `backlog` resource is a research target.

## relevance

```yaml
role: text
merge: firstWriteWin
```

A 1–2 sentence note on why the resource matters.

## granularity

```yaml
role: meta
merge: immutable
```

`yearly` or `monthly`.

## entries

```yaml
role: link
merge: append
```

The `source` nodes whose `published` date falls in this period, ordered by date.

## heading

```yaml
role: meta
merge: firstWriteWin
```

A human-readable title for the period, shown as H1 in place of the bare `@id` (period code).

## role

```yaml
role: meta
merge: immutable
```

One of `meta`/`text`/`href`/`edge`/`link` (§5): the predicate's serialization position.

## merge

```yaml
role: meta
merge: immutable
```

One of the merge behaviors (§9.3): how contributions to this predicate combine.

## label

```yaml
role: meta
merge: firstWriteWin
```

Human-readable title shown as a `link`-role predicate's `## ` heading; defaults to the predicate name, capitalized.

## aligned

```yaml
role: meta
merge: firstWriteWin
```

The standard-vocabulary term this predicate maps to (e.g. `dcterms:isPartOf`), or `arc:<name>` if graph-native.

## description

```yaml
role: text
merge: firstWriteWin
```

Prose describing the predicate's or type's meaning — the body text of a `Property`/`Class` node.

## required

```yaml
role: link
merge: union
label: "Requires"
```

Asserts that the class requires the target predicate on every conforming instance. Recorded under the class's own `## Requires` block.

## optional

```yaml
role: link
merge: union
```

Asserts that the class permits the target predicate. Recorded under the class's own `## Optional` block.

# Class

## source

A node for one ingested document — the provenance origin other nodes derive from.

**Requires**
- required:: [[title]]
- required:: [[published]]
- required:: [[abstract]]
- required:: [[mentions]]

**Optional**
- optional:: [[authors]]
- optional:: [[url]]
- optional:: [[cites]]
- optional:: [[tags]]
- optional:: [[doi]]

## entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.7).

**Requires**
- required:: [[category]]
- required:: [[definition]]
- required:: [[mentionedIn]]

**Optional**
- optional:: [[aliases]]
- optional:: [[tags]]
- optional:: [[notes]]
- any §10.5 semantic predicate, as applicable

## resource

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research.

**Requires**
- required:: [[ref]]
- required:: [[relevance]]

**Optional**
- optional:: [[url]]
- optional:: [[isCitedBy]]
- optional:: [[authors]]
- optional:: [[year]]
- optional:: [[doi]]
- optional:: [[status]]
- optional:: [[notes]]

## timeline

A production-date index of ingested documents.

**Requires**
- required:: [[granularity]]
- required:: [[entries]]

**Optional**
- optional:: [[heading]]
