# AST — In-Memory Representation of the Graph

**Status:** Accepted · **Version:** 0.4 · **Date:** 2026-06-23
**Extends:** [`CORE.md`](ARCNET-CORE.md)

This document specifies a **runtime-agnostic in-memory model** for the Markdown knowledge graph defined by [CORE](ARCNET-CORE.md). It is a *model definition*, not a wire or storage format: it fixes the shapes an application holds in memory so producers and consumers in any language interoperate. The model is a **lossless projection** of the on-disk graph — the Markdown files (CORE §3.1), the document patch (CORE §12), and this model convert without loss of content or connectivity.

The model is **plain JSON**. The semantic alignment of predicates to standard vocabularies lives in CORE's predicate registry (CORE §7.3/§7.4), not in the runtime shapes; this model does not carry it. The model depends on no program, library, or language.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in RFC 2119. As in CORE §1, each element is declared at three levels — **Mandatory**, **Recommended**, **Optional**. JSON value types are named per RFC 8259. This document **represents** CORE mechanism (identity §6, edges §7, citations §8, merge §10) and **MUST NOT** redefine it.

## 2. Purpose and Scope

The model is the **AST every application loads into**. The Markdown graph (CORE §3.1) and a patch (CORE §12) are two human-readable on-disk forms of the same node objects: an application parses either into the model, operates on it, and serializes it back. Applying a patch to the graph (CORE §12.3) is the model's central operation — load the patch nodes and the target nodes, then reconcile each by the kind's merge (CORE §10). Conversion between the forms is lossless (§3.6).

CORE §4 defines a node as **front-matter scalars + a Markdown body**. In the graph the body is human-readable prose plus **edges** (`predicate:: [[Target]]`) and **literals** (plain bullets, possibly naming a node inline). From the *representation and navigation* perspective this collapses to exactly **two kinds of thing**:

1. **Text** — a content block, opaque to a program and carried verbatim. A consumer **MUST NOT** parse it. Prose, literals (CORE §4), and any inline `[[wikilink]]` markup authored inside them are all Text. A node carries Text in at most two named places: a leading block (§6.1) and an optional trailing block (§6.1).
2. **Link** — the one abstraction for every connection: wikilinks, predicated edges, and citations. A citation is a Link whose `predicate` is a citation type (CORE §8).

A Link is stored on the node for one of three distinct purposes, and never two at once for the same purpose:

- **`hrefs`** (§6.3) — every link literally embedded inside `text`/`notes`, kept only so an in-memory application can iterate them without parsing Text. It is a cache of what is written, not the graph.
- **`edges`** (§6.4) — the node's structural edges that the producer did not group under a heading: a flat, ordered, possibly mixed-predicate list.
- **`links`** (§6.5) — structural edges the producer *did* group under a heading, one block per predicate.

There is no fourth, structural kind of element. A Markdown heading that groups a predicate's edges (e.g. `## Mentions`) is the **display title of that predicate's block** (§6.5), carried as an attribute of the block, never as a sibling element. A heading and the edges it labels are one object; a heading cannot exist without edges, edges cannot exist without an implied heading, and no other content can interpose between a heading and the edges it labels.

The central design rule follows from this: connectivity is navigable only because a Link sits in `edges` or `links` — never because a consumer scans Text, and never via `hrefs`, which is a derived convenience, not an edge source.

Any program-relevant meaning a document carries **MUST** be expressed as an **attribute** or a **Link**; **exceptionally**, and only with profile justification, as a new top-level Node member (§8). Deep parsing of Markdown by a consumer is out of scope and is to be avoided.

This document does **not** define how nodes are produced (CORE's non-scope) and does not mandate a language binding; it fixes the abstract shapes a binding represents.

## 3. Design Invariants

1. **One node, one object.** Each CORE node (one `.md` file) is one **node object** (§4).
2. **Two element kinds only.** Content is **Text** (`text`, `notes` — §6.1) or **Link** (§6.2), held in `hrefs`, `edges`, or `links` (§6.3–§6.5). No structural element kind exists in this version; a heading is the `title` of the `links` block it labels, never a separate sibling.
3. **No consumer-side Markdown parsing for edges.** Text is opaque (§6.1). A consumer reads connectivity from `edges`/`links` alone; it never extracts an edge from Text. `hrefs` is populated by the producer by scanning Text, but a consumer treats it as already computed and **MUST NOT** itself parse Text to derive or verify it (§6.3); `hrefs` is also never treated as a source of navigable edges.
4. **Item order is preserved; block order is not.** Order within `hrefs`, within `edges`, and within one `links[predicate].seq` is the on-disk order and is significant wherever a kind's schema assigns it meaning (e.g. CORE §9.4's chronological entries, modeled as `edges`). Which predicate's `links` block precedes which, however, is **not** stored: the canonical order is deterministic — `edges` first, then `links` blocks sorted by `title` — and is a rendering rule (§6.5), not data.
5. **Open vocabularies, preserve unknowns.** `kind`, attribute names, and predicate names (whether on a `link.predicate` field or as a `links` key) are open. A consumer **MUST** preserve attributes and predicates it does not recognize (CORE §4), so a profile (CORE §13) extends the model with data alone (§8).
6. **Lossless conversion.** Markdown ⇄ model ⇄ patch (CORE §12) preserves content, connectivity, front-matter, item order within `hrefs`/`edges`/a `links` block, and every Link's `predicate`/`target`/`alias`. Insignificant whitespace and the relative order of `links` blocks **MAY** be normalized (§3.4).

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
  "links": { /* predicate-keyed blocks (§6.5) */ }
}
```

**Members**
- `id` (mandatory) — the node identity, equal to the file basename without `.md` (CORE §3.2, §6).
- `kind` (mandatory) — the node kind (CORE §4); a core kind or a profile kind. Open vocabulary.
- `attrs` (mandatory) — the front-matter scalar attributes (§7), as a JSON object, **excluding** `kind`. The identity field (`id`/`title`) **MAY** be repeated here for convenience.
- `text` (optional) — the leading opaque prose block (§6.1). Mandatory or recommended per the kind's own schema (e.g. CORE §9.1's `abstract`); absent when the kind has none (e.g. `timeline`).
- `notes` (optional) — a trailing opaque prose block (§6.1), distinct from `text` because it renders **after** `edges`/`links`, never before.
- `hrefs` (optional) — an ordered array of Links embedded inline in `text`/`notes` (§6.3). **MAY** be empty or absent.
- `edges` (optional) — an ordered, possibly mixed-predicate array of Links not grouped under a heading (§6.4). **MAY** be empty or absent.
- `links` (optional) — an object keyed by predicate name, each value a block (§6.5). **MAY** be empty `{}` or absent.

The leading `# <title>` heading of an on-disk node repeats `attrs.title`/`id` and is **derivable**; it is **not** stored. A `links` block's display heading (e.g. `## Mentions`) is likewise not stored independently — it is `links["mentions"].title` (§6.5).

A consumer that traverses the graph collects a node's outgoing edges by concatenating `edges` with the flattened `seq` of every `links` value (and any link-valued attribute, §7) — an O(n) walk with no Markdown parsing. `hrefs` is excluded from this walk; it answers "what does the prose embed," not "what does this node connect to." A binding **MAY** additionally cache a combined flat link list, but `edges`/`links` remain authoritative.

## 5. Graph

A **graph** is an ordered sequence of node objects — a plain JSON array.

```json
{ "nodes": [ /* node objects (§4) */ ] }
```

- `nodes` (mandatory) — array of node objects. Order is filing convenience; identity (CORE §6) is the sole index. Basenames are unique (CORE §3.2), so a consumer **MAY** key nodes by `id`. A binding **MAY** represent the graph as a bare top-level array instead of the wrapper object.

A graph **MAY** be partial; a Link target absent from the in-memory set is a dangling reference resolved against the full on-disk graph, not a model error.

## 6. Text and Links

### 6.1 `text` and `notes`

```json
"text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption."
```

- Both are an opaque Markdown run, carried verbatim. The model does **not** decompose either into paragraphs, emphasis, lists, or marks, and a consumer **MUST NOT** parse them (§3.3).
- `text` **MUST** render before `edges`/`links`; `notes` **MUST** render after. A kind whose CORE schema defines only one prose field (e.g. `source`'s `abstract`, `entity`'s `definition`) carries it as `text`; a kind whose schema additionally defines a closing field (e.g. `entity`/`resource`'s `notes`, CORE §9.2/§9.3) carries that as `notes`. A kind with no prose field at all (`timeline`, CORE §9.4) carries neither.
- Either **MAY** contain inline `[[links]]` or the inline predicate form (CORE §7.2) as authored Markdown. These are mirrored into `hrefs` (§6.3); they are not themselves navigable connectivity unless also present in `edges`/`links`.
- Emphasis and other inline Markdown formatting are part of Text. The model does **not** elevate them. If an application needs a piece of Markdown as structured data, it lifts it into an **attribute** (§7) — a producer/profile concern, never a Text field.

### 6.2 `link` — the Unified Edge

A Link is the one abstraction for every connection. Its shape is the same wherever it is stored (§6.3–§6.5):

```json
{ "predicate": "mentions", "target": "Transport Layer Security" }
```

- `target` (mandatory) — the target basename (CORE §3.2).
- `predicate` (recommended in `hrefs`/`edges`; redundant in `links`) — the edge predicate (camelCase, registered, CORE §7.3). Absent for an untyped mention. Inside a `links` block the predicate is already the map key and **SHOULD** be omitted from the item; if present it **MUST** match the key.
- `alias` (optional) — the display text of `[[Target|text]]` (CORE §7.1).
- A **citation** is a Link whose `predicate` is a citation type (CORE §8); it needs no separate shape, though a binding **MAY** classify it from the predicate's `cito:` namespace.

CORE's forms map onto the three containers as follows:

| CORE form (§7, §8)                                              | Representation                                                                                  |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| untyped mention `[[Target]]`, a standalone body bullet              | `{ "target": "Target" }` in `edges` (§6.4)                                                          |
| display alias `[[Target\|text]]`                                    | adds `"alias": "text"` to the Link wherever it is filed                                              |
| body/list predicated edge `pred:: [[Target]]`, no heading            | `{ "predicate": "pred", "target": "Target" }` in `edges` (§6.4)                                     |
| predicated edges grouped under a heading (e.g. `## Mentions`)       | `{ "target": "Target" }` in `links["pred"].seq` (§6.5) — predicate implied by the key               |
| citation `citesAsEvidence:: [[X]]`                                   | `{ "predicate": "citesAsEvidence", "target": "X" }`, in `edges` or grouped in `links` if headed     |
| inline predicated edge `[pred:: [[Target]]]` inside prose, navigable | `{ "predicate": "pred", "target": "Target" }` in **both** `hrefs` (§6.3) and `edges`/`links`         |
| bare inline mention `[[Target]]` inside prose, not meant to be a navigable edge | `{ "target": "Target" }` in `hrefs` **only** (CORE §4's literal-with-embedded-link case) |

### 6.3 `hrefs` — the Inline Link Cache

```json
"hrefs": [
  { "predicate": "citesAsEvidence", "target": "RFC 8446" }
]
```

- `hrefs` is an ordered array of `link` elements (§6.2), one per literal `[[Target]]` occurrence (bare or inline-predicated, CORE §7.2) physically embedded inside `text`/`notes`.
- It exists **only** so an in-memory application can iterate a node's inline links in O(n), without parsing `text`/`notes`. It is **not** the graph's edge structure.
- The producer **MUST** populate `hrefs` by scanning the Markdown it converts. A consumer **MUST** treat `hrefs` as already computed and **MUST NOT** parse Text to verify or rebuild it.
- An inline link that is also meant to be a navigable graph edge **MUST** additionally appear in `edges` or `links` (§6.4/§6.5), carrying the same `predicate`/`target` — duplication between `hrefs` and `edges`/`links` is expected for that case. A bare inline mention with no intended navigability (CORE §4's literal-with-embedded-link case) **MAY** exist only in `hrefs`.
- `hrefs` **MAY** be empty or absent when neither `text` nor `notes` embeds a link.

### 6.4 `edges` — the Flat Predicate List

```json
"edges": [
  { "predicate": "replaces", "target": "SSL Protocol" },
  { "predicate": "conformsTo", "target": "RFC 8446" }
]
```

- `edges` is an ordered array of `link` elements (§6.2) — structural edges the producer did **not** group under a display heading.
- Each item **MAY** carry `predicate`; absence means an untyped mention.
- `edges` **MAY** mix any number of distinct predicates. This is exactly the case a single-predicate `links` block cannot represent, e.g. `entity`'s bare semantic edges (`replaces`, `conformsTo`, …, CORE §9.2) that sit directly under `text` with no preceding `##`.
- Item order is preserved and is significant wherever a kind's schema assigns it meaning. `timeline`'s entries (CORE §9.4) are exactly `edges` with no `predicate`, in chronological order; the per-entry display annotation (title, author, date) is rendered from each target's own `attrs`, never stored on the `timeline` node.
- `edges` renders immediately after `text`, before any `links` block, and before `notes`.

### 6.5 `links` — Predicate-Grouped Blocks

```json
"links": {
  "mentions": {
    "title": "Mentions",
    "seq": [
      { "target": "Transport Layer Security" },
      { "target": "Forward Secrecy" }
    ]
  },
  "cites": {
    "title": "Cites",
    "seq": [
      { "target": "RFC 8446" }
    ]
  }
}
```

- `links` is a JSON object keyed by **predicate name** (camelCase, registered, CORE §7.3).
- Each value is a **block**: `title` (optional, defaults to the predicate name capitalized) and `seq` (mandatory, non-empty, ordered array of `link` elements, §6.2).
- Every item in one block shares the block's predicate, so the predicate is read from the key, not repeated on the item (§6.2).
- A `links` block exists exactly when the producer wants a `## <title>` heading grouping that predicate's edges (CORE §9's `## Mentions`, `## Cites`, `## isCitedBy`, `## mentionedIn`). A predicate's edges that are not meant to render under a heading belong in `edges` (§6.4) instead — even if other edges of the same predicate elsewhere do have one.
- **Canonical block order** (§3.4) for rendering: `edges` first, then `links` blocks sorted by `title`, then `notes`. This order is derived at render time and is never stored.
- **Item order within a block is preserved.**
- **Merge (CORE §10 `union`).** A contribution's block for predicate `P` unions into the existing block for `P` by `target`-deduplicated union of `seq`; `title` follows first-writer precedence. `edges` similarly unions by the `(predicate, target)` pair, since two Links to the same target under different predicates are distinct.

### 6.6 Worked Examples

`source` (CORE §9.1) — leading `text`, two titled blocks, no bare `edges`, no `notes`:

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
        { "target": "Transport Layer Security" },
        { "target": "Forward Secrecy" }
      ]
    },
    "cites": {
      "title": "Cites",
      "seq": [
        { "target": "RFC 8446" }
      ]
    }
  }
}
```

`entity` (CORE §9.2) — leading `text`, bare mixed-predicate `edges`, one titled block, trailing `notes`:

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
  "edges": [
    { "predicate": "replaces", "target": "SSL Protocol" },
    { "predicate": "conformsTo", "target": "RFC 8446" }
  ],
  "links": {
    "mentionedIn": {
      "title": "mentionedIn",
      "seq": [
        { "target": "rescorla-2026-tls13" }
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
        { "target": "rescorla-2026-tls13" }
      ]
    }
  }
}
```

`timeline` (CORE §9.4) — no `text`/`notes`/`hrefs`/`links`; entries are untyped, chronologically-ordered `edges`, with display annotation rendered from each target's own `attrs`:

```json
{
  "id": "2026-04",
  "kind": "timeline",
  "attrs": {
    "period": "2026-04",
    "granularity": "monthly"
  },
  "edges": [
    { "target": "rescorla-2026-tls13" },
    { "target": "chen-2026-pqkex" }
  ]
}
```

An inline citation (CORE §7.2) — the producer keeps the markup verbatim in `text`, mirrors it into `hrefs`, and records the same edge in `edges` so it is navigable:

```json
{
  "text": "...the residual risk of zero round-trip resumption, a property [citesAsEvidence:: [[RFC 8446]]] explicitly documents.",
  "hrefs": [
    { "predicate": "citesAsEvidence", "target": "RFC 8446" }
  ],
  "edges": [
    { "predicate": "citesAsEvidence", "target": "RFC 8446" }
  ]
}
```

## 7. Attributes

`attrs` is a JSON object carrying the node's front-matter scalar attributes (CORE §4), keyed by attribute name. Values are JSON scalars or arrays of scalars, as authored.

- The known attributes of a `kind` and their types are declared by the kind's three-level element table in CORE §9 or the profile (CORE §13); that table is the attribute schema.
- A consumer **MUST** preserve attributes it does not recognize (CORE §4). A typesafe binding represents the known attributes as typed fields and unknown attributes in a single overflow map.
- Unknown attributes **MUST** survive a round-trip unchanged (§3.6).

## 8. Extensibility

A profile (CORE §13) extends the model with **data alone**, in priority order:

| Need                                              | Model representation                                                              |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------- |
| program-relevant scalar / flag                        | a new **attribute** (§7) + its declared type in the profile schema                     |
| connectivity / navigation, ungrouped or mixed-predicate | a new **predicate** on a Link (§6.2), placed in `edges` (§6.4)                       |
| connectivity / navigation, grouped under a heading     | a new **predicate**, registered per CORE §7.3, as a `links` key (§6.5)                |
| display heading for a predicate's block                | the block's `title` (§6.5) — existing, no shape change                                |
| new node kind                                           | a new `kind` value (§4) — no shape change                                             |
| new node-level content area                             | **exceptionally**, a new top-level Node member — see below                            |

A new top-level **Node member** is permitted only when the need cannot be met by an attribute, `edges`, or `links`, and the profile **MUST** justify it. Because consumers preserve unknown `attrs` keys and unknown predicates (§3.5), an extension that stays within `attrs`/`edges`/`links` never breaks a conforming binding. A profile **MUST NOT** redefine the node object, the Link abstraction, or the attribute encoding (consistent with CORE §13).

## 9. Conformance Checklist

- [ ] Each CORE node is one node object with `id`, `kind`, `attrs` (§4); `text`/`notes`/`hrefs`/`edges`/`links` present when the kind's schema and content require them.
- [ ] `id` equals the file basename; `kind` is preserved verbatim (§4).
- [ ] `hrefs`, if present, mirrors exactly the links embedded in `text`/`notes`, and is never read by a consumer as a source of navigable edges (§6.3).
- [ ] `edges`, if present, is a flat, ordered array of Links not grouped under a heading; it **MAY** mix predicates (§6.4).
- [ ] `links`, if present, is a JSON object keyed by predicate name; every value is a block of `title` (optional) and `seq` (a non-empty ordered array of Links whose predicate is the key, §6.5).
- [ ] `text` and `notes` are opaque and carried verbatim; no consumer parses them to discover edges (§3.3, §6.1).
- [ ] Every navigable connection — wikilink, predicate, or citation, including the inline predicate form — is a Link present in `edges` or `links` (§6.2, §6.3).
- [ ] Item order within `hrefs`, `edges`, and a `links` block is preserved; the relative order of `edges` and `links` blocks is the canonical rendering order, not stored data (§3.4, §6.5).
- [ ] `attrs` is a JSON object keyed by attribute name; known attributes follow the per-kind schema and unknown attributes are preserved (§7).
- [ ] Unknown attributes and unknown predicates are preserved; extensions add attributes or predicates first, a new top-level Node member only exceptionally (§3.5, §8).
- [ ] Markdown ⇄ model ⇄ patch round-trips without loss of content or connectivity; cosmetic `links`-block order MAY be normalized (§3.6).
