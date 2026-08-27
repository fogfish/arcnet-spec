---
"@type": patch
document: ARCNET-CORE.md
title: "CORE Schema Bootstrap"
published: 2026-08-23
stats: { nodes: 48, edges: 24 }
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

Generic prose predicate. Each contribution appends to the existing prose rather than overwriting it, since separate documents may each add relevant text about the same subject over time. A type MAY instead declare its own, more specific text predicate (e.g. `abstract`, §10.2) when a precise name aids reading and a single, first-fixed value is wanted instead.

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
label: Mentions
```

Asserts that the source document mentions the entity; recorded under the source's own `## Mentions` block.

## mentionedIn

```yaml
role: link
merge: union
aligned: "schema:subjectOf"
label: Mentioned In
```

The inverse of `mentions` when carried by an `Entity`; more generally, any derived node's backlink to the source it was drawn from (`Resource` uses it this way too) — recorded under the node's own `## Mentioned In` block.

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

**Standard adherence.** `X conformsTo:: [[Y]]` asserts X complies with a named specification or schema Y (typically a reference). *e.g.* `Transport Layer Security` → `conformsTo:: [[RFC 8446]]`.

## related

```yaml
role: edge
merge: union
aligned: "skos:related"
```

**Associative link.** A non-hierarchical, non-compositional association between two connected subjects where none of the above applies. Last resort; prefer a specific predicate whenever one fits.

## referencedBy

```yaml
role: edge
merge: union
```

**Associative link.** A non-hierarchical, non-compositional asymmetric association when the object's own node doesn't explicitly link the subject back.

## cites

```yaml
role: link
merge: union
aligned: "cito:cites / schema:citation"
```

The general-purpose citation type; also the source's own structural link to a cited reference, recorded under its `## Cites` block.

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
aligned: "schema:title"
```

The title of document or creative work as originally published (e.g. full article title for `Source` or `Reference`).

## abstract

```yaml
role: text
merge: firstWriteWin
aligned: "schema:abstract"
```

An abstract is a short description that summarizes a creative work.

## author

```yaml
role: meta
merge: union
aligned: "schema:author"
```

The author of the content.

## url

```yaml
role: meta
merge: fillIfEmpty
aligned: "schema:url"
```

Canonical location of the document/work.

## doi

```yaml
role: meta
merge: fillIfEmpty
aligned: "schema:doi"
```

Digital object identifier.

## category

```yaml
role: meta
merge: firstWriteWin
```

Records John F. Sowa's top-level category:
- Level 1: independent · relative · mediating
- Level 2: physical · abstract
- Level 3: continuant · occurrent
- Level 4 (Leaf): object · process · schema · script · juncture · participation · description · history · structure · situation · reason · purpose

The `category` predicate MUST contain the four decoded words. Allowed combinations, following John F. Sowa's taxonomy:
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

## about

```yaml
role: meta
merge: union
```

The subject matter of a node: `technique`/`theory`/`platform`/`system`/`technology`/`language`/`framework`/`field`.

## genre

```yaml
role: meta
merge: union
```

Genre of the node: `paper`/`standard`/`tool`/`dataset`/`post`.

## aliases

```yaml
role: meta
merge: union
```

Alternative identities (`skos:altLabel`, §7.4).

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

## Source

A node for one ingested document — the provenance origin other nodes derive from.

**Requires**
- required:: [[title]]
- required:: [[published]]
- required:: [[abstract]]
- required:: [[mentions]]

**Optional**
- optional:: [[author]]
- optional:: [[url]]
- optional:: [[cites]]
- optional:: [[tags]]
- optional:: [[doi]]

## Entity

A node for a subject occurring in sources, typed by Sowa category (`category`, §10.2).

**Requires**
- required:: [[category]]
- required:: [[text]]
- required:: [[mentionedIn]]

**Optional**
- optional:: [[aliases]]
- optional:: [[tags]]
- any §10.5 semantic predicate, as applicable

## Resource

A fragment of an ingested document's content that is relevant to the graph but does not warrant its own dedicated type.

**Requires**
- required:: [[text]]
- required:: [[tags]]
- required:: [[mentionedIn]]

## Timeline

A production-date index of ingested documents.

**Requires**
- required:: [[cites]]

## Reference

A node for an external work the graph points to but has not ingested, or a topic/area tracked for reading or research.

**Requires**
- required:: [[title]]

**Optional**
- optional:: [[url]]
- optional:: [[author]]
- optional:: [[published]]
- optional:: [[doi]]
- optional:: [[isCitedBy]]
