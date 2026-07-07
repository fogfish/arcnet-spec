# AST — In-Memory Representation of the Graph

**Status:** Draft · **Version:** 0.6 · **Date:** 2026-07-07
**Extends:** [`CORE.md`](ARCNET-CORE.md)

> Revision note (0.4 → 0.5): follows CORE v0.5's move to a predicate-first model. `kind` → `type`. The fixed `text`/`notes` duo is retired: CORE now registers an *open* set of `text`-role predicates (`text`, `abstract`, `definition`, `relevance`, `notes`, and whatever a profile adds), so `texts` becomes a predicate-keyed map instead of two hardcoded slots. `links` is retired as a separate container: CORE now fixes a predicate's role (`edge` vs `link`) once, on its own schema node, rather than letting a document choose per occurrence — so the heading-grouped-vs-flat distinction is a rendering derivation from the predicate's own schema, not a second storage shape. `attrs` becomes a map of predicate name to an ordered, non-empty array of `Predicate` (§7) — a predicate's cardinality (single- vs multi-valued) is a merge/schema concern (CORE §9.3), not a storage-shape concern, mirroring how RDF itself carries no built-in cardinality distinction. A new **Schema Index** (§8) formalizes how a consumer resolves a predicate's role/merge — sourced from CORE's own `_schema/predicates/`/`_schema/types/` nodes, which need no special AST shape: they are ordinary Node Objects.
>
> Revision note (0.5 → 0.6): follows CORE v0.7's retirement of the `## Recommended` tier. A `Class` node's `edges` now carry only `required`/`optional` predicates (§6.5, §8) — the `recommended` predicate no longer exists.

This document specifies a **runtime-agnostic in-memory model** for the Markdown knowledge graph defined by [CORE](ARCNET-CORE.md). It is a *model definition*, not a wire or storage format: it fixes the shapes an application holds in memory so producers and consumers in any language interoperate. The model is a **lossless projection** of the on-disk graph — the Markdown files (CORE §6), the document patch (CORE §14), and this model convert without loss of content or connectivity.

The model is **plain JSON**. The semantic alignment of predicates to standard vocabularies lives in CORE's predicate registry (CORE §9.1/§10), not in the runtime shapes; this model does not carry it. The model depends on no program, library, or language.

## 1. Conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are used as in RFC 2119. As in CORE §1, each element is declared at three levels — **Mandatory**, **Recommended**, **Optional**. JSON value types are named per RFC 8259. This document **represents** CORE mechanism (identity §7, edges §8, schema/merge §9, citations §12) and **MUST NOT** redefine it.

## 2. Purpose and Scope

The model is the **AST every application loads into**. The Markdown graph (CORE §6) and a patch (CORE §14) are two human-readable on-disk forms of the same node objects: an application parses either into the model, operates on it, and serializes it back. Applying a patch to the graph (CORE §14.3) is the model's central operation — load the patch nodes and the target nodes, then reconcile each predicate by its own declared merge behavior (CORE §9.3). Conversion between the forms is lossless (§3.6).

CORE §5 defines a node as a bag of predicates, each rendered at a position fixed by its declared **role** — `meta`, `text`, `href`, `edge`, or `link`. From the *representation and navigation* perspective this collapses to four containers, because two of the five roles (`href` and `edge`/`link`) already share one shape:

1. **`attrs`** — role `meta`. A predicate-keyed map, each value an ordered sequence of **`Predicate`** (§7) — a predicate's value can be a plain scalar or link-shaped, and its cardinality is a schema/merge concern, not a shape concern.
2. **`texts`** — role `text`. A predicate-keyed map of opaque prose strings (§6.1).
3. **`hrefs`** — role `href`. An ordered array of **`Link`** (§6.2) — a cache of what prose embeds, never itself navigable.
4. **`edges`** — roles `edge` and `link`, unified. An ordered, possibly mixed-predicate array of `Link` (§6.2) — CORE v0.5 fixes a predicate's role once, on its own schema node (CORE §9.1), so whether a predicate's occurrences render flat or grouped under a heading is a rendering derivation from the Schema Index (§8), never a storage choice.

Any program-relevant meaning a document carries **MUST** be expressed as a `Predicate` (in `attrs`), a text value (in `texts`), or a `Link` (in `hrefs`/`edges`); **exceptionally**, and only with profile justification, as a new top-level Node member (§9). Deep parsing of Markdown by a consumer is out of scope and is to be avoided.

This document does **not** define how nodes are produced (CORE's non-scope) and does not mandate a language binding; it fixes the abstract shapes a binding represents.

## 3. Design Invariants

1. **One node, one object.** Each CORE node (one `.md` file) is one **node object** (§4).
2. **Three element kinds.** Content is a **Predicate** (role `meta` — §7), **Text** (role `text` — §6.1), or **Link** (roles `href`/`edge` — the latter absorbing what CORE used to render as a separate `link` role — §6.2), held in `attrs`, `texts`, `hrefs`, or `edges` (§4). No other structural container exists in this version.
3. **No consumer-side Markdown parsing for connectivity.** Text is opaque (§6.1). A consumer reads connectivity from `edges` **alone**; it never extracts an edge from Text. `hrefs` is populated by the producer by scanning Text, but a consumer treats it as already computed and **MUST NOT** itself parse Text to derive or verify it (§6.3). Neither `hrefs` nor a link-shaped `Predicate` inside `attrs` (§7) is ever treated as a source of navigable edges — both are informative references a producer may promote into an `edges` entry, never graph structure themselves.
4. **Item order is preserved; grouping is derived, not stored.** Order within `hrefs`, within `edges`, and within one `attrs[predicate]` sequence is the on-disk order and is significant wherever a predicate's schema assigns it meaning (e.g. `entries`'s chronological order, CORE §10.7). Whether a predicate's `edges` occurrences render as flat, heterogeneous bullets or as one grouped `## <label>` block, and the relative order of any such groups, is derived at render time from the Schema Index (§8) — never stored.
5. **Open vocabularies, preserve unknowns.** `type`, attribute names, and predicate names are open. A consumer **MUST** preserve `attrs` entries and predicates it does not recognize (CORE §5), so a profile (CORE §15) extends the model with data alone (§9).
6. **Lossless conversion.** Markdown ⇄ model ⇄ patch (CORE §14) preserves content, connectivity, front-matter, item order within `attrs`/`hrefs`/`edges`, and every `Predicate`'s/`Link`'s fields. Insignificant whitespace and the relative order/grouping of `edges` **MAY** be normalized (§3.4).

## 4. Node Object

```json
{
  "id": "rescorla-2026-tls13",
  "type": "source",
  "attrs": {
    "title": [ { "value": "TLS 1.3: Design and Rationale" } ],
    "authors": [ { "value": "Eric Rescorla" } ],
    "published": [ { "value": "2026-04-12" } ],
    "url": [ { "value": "https://example.org/tls13-design" } ],
    "tags": [ { "value": "tls" }, { "value": "protocols" } ]
  },
  "texts": {
    "abstract": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption."
  },
  "edges": [
    { "predicate": "mentions", "target": "Transport Layer Security" },
    { "predicate": "mentions", "target": "Forward Secrecy" },
    { "predicate": "cites", "target": "RFC 8446" }
  ]
}
```

**Members**
- `id` (mandatory) — the node identity, equal to the file basename without `.md`; mirrors CORE's `@id` (CORE §7, §10.1).
- `type` (mandatory) — the node's class; mirrors CORE's `@type` (CORE §5, §10.1). A core type or a profile type. Open vocabulary.
- `attrs` (optional) — every role-`meta` predicate, keyed by predicate name, each value a non-empty, ordered array of `Predicate` (§7). Excludes `id`/`type` — CORE's `@id`/`@type` are always the two dedicated top-level members above, never duplicated into `attrs`. **MAY** be empty `{}` or absent.
- `texts` (optional) — every role-`text` predicate, keyed by predicate name, each value the opaque prose string (§6.1). **MAY** be empty `{}` or absent.
- `hrefs` (optional) — an ordered array of `Link` (§6.2) embedded inline in any `texts` value (§6.3). **MAY** be empty or absent.
- `edges` (optional) — an ordered, possibly mixed-predicate array of `Link` (§6.2): every role-`edge` and role-`link` predicate's occurrences, unified (§6.4). **MAY** be empty or absent.

The leading `# <heading>` of an on-disk node is derivable — from `id`, or from a type-specific display predicate (e.g. `heading` for `timeline`, CORE §10.7) — and is **not** stored.

A consumer that traverses the graph collects a node's outgoing edges by reading `edges` **alone** — an O(n) walk with no Markdown parsing. `hrefs`, and any `target`-bearing `Predicate` inside `attrs` (§7), are excluded from this walk: `hrefs` answers "what does the prose embed," not "what does this node connect to," and a link-shaped `attrs` entry is informative raw data, not yet an asserted relationship (§3 invariant 3) — a producer that wants either navigable promotes it into an `edges` entry (mirroring how the inline predicated form promotes an `href` into `edges`, §6.2). A binding **MAY** additionally group `edges` by predicate at load time, using the Schema Index (§8), for ergonomic per-predicate access — but the stored shape stays one flat array.

## 5. Graph

A **graph** is an ordered sequence of node objects — a plain JSON array.

```json
{ "nodes": [ /* node objects (§4) */ ] }
```

- `nodes` (mandatory) — array of node objects. Order is filing convenience; identity (CORE §7) is the sole index. Basenames are unique (CORE §7.1), so a consumer **MAY** key nodes by `id`. A binding **MAY** represent the graph as a bare top-level array instead of the wrapper object.

A graph **MAY** be partial; a Link target absent from the in-memory set is a dangling reference resolved against the full on-disk graph, not a model error.

## 6. Text and Links

### 6.1 `texts`

```json
"texts": {
  "definition": "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network.",
  "notes": "Widely deployed since 2018; superseded TLS 1.2 as the IETF-recommended baseline."
}
```

- `texts` is a JSON object keyed by **predicate name** (role `text`; CORE §10.2/§10.7), each value an opaque Markdown run, carried verbatim. The model does **not** decompose it into paragraphs, emphasis, lists, or marks, and a consumer **MUST NOT** parse it (§3.3).
- CORE registers an *open* set of role-`text` predicates (`text`, `abstract`, `definition`, `relevance`, `notes`, `description`, and whatever a profile adds — CORE §10.2/§10.7/§10.8); a node carries exactly the ones its type's schema requires/recommends/permits (CORE §11), each under its own key. This replaces AST v0.4's fixed leading `text`/trailing `notes` pair, which assumed at most two text-role fields per kind — no longer true once text-role predicates are independently named and open-ended.
- Render order among multiple `texts` entries on one node follows CORE §5's placement rule (each `text`-role predicate is its own paragraph) and the type's own schema declaration order; it is not carried by the model.
- Either **MAY** contain inline `[[links]]` or the inline predicate form (CORE §8.2) as authored Markdown. These are mirrored into `hrefs` (§6.3); they are not themselves navigable connectivity unless also present in `edges`.
- Emphasis and other inline Markdown formatting are part of Text. The model does **not** elevate them. If an application needs a piece of Markdown as structured data, it lifts it into an **attribute** (§7) — a producer/profile concern, never a Text field.

### 6.2 `Link` — the Unified Edge/Href Shape

A `Link` is the one abstraction for every connection. Its shape is the same wherever it is stored (§6.3, §6.4):

```json
{ "predicate": "mentions", "target": "Transport Layer Security" }
```

- `target` (mandatory) — the target basename (CORE §7.1).
- `predicate` (recommended) — the edge predicate (camelCase, registered, CORE §8.3). Absent for an untyped mention.
- `alias` (optional) — the display text of `[[Target|text]]` (CORE §8.1).
- A **citation** is a `Link` whose `predicate` is a citation type (CORE §10.6); it needs no separate shape.

CORE's forms map onto `hrefs`/`edges` as follows. Unlike AST v0.4, there is no third row for "grouped under a heading" — that is now a rendering derivation from the predicate's own schema role (§8), not a distinct representation:

| CORE form (§8, §10.6)                                                           | Representation                                                                       |
| -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| untyped mention `[[Target]]`, a standalone body bullet                             | `{ "target": "Target" }` in `edges` (§6.4)                                              |
| display alias `[[Target\|text]]`                                                   | adds `"alias": "text"` to the Link wherever it is filed                                 |
| predicated edge `pred:: [[Target]]` — bare, list, or grouped under a heading (role `edge` or `link`, resolved via §8) | `{ "predicate": "pred", "target": "Target" }` in `edges` (§6.4)                          |
| citation `citesAsEvidence:: [[X]]`                                                  | `{ "predicate": "citesAsEvidence", "target": "X" }` in `edges`                           |
| inline predicated edge `[pred:: [[Target]]]` inside prose, navigable                | `{ "predicate": "pred", "target": "Target" }` in **both** `hrefs` (§6.3) and `edges`     |
| bare inline mention `[[Target]]` inside prose, not meant to be a navigable edge     | `{ "target": "Target" }` in `hrefs` **only** (CORE §5's literal-with-embedded-link case) |

### 6.3 `hrefs` — the Inline Link Cache

```json
"hrefs": [
  { "predicate": "citesAsEvidence", "target": "RFC 8446" }
]
```

- `hrefs` is an ordered array of `Link` (§6.2), one per literal `[[Target]]` occurrence (bare or inline-predicated, CORE §8.2) physically embedded inside any `texts` value.
- It exists **only** so an in-memory application can iterate a node's inline links in O(n), without parsing `texts`. It is **not** the graph's edge structure.
- The producer **MUST** populate `hrefs` by scanning the Markdown it converts. A consumer **MUST** treat `hrefs` as already computed and **MUST NOT** parse Text to verify or rebuild it.
- An inline link that is also meant to be a navigable graph edge **MUST** additionally appear in `edges` (§6.4), carrying the same `predicate`/`target` — duplication between `hrefs` and `edges` is expected for that case. A bare inline mention with no intended navigability (CORE §5's literal-with-embedded-link case) **MAY** exist only in `hrefs`.
- `hrefs` **MAY** be empty or absent when no `texts` value embeds a link. `hrefs` is a single flat array for the whole node — it is **not** scoped per `texts` key, since it is a non-navigable cache, not structural data (§3.3).

### 6.4 `edges` — the Unified Predicate List

```json
"edges": [
  { "predicate": "mentions", "target": "Transport Layer Security" },
  { "predicate": "mentions", "target": "Forward Secrecy" },
  { "predicate": "cites", "target": "RFC 8446" },
  { "predicate": "replaces", "target": "SSL Protocol" },
  { "predicate": "conformsTo", "target": "RFC 8446" }
]
```

- `edges` is a single ordered array of `Link` (§6.2) carrying **every** structural connection on the node, regardless of whether its predicate's schema declares role `edge` (heterogeneous) or role `link` (grouped under a heading). AST v0.4 kept these as two containers (`edges`/`links`) because a producer could choose, per document, whether a predicate's occurrences got a heading. CORE v0.5 fixes a predicate's role once, on its own schema node (CORE §9.1) — so the split is now a pure rendering derivation (§8), not worth storing twice.
- Each item **MAY** carry `predicate`; absence means an untyped mention. `edges` **MAY** mix any number of distinct predicates — this is exactly the case a single-predicate grouped block cannot represent, and is also how a type's bare, ungrouped semantic edges (e.g. `entity`'s `replaces`, `conformsTo`, CORE §10.5) sit alongside grouped ones like `mentionedIn` in one array.
- Item order is preserved and is significant wherever a predicate's schema assigns it meaning. `timeline`'s `entries` (CORE §10.7) is exactly `edges` filtered to that one predicate, in chronological order; the per-entry display annotation (title, author, date) is rendered from each target's own `attrs`, never stored on the `timeline` node.
- **Rendering.** To serialize back to Markdown, a producer partitions `edges` by looking up each item's predicate in the Schema Index (§8): role-`edge` predicates render as heterogeneous bullets in on-disk order; role-`link` predicates are grouped by predicate into one `## <label>` block each (`label` from the predicate's own schema node, defaulting to the capitalized predicate name), blocks ordered by label. When a type's whole body is one role-`link` predicate, the `## ` heading **MAY** be omitted (CORE §5).
- **Merge (CORE §9.3 `union`).** A contribution's edges union into the existing `edges` by the `(predicate, target)` pair — two Links to the same target under different predicates are distinct and both survive. This applies uniformly regardless of the predicate's role.

### 6.5 Worked Examples

`source` (CORE §11.2) — `attrs` and `texts`, mixed-predicate `edges` (two `mentions`, one `cites` — the former group under `## Mentions`, the latter under `## Cites` at render time, per each predicate's own schema role):

```json
{
  "id": "rescorla-2026-tls13",
  "type": "source",
  "attrs": {
    "title": [ { "value": "TLS 1.3: Design and Rationale" } ],
    "authors": [ { "value": "Eric Rescorla" } ],
    "published": [ { "value": "2026-04-12" } ],
    "url": [ { "value": "https://example.org/tls13-design" } ],
    "tags": [ { "value": "tls" }, { "value": "protocols" } ]
  },
  "texts": {
    "abstract": "A design retrospective on the TLS 1.3 handshake and the residual risk of zero round-trip resumption."
  },
  "edges": [
    { "predicate": "mentions", "target": "Transport Layer Security" },
    { "predicate": "mentions", "target": "Forward Secrecy" },
    { "predicate": "cites", "target": "RFC 8446" }
  ]
}
```

`entity` (CORE §11.3) — two `texts` entries (`definition`, `notes`), `edges` mixing bare semantic predicates (`replaces`, `conformsTo` — render flat) with a grouped one (`mentionedIn` — renders under its own `## mentionedIn` heading):

```json
{
  "id": "Transport Layer Security",
  "type": "entity",
  "attrs": {
    "category": [
      { "value": "independent" },
      { "value": "abstract" },
      { "value": "occurrent" },
      { "value": "script" }
    ],
    "aliases": [ { "value": "TLS" }, { "value": "TLS 1.3" } ],
    "tags": [ { "value": "cryptography" } ]
  },
  "texts": {
    "definition": "A cryptographic protocol that establishes an authenticated, confidential channel over an untrusted network.",
    "notes": "Widely deployed since 2018; superseded TLS 1.2 as the IETF-recommended baseline."
  },
  "edges": [
    { "predicate": "replaces", "target": "SSL Protocol" },
    { "predicate": "conformsTo", "target": "RFC 8446" },
    { "predicate": "mentionedIn", "target": "rescorla-2026-tls13" }
  ]
}
```

`resource` (CORE §11.4):

```json
{
  "id": "RFC 8446",
  "type": "resource",
  "attrs": {
    "ref": [ { "value": "standard" } ],
    "authors": [ { "value": "Eric Rescorla" } ],
    "year": [ { "value": 2018 } ],
    "url": [ { "value": "https://www.rfc-editor.org/rfc/rfc8446" } ],
    "status": [ { "value": "read" } ]
  },
  "texts": {
    "relevance": "The normative specification of TLS 1.3."
  },
  "edges": [
    { "predicate": "isCitedBy", "target": "rescorla-2026-tls13" }
  ]
}
```

`timeline` (CORE §11.5) — no `texts`/`hrefs`; a single role-`link` predicate (`entries`), so it renders unheaded (CORE §5):

```json
{
  "id": "2026-04",
  "type": "timeline",
  "attrs": {
    "granularity": [ { "value": "monthly" } ]
  },
  "edges": [
    { "predicate": "entries", "target": "rescorla-2026-tls13" },
    { "predicate": "entries", "target": "chen-2026-pqkex" }
  ]
}
```

`Property`/`Class` schema nodes (CORE §9.1/§9.2) need **no special shape**: a predicate or type is an ordinary node object — `attrs` carries its own `role`/`merge`/`aligned` (themselves registered predicates, CORE §10.8), `texts.description` carries its prose, and — for a `Class` — `edges` carries its `required`/`optional` relations to other predicates, exactly like any other role-`link`/role-`edge` predicate:

```json
{
  "id": "isPartOf",
  "type": "Property",
  "attrs": {
    "role": [ { "value": "edge" } ],
    "merge": [ { "value": "union" } ],
    "aligned": [ { "value": "dcterms:isPartOf" } ]
  },
  "texts": {
    "description": "Asserts that the subject is a component or member of the whole named by the target — composition (part–whole), not generalization. See [[broader]] for is-a."
  },
  "hrefs": [
    { "target": "broader" }
  ]
}
```

```json
{
  "id": "entity",
  "type": "Class",
  "texts": {
    "description": "A node for a subject occurring in sources, typed by Sowa category."
  },
  "edges": [
    { "predicate": "required", "target": "category" },
    { "predicate": "required", "target": "definition" },
    { "predicate": "required", "target": "mentionedIn" },
    { "predicate": "optional", "target": "aliases" },
    { "predicate": "optional", "target": "tags" }
  ]
}
```

An inline citation (CORE §8.2) — the producer keeps the markup verbatim in `texts`, mirrors it into `hrefs`, and records the same edge in `edges` so it is navigable:

```json
{
  "texts": {
    "text": "...the residual risk of zero round-trip resumption, a property [citesAsEvidence:: [[RFC 8446]]] explicitly documents."
  },
  "hrefs": [
    { "predicate": "citesAsEvidence", "target": "RFC 8446" }
  ],
  "edges": [
    { "predicate": "citesAsEvidence", "target": "RFC 8446" }
  ]
}
```

## 7. Attributes

`attrs` is a JSON object carrying the node's role-`meta` predicates (CORE §5), keyed by predicate name. Each value is a non-empty, ordered array of `Predicate`:

```json
{ "value": "Eric Rescorla" }
```

- `value` (present for a scalar predicate) — a JSON scalar (string, number, or boolean), as authored.
- `target` (present instead of `value`, for a link-shaped predicate) — the target basename (CORE §7.1), for a meta-role predicate whose value is a reference to another node rather than a plain scalar (e.g. an attribute a profile later resolves into a structural edge). This is informative only — like `hrefs` (§6.3), it is never itself treated as a source of navigable edges (§3 invariant 3); a producer that wants it navigable promotes it into an `edges` entry.
- `alias` (optional) — a display alias, meaningful only alongside `target`.

Exactly one of `value`/`target` is present per `Predicate`.

- **Every `attrs` entry is a sequence, even single-valued predicates** — e.g. `attrs.published` is `[ { "value": "2026-04-12" } ]`, never a bare value. This is deliberate: RDF itself carries no built-in cardinality distinction — a subject may carry any number of triples for the same predicate, and "at most one value" is a schema-level constraint, not a shape one (mirrored by `owl:FunctionalProperty` needing engine support beyond RDFS itself). Keeping `attrs` uniformly a sequence means the merge algorithm (CORE §9.3) never branches on "is this predicate single- or multi-valued" — `firstWriteWin`/`lastWriteWin` simply keep the sequence at length 1; `union` grows it. A binding **MAY** still expose a single-valued convenience accessor for a predicate it knows to be cardinality-1, derived from `attrs[predicate][0]`.
- The known attributes of a `type` and each attribute's `Predicate` shape are declared by the type's schema node (CORE §9.2, §11) and each predicate's own schema node (CORE §9.1). A consumer **MUST** preserve `attrs` entries it does not recognize (CORE §5). A typesafe binding represents known attributes as typed fields and unknown ones in a single overflow map.
- Unknown `attrs` entries **MUST** survive a round-trip unchanged (§3.6).

## 8. Schema Index

CORE v0.5 makes predicates and types ordinary graph nodes (`Property`/`Class`, CORE §9) rather than external prose — so they need **no dedicated AST shape**; a `_schema/predicates/<name>.md` or `_schema/types/<name>.md` file loads as an ordinary Node Object (§4), exactly as shown in §6.5.

A consumer builds a derived, in-memory **Schema Index** by reading every `Property`/`Class` node once per graph load — a plain map from predicate/type name to its declared shape, used to resolve the things this model deliberately does not store on each occurrence:

```json
{
  "predicates": {
    "isPartOf": { "role": "edge", "merge": "union", "aligned": "dcterms:isPartOf", "label": null },
    "mentionedIn": { "role": "link", "merge": "union", "aligned": "schema:subjectOf", "label": null },
    "required": { "role": "link", "merge": "union", "aligned": null, "label": "Requires" }
  },
  "types": {
    "entity": {
      "required": ["category", "definition", "mentionedIn"],
      "optional": ["aliases", "tags"]
    }
  }
}
```

- `predicates[name].role`/`.merge` drive §6.4's rendering partition and §6.4/§7's merge behavior; `.label` overrides the default (capitalized name) heading for a role-`link` predicate.
- `types[name]` is read directly off the type's `Class` node — its `required`/`optional` `edges` (§6.5) — not duplicated into a separate schema description.
- The index is a **read-time convenience**, not part of the wire model: nothing in §4's Node Object depends on it existing, and a consumer that never needs to render Markdown or validate a type **MAY** skip building it entirely.

## 9. Extensibility

A profile (CORE §15) extends the model with **data alone**, in priority order:

| Need                                                    | Model representation                                                                                                                        |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| program-relevant scalar / flag                              | a new **`attrs`** entry (§7) + its predicate registered per CORE §9.1                                                                          |
| a new prose field                                           | a new **`texts`** key (§6.1), its predicate registered role `text`                                                                              |
| connectivity / navigation, any placement                    | a new **predicate** on a `Link` (§6.2), placed in `edges` (§6.4) — flat vs grouped is a schema/rendering concern, not a model-shape choice     |
| display heading for a role-`link` predicate's block          | the predicate's own `label` (CORE §9.1, §10.8) — existing mechanism, no shape change                                                            |
| new node type                                                | a new `type` value (§4) — no shape change                                                                                                       |
| new node-level content area                                  | **exceptionally**, a new top-level Node member — see below                                                                                      |

A new top-level **Node member** is permitted only when the need cannot be met by `attrs`, `texts`, `hrefs`, or `edges`, and the profile **MUST** justify it. Because consumers preserve unknown `attrs` entries and unknown predicates (§3.5), an extension that stays within these four members never breaks a conforming binding. A profile **MUST NOT** redefine the node object, the `Predicate`/`Link` abstractions, or the attribute encoding (consistent with CORE §15).

## 10. Conformance Checklist

- [ ] Each CORE node is one node object with `id`, `type` (§4); `attrs`/`texts`/`hrefs`/`edges` present when the type's schema and content require them.
- [ ] `id` equals the file basename; `type` is preserved verbatim (§4).
- [ ] Every `attrs` entry is a non-empty, ordered array of `Predicate`, each with exactly one of `value`/`target` (§7).
- [ ] `hrefs`, if present, mirrors exactly the links embedded in any `texts` value; neither `hrefs` nor a link-shaped `attrs` entry is ever read by a consumer as a source of navigable edges (§3, §6.3, §7).
- [ ] `edges`, if present, is a single flat, ordered array of `Link` carrying every role-`edge` and role-`link` predicate's occurrences; it **MAY** mix predicates (§6.4).
- [ ] `texts` values are opaque and carried verbatim; no consumer parses them to discover edges (§3.3, §6.1).
- [ ] Every navigable connection — wikilink, predicate, or citation, including the inline predicate form — is a `Link` present in `edges` (§6.2, §6.3).
- [ ] Item order within `attrs[predicate]`, `hrefs`, and `edges` is preserved; whether/how `edges` groups under a heading at render time comes from the Schema Index (§8), not stored data (§3.4).
- [ ] Unknown `attrs` entries and unknown predicates are preserved; extensions add an `attrs`/`texts` entry or an `edges` predicate first, a new top-level Node member only exceptionally (§3.5, §9).
- [ ] Markdown ⇄ model ⇄ patch round-trips without loss of content or connectivity; cosmetic `edges`-grouping order **MAY** be normalized (§3.6).
