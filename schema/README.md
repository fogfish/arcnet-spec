# Schema Bootstrap Patches

Three [document patches](../../ARCNET-CORE.md) (CORE §14), one per specification, that together
serialize the *entire* `_schema/predicates/` and `_schema/types/` registry those specifications
define. An app applies them (CORE §14.3) to populate — or update — a graph's `_schema/` folder,
instead of hand-authoring each `Property`/`Class` node.

| Patch                         | Serializes                                                          | Depends on          |
| ------------------------------ | -------------------------------------------------------------------- | -------------------- |
| [`core.md`](core.md)                             | [`../../ARCNET-CORE.md`](../../ARCNET-CORE.md)'s full core vocabulary — 49 predicates (§10) + 4 types (§11: `source`, `entity`, `resource`, `timeline`) | none |
| [`domain-article.md`](domain-article.md)         | [`../../ARCNET-DOMAIN-ARTICLE.md`](../../ARCNET-DOMAIN-ARTICLE.md) — 17 predicates + `hypothesis`/`aporia` (§4, §2), plus its extension of `source` (§3.1) | `core.md` |
| [`domain-core-thought.md`](domain-core-thought.md) | [`../../ARCNET-DOMAIN-CORE-THOUGHT.md`](../../ARCNET-DOMAIN-CORE-THOUGHT.md) — 6 new predicates + `thought` (§4, §3); reuses `derivedFrom`/`concerns` (registered by `domain-article.md`) and `citesAsEvidence` (registered by `core.md`) rather than re-defining them | `core.md`; `domain-article.md` if reuse is to resolve, though CORE §7.4's own "thought composes with any domain profile" claim holds even without it, as long as *something* eventually registers `derivedFrom`/`concerns` |

## Applying them

`core.md` MUST be applied first — every other patch's `Class` nodes reference core predicates
(`title`, `mentions`, `notes`, `tags`, …) by `[[wikilink]]`, and CORE §8.3 requires reusing a
registered predicate before introducing a new one. `domain-article.md` and
`domain-core-thought.md` are independent of each other and may be applied in either order, or
either may be applied alone — this mirrors [DOMAIN-CORE-THOUGHT](../../ARCNET-DOMAIN-CORE-THOUGHT.md) §1's
own claim to compose with any domain profile, or none.

Applying all three to an empty graph yields exactly the `_schema/` folder shown under
[`../../examples/graph/_schema/`](../../examples/graph/_schema/) (minus `aliases.md`, which is a
plain aggregate file — CORE §7.4 — not a node, so it is never part of a patch).

## A source-extending patch has no front-matter body

`domain-article.md`'s `## source` section carries only a `**Optional**` block, no `description`
and no `**Requires**` — it is a *contribution*, not a fresh definition. Applied after `core.md`'s
own `## source` (which does carry the description and the required/optional predicates CORE
defines), the two union into one `_schema/types/source.md` whose `Optional` block ends up
recommending `authors`/`url`/`cites`/`tags`/`doi` (from CORE) and `proposes`/`raises` (from the
profile) alike — CORE §9.3's `union` merge, applied to `Class` nodes exactly as it is to any other
role-`link` predicate.
