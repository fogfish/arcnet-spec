---
"@type": patch
document: ARCNET-DOMAIN-CORE-THOUGHT.md
title: "DOMAIN CORE THOUGHT Schema Bootstrap"
published: 2026-07-07
stats: { nodes: 7, edges: 9 }
---

# Property

## generatedThought

```yaml
role: link
merge: union
aligned: "prov:hadDerivation"
```

The inverse of `derivedFrom` — recorded as a backlink under the source's own `## generatedThought` block, the navigational convenience for discovering what a source has yielded. Both directions MUST be kept consistent; the canonical direction for provenance queries is `thought → source` via `derivedFrom`.

## cognition

```yaml
role: meta
merge: immutable
```

The thought's cognitive role, set once at creation: `insight` | `hypothesis` | `principle` | `question` | `direction` | `decision`. Deliberately not named `class` — that is DOMAIN-ARTICLE's validation-pass predicate (`established`/`extended`/`novel`/…), a different meaning and merge behavior; the two must not collide under one global predicate name.

## maturity

```yaml
role: meta
merge: lastWriteWin
```

How developed the thought is, legitimately changing over time: `emerging` (early intuition, weakly supported) | `developing` (coherent, supported by multiple points in the paper) | `mature` (well-developed line of reasoning).

## about

```yaml
role: text
merge: firstWriteWin
```

A 2–4 sentence rationale: why the thought matters, what problem it addresses.

## motivation

```yaml
role: text
merge: firstWriteWin
```

The underlying problem or curiosity that likely motivated the notes.

## next

```yaml
role: text
merge: firstWriteWin
```

The natural next question, experiment, or line of reasoning the thought implies.

# Class

## thought

A node distilling the central insight, hypothesis, principle, question, direction, or decision an author draws from a body of notes.

**Requires**
- required:: [[derivedFrom]]
- required:: [[claim]]
- required:: [[cognition]]

**Optional**
- optional:: [[maturity]]
- optional:: [[about]]
- optional:: [[motivation]]
- optional:: [[concerns]]
- optional:: [[citesAsEvidence]]
- optional:: [[next]]
