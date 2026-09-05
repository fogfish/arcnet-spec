# DOMAIN INCIDENT - Markdown Knowledge Graph extension with Incidents and Postmortems

**Status:** Draft · **Version:** 0.5 · **Date:** 2026-09-05
**Extends:** [`ARCNET-CORE.md`](ARCNET-CORE.md)

> Revision note (0.1 → 0.2): `ContributingFactor` is promoted from a `tags` classification on
> `Cause` to a node type of its own (§3.3), bound to `Cause` by the `contributingFactor` predicate —
> the chain is `Incident → Cause → ContributingFactor`, and factors are shared across causes.
> `Incident`'s lifecycle predicates are stated in full (`startedAt`/`detectedAt`/`acknowledgedAt`/
> `mitigatedAt`/`resolvedAt`/`closedAt`), replacing 0.1's four-timestamp set; `affects`, `causedBy`,
> `triggeredBy`, and `detectedBy` are all role `link`; `customerImpact`/`businessImpact`/
> `technicalImpact` collapse into one `impact` predicate carrying a list of statements, one per
> impact, each prefixed by its facet; `summary` is retired in favour of CORE's
> own `text` (CORE §8.3 requires reusing a registered predicate before introducing a new one);
> `aggravatedBy` is retired, replaced by `contributingFactor` asserted from `Cause`. Every type is
> now stated as an explicit predicate table (role, merge, cardinality, target) before any rationale.
>
> Revision note (0.2 → 0.3): follows CORE v0.11's folder rule (a type's folder name MUST be
> character-for-character identical to the type name, §6): `sources/`/`entities/`/`incidents/`/
> `causes/`/`factors/`/`actions/`/`learnings/`/`references/`/`resources/` are renamed
> `Source/`/`Entity/`/`Incident/`/`Cause/`/`ContributingFactor/`/`CorrectiveAction/`/`Learning/`/
> `Reference/`/`Resource/`; `_schema/predicates/`→`_schema/Property/`, `_schema/types/`→
> `_schema/Class/`. `authors` is corrected to the registered singular `author` (CORE v0.12) in the
> `Source` worked example. `notes`, used optionally by `Incident`/`Cause`/`ContributingFactor`/
> `CorrectiveAction`, was cited as reused from CORE, but CORE retired its own `notes` predicate in
> v0.12 without reinstating it — it is now registered as this profile's own `_schema/Property/`
> node (§5.2). `actionStatus`'s rationale for avoiding the name `status` cited a `Reference`
> reading-state predicate of that name; CORE v0.12 retired `status` from `Reference` outright, so
> the rationale is corrected to note the name stays reserved on principle, not against a predicate
> that still exists.
>
> Revision note (0.3 → 0.4): 0.2 → 0.3 re-registered `notes` as this profile's own predicate after
> discovering CORE retired it in v0.12 — but re-registering it just carried the same badly-inherited
> field forward under new ownership. `notes` is now removed outright: dropped from `Incident`'s,
> `Cause`'s, `ContributingFactor`'s, `CorrectiveAction`'s, and `Learning`'s Optional lists and their
> predicate tables, and its `_schema/Property/` registration (§5.3) is deleted. Free-form prose that
> doesn't fit a type's other predicates belongs in `text` (`Incident`/`Cause`/`ContributingFactor`/
> `CorrectiveAction`, which already merges by accumulation) or `overview` (`Learning`).
>
> Revision note (0.4 → 0.5): merge behaviors stated twice — once in a type's §3 predicate table and
> once in the predicate's §5 registration — had drifted apart. §5 is authoritative (a predicate's
> merge behavior is a property of the predicate, not of the type using it, CORE §4 invariant 4), so
> the §3 tables are corrected to match it: `severity` is `immutable` (was `firstWriteWin` in §3.1),
> `resolution` is `append` (§3.1), `causeType` is `union` (§3.2, §3.3), and `lesson` and `overview`
> are `append` (§3.5). No `firstWriteWin` merge remains in the profile. §6.1's `severity` vocabulary
> is restated as `SEV1`/`SEV2`/`SEV3`, the values §5.2 already named, with named ladders
> (`critical`/`major`/`minor`) becoming the local labels that MUST be mapped onto them — the
> inverse of 0.4's direction; §3.1's worked example is updated accordingly.

This profile ingests **postmortem documents** into a knowledge graph. It adopts the CORE types
(`Source`, `Entity`, `Resource`, `Reference`, `Timeline`, [CORE §11](ARCNET-CORE.md)) and adds five
domain types: `Incident`, `Cause`, `ContributingFactor`, `CorrectiveAction`, and `Learning`. All
CORE mechanism (identity §7, edges §8, schema/merge §9, citations §12, version control §13, patch
§14) applies unchanged.

The profile's purpose is **indexing**: making a body of postmortems queryable as one graph. It is
not a runtime incident-response model. §9 records what it deliberately does not model, and why.

## 1. Model

### 1.1 Types

| Type                 | Definition                                                                                         | Folder                | Identity (CORE §7.3)                                      |
| -------------------- | -------------------------------------------------------------------------------------------------- | --------------------- | --------------------------------------------------------- |
| `Incident`           | The operational situation a postmortem analyzes — a recognized disruption with a lifecycle.        | `Incident/`           | Situation name, qualified by date or incident identifier. |
| `Cause`              | A condition that made an incident possible or initiated it.                                        | `Cause/`              | The condition, as a recurring condition.                  |
| `ContributingFactor` | A condition that increased an incident's probability, severity, or duration, but did not cause it. | `ContributingFactor/` | The factor, as a recurring condition.                     |
| `CorrectiveAction`   | A remedy a postmortem commits to.                                                                  | `CorrectiveAction/`   | The action, as an imperative phrase.                      |
| `Learning`           | The transferable knowledge a postmortem produced.                                                  | `Learning/`           | The lesson, in short form.                                |

`Service`, `Alert`, `Team`, and every other operational subject are CORE `Entity` nodes, not types
of this profile (§4.2).

### 1.2 Edges

The complete edge map of the profile. Every edge is registered in §5.

```
Source ──analyzes──────────────> Incident
Incident ──derivedFrom─────────> Source
Incident ──affects─────────────> Entity (Service)
Incident ──detectedBy──────────> Entity (Alert)
Incident ──causedBy────────────> Cause
Incident ──triggeredBy─────────> Cause
Incident ──learned─────────────> Learning
Incident ──addressedBy─────────> CorrectiveAction

Cause ──contributingFactor─────> ContributingFactor
Cause ──concerns───────────────> Entity
Cause ──addressedBy────────────> CorrectiveAction
Cause ──derivedFrom────────────> Source

ContributingFactor ──concerns──────> Entity
ContributingFactor ──addressedBy───> CorrectiveAction
ContributingFactor ──derivedFrom───> Source

CorrectiveAction ──addresses───> Cause | ContributingFactor | Incident
CorrectiveAction ──owner───────> Entity (Team)
CorrectiveAction ──derivedFrom─> Source

Learning ──concerns────────────> Entity
Learning ──derivedFrom─────────> Source
```

Two structural rules follow, and both are normative:

1. **A contributing factor attaches to a `Cause`, never directly to an `Incident`.** The chain is
   `Incident → Cause → ContributingFactor`. An incident whose postmortem identified factors but no
   cause MUST record a `Cause` for the factors to bind to.
2. **Every domain node carries `derivedFrom`** to every `Source` that asserts it (CORE §4.6). This
   is the profile's entire provenance model.

### 1.3 Shared nodes

`Cause`, `ContributingFactor`, and `Learning` are **shared**: the same node is asserted by many
postmortems and reached from many incidents. `derivedFrom` merges by `union` and `text` by `append`,
so a second postmortem restating a condition adds provenance and prose to the existing node instead
of creating a second one. That accumulation is what the graph is for — recurrence is read off node
degree, not by re-reading documents.

`Incident` and `CorrectiveAction` are **not** shared: each belongs to one situation.

### 1.4 Cardinality notation

Each type below states its predicates in a table. `Card.` is the cardinality a conforming instance
MUST satisfy — `1` exactly one, `0..1` at most one, `1..n` at least one, `0..n` any number. CORE's
`Class` node records only the Requires/Optional distinction (CORE §9.2); the cardinality column is
this profile's statement to producers, and `1`/`1..n` correspond exactly to `## Requires`.

## 2. Folder Layout

```
graph/
├── Source/                # postmortem documents  (CORE §11.2, extended §4.1)
├── Entity/                # services, alerts, teams, changes (CORE §11.3, §4.2)
├── Incident/              # incident nodes        (§3.1)
├── Cause/                 # cause nodes           (§3.2)
├── ContributingFactor/    # contributing factors  (§3.3)
├── CorrectiveAction/      # corrective actions    (§3.4)
├── Learning/              # learning nodes        (§3.5)
├── Reference/             # runbooks, standards, dashboards (CORE §11.6)
├── Resource/              # resource nodes        (CORE §11.4)
├── timeline/              # production index      (CORE §11.5)
│   ├── yearly/
│   └── monthly/
└── _schema/
    ├── Property/           # this profile's own + reused predicates (§5)
    ├── Class/              # Incident.md, Cause.md, ContributingFactor.md,
    │                       #   CorrectiveAction.md, Learning.md, Source.md's extension
    └── aliases.md          # entity alias table — not a node (CORE §7.4)
```

Every folder is flat: `severity`, `causeType`, and `actionStatus` live in predicates (§6), so none
of them determines file location (CORE §4.7).

## 3. Domain Types

### 3.1 `Incident`

The operational situation a postmortem analyzes.

**Identity.** `@id` (CORE §7.3) is the situation's short name. Incident names recur, so the `@id`
MUST be qualified (CORE §7.1) by the incident's date — `Checkout API Outage (2026-04-12)` — or by
the organization's incident identifier where one exists — `Checkout API Outage (INC-4471)`.

| Predicate        | Role | Merge         | Card. | Target / value                           |
| ---------------- | ---- | ------------- | ----- | ---------------------------------------- |
| `derivedFrom`    | edge | union         | 1..n  | `Source`                                 |
| `text`           | text | append        | 1     | the issue summary — leading paragraph    |
| `severity`       | meta | immutable     | 1     | §6.1                                     |
| `startedAt`      | meta | immutable     | 0..1  | ISO-8601 timestamp                       |
| `detectedAt`     | meta | immutable     | 1     | ISO-8601 timestamp                       |
| `acknowledgedAt` | meta | immutable     | 0..1  | ISO-8601 timestamp                       |
| `mitigatedAt`    | meta | immutable     | 0..1  | ISO-8601 timestamp                       |
| `resolvedAt`     | meta | immutable     | 0..1  | ISO-8601 timestamp                       |
| `closedAt`       | meta | immutable     | 0..1  | ISO-8601 timestamp                       |
| `chronology`     | text | append        | 0..1  | timed bullets                            |
| `impact`         | text | append        | 1..n  | one statement per impact, facet-prefixed |
| `resolution`     | text | append        | 0..1  | how the incident was ended               |
| `affects`        | link | union         | 1..n  | `Entity` (Service, §4.2)                 |
| `causedBy`       | link | union         | 0..n  | `Cause`                                  |
| `triggeredBy`    | link | union         | 0..n  | `Cause`                                  |
| `detectedBy`     | link | union         | 0..n  | `Entity` (Alert, §4.2)                   |
| `learned`        | link | union         | 0..n  | `Learning`                               |
| `addressedBy`    | edge | union         | 0..n  | `CorrectiveAction`                       |
| `tags`           | meta | union         | 0..n  | CORE §10.2                               |

`causedBy` is `0..n`: a postmortem that reached no conclusion still produces a valid `Incident`, and
*incidents with no identified root cause* is a query this graph exists to answer.

```markdown
---
"@id": Incident
"@type": Class
---
# Incident

The operational situation a postmortem analyzes — a recognized disruption with a lifecycle, not a
single event.

## Requires
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[severity]]
- required:: [[detectedAt]]
- required:: [[impact]]
- required:: [[affects]]

## Optional
- optional:: [[startedAt]]
- optional:: [[acknowledgedAt]]
- optional:: [[mitigatedAt]]
- optional:: [[resolvedAt]]
- optional:: [[closedAt]]
- optional:: [[chronology]]
- optional:: [[resolution]]
- optional:: [[causedBy]]
- optional:: [[triggeredBy]]
- optional:: [[detectedBy]]
- optional:: [[learned]]
- optional:: [[addressedBy]]
- optional:: [[tags]]
```

```markdown
---
"@id": Checkout API Outage (2026-04-12)
"@type": Incident
severity: SEV1
startedAt: 2026-04-12T09:14:00Z
detectedAt: 2026-04-12T09:31:00Z
acknowledgedAt: 2026-04-12T09:33:00Z
mitigatedAt: 2026-04-12T10:02:00Z
resolvedAt: 2026-04-12T10:48:00Z
closedAt: 2026-04-12T11:40:00Z
tags: [checkout, configuration]
---
A configuration push removed the checkout service's upstream timeout, saturating its connection pool
and failing 41% of checkout requests for 48 minutes.

- derivedFrom:: [[almeida-2026-checkout]]

**Chronology**
- 09:14 — a manual configuration push drops the `upstream_timeout` key from the checkout profile.
- 09:22 — connection pool saturation begins; error rate crosses 5%.
- 09:31 — the latency alert fires; the on-call engineer acknowledges at 09:33.
- 09:48 — the rollback runbook is found to reference a decommissioned deploy tool.
- 10:02 — the previous configuration revision is restored by hand; error rate recovers.
- 11:40 — pool metrics confirmed nominal; incident closed.

**Impact**
- Customer: approximately 125,000 checkout attempts failed across the EU region, shown a generic
  error page with no indication that a retry would succeed.
- Business: an estimated 6,800 orders abandoned, worth ~€410,000 in gross merchandise value.
- Business: 62% of the checkout SLO's remaining quarterly error budget consumed.
- Technical: four services degraded, one of them (the order ledger) through retry amplification.
- Technical: no data lost and no manual reconciliation required.

**Resolution**
The previous configuration revision was restored by hand and the connection pool drained; the
configuration service was frozen to reviewed pushes until schema validation shipped.

## Affects
- affects:: [[Checkout API]]
- affects:: [[Order Ledger]]
- affects:: [[Configuration Service]]

## Caused By
- causedBy:: [[Configuration Applied Without Schema Validation]]

## Triggered By
- triggeredBy:: [[Manual Configuration Push]]

## Detected By
- detectedBy:: [[Checkout Latency Alert]]

## Learnings
- learned:: [[Configuration Needs Machine-Checked Schemas]]
- learned:: [[Alert Thresholds Must Track The SLO They Defend]]
```

### 3.2 `Cause`

A condition that made an incident possible (`causedBy` — the root cause) or initiated it
(`triggeredBy` — the trigger).

**Identity.** `@id` (CORE §7.3) is the condition stated as a **recurring condition, not a dated
occurrence**: `Manual Configuration Push`, never `Config Push On 12 April`. The occurrence belongs
in the incident's `chronology`; the condition is the node, and the next incident arising from the
same condition links to that same node (§1.3).

**Role.** Whether a cause was a root cause or a trigger is carried by the **edge from the incident**
(`causedBy`, `triggeredBy`), not by any predicate on the cause: the same condition is a root cause in
one incident and a trigger in the next. What is intrinsic to the condition is its `causeType` (§6.2).

| Predicate            | Role | Merge         | Card. | Target / value             |
| -------------------- | ---- | ------------- | ----- | -------------------------- |
| `derivedFrom`        | edge | union         | 1..n  | `Source`                   |
| `text`               | text | append        | 1     | statement of the condition |
| `causeType`          | meta | union         | 1     | §6.2                       |
| `contributingFactor` | link | union         | 0..n  | `ContributingFactor`       |
| `concerns`           | edge | union         | 0..n  | `Entity`                   |
| `addressedBy`        | edge | union         | 0..n  | `CorrectiveAction`         |
| `tags`               | meta | union         | 0..n  | CORE §10.2                 |

```markdown
---
"@id": Cause
"@type": Class
---
# Cause

A condition that made an incident possible or initiated it, stated as a recurring condition rather
than a dated occurrence.

## Requires
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[causeType]]

## Optional
- optional:: [[contributingFactor]]
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[tags]]
```

```markdown
---
"@id": Configuration Applied Without Schema Validation
"@type": Cause
causeType: technical
---
The configuration service accepts and applies any syntactically valid document, so a key removed by
an operator propagates to every consumer without any check that the resulting profile is complete.

- derivedFrom:: [[almeida-2026-checkout]]

**Concerns**
- concerns:: [[Configuration Service]]

**Addressed by**
- addressedBy:: [[Validate Configuration Against Schema At Push Time]]

## Contributing Factors
- contributingFactor:: [[Coarse Latency Alert Threshold]]
- contributingFactor:: [[Stale Rollback Runbook]]
```

### 3.3 `ContributingFactor`

A condition that increased an incident's probability, severity, or duration without causing or
initiating it. Detection gaps (something that delayed detection) and control failures (something
that should have prevented the incident and did not) are `ContributingFactor` nodes; a graph MAY
distinguish them with `tags` until a pattern justifies promotion (CORE §11.4).

**Identity.** As `Cause` (§3.2): a recurring condition, never a dated occurrence.

**Binding.** A factor is bound to a `Cause` by `contributingFactor` (§5.1) and reached from an
incident only through that cause: `Incident → Cause → ContributingFactor`. One factor binds to many
causes, and one cause to many factors; a factor recurring under several causes across several
incidents is exactly the pattern this type exists to make visible.

| Predicate     | Role | Merge         | Card. | Target / value             |
| ------------- | ---- | ------------- | ----- | -------------------------- |
| `derivedFrom` | edge | union         | 1..n  | `Source`                   |
| `text`        | text | append        | 1     | statement of the condition |
| `causeType`   | meta | union         | 1     | §6.2                       |
| `concerns`    | edge | union         | 0..n  | `Entity`                   |
| `addressedBy` | edge | union         | 0..n  | `CorrectiveAction`         |
| `tags`        | meta | union         | 0..n  | CORE §10.2                 |

```markdown
---
"@id": ContributingFactor
"@type": Class
---
# ContributingFactor

A condition that increased an incident's probability, severity, or duration without causing or
initiating it. Bound to a [[Cause]] by [[contributingFactor]].

## Requires
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[causeType]]

## Optional
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[tags]]
```

```markdown
---
"@id": Coarse Latency Alert Threshold
"@type": ContributingFactor
causeType: technical
tags: [detection-gap]
---
The checkout latency alert fires on a five-minute p99 average, so a total failure of the endpoint
takes up to nine minutes to breach it.

- derivedFrom:: [[almeida-2026-checkout]]

**Concerns**
- concerns:: [[Checkout Latency Alert]]

**Addressed by**
- addressedBy:: [[Alert On Error Rate Within One Minute]]
```

### 3.4 `CorrectiveAction`

A remedy a postmortem commits to.

**Identity.** `@id` (CORE §7.3) is the action as an imperative phrase, ≤ ~8 words.

**Target.** `addresses` normally targets a `Cause` or a `ContributingFactor` — an action addressing
neither is the anomaly the requirement is meant to surface. An action responding to the incident as
a whole (customer communication, a one-off reconciliation) MAY target the `Incident`.

The type is `CorrectiveAction`, not `Action`: type names are registered globally, one node per name
(CORE §9.2), and `Action` is a name another profile would plausibly claim for an unrelated meaning.

| Predicate      | Role | Merge        | Card. | Target / value                                |
| -------------- | ---- | ------------ | ----- | --------------------------------------------- |
| `derivedFrom`  | edge | union        | 1..n  | `Source`                                      |
| `text`         | text | append       | 1     | statement of the remedy                       |
| `addresses`    | edge | union        | 1..n  | `Cause` \| `ContributingFactor` \| `Incident` |
| `actionStatus` | meta | lastWriteWin | 1     | §6.3                                          |
| `owner`        | edge | union        | 0..n  | `Entity` (Team, §4.2)                         |
| `due`          | meta | lastWriteWin | 0..1  | ISO-8601 date                                 |
| `tags`         | meta | union        | 0..n  | CORE §10.2                                    |

```markdown
---
"@id": CorrectiveAction
"@type": Class
---
# CorrectiveAction

A remedy a postmortem commits to, addressing an identified cause or contributing factor.

## Requires
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[addresses]]
- required:: [[actionStatus]]

## Optional
- optional:: [[owner]]
- optional:: [[due]]
- optional:: [[tags]]
```

```markdown
---
"@id": Validate Configuration Against Schema At Push Time
"@type": CorrectiveAction
actionStatus: inProgress
due: 2026-05-15
---
Reject a configuration push whose resulting profile does not satisfy the consumer's declared schema,
at push time rather than at consumption time.

- derivedFrom:: [[almeida-2026-checkout]]

**Addresses**
- addresses:: [[Configuration Applied Without Schema Validation]]

**Owner**
- owner:: [[Platform Team]]
```

### 3.5 `Learning`

The transferable knowledge a postmortem produced, generalized beyond the incident that occasioned
it.

**Identity.** `@id` (CORE §7.3) is the lesson in short form, phrased so that a later incident
teaching the same lesson yields the same title and merges into the same node (§1.3).

| Predicate         | Role | Merge         | Card. | Target / value                  |
| ----------------- | ---- | ------------- | ----- | ------------------------------- |
| `derivedFrom`     | edge | union         | 1..n  | `Source`                        |
| `lesson`          | text | append        | 1     | one-sentence lesson, emphasized |
| `concerns`        | edge | union         | 1..n  | `Entity`                        |
| `overview`        | text | append        | 0..1  | paragraph of context            |
| `citesAsEvidence` | edge | union         | 0..n  | `Reference` (CORE §10.6)        |
| `tags`            | meta | union         | 0..n  | CORE §10.2                      |

`concerns` is `1..n`: a lesson generalizing to nothing is a note about one incident and belongs in
that incident's `text`. The entities it concerns are how a later reader finds it.

```markdown
---
"@id": Learning
"@type": Class
---
# Learning

The transferable knowledge a postmortem produced, generalized beyond the incident that occasioned
it.

## Requires
- required:: [[derivedFrom]]
- required:: [[lesson]]
- required:: [[concerns]]

## Optional
- optional:: [[overview]]
- optional:: [[citesAsEvidence]]
- optional:: [[tags]]
```

```markdown
---
"@id": Configuration Needs Machine-Checked Schemas
"@type": Learning
tags: [configuration, validation]
---
*Configuration that is only checked for syntax will eventually be applied incomplete; the check that
matters is against the consumer's declared schema.*

- derivedFrom:: [[almeida-2026-checkout]]

**Overview**
Two of the last four checkout incidents originated in a configuration document that parsed cleanly
and meant something different from what its author intended. Syntactic validity is what the
configuration service checks; semantic completeness is what its consumers depend on.

**Concerns**
- concerns:: [[Configuration Service]]
- concerns:: [[Checkout API]]
```

## 4. CORE Types

A profile MAY extend a CORE type's `_schema/Class/` node with additional Optional predicates,
without changing the type's identity or any existing predicate's merge behavior — the contribution
unions into the existing node (CORE §9.3 `union`). This profile extends `Source` (§4.1) and
prescribes how `Entity` is used (§4.2); `Resource`, `Reference`, and `Timeline` are used as CORE
defines them.

### 4.1 `Source` (extends CORE §11.2)

The postmortem document is an ordinary `Source`. Identity is the citekey (CORE §7.2), derived from
the postmortem's author and **its own publication date**, not the incident's. `title`, `published`,
`abstract`, and `mentions` are required as CORE requires them. The profile adds one navigation
predicate.

| Predicate  | Role | Merge | Card. | Target     |
| ---------- | ---- | ----- | ----- | ---------- |
| `analyzes` | link | union | 0..n  | `Incident` |

```markdown
---
"@id": Source
"@type": Class
---
## Optional
- optional:: [[analyzes]]
```

```markdown
---
"@id": almeida-2026-checkout
"@type": Source
title: "Postmortem: Checkout API Outage, 12 April 2026"
author: [Rita Almeida]
published: 2026-04-19
tags: [postmortem, checkout, configuration]
---
# Postmortem: Checkout API Outage, 12 April 2026

A blameless review of the 48-minute checkout failure of 12 April 2026, its configuration root cause,
and the validation work it commits the platform team to.

## Mentions
- mentions:: [[Checkout API]]
- mentions:: [[Configuration Service]]
- mentions:: [[Order Ledger]]
- mentions:: [[Platform Team]]

## Analyzes
- analyzes:: [[Checkout API Outage (2026-04-12)]]

## Cites
- cites:: [[Checkout Service Level Objectives]]
```

Two rules govern the pair:

- Every entity an incident `affects` MUST also be `mentions`ed by the postmortem that asserts it.
  The two are not redundant: `mentions` records what the document talks about, `affects` records
  what broke.
- `published` is the postmortem's publication date and is what places the document in the `Timeline`
  (CORE §11.5). The incident's own date is `detectedAt` and is queried from the `Incident` node,
  never from the timeline index.

### 4.2 `Entity` (CORE §11.3)

Every operational subject — service, alert, team, component, kind of change — is a CORE `Entity`
node with a four-word decoded Sowa `category` (CORE §10.2). This profile defines no `Service`,
`Alert`, `Team`, or `Change` type; it fixes the `Entity` shape each takes.

| Operational subject                        | `category`                                      | `tags` includes |
| ------------------------------------------ | ----------------------------------------------- | --------------- |
| **Service**, system, component, data store | `[independent, physical, continuant, object]`   | `service`       |
| **Alert**, monitor, dashboard definition   | `[relative, abstract, continuant, description]` | `alert`         |
| **Team**, organization, on-call rotation   | `[mediating, physical, continuant, structure]`  | `team`          |
| Kind of change — deployment, config push   | `[independent, physical, occurrent, process]`   | `change`        |
| Protocol, procedure, runbook               | `[independent, abstract, occurrent, script]`    | `procedure`     |
| SLO, policy, error budget                  | `[mediating, abstract, continuant, reason]`     | `policy`        |

The first three rows are normative for the two subjects this profile links to directly — an
`Incident` `affects` a **Service** and is `detectedBy` an **Alert**, and a `CorrectiveAction`
`owner` is a **Team**. The remaining rows are guidance, so that a graph of postmortems stays
internally consistent.

A **Service** entity SHOULD record its telemetry identifier — the OpenTelemetry `service.name`, or
the equivalent in the organization's own convention — in `aliases` (CORE §7.4), so that a node in
this graph and a series in the observability system are known to be the same subject. This is the
profile's whole position on telemetry vocabulary: it does not restate the observability model, it
names the same subjects and stores the join key.

```markdown
---
"@id": Checkout API
"@type": Entity
category: [independent, physical, continuant, object]
aliases: [checkout-api, checkout.api.svc]
tags: [service, payments]
---
# Checkout API

The customer-facing service that reserves inventory, authorizes payment, and creates an order.

- requires:: [[Configuration Service]]

## mentionedIn
- mentionedIn:: [[almeida-2026-checkout]]
```

```markdown
---
"@id": Checkout Latency Alert
"@type": Entity
category: [relative, abstract, continuant, description]
tags: [alert, checkout]
---
# Checkout Latency Alert

Fires when the checkout endpoint's five-minute p99 latency exceeds 2s.

- concerns:: [[Checkout API]]

## mentionedIn
- mentionedIn:: [[almeida-2026-checkout]]
```

## 5. Predicates

In addition to the core vocabulary (CORE §10), this profile registers the following as
`_schema/Property/` nodes (CORE §9.1), one per heading. Namespaces: `schema:`, `prov:`, `cito:`,
`arc:` (graph-native).

### 5.1 Structural predicates

#### `analyzes`
**role:** `link` · **merge:** `union` · **label:** `Analyzes` · **aligned:** `arc:analyzes` · **from → to:** Source → Incident

Asserts that the source document is a postmortem of the incident; recorded under the source's own
`## Analyzes` block. Its inverse is `derivedFrom`, which is the canonical direction for provenance
queries; both MUST be kept consistent.

#### `affects`
**role:** `link` · **merge:** `union` · **label:** `Affects` · **aligned:** `arc:affects` · **from → to:** Incident → Entity

The services and systems the incident degraded; recorded under the incident's own `## Affects`
block.

#### `causedBy`
**role:** `link` · **merge:** `union` · **label:** `Caused By` · **aligned:** `arc:causedBy` · **from → to:** Incident → Cause

The **root cause** — the underlying condition that made the incident possible; recorded under the
incident's own `## Caused By` block.

#### `triggeredBy`
**role:** `link` · **merge:** `union` · **label:** `Triggered By` · **aligned:** `arc:triggeredBy` · **from → to:** Incident → Cause

The **trigger** — the condition whose occurrence immediately initiated the incident, as distinct
from the root cause that made it possible; recorded under the incident's own `## Triggered By`
block.

#### `detectedBy`
**role:** `link` · **merge:** `union` · **label:** `Detected By` · **aligned:** `arc:detectedBy` · **from → to:** Incident → Entity

What surfaced the incident — an alert, a dashboard, a support channel; recorded under the incident's
own `## Detected By` block.

#### `contributingFactor`
**role:** `link` · **merge:** `union` · **label:** `Contributing Factors` · **aligned:** `arc:contributingFactor` · **from → to:** Cause → ContributingFactor

The conditions that made this cause more likely to produce an incident, or its consequences worse;
recorded under the cause's own `## Contributing Factors` block. Asserted from the `Cause`, never
from the `Incident` (§1.2).

#### `learned`
**role:** `link` · **merge:** `union` · **label:** `Learnings` · **aligned:** `arc:learned` · **from → to:** Incident → Learning

The learnings the incident produced; recorded under the incident's own `## Learnings` block. Because
the target is shared, a lesson taught by several incidents is one node reached from each.

#### `owner`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:agent` · **from → to:** CorrectiveAction → Entity

The team or person accountable for the corrective action. Displayed under the bold label **Owner**.

### 5.2 Type-specific predicates

#### `chronology`
**Used by:** `Incident` · **role:** `text` · **merge:** `append`

The sequence of events surrounding the incident, one per bullet, each prefixed by its time.
Displayed under the bold label **Chronology**. Individual timestamped occurrences live here, not as
nodes (§9).

#### `impact`
**Used by:** `Incident` · **role:** `text` · **merge:** `append`

What the incident cost — a **list** of statements, one per bullet, each prefixed by its facet.
Displayed under the bold label **Impact**. Exactly three facet prefixes are defined:

| Prefix       | States                                                                   |
| ------------ | ------------------------------------------------------------------------ |
| `Customer:`  | Who was affected, how many, in which regions, and what they experienced. |
| `Business:`  | Revenue, orders, and any SLO, error-budget, or contractual consequence.  |
| `Technical:` | Degradation, data loss, backlog, and manual repair the systems incurred. |

A conforming `Incident` MUST carry at least one bullet; each facet MAY appear more than once, and a
facet MAY be omitted where the postmortem did not establish it. Because `impact` merges by `append`,
a later postmortem restating the same incident adds bullets rather than replacing the assessment —
which is why it is a list of statements and not one paragraph.

#### `resolution`
**Used by:** `Incident` · **role:** `text` · **merge:** `append`

How the incident was brought to an end — distinct from a `CorrectiveAction`, which prevents its
recurrence. Displayed under the bold label **Resolution**.

#### `lesson`
**Used by:** `Learning` · **role:** `text` · **merge:** `append`

A one-sentence statement of the transferable lesson, rendered emphasized (`*lesson*`). Phrased
generally: it must remain true of the next incident that teaches it.

#### `severity`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

The incident's severity: `SEV1`/`SEV2`/`SEV3` (§6.1).


#### `startedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which impact began, whether or not anyone knew.

#### `detectedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which the incident became known. Required on every `Incident`: it is the
temporal anchor every incident query starts from.

#### `acknowledgedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which a responder took ownership of the incident.

#### `mitigatedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which customer impact ended, whether or not the underlying condition was
fixed.

#### `resolvedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which the system was returned to its intended state.

#### `closedAt`
**Used by:** `Incident` · **role:** `meta` · **merge:** `immutable`

ISO-8601 timestamp at which the incident was formally closed.

The six lifecycle timestamps are ordered
`startedAt ≤ detectedAt ≤ acknowledgedAt ≤ mitigatedAt ≤ resolvedAt ≤ closedAt`, and make the
durations that matter — time to detect, time to acknowledge, time to mitigate, total impact —
arithmetic on the node rather than a reading of the narrative.

#### `causeType`
**Used by:** `Cause`, `ContributingFactor` · **role:** `meta` · **merge:** `union`

The nature of the condition (§6.2) — intrinsic to it, unlike the role it played in any one incident,
which the edge carries (§3.2).

#### `actionStatus`
**Used by:** `CorrectiveAction` · **role:** `meta` · **merge:** `lastWriteWin`

The action's state of completion (§6.3). The one legitimately mutable predicate in this profile.
Deliberately **not** named `status`: an earlier CORE draft used that name for a `Reference`'s
reading state before retiring it in CORE v0.12 without reinstating it. Kept distinct here anyway —
predicates are registered globally under one name (CORE §9.1), and a future CORE revision could
reintroduce a generic `status` with different meaning and merge behavior.

#### `due`
**Used by:** `CorrectiveAction` · **role:** `meta` · **merge:** `lastWriteWin`

ISO-8601 date the action is expected to complete. Mutable: dates are re-committed.

### 5.3 Reused predicates

The following are reused unchanged. Where a co-located profile has already registered one, reuse
that node; otherwise this document is where it is registered, with exactly the role/merge/aligned
values given here — a predicate's meaning and merge behavior do not change with the type that uses
it (CORE §4, invariant 4).

#### `derivedFrom`
**role:** `edge` · **merge:** `union` · **aligned:** `prov:wasDerivedFrom` · **from → to:** Incident/Cause/ContributingFactor/CorrectiveAction/Learning → Source

The provenance edge to the postmortem(s) a node was distilled from. Identical to
[DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1's predicate of the same name.

#### `concerns`
**role:** `edge` · **merge:** `union` · **aligned:** `schema:about` · **from → to:** Cause/ContributingFactor/Learning → Entity

The entities a condition involves or a lesson generalizes to. Identical to
[DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1's predicate of the same name.

#### `addresses`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addresses` · **from → to:** CorrectiveAction → Cause/ContributingFactor/Incident

What a corrective action tackles. Identical to [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.1's
predicate of the same name (there, Hypothesis → Aporia); this profile extends the targets it applies
to, not its meaning. Displayed under the bold label **Addresses**.

#### `addressedBy`
**role:** `edge` · **merge:** `union` · **aligned:** `arc:addressedBy` · **from → to:** Cause/ContributingFactor/Incident → CorrectiveAction

The inverse of `addresses`, displayed under the bold label **Addressed by**. The canonical direction
is `addresses`, asserted by the action; where both are asserted they MUST be consistent.

#### `overview`
**Used by:** `Learning` · **role:** `text` · **merge:** `append`

A short paragraph of context. Identical to [DOMAIN-ARTICLE](ARCNET-DOMAIN-ARTICLE.md) §4.2's
predicate of the same name.

CORE's `text`, `tags`, and citation vocabulary (§10.6 — e.g. `citesAsEvidence` pointing a
`Learning` at a `Reference`) are used unchanged. `text` carries the issue summary on an `Incident`
and the statement of the condition or remedy on `Cause`, `ContributingFactor`, and
`CorrectiveAction`; it renders as the node's leading, unlabeled paragraph, and merges by `append` so
that a second postmortem restating a shared condition adds its own reading of it.

## 6. Class Vocabularies

### 6.1 `severity`

| Value  | Definition                                                      |
| ------ | --------------------------------------------------------------- |
| `SEV1` | Customer-facing failure of a primary function, or data at risk. |
| `SEV2` | Significant degradation, or failure of a secondary function.    |
| `SEV3` | Limited or internal impact, no customer consequence.            |

A graph whose organization runs a named ladder MAY map onto these — typically `critical` → SEV1,
`major` → SEV2, `minor` and below → SEV3 — but MUST record the mapped value, not the local label, so
severity stays comparable across the graph.

### 6.2 `causeType`

| Value            | Definition                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------- |
| `technical`      | A property of the system: a defect, a missing check, a capacity limit.                               |
| `process`        | A property of how work is done: an unreviewed change, a stale runbook, a missing gate.               |
| `organizational` | A property of how the organization is arranged: unclear ownership, no coverage, a priority conflict. |
| `human`          | A property of the moment: a mistaken action, under conditions that permitted it.                     |
| `external`       | A property of something outside the organization's control: a provider failure, an upstream change.  |

`human` classifies the condition, never the person. A blameless postmortem records that a manual
action was possible; it does not create an `Entity` for the individual and attach the condition to
them.

### 6.3 `actionStatus`

| Value        | Definition                                                                       |
| ------------ | -------------------------------------------------------------------------------- |
| `proposed`   | Written down, not yet committed to.                                              |
| `accepted`   | Committed to and owned.                                                          |
| `inProgress` | Under way.                                                                       |
| `done`       | Complete.                                                                        |
| `dropped`    | Deliberately abandoned — recorded rather than deleted, so the decision survives. |

## 7. Mapping a Postmortem Document

A postmortem structured as *Issue Summary · Root Causes · Impact · Resolution · Actions · Key
Learning* maps onto the graph as follows. One document is one commit (CORE §13.1), producing one
`Source`, one `Incident`, and as many `Cause`/`ContributingFactor`/`CorrectiveAction`/`Learning`
nodes as the document asserts — new where the condition or lesson is new, merged into the existing
node where it is not.

| Postmortem section  | Graph                                                                                                                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| the document itself | `Source` (§4.1) — citekey identity, `analyzes` the incident, indexed in the `Timeline` by `published`                                                                                 |
| Issue Summary       | `Incident` `text`, `severity`, and the six lifecycle timestamps; the narrative sequence → `chronology`                                                                                |
| Root Causes         | one `Cause` per condition, linked `causedBy` (underlying) or `triggeredBy` (initiating); each aggravating condition a `ContributingFactor` bound to its cause by `contributingFactor` |
| Impact              | one `impact` bullet per statement, prefixed `Customer:` / `Business:` / `Technical:`; the systems degraded → `affects` → `Entity` (Service)                                           |
| Resolution          | `Incident` `resolution`                                                                                                                                                               |
| Actions             | one `CorrectiveAction` per item, each `addresses` the `Cause` or `ContributingFactor` it remedies, with `owner`, `due`, `actionStatus`                                                |
| Key Learning        | one `Learning` per lesson, reached by `learned` from the incident, `concerns` the entities it generalizes to                                                                          |

Merge behavior does the accumulation. Ingesting a second postmortem that implicates
`Manual Configuration Push` adds a `derivedFrom` to that existing node (`union`) and appends its own
reading to its `text` (`append`); it does not create a second node. That is what turns *n* documents
into a graph: which conditions recur, which factors keep appearing under different causes, which
lessons were already recorded, which causes have no corrective action, and which services appear
across incidents are all answered by traversal.

## 8. Conformance

In addition to the CORE checklist ([CORE §16](ARCNET-CORE.md)):

- [ ] Every `Incident`, `Cause`, `ContributingFactor`, `CorrectiveAction`, and `Learning` carries
      `derivedFrom` to at least one `Source` (§1.2).
- [ ] Every `Incident` satisfies §3.1's table: `text`, `severity`, `detectedAt`, `impact`, and at
      least one `affects`; its `@id` is qualified by date or incident identifier.
- [ ] Every `Incident` `impact` bullet is prefixed by one of `Customer:`, `Business:`, `Technical:`
      (§5.2).
- [ ] Every `Incident` timestamp present respects the order
      `startedAt ≤ detectedAt ≤ acknowledgedAt ≤ mitigatedAt ≤ resolvedAt ≤ closedAt` (§5.2).
- [ ] Every `ContributingFactor` is reached from an `Incident` only through a `Cause`, via
      `contributingFactor` (§1.2, §3.3). No `Incident` links a factor directly.
- [ ] Every `Cause` and `ContributingFactor` `@id` names a recurring condition, not a dated
      occurrence, and carries a `causeType` from §6.2 (§3.2, §3.3).
- [ ] The role a cause played is carried by the edge (`causedBy`/`triggeredBy`), never by a predicate
      on the cause node (§3.2).
- [ ] Every `CorrectiveAction` has at least one `addresses` edge and an `actionStatus` from §6.3
      (§3.4).
- [ ] Every `Learning` has a `lesson` and at least one `concerns` edge (§3.5).
- [ ] Every entity an `Incident` `affects` is also `mentions`ed by the postmortem that asserts it,
      and every `analyzes` edge has its matching `derivedFrom` (§4.1).
- [ ] Every `Entity` reached by `affects` is shaped as a Service, and by `detectedBy` as an Alert,
      per §4.2's table.
- [ ] Every postmortem document appears in the `Timeline` files for its `published` period — the
      document's publication date, never the incident's (§4.1).
- [ ] No individual occurrence is a node: timestamped events appear in `chronology` (§9).
- [ ] Every predicate used is registered as a `_schema/Property/` node (§5 or CORE §10); all five
      types, and the extension to `Source` (§4.1), are registered as `_schema/Class/` nodes (§3, §4.1).

## 9. Design Notes

The rationale behind the model above, kept separate from it. Two rules decided most of it:

1. **A node type earns its place when its instances are shared across documents.** `Cause`,
   `ContributingFactor`, and `Learning` recur across postmortems and accumulate provenance;
   `Incident` and `CorrectiveAction` do not, but each is the subject a document is about or commits
   to. Anything that appears in exactly one document, forever, is prose.
2. **The postmortem document is the `Source`.** CORE already has a type for an ingested document; a
   separate `Postmortem` node would duplicate its identity and split its provenance.

What follows from those, and was deliberately left out:

- **`Event` / `IncidentEvent`.** An individual timestamped occurrence is asserted by exactly one
  document and queried in no other way, so it is `chronology` prose. The lifecycle moments that must
  be arithmetic are the six timestamps (§5.2). What recurs is the *kind* of event — that is a
  `Cause` or an `Entity`.
- **`Impact` as a node.** No impact instance is shared between two incidents; an impact node would
  have degree one, permanently. `impact` plus `affects` carries the same content.
- **Separate customer / business / technical impact predicates.** Three predicates would fix the
  facets in the schema and force a producer to decide, per postmortem, which of them a sentence
  belongs to before it can be written down. One `impact` list keeps the facets as bullet prefixes:
  a statement stays one statement, several may share a facet, and an unestablished facet is simply
  absent rather than an empty predicate.
- **`Service`, `Alert`, `Team`, `Change` types.** CORE's `Entity` covers them; §4.2 fixes their
  shape.
- **Quantitative impact fields** (`affectedCustomers`, `revenueImpact`, `errorRate`). They are
  organization-specific, rarely comparable across incidents, and belong to the systems that produced
  them. A graph needing one comparable should register a `meta` predicate of its own rather than
  have this profile guess a unit.
- **Incident-to-incident links.** Two incidents sharing a `Cause`, a `ContributingFactor`, or a
  `Learning` *are* the recurrence, discoverable by traversal without anyone noticing the
  relationship at ingestion time.
