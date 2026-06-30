# CORE THOUGHT — Markdown Knowledge Graph extension with Core Thoughts

**Status:** Draft · **Version:** 0.1 · **Date:** 2026-06-29
**Extends:** [`ARCNET-CORE.md`](ARCNET-CORE.md)

This extension adds one new node kind, `thought` ([CORE](ARCNET-CORE.md) §13), for the
synthesized insights, hypotheses, principles, questions, directions, and decisions an author
distills from a body of notes — collectively, **Core Thoughts**. Unlike a domain profile
(`DOMAIN-<name>.md`), `thought` relates only to the CORE kinds (`source`, `entity`, `resource`;
CORE §9) and never to a domain profile's own kinds. It therefore composes with **any** domain
profile, including [`DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md), without depending on one —
a graph may adopt this extension with no domain profile at all.

A **Core Thought** is the minimal *generative cognitive unit* — the central insight, hypothesis, design principle, research
direction, decision, or question that motivated the notes. It is not a summary. It answers "what was the author trying to figure out, understand, design, or express?".

## 1. Two-Phase Workflow

Core Thought extraction is a distinct second pass, run **after** core processing, never
interleaved with it:

1. **Core processing** ([CORE](ARCNET-CORE.md) §9) ingests a document into `source`, `entity`,
   `resource`, and `timeline` nodes. A domain profile, if adopted, derives its own kinds in this
   same pass.
2. **Core Thought extraction** then reads one or more `source` nodes' literals and synthesizes `thought` nodes from them,
   linked back by provenance (§3). This pass is idempotent and may be re-run as a source's body
   grows; it has no dependency on, and no ordering requirement relative to, any domain profile's
   own derivation step.

## 2. Folder Layout

```
graph/
├── sources/              # source nodes        (CORE §9.1)
├── entities/             # entity nodes, Sowa   (CORE §9.2)
├── resources/            # resource nodes       (CORE §9.3)
├── thoughts/             # thought nodes        (§3)
├── timeline/             # production index     (CORE §9.4)
│   ├── yearly/
│   └── monthly/
└── _meta/
    ├── predicates.md
    └── aliases.md
```

`thoughts/` is flat: a thought's `class` (§3) lives in front-matter, so it never determines file
location (CORE §3.5).

## 3. Node Kind: `thought`

A node distilling the central insight, hypothesis, principle, question, direction, or decision. **Identity:** title (CORE
§6.3). **Merge:** union (CORE §10).

**Front-matter**
- `kind` (mandatory) — the literal `thought`
- `title` (mandatory) — concise name, ≤ 6 words, equal to the basename
- `source` (mandatory) — `wasDerivedFrom` link(s) to the originating source node(s)
- `class` (mandatory) — the thought's cognitive role: `insight` | `hypothesis` | `principle` |
  `question` | `direction` | `decision`
- `maturity` (recommended) — how developed the thought is: `emerging` (early intuition, weakly
  supported) | `developing` (coherent, supported by multiple points in the paper) | `mature` (well-
  developed line of reasoning)

**Body**
- `claim` (mandatory) — the one-sentence central statement (rendered emphasized)
- `about` (recommended) — a 2–4 sentence rationale: why the thought matters, what problem it
  addresses.
- `motivation` (recommended) — the underlying problem or curiosity that likely motivated the
  notes
- `**Concerns**` (recommended) — `concerns::` edges to the entities the thought concerns
- `**Evidence**` (optional) — citation edges (`citesAsEvidence::`, …; CORE §8) to resources the
  thought draws on
- `next` (optional) — the natural next question, experiment, or line of reasoning the thought
  implies

```markdown
---
kind: thought
title: Latency Cost Justifies 1-RTT Risk
source: [[rescorla-2026-tls13]]
class: decision
maturity: developing
---
*Accepting the residual replay risk of zero-round-trip resumption is worth the latency win,
provided application-layer idempotency is enforced upstream.*

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

## 4. Predicate Vocabulary

This extension introduces no new predicate — every edge a `thought` carries reuses a predicate
already registered by CORE or a co-located domain profile. Register these in
`graph/_meta/predicates.md` if not already present:

| Predicate         | From → To          | Aligned term         |
| ----------------- | ------------------ | -------------------- |
| `wasDerivedFrom`  | thought → source   | prov:wasDerivedFrom  |
| `concerns`        | thought → entity   | schema:about         |
| `citesAsEvidence` | thought → resource | cito:citesAsEvidence |

`concerns` and `citesAsEvidence` carry the same meaning here as wherever else they are
registered (CORE §8, [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4) — a predicate's meaning does
not change with the kind that uses it (CORE §7.3).

## 5. Conformance

In addition to the CORE checklist (CORE §14):

- [ ] Every `thought` has a `source` link and a title-based basename (CORE §6.3).
- [ ] Every `thought.class` is one of the six §3 values; `maturity`, where present, is one of the
      three §3 values.
- [ ] No `thought` links to a domain-profile kind (e.g. `hypothesis`, `aporia`) — only to
      `source`, `entity`, or `resource` (§1).
- [ ] Core Thought extraction ran strictly after core processing for the same source (§1).
- [ ] Every predicate used is registered in `graph/_meta/predicates.md` (§4).
