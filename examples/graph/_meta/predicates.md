---
kind: meta
title: Predicate Registry
---
# Predicate Registry

The controlled vocabulary of edge predicates used in this graph (mechanism + core vocabulary:
`CORE.md` §7–§8; domain vocabulary: `DOMAIN-ARTICLE.md` §3).
Names are camelCase and aligned to standard vocabularies where a term exists; otherwise they
are graph-native (`kg:`). Producers reuse a registered predicate before introducing a new one.

Namespaces: `schema:` schema.org · `dcterms:` DCMI Terms · `cito:` Citation Typing Ontology ·
`skos:` SKOS · `prov:` PROV-O · `kg:` graph-native.

## Structural predicates

| Predicate        | From → To                    | Aligned term         |
| ---------------- | ---------------------------- | -------------------- |
| `mentions`       | source → entity              | schema:mentions      |
| `mentionedIn`    | entity → source              | schema:subjectOf     |
| `proposes`       | source → hypothesis          | kg:proposes          |
| `raises`         | source → aporia              | kg:raises            |
| `wasDerivedFrom` | hypothesis/aporia → source   | prov:wasDerivedFrom  |
| `assumes`        | hypothesis → entity          | kg:assumes           |
| `concerns`       | aporia → entity              | schema:about         |
| `addresses`      | hypothesis → aporia          | kg:addresses         |
| `addressedBy`    | aporia → hypothesis          | kg:addressedBy       |
| `solvedBy`       | aporia → resource/hypothesis | kg:solvedBy          |

The `source:` front-matter field on hypothesis/aporia nodes carries the `wasDerivedFrom`
(`dcterms:source`) provenance edge.

## Semantic predicates (entity ↔ entity / resource)

| Predicate      | Aligned term         |
| -------------- | -------------------- |
| `broader`      | skos:broader         |
| `narrower`     | skos:narrower        |
| `related`      | skos:related         |
| `isPartOf`     | dcterms:isPartOf     |
| `hasPart`      | schema:hasPart       |
| `replaces`     | dcterms:replaces     |
| `isReplacedBy` | dcterms:isReplacedBy |
| `requires`     | dcterms:requires     |
| `conformsTo`   | dcterms:conformsTo   |

## Citation predicates (higher-order; placed inline, see SPEC §10)

| Predicate          | Aligned term                 |
| ------------------ | ---------------------------- |
| `cites`            | cito:cites / schema:citation |
| `citesAsEvidence`  | cito:citesAsEvidence         |
| `citesAsAuthority` | cito:citesAsAuthority        |
| `supports`         | cito:supports                |
| `confirms`         | cito:confirms                |
| `extends`          | cito:extends                 |
| `critiques`        | cito:critiques               |
| `disputes`         | cito:disputes                |
| `refutes`          | cito:refutes                 |
| `isCitedBy`        | cito:isCitedBy               |

## Domain predicates (graph-native extensions)

| Predicate  | From → To         | Aligned term |
| ---------- | ----------------- | ------------ |
| `secures`  | entity → entity   | kg:secures   |
| `verifies` | entity → entity   | kg:verifies  |
