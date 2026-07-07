---
"@type": patch
document: ARCNET-DOMAIN-ARTICLE.md
title: "DOMAIN ARTICLE Schema Bootstrap"
published: 2026-07-07
stats: { nodes: 20, edges: 20 }
---

# Property

## proposes

```yaml
role: link
merge: union
aligned: "arc:proposes"
```

Asserts that the source document proposes the hypothesis; recorded under the source's own `## Proposes` block.

## raises

```yaml
role: link
merge: union
aligned: "arc:raises"
```

Asserts that the source document raises the aporia; recorded under the source's own `## Raises` block.

## derivedFrom

```yaml
role: edge
merge: union
aligned: "prov:wasDerivedFrom"
```

The provenance edge to the originating source(s) this node was distilled from — replaces the old front-matter `source` field, which never fit CORE's scalar-only `meta` role.

## assumes

```yaml
role: edge
merge: union
aligned: "arc:assumes"
```

The entities a hypothesis's premises depend on. Displayed under the bold label **Depends on**.

## concerns

```yaml
role: edge
merge: union
aligned: "schema:about"
```

The entities an aporia's open problem involves.

## addresses

```yaml
role: edge
merge: union
aligned: "arc:addresses"
```

The aporia a hypothesis's claim tackles.

## addressedBy

```yaml
role: edge
merge: union
aligned: "arc:addressedBy"
```

The inverse of `addresses` — the hypotheses that tackle this aporia. Displayed under the bold label **Addressed by**.

## solvedBy

```yaml
role: edge
merge: union
aligned: "arc:solvedBy"
```

The resource or hypothesis that resolves an aporia. Displayed under the bold label **Solved by**.

## claim

```yaml
role: text
merge: firstWriteWin
```

A one-sentence statement of the conclusion, rendered emphasized (`*claim*`).

## tension

```yaml
role: text
merge: firstWriteWin
```

A one-sentence statement of the open problem, rendered emphasized (`*tension*`).

## overview

```yaml
role: text
merge: firstWriteWin
```

A short paragraph of context.

## assumptions

```yaml
role: text
merge: append
```

Literal premise statements the hypothesis depends on, one per bullet. Displayed under the bold label **Assumptions**.

## issues

```yaml
role: text
merge: append
```

Literal statements decomposing the open problem, one per bullet. Displayed under the bold label **Issues**.

## class

```yaml
role: meta
merge: validatedOverwrite
```

The node's validation class — enum values differ per type. Produced only by an optional validation pass; an unvalidated `hypothesis`/`aporia` simply has none, a valid state.

## confidence

```yaml
role: meta
merge: validatedOverwrite
```

A 0–1 numeric confidence score assigned by the validation pass.

## secures

```yaml
role: edge
merge: union
aligned: "arc:secures"
```

Asserts the subject entity secures the target entity.

## verifies

```yaml
role: edge
merge: union
aligned: "arc:verifies"
```

Asserts the subject entity verifies the target entity.

# Class

## hypothesis

A conclusion distilled from sources.

**Requires**
- required:: [[derivedFrom]]
- required:: [[claim]]

**Optional**
- optional:: [[overview]]
- optional:: [[assumptions]]
- optional:: [[assumes]]
- optional:: [[addresses]]
- optional:: [[notes]]
- optional:: [[class]]
- optional:: [[confidence]]
- any CORE §10.6 citation predicate, as applicable

## aporia

An open problem or unresolved tension.

**Requires**
- required:: [[derivedFrom]]
- required:: [[tension]]

**Optional**
- optional:: [[overview]]
- optional:: [[issues]]
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[solvedBy]]
- optional:: [[notes]]
- optional:: [[class]]

## source

**Optional**
- optional:: [[proposes]]
- optional:: [[raises]]
