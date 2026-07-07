# DOMAIN CORE THOUGHT - Markdown Knowledge Graph extension with Core Thoughts

**Status:** Draft · **Version:** 0.3 · **Date:** 2026-07-07
**Extends:** [`ARCNET-CORE.md`](ARCNET-CORE.md)

> Revision note (0.1 → 0.2): follows CORE v0.6's predicate-first model. `kind`/`id`/`title` → `@type`/`@id`. `thought`'s front-matter `source` field was a link value stored where CORE's `meta` role only ever holds scalars — it is now the `derivedFrom` predicate itself, asserted as a body edge. `thought`'s cognitive-role field is renamed from `class` to `cognition`: CORE registers predicates globally, one node per name (CORE §9.1), and this extension's `class` (`insight`/`hypothesis`/`principle`/`question`/`direction`/`decision`, set once at creation) is a different predicate from [`DOMAIN-ARTICLE`](ARCNET-DOMAIN-ARTICLE.md)'s `class` (`established`/`extended`/`novel`/…, produced only by a later validation pass) — the two would otherwise collide under one global name when both extensions are adopted together, which §1 says must work. Predicates are now registered as individual `_schema/predicates/`/`_schema/types/` nodes (CORE §9), described one-per-heading (CORE §10's style) rather than in a table.
>
> Revision note (0.2 → 0.3): follows CORE v0.7's retirement of the `## Recommended` tier. `maturity`/`about`/`motivation`/`concerns` are now optional rather than recommended — none is essential to what a `thought` minimally asserts (a claim, its cognitive role, and its provenance).

This extension adds one new node type, `thought` ([CORE](ARCNET-CORE.md) §15), for the
synthesized insights, hypotheses, principles, questions, directions, and decisions an author
distills from a body of notes — collectively, **Core Thoughts**. Unlike a domain profile
(`DOMAIN-<name>.md`), `thought` relates only to the CORE types (`source`, `entity`, `resource`;
CORE §11) and never to a domain profile's own types. It therefore composes with **any** domain
profile, including [`DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md), without depending on one —
a graph may adopt this extension with no domain profile at all.

A **Core Thought** is the minimal *generative cognitive unit* — the central insight, hypothesis, design principle, research
direction, decision, or question that motivated the notes. It is not a summary. It answers "what was the author trying to figure out, understand, design, or express?".

## 1. Two-Phase Workflow

Core Thought extraction is a distinct second pass, run **after** core processing, never
interleaved with it:

1. **Core processing** ([CORE](ARCNET-CORE.md) §11) ingests a document into `source`, `entity`,
   `resource`, and `timeline` nodes. A domain profile, if adopted, derives its own types in this
   same pass.
2. **Core Thought extraction** then reads one or more `source` nodes' literals and synthesizes `thought` nodes from them,
   linked back by provenance (§4). This pass is idempotent and may be re-run as a source's body
   grows; it has no dependency on, and no ordering requirement relative to, any domain profile's
   own derivation step.

## 2. Folder Layout

```
graph/
├── sources/               # source nodes         (CORE §11.2)
├── entities/              # entity nodes, Sowa    (CORE §11.3)
├── resources/              # resource nodes       (CORE §11.4)
├── thoughts/               # thought nodes        (§3)
├── timeline/               # production index     (CORE §11.5)
│   ├── yearly/
│   └── monthly/
└── _schema/
    ├── predicates/          # this extension's own + reused predicates (§4)
    ├── types/               # thought.md (§3)
    └── aliases.md           # entity alias table — not a node (CORE §7.4)
```

`thoughts/` is flat: a thought's `cognition` (§3) lives in a predicate, so it never determines
file location (CORE §4.7).

## 3. Node Type: `thought`

A node distilling the central insight, hypothesis, principle, question, direction, or decision. **Identity:** `@id` (CORE §7.3) — a concise name, ≤ 6 words, equal to the basename; carries no separate title predicate, same as `entity`/`resource`.

```markdown
---
"@id": thought
"@type": Class
---
# thought

A node distilling the central insight, hypothesis, principle, question, direction, or decision an
author draws from a body of notes.

## Requires
- required:: [[derivedFrom]]
- required:: [[claim]]
- required:: [[cognition]]

## Optional
- optional:: [[maturity]]
- optional:: [[about]]
- optional:: [[motivation]]
- optional:: [[concerns]]
- optional:: [[citesAsEvidence]]
- optional:: [[next]]
```

```markdown
---
"@id": Latency Cost Justifies 1-RTT Risk
"@type": thought
cognition: decision
maturity: developing
---
*Accepting the residual replay risk of zero-round-trip resumption is worth the latency win,
provided application-layer idempotency is enforced upstream.*

- derivedFrom:: [[rescorla-2026-tls13]]

**About**
The 1-RTT handshake's latency savings are why TLS 1.3 standardized it despite a known replay
exposure; the design bet is that idempotency enforcement is cheaper than losing the round trip.

**Motivation**
Whether the performance gain of 0/1-RTT resumption is defensible once its replay weakness is
known.

**Concerns**
- concerns:: [[Forward Secrecy]]
- concerns:: [[Handshake Protocol]]

**Evidence**
- citesAsEvidence:: [[RFC 8446]]

**Next**
Quantify how much idempotency enforcement actually costs at the application layer before
treating this trade-off as settled.
```

## 4. Predicates

This extension registers `generatedThought`, `cognition`, `maturity`, `about`, `motivation`, and
`next` as `_schema/predicates/` nodes (CORE §9.1) — each below, one per heading. It reuses
`derivedFrom`, `concerns`, and `citesAsEvidence` if a co-located profile (e.g.
[DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1) already registered them; otherwise this
document is where they get registered, with the same role/merge/aligned values given here — a
predicate's meaning and merge behavior do not change with the type that uses it (CORE §4
invariant 4).

### 4.1 Structural predicates

#### `derivedFrom`
**role:** `edge` · **merge:** `union` · **aligned:** `prov:wasDerivedFrom` · **from → to:** thought → source

The provenance edge to the originating source(s) — replaces the old front-matter `source` field, which never fit CORE's scalar-only `meta` role. Identical to [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1's predicate of the same name; reuse it, do not re-register it with different values.

#### `generatedThought`
**role:** `link` · **merge:** `union` · **aligned:** `prov:hadDerivation` (inverse) · **from → to:** source → thought

The inverse of `derivedFrom` — recorded as a backlink under the source's own `## generatedThought` block, the navigational convenience for discovering what a source has yielded. Both directions MUST be kept consistent; the canonical direction for provenance queries is `thought → source` via `derivedFrom`.

```markdown
## generatedThought
- generatedThought:: [[Latency Cost Justifies 1-RTT Risk]]
```

#### `concerns`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:about` · **from → to:** thought → entity

The entities the thought concerns. Identical to [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1's predicate of the same name (there, aporia → entity); reuse it if already registered.

### 4.2 Type-specific predicates

#### `claim`
**Used by:** `thought` · **role:** `text` · **merge:** `firstWriteWin`

The one-sentence central statement, rendered emphasized (`*claim*`). Identical to [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.2's predicate of the same name; reuse it if already registered.

#### `cognition`
**Used by:** `thought` · **role:** `meta` · **merge:** `immutable`

The thought's cognitive role, set once at creation: `insight` | `hypothesis` | `principle` | `question` | `direction` | `decision`. Deliberately **not** named `class` — that name is [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.2's validation-pass predicate (`established`/`extended`/`novel`/…), a different meaning and merge behavior; the two must not collide under one global predicate name.

#### `maturity`
**Used by:** `thought` · **role:** `meta` · **merge:** `lastWriteWin`

How developed the thought is, legitimately changing over time: `emerging` (early intuition, weakly supported) | `developing` (coherent, supported by multiple points in the paper) | `mature` (well-developed line of reasoning).

#### `about`
**Used by:** `thought` · **role:** `text` · **merge:** `firstWriteWin`

A 2–4 sentence rationale: why the thought matters, what problem it addresses.

#### `motivation`
**Used by:** `thought` · **role:** `text` · **merge:** `firstWriteWin`

The underlying problem or curiosity that likely motivated the notes.

#### `next`
**Used by:** `thought` · **role:** `text` · **merge:** `firstWriteWin`

The natural next question, experiment, or line of reasoning the thought implies.

### 4.3 Citation and reused predicates

`citesAsEvidence` and the other CORE citation types (CORE §10.6) are used unchanged in **Evidence**.

## 5. Conformance

In addition to the CORE checklist (CORE §16):

- [ ] Every `thought` has a `derivedFrom` edge and a claim-based basename (CORE §7.3).
- [ ] Every `thought.cognition` is one of the six §4.2 values; `maturity`, where present, is one of the three §4.2 values.
- [ ] No `thought` links to a domain-profile type (e.g. `hypothesis`, `aporia`) — only to
      `source`, `entity`, or `resource` (§1).
- [ ] Core Thought extraction ran strictly after core processing for the same source (§1).
- [ ] Every source referenced by a `thought`'s `derivedFrom` edge carries a matching `generatedThought::` edge back to that thought (§4.1).
- [ ] Every predicate used is registered as a `_schema/predicates/` node (§4); `thought` itself is registered as a `_schema/types/` node (§3).
