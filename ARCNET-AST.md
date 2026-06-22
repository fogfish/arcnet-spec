# AST — In-Memory Representation of the Graph

**Status:** Accepted · **Version:** 0.4 · **Date:** 2026-06-22
**Extends:** [`CORE.md`](ARCNET-CORE.md)

This document specifies a **runtime-agnostic in-memory model** for the Markdown knowledge graph defined by [CORE](ARCNET-CORE.md). It is a *model definition*, not a wire or storage format: it fixes the shapes an application holds in memory so producers and consumers in any language interoperate. The model is a **lossless projection** of the on-disk graph — the Markdown files (CORE §3.1), the document patch (CORE §12), and this model convert without loss of content or connectivity.

The model is **plain JSON**. The semantic alignment of predicates to standard vocabularies lives in CORE's predicate registry (CORE §7.3/§7.4), not in the runtime shapes; this model does not carry it. The model depends on no program, library, or language.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in RFC 2119. As in CORE §1, each element is declared at three levels — **Mandatory**, **Recommended**, **Optional**. JSON value types are named per RFC 8259. This document **represents** CORE mechanism (identity §6, edges §7, citations §8, merge §10) and **MUST NOT** redefine it.

## 2. Purpose and Scope

The model is the **AST every application loads into**. The Markdown graph (CORE §3.1) and a patch (CORE §12) are two human readable on-disk forms of the same node objects: an application parses either into the model, operates on it, and serializes it back. Applying a patch to the graph (CORE §12.3) is the model's central operation — load the patch nodes and the target nodes, then reconcile each by the kind's merge (CORE §10). Conversion between the forms is lossless (§3.6).

CORE §4 defines a node as **front-matter scalars + a Markdown body**. In the graph the body is human readable prose plus **edges** (`predicate:: [[Target]]`) and **literals** (plain bullets, possibly naming a node inline). From the *representation and navigation* perspective this collapses to exactly **two kinds of thing**:

1. **Text** — a content block, opaque to a program and carried verbatim. A consumer **MUST NOT** parse it. Prose, literals (CORE §4), and any inline `[[wikilink]]` markup authored inside them are all Text. A node carries Text in at most two named places: a leading block (§6.1) and an optional trailing block (§6.1).
2. **Links** — the elements that establish connectivity: wikilinks, predicated edges, and citations. A **single Link abstraction** (§6.2) carries its own `predicate` and represents all three; a citation is a Link whose `predicate` is a citation type (CORE §8). Links are presented in **`links`** (§6.3) — an object of role-keyed **blocks**, each an ordered unit of **navigation**, not formatting. A block's role label is **not** the same axis as a Link's predicate (§6.3).

The central design rule follows from this: connectivity is navigable only because Links are first-class elements.

Any program-relevant meaning a document carries **MUST** be expressed as an **attribute** or a **Link**; **exceptionally**, and only with profile justification, as a new top-level Node member (§8). Deep parsing of Markdown by a consumer is out of scope and is to be avoided.

This document does **not** define how nodes are produced (CORE's non-scope) and does not mandate a language binding; it fixes the abstract shapes a binding represents.

## 3. Design Invariants

1. **One node, one object.** Each CORE node (one `.md` file) is one **node object** (§4).
2. **Two element kinds only.** Content is **Text** (`text`, `notes` — §6.1) or **Links**, presented in `links` blocks keyed by role (§6.3). 
3. **No consumer-side Markdown parsing.** Text is opaque (§6.1). A consumer reads connectivity from `links` alone. A consumer **MUST NOT** scan Text to *discover* edges; finding where a *known* edge's target sits inside Text. 
4. **Item order is preserved; block order is not.** Within one `links[role].seq` array, order is the application specific order where a kind's schema assigns it meaning (e.g. CORE §9.4's chronological entries). Blocks are **not** stored in the canonical order. The rendering rule (§6.3) provides a recommendation about ordering.
5. **Open vocabularies, preserve unknowns.** `kind`, attribute names, a Link's `predicate`, a block's role key, and a block's `title` are each independently open. A consumer **MUST** preserve attributes, role keys, and predicates it does not recognize (CORE §4), so a profile (CORE §13) extends the model with data alone (§8).
6. **Predicate and role are independent axes.** A Link's `predicate` (§6.2) names the edge's semantics (CORE §7.3); the `links` key it is filed under (§6.3) names only how it is grouped for navigation and display. A consumer **MUST NOT** infer one from the other. A role MAY hold Links of several predicates, or none; the same predicate MAY appear under more than one role.
7. **Lossless conversion.** Markdown ⇄ model ⇄ patch (CORE §12) preserves content, connectivity, front-matter, item order within a block, and every Link's `predicate`/`target`/`alias`. Insignificant whitespace and the relative order of `links` blocks **MAY** be normalized (§3.4).

## 4. Node Object

```json
{
  "id": "rescorla-2026-tls13",
  "kind": "source",
  "attrs": {
    "title": "TLS 1.3: Design and Rationale",
    "authors": ["Eric Rescorla"],
    "published": "2026-04-12",
    "url": "https://example.org/tls13-design",
    "tags": ["tls", "protocols"]
  },
  "text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption.",
  "links": { /* role-keyed blocks (§6.3) */ }
}
```

**Members**
- `id` (mandatory) — the node identity, equal to the file basename without `.md` (CORE §3.2, §6).
- `kind` (mandatory) — the node kind (CORE §4); a core kind or a profile kind. Open vocabulary.
- `attrs` (mandatory) — the front-matter scalar attributes (§7), as a JSON object, **excluding** `kind`. The identity field (`id`/`title`) **MAY** be repeated here for convenience.
- `text` (optional) — the leading opaque prose block (§6.1). Mandatory or recommended per the kind's own schema (e.g. CORE §9.1's `abstract`); absent when the kind has none (e.g. `timeline`).
- `notes` (optional) — a trailing opaque prose block (§6.1), distinct from `text` because it renders **after** `links`, never before.
- `links` (mandatory) — an object keyed by an open **role** label (a presentation grouping, §6.3), each value a block (§6.3). **MAY** be empty `{}`. The reserved key `""` is the default unlabeled role.

The leading `# <title>` heading of an on-disk node repeats `attrs.title`/`id` and is **derivable**; it is **not** stored. A block's display heading (e.g. `## Mentions`) is likewise not stored independently — it is `links["mentions"].title` (§6.3) — and is a property of the role's block, not of the predicate.

A consumer that traverses the graph collects a node's outgoing edges by iterating the values of `links` and flattening their `seq` (and any link-valued attribute, §7) — an O(n) walk with no Markdown parsing and no element-type filtering, since `links` holds only Links by construction. To collect every edge of one **predicate**, filter the flattened `seq` by `link.predicate`: a role's `seq` MAY mix predicates, so the role key alone is not a reliable filter (§6.3). A binding **MAY** additionally cache a flat link list, but `links` remains authoritative.

## 5. Graph

A **graph** is an ordered sequence of node objects — a plain JSON array.

```json
{ "nodes": [ /* node objects (§4) */ ] }
```

- `nodes` (mandatory) — array of node objects. Order is filing convenience; identity (CORE §6) is the sole index. Basenames are unique (CORE §3.2), so a consumer **MAY** key nodes by `id`. A binding **MAY** represent the graph as a bare top-level array instead of the wrapper object.

A graph **MAY** be partial; a Link target absent from the in-memory set is a dangling reference resolved against the full on-disk graph, not a model error.

## 6. Text, Notes, and Links

### 6.1 `text` and `notes`

```json
"text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption."
```

- Both are an opaque Markdown run, carried verbatim. The model does **not** decompose either into paragraphs, emphasis, lists, or marks, and a consumer **MUST NOT** parse them (§3.3).
- `text` **MUST** render before `links`; `notes` **MUST** render after. A kind whose CORE schema defines only one prose field (e.g. `source`'s `abstract`, `entity`'s `definition`) carries it as `text`; a kind whose schema additionally defines a closing field (e.g. `entity`/`resource`'s `notes`, CORE §9.2/§9.3) carries that as `notes`. 
- Either **MAY** contain inline `[[links]]` or the inline predicate form (CORE §7.2) as authored Markdown. These are **content**, not navigable connectivity by themselves; see §6.4.
- Emphasis and other inline Markdown formatting are part of Text. The model does **not** elevate them. If an application needs a piece of Markdown as structured data, it lifts it into an **attribute** (§7) — a producer/profile concern, never a Text field.

### 6.2 `link` — the Unified Edge

A Link is the one abstraction for every connection, and it **carries its own predicate** — the
predicate is never inferred from whichever `links` role (§6.3) the Link happens to be filed under. CORE's forms map onto it as follows:

| CORE form (§7, §8)                                        | Representation                                                                                          |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| untyped mention `[[Target]]`                              | `{ "target": "Target" }` — no `predicate`; conventionally filed under the reserved `""` role            |
| display alias `[[Target\|text]]`                          | `{ "target": "Target", "alias": "text" }`                                                               |
| list/body predicated edge `pred:: [[Target]]`             | `{ "predicate": "pred", "target": "Target" }`, conventionally filed under a role named `"pred"`         |
| citation `citesAsEvidence:: [[X]]`                        | `{ "predicate": "citesAsEvidence", "target": "X" }`                                                     |
| inline predicated edge `[pred:: [[Target]]]` inside prose | the same Link shape, filed under the default `""` role; the markup also stays verbatim in `text`/`notes` (§6.4) |

- `target` (mandatory) — the target basename (CORE §3.2).
- `predicate` (recommended) — the edge predicate (camelCase, registered, CORE §7.3). Absent for an untyped mention. A consumer **MUST** read predicate only from this field (§3.6) — never from the `links` role key.
- `alias` (optional) — the display text of `[[Target|text]]` (CORE §7.1).
- A **citation** is a Link whose `predicate` is a citation type (CORE §8); it needs no separate shape, though a binding **MAY** classify it from the predicate's `cito:` namespace.

### 6.3 `links` — Role-Keyed Blocks

```json
"links": {
  "mentions": {
    "title": "Mentions",
    "seq": [
      { "predicate": "mentions", "target": "Transport Layer Security" },
      { "predicate": "mentions", "target": "Forward Secrecy" }
    ]
  },
  "cites": {
    "title": "Cites",
    "seq": [
      { "predicate": "cites", "target": "RFC 8446" }
    ]
  }
}
```

- `links` is a JSON object keyed by an open **role** label — a presentation grouping, not a predicate — or the reserved key `""` for the default, unlabeled role.
- Each value is a **block**: `title` (optional) and `seq` (mandatory, an ordered, non-empty array of `link` elements, §6.2).
- `title` is the block's display heading text (rendered as `## <title>` in a graph file; as a bold label in a patch, CORE §12.2). It defaults to nothing — **absent** `title` means the block renders as plain bullets with **no heading**.
- `seq` **MUST NOT** be empty; a producer with no edges for a role **MUST** omit that key rather than emit an empty block.

**A role names a grouping, not a predicate.** By convention — and in every CORE §9 worked example — a role's name matches the single predicate its Links share (`mentions`, `cites`, `isCitedBy`, `mentionedIn`), and `title` is that name capitalized. This convention is **not** a constraint:
  - A role's `seq` **MAY** mix several predicates under one heading, e.g. a profile that wants one `## Citations` block covering several CITO predicates instead of one block per predicate:
    ```json
    "Citations": {
      "title": "Citations",
      "seq": [
        { "predicate": "citesAsEvidence", "target": "RFC 8446" },
        { "predicate": "disputes", "target": "Bellovin 2019" }
      ]
    }
    ```
  - A role's `seq` **MAY** hold untyped Links alongside predicated ones. **A Link authored inline in prose (§6.4) is filed under the reserved `""` role by default, regardless of its predicate** — being authored inline, rather than as a list/body-form edge, is a presentation fact independent of any headed block, so it does not by itself belong to that predicate's named role.
  - The same predicate **MAY** be split across more than one role (e.g. some `mentions` edges in a headed block, others left in the default `""` role because they were authored inline) without changing what they mean; `predicate` on the Link is what a consumer queries by, never the role.
- **Canonical block order** (§3.4) for rendering: headerless blocks first, sorted by role name; then titled blocks, sorted by `title`; `text` always precedes every block, `notes` always follows every block. This order is derived at render time and is never stored.
- **Item order within a block is preserved.** It is significant wherever the kind's schema assigns it meaning — e.g. a `timeline` node's `links[""].seq` is the chronological entry order (CORE §9.4); a generic consumer otherwise treats it as the producer's authored order.
- **Merge (CORE §10 `union`).** When a contribution merges into an existing node, blocks with the same role key combine by union of `seq`, deduplicated by the `(predicate, target)` pair — two Links to the same target under different predicates are distinct edges and both survive; `title` follows first-writer precedence, consistent with CORE's per-field commutative-merge requirement.

### 6.4 Inline Links within Text

A wikilink target is always a node title (CORE §6.3), so it is always a literal substring of the Text that authored it. This lets the inline predicate form (CORE §7.2) and any other in-prose mention stay fully opaque as Text while remaining recoverable for both navigation and rendering, without a run-encoded or position-indexed Text structure:

- The producer **MUST** keep the inline markup (`[pred:: [[Target]]]`, or a bare `[[Target]]`)
  verbatim inside `text`/`notes`, and, if the edge is meant to be navigable through this model, also record it as a Link carrying that predicate (or none, for an untyped mention). **By default this Link goes in the reserved `""` role (§6.3)** — the default destination for every inline-authored Link, regardless of predicate — the same rule CORE already applies to a literal that names a node (CORE §4). A producer **MAY** instead file it under its predicate's own headed role if it wants the edge to also surface in that block's rendering.
- A renderer reproducing Markdown gets the inline mention's navigability for free: the literal
  `[[...]]` substring round-trips inside Text and any Markdown/wiki viewer renders it as a link without help from this model.
- `links` remains the **sole authoritative source** for programmatic graph traversal. A consumer **MUST NOT** derive a new edge by scanning Text (§3.3); it **MAY**, for rendering or highlighting, locate a substring inside Text matching a target it already obtained from `links` — a bounded, deterministic string match against a known set of targets, not Markdown parsing.

### 6.5 Worked Examples

`source` (CORE §9.1) — leading `text`, two titled blocks, no `notes`:

```json
{
  "id": "rescorla-2026-tls13",
  "kind": "source",
  "attrs": {
    "title": "TLS 1.3: Design and Rationale",
    "authors": ["Eric Rescorla"],
    "published": "2026-04-12",
    "url": "https://example.org/tls13-design",
    "tags": ["tls", "protocols"]
  },
  "text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption.",
  "links": {
    "mentions": {
      "title": "Mentions",
      "seq": [
        { "predicate": "mentions", "target": "Transport Layer Security" },
        { "predicate": "mentions", "target": "Forward Secrecy" }
      ]
    },
    "cites": {
      "title": "Cites",
      "seq": [
        { "predicate": "cites", "target": "RFC 8446" }
      ]
    }
  }
}
```

`entity` (CORE §9.2) — leading `text`, a **headerless** block, a titled block, trailing `notes`:

```json
{
  "id": "Transport Layer Security",
  "kind": "entity",
  "attrs": {
    "title": "Transport Layer Security",
    "category": ["independent", "abstract", "occurrent", "script"],
    "aliases": ["TLS", "TLS 1.3"],
    "tags": ["cryptography"]
  },
  "text": "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network.",
  "links": {
    "replaces": {
      "seq": [
        { "predicate": "replaces", "target": "SSL Protocol" }
      ]
    },
    "conformsTo": {
      "seq": [
        { "predicate": "conformsTo", "target": "RFC 8446" }
      ]
    },
    "mentionedIn": {
      "title": "mentionedIn",
      "seq": [
        { "predicate": "mentionedIn", "target": "rescorla-2026-tls13" }
      ]
    }
  },
  "notes": "Widely deployed since 2018; superseded TLS 1.2 as the IETF-recommended baseline."
}
```

`resource` (CORE §9.3) — leading `text`, one titled block:

```json
{
  "id": "RFC 8446",
  "kind": "resource",
  "attrs": {
    "title": "RFC 8446",
    "ref": "standard",
    "authors": ["Eric Rescorla"],
    "year": 2018,
    "url": "https://www.rfc-editor.org/rfc/rfc8446",
    "status": "read"
  },
  "text": "The normative specification of TLS 1.3.",
  "links": {
    "isCitedBy": {
      "title": "isCitedBy",
      "seq": [
        { "predicate": "isCitedBy", "target": "rescorla-2026-tls13" }
      ]
    }
  }
}
```

`timeline` (CORE §9.4) — no `text`/`notes`. Entries are untyped, so they sit in the default `""` role. Item order is chronological. The display annotation (title, author, date) is rendered from each target's own `attrs`, never stored on the timeline node itself:

```json
{
  "id": "2026-04",
  "kind": "timeline",
  "attrs": {
    "period": "2026-04",
    "granularity": "monthly"
  },
  "links": {
    "": {
      "seq": [
        { "target": "rescorla-2026-tls13" },
        { "target": "chen-2026-pqkex" }
      ]
    }
  }
}
```

## 7. Attributes

`attrs` is a JSON object carrying the node's front-matter scalar attributes (CORE §4), keyed by attribute name. Values are JSON scalars or arrays of scalars, as authored.

- The known attributes of a `kind` and their types are declared by the kind's three-level element table in CORE §9 or the profile (CORE §13); that table is the attribute schema.
- A consumer **MUST** preserve attributes it does not recognize (CORE §4). A typesafe binding represents the known attributes as typed fields and unknown attributes in a single overflow map.
- Unknown attributes **MUST** survive a serailizations unchanged (§3.6).

## 8. Extensibility

A profile (CORE §13) extends the model with **data alone**, in priority order:

| Need                                          | Model representation                                                          |
| --------------------------------------------- | ----------------------------------------------------------------------------- |
| program-relevant scalar / flag                | a new **attribute** (§7) + its declared type in the profile schema            |
| connectivity / navigation, 1:N or 1:1         | a new **predicate** on a Link (§6.2), registered per CORE §7.3                |
| new presentational grouping of existing Links | a new **role** key in `links` (§6.3) — open, no registration, no shape change |
| display heading for a role's block            | the block's `title` (§6.3) — existing, no shape change                        |
| new node kind                                 | a new `kind` value (§4) — no shape change                                     |
| new node-level content area                   | **exceptionally**, a new top-level Node member — see below                    |

A new top-level **Node member** is permitted only when the need cannot be met by an attribute or a `links` entry, and the profile **MUST** justify it. Because consumers preserve unknown `attrs` keys, unknown `links` role keys, and unknown `predicate` values (§3.5), an extension that stays within `attrs`/`links` never breaks a conforming binding. A profile **MUST NOT** redefine the node object, the Link abstraction, or the attribute encoding (consistent with CORE §13).

## 9. Conformance Checklist

- [ ] Each CORE node is one node object with `id`, `kind`, `attrs`, `links` (§4); `text`/`notes` present when the kind's schema defines a corresponding prose field.
- [ ] `id` equals the file basename; `kind` is preserved verbatim (§4).
- [ ] `links` is a JSON object keyed by an open role label, or the reserved `""` default role; every value is a block of `title` (optional) and `seq` (a non-empty ordered array of Links, §6.3).
- [ ] Every Link carries its own `predicate` (or none, for an untyped mention); a consumer **MUST NOT** infer a Link's predicate from the role key it is filed under (§6.2, §6.3).
- [ ] `text` and `notes` are opaque and carried verbatim; no consumer parses them to discover edges, even though a known Link's target MAY also appear there as a literal substring (§3.3, §6.4).
- [ ] Every navigable connection — wikilink, predicate, or citation, including the inline predicate form — is a Link present somewhere in `links` with `predicate` set accordingly (§6.2, §6.4).
- [ ] Item order within a `links` block is preserved; the relative order of blocks themselves, and of `text`/`notes` around them, is the canonical rendering order, not stored data (§3.4, §6.3).
- [ ] `attrs` is a JSON object keyed by attribute name; known attributes follow the per-kind schema and unknown attributes are preserved (§7).
- [ ] Unknown attributes, role keys, and predicates are preserved; extensions add attributes, predicates, or roles first, a new top-level Node member only exceptionally (§3.5, §8).
- [ ] Markdown ⇄ model ⇄ patch round-trips without loss of content or connectivity; cosmetic block order MAY be normalized (§3.6).
