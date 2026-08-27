---
"@type": patch
document: ARCNET-DOMAIN-CORE-THOUGHT.md
published: 2026-08-27
stats: { nodes: 8, edges: 10 }
---

# Property

## generatedThought

```yaml
role: link
merge: union
label: "Generated Thought"
aligned: "prov:hadDerivation"
```

The inverse of `derivedFrom` — recorded as a backlink under the source's own `## Generated Thought` block, the navigational convenience for discovering what a source has yielded. Both directions MUST be kept consistent; the canonical direction for provenance queries is `Thought → Source` via `derivedFrom`. Registered as `Optional` on `Source`'s own Class node.

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

## rationale

```yaml
role: text
merge: firstWriteWin
```

A 2–4 sentence rationale: why the thought matters, what problem it addresses. Deliberately not named `about` — CORE registers its own `about` predicate (role `meta`, a subject-matter enum), a different meaning and merge behavior.

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

## Thought

```yaml
```

A node distilling the central insight, hypothesis, principle, question, direction, or decision an author draws from a body of notes.

**Requires**
- required:: [[derivedFrom]]
- required:: [[claim]]
- required:: [[cognition]]

**Optional**
- optional:: [[maturity]]
- optional:: [[rationale]]
- optional:: [[motivation]]
- optional:: [[concerns]]
- optional:: [[citesAsEvidence]]
- optional:: [[next]]

## Source

```yaml
```

**Optional**
- optional:: [[generatedThought]]
