# AST — In-Memory Representation of the Graph

**Status:** Accepted · **Version:** 0.3 · **Date:** 2026-06-18
**Extends:** [`CORE.md`](ARCNET-CORE.md)

This document specifies a **runtime-agnostic in-memory model** for the Markdown knowledge graph
defined by [CORE](ARCNET-CORE.md). It is a *model definition*, not a wire or storage format: it
fixes the shapes an application holds in memory so producers and consumers in any language
interoperate. The model is a **lossless projection** of the on-disk graph — the Markdown files
(CORE §3.1), the document patch (CORE §12), and this model converts without loss of content,
order, or connectivity.

The model is **plain JSON**. The semantic alignment of predicates to standard vocabularies lives in
CORE's predicate registry (CORE §7.3/§7.4), not in the runtime shapes; this model does not carry it.
The model depends on no program, library, or language.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in
RFC 2119. As in CORE §1, each element is declared at three levels — **Mandatory**, **Recommended**,
**Optional**. JSON value types are named per RFC 8259. This document **represents** CORE mechanism
(identity §6, edges §7, citations §8, merge §10) and **MUST NOT** redefine it.

## 2. Purpose and Scope

The model is the **AST every application loads into**. The Markdown graph (CORE §3.1) and a patch
(CORE §12) are two on-disk forms of the same node objects: an application parses either into the
model, operates on it, and serializes it back. Applying a patch to the graph (CORE §12.3) is the
model's central operation — load the patch nodes and the target nodes, then reconcile each by the
kind's merge (CORE §10). Conversion between the forms is lossless (§3.6).

CORE §4 defines a node as **front-matter scalars + a Markdown body**. The body's Markdown semantics
are deliberately loose in CORE; from the *representation* perspective a graph is built from exactly
**three kinds of thing**, and nothing else:

1. **Text** — a content block, opaque to a program and carried verbatim. A consumer **MUST NOT**
   parse it. Prose, atomic statements (CORE §4 literals), emphasized claims, and bullet text are
   all Text.
2. **Links** — the elements that establish connectivity: wikilinks, predicated edges, and
   citations. A **single Link abstraction** (§6.2) represents all three; a citation is a Link whose
   predicate is a citation type (CORE §8). An ordered **sequence of Links** (§6.3) models a 1:N
   relation and is itself a unit of **navigation**, not mere formatting.
3. **Structural elements** — they control grouping and rendering flow. As of this version there is
   exactly **one**: the **Header** (§6.4).

The central design rule follows from this: **connectivity is navigable only because Links are
first-class elements** — never because a consumer scans Text. Any program-relevant meaning a
document carries **MUST** be expressed as an **attribute** or a **Link**; **exceptionally**, and
only with profile justification, as a new structural element (§8). Deep parsing of Markdown by a
consumer is out of scope and is to be avoided.

This document does **not** define how nodes are produced (CORE's non-scope) and does not mandate a
language binding; it fixes the abstract shapes a binding represents.

## 3. Design Invariants

1. **One node, one object.** Each CORE node (one `.md` file) is one **node object** (§4).
2. **Three element kinds only.** A body is a sequence whose elements are **Text**, **Link**,
   **Links** (a collection of Link), or **Header** (§6). No other body element kind exists in this
   version.
3. **No consumer-side Markdown parsing.** Text is opaque (§2.1). A consumer reads connectivity from
   **Link** elements alone; it never extracts edges from Text.
4. **Order is preserved.** The body is an array; element order in memory equals the Markdown file.
   Attribute maps and the link set are unordered.
5. **Open vocabularies, preserve unknowns.** `kind`, attribute names, predicate names, header
   text, and any future element `type` are open. A consumer **MUST** preserve attributes and
   element types it does not recognize (CORE §4), so a profile (CORE §13) extends the model with
   data alone (§8).
6. **Lossless conversion.** Markdown ⇄ model ⇄ patch (CORE §12) preserves content, order,
   front-matter, headers, and every Link. Insignificant whitespace **MAY** be normalized.

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
  "body": [ /* elements (§6) */ ]
}
```

**Members**
- `id` (mandatory) — the node identity, equal to the file basename without `.md` (CORE §3.2, §6).
- `kind` (mandatory) — the node kind (CORE §4); a core kind or a profile kind. Open vocabulary.
- `attrs` (mandatory) — the front-matter scalar attributes (§7), as a JSON object, **excluding**
  `kind`. The identity field (`id`/`title`) **MAY** be repeated here for convenience.
- `body` (mandatory) — the body as an ordered element sequence (§6). **MAY** be empty `[]`.

The leading `# <title>` heading of an on-disk node repeats `attrs.title`/`id` and is **derivable**;
it is **not** stored as a body element (consistent with the patch format, CORE §12.2). It is **not**
a structural Header in the §6.4 sense.

A consumer that traverses the graph collects a node's outgoing edges by iterating `body` for `link`
and `links` elements (and any link-valued attribute, §7) — an O(n) walk over typed elements, with no
Markdown parsing. A binding **MAY** additionally cache a flat link list, but the body remains
authoritative.

## 5. Graph

A **graph** is an ordered sequence of node objects — a plain JSON array.

```json
{ "nodes": [ /* node objects (§4) */ ] }
```

- `nodes` (mandatory) — array of node objects. Order is filing convenience; identity (CORE §6) is
  the sole index. Basenames are unique (CORE §3.2), so a consumer **MAY** key nodes by `id`. A
  binding **MAY** represent the graph as a bare top-level array instead of the wrapper object.

A graph **MAY** be partial; a Link target absent from the in-memory set is a dangling reference
resolved against the full on-disk graph, not a model error.

## 6. Body Elements

`body` is an array of **elements**. Every element is an object with a `type` discriminator naming
one of the four element kinds. A `type` a consumer does not recognize is preserved verbatim (§3.5).

| `type`   | Category (§2)      | Members                        |
| -------- | ------------------ | ------------------------------ |
| `text`   | Text               | `text`                         |
| `link`   | Link               | `predicate`, `target`, `alias` |
| `links`  | Link (1:N)         | `items` (array of `link`)      |
| `header` | Structural element | `level`, `text`                |

### 6.1 `text`

```json
{ "type": "text", "text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption." }
```

- `text` (mandatory) — an opaque Markdown run, carried verbatim. The model does **not** decompose
  it into paragraphs, emphasis, lists, or marks, and a consumer **MUST NOT** parse it (§3.3).
- A Text block **MAY** contain inline `[[links]]` as authored Markdown. These are **content**, not
  navigable connectivity; if connectivity is intended to be navigable, the producer **MUST** also
  emit it as a `link`/`links` element (§2). A consumer never derives edges from Text.
- An emphasized text, innher Markdown formatting is **Text**. The model does **not**
  elevate it. If an application needs the element of Markdown as structured data, it lifts it into an **attribute**
  (§7) — a producer/profile concern, never a body element.

### 6.2 `link` — the Unified Edge

A Link is the one abstraction for every connection. CORE's three link kinds map to a single shape:

| CORE form (§7, §8)                  | `link` representation                                            |
| ----------------------------------- | ---------------------------------------------------------------- |
| untyped mention `[[Target]]`        | `{ "type":"link", "target":"Target" }`                           |
| display alias `[[Target\|text]]`    | `{ …, "target":"Target", "alias":"text" }`                       |
| predicated edge `pred:: [[Target]]` | `{ "type":"link", "predicate":"pred", "target":"Target" }`       |
| citation `citesAsEvidence:: [[X]]`  | `{ "type":"link", "predicate":"citesAsEvidence", "target":"X" }` |

- `target` (mandatory) — the target basename (CORE §3.2).
- `predicate` (recommended) — the edge predicate (camelCase, registered, CORE §7.3). Absent for an
  untyped mention. A **citation** is a Link whose `predicate` is a citation type (CORE §8); it needs
  no separate element kind, though a binding **MAY** classify it from the predicate's `cito:`
  namespace.
- `alias` (optional) — the display text of `[[Target|text]]` (CORE §7.1).

### 6.3 `links` — a Collection of Links

```json
{ "type": "links", "items": [
  { "type": "link", "predicate": "mentions", "target": "Transport Layer Security" },
  { "type": "link", "predicate": "mentions", "target": "Forward Secrecy" } ] }
```

- `items` (mandatory) — an ordered array of `link` elements. A `links` models a **1:N relation** and
  is a unit of navigation (§2.2), not formatting. A single CORE body-form edge **MAY** be a `link`
  directly or a one-item `links`; producers **SHOULD** use `links` for a bulleted list of edges.
- A `links` contains only `link` elements; it does not nest `text`, `header`, or `links`.

### 6.4 `header` — the Structural Element

```json
{ "type": "header", "level": 2, "text": "Mentions" }
```

- `level` (mandatory) — the Markdown heading depth (e.g. `2` for `## Mentions`).
- `text` (mandatory) — the heading text.
- A Header is a **flat marker** in the body sequence that controls grouping and rendering flow; it
  is **not** a container. The elements it groups are simply those that follow it until the next
  Header. This is the **only** structural element in this version (§2.3, §8).

### 6.5 Worked Example

The `source` node of CORE §9.1 in full:

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
  "body": [
    { "type": "text", "text": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption." },
    { "type": "header", "level": 2, "text": "Mentions" },
    { "type": "links", "items": [
      { "type": "link", "predicate": "mentions", "target": "Transport Layer Security" },
      { "type": "link", "predicate": "mentions", "target": "Forward Secrecy" } ] },
    { "type": "header", "level": 2, "text": "Cites" },
    { "type": "links", "items": [
      { "type": "link", "predicate": "cites", "target": "RFC 8446" } ] }
  ]
}
```

## 7. Attributes

`attrs` is a JSON object carrying the node's front-matter scalar attributes (CORE §4), keyed by
attribute name. Values are JSON scalars or arrays of scalars, as authored.

- The known attributes of a `kind` and their types are declared by the kind's three-level element
  table in CORE §9 or the profile (CORE §13); that table is the attribute schema.
- A consumer **MUST** preserve attributes it does not recognize (CORE §4). A typesafe binding
  represents the known attributes as typed fields and unknown attributes in a single overflow map.
- Unknown attributes **MUST** survive a round-trip unchanged (§3.6).

## 8. Extensibility

A profile (CORE §13) extends the model with **data alone**, in priority order:

| Need                             | Model representation                                                 |
| -------------------------------- | -------------------------------------------------------------------- |
| program-relevant scalar / flag   | a new **attribute** (§7) + its declared type in the profile schema   |
| connectivity / navigation        | a new **Link** `predicate` (§6.2), registered per CORE §7.3          |
| 1:N navigation                   | a **`links`** collection under a **`header`** (§6.3/§6.4) — existing |
| new node kind                    | a new `kind` value (§4) — no shape change                            |
| new rendering/grouping structure | **exceptionally**, a new body element `type` — see below             |

A new **structural element type** is permitted only when the need cannot be met by an attribute or a
Link, and the profile **MUST** justify it. Because consumers preserve unknown element `type`s and
unknown attributes (§3.5), such an extension never breaks a conforming binding: a typesafe binding
models body elements as a tagged union over the §6 `type`s plus an *unknown* variant that retains
the raw object. A profile **MUST NOT** redefine the node object, the four element kinds, the Link
abstraction, or the attribute encoding (consistent with CORE §13).

## 9. Conformance Checklist

- [ ] Each CORE node is one node object with `id`, `kind`, `attrs`, `body` (§4).
- [ ] `id` equals the file basename; `kind` is preserved verbatim (§4).
- [ ] `body` is an ordered array of only `text`, `link`, `links`, `header` elements (§6); the leading
      title heading is derived, not stored (§4).
- [ ] Text is opaque and carried verbatim; no consumer parses it for edges (§3.3, §6.1).
- [ ] Every navigable connection is a `link` (wikilink, predicate, or citation, unified) and 1:N
      relations use `links` (§6.2, §6.3).
- [ ] `attrs` is a JSON object keyed by attribute name; known attributes follow the per-kind
      schema and unknown attributes are preserved (§7).
- [ ] Unknown attributes and unknown element `type`s are preserved; extensions add attributes or
      links first, structural elements only exceptionally (§3.5, §8).
- [ ] Markdown ⇄ model ⇄ patch round-trips without loss of Text, order, or Links (§3.6).
