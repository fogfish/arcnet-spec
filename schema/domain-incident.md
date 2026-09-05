---
"@type": patch
document: ARCNET-DOMAIN-INCIDENT.md
published: 2026-09-05
stats: { nodes: 33, edges: 46 }
---

# Property

## analyzes

```yaml
role: link
merge: union
label: "Analyzes"
aligned: "arc:analyzes"
```

Asserts that the source document is a postmortem of the incident; recorded under the source's own `## Analyzes` block. Its inverse is `derivedFrom`, which is the canonical direction for provenance queries; both MUST be kept consistent.

## affects

```yaml
role: link
merge: union
label: "Affects"
aligned: "arc:affects"
```

The services and systems the incident degraded; recorded under the incident's own `## Affects` block.

## causedBy

```yaml
role: link
merge: union
label: "Caused By"
aligned: "arc:causedBy"
```

The root cause — the underlying condition that made the incident possible; recorded under the incident's own `## Caused By` block.

## triggeredBy

```yaml
role: link
merge: union
label: "Triggered By"
aligned: "arc:triggeredBy"
```

The trigger — the condition whose occurrence immediately initiated the incident, as distinct from the root cause that made it possible; recorded under the incident's own `## Triggered By` block.

## detectedBy

```yaml
role: link
merge: union
label: "Detected By"
aligned: "arc:detectedBy"
```

What surfaced the incident — an alert, a dashboard, a support channel; recorded under the incident's own `## Detected By` block.

## contributingFactor

```yaml
role: link
merge: union
label: "Contributing Factors"
aligned: "arc:contributingFactor"
```

The conditions that made this cause more likely to produce an incident, or its consequences worse; recorded under the cause's own `## Contributing Factors` block. Asserted from the `Cause`, never from the `Incident`.

## learned

```yaml
role: link
merge: union
label: "Learnings"
aligned: "arc:learned"
```

The learnings the incident produced; recorded under the incident's own `## Learnings` block. Because the target is shared, a lesson taught by several incidents is one node reached from each.

## owner

```yaml
role: edge
merge: union
aligned: "schema:agent"
```

The team or person accountable for the corrective action. Displayed under the bold label **Owner**.

## chronology

```yaml
role: text
merge: append
```

The sequence of events surrounding the incident, one per bullet, each prefixed by its time. Displayed under the bold label **Chronology**. Individual timestamped occurrences live here, not as nodes.

## impact

```yaml
role: text
merge: append
```

What the incident cost — a list of statements, one per bullet, each prefixed by its facet (`Customer:`/`Business:`/`Technical:`). Displayed under the bold label **Impact**. A conforming `Incident` MUST carry at least one bullet.

## resolution

```yaml
role: text
merge: append
```

How the incident was brought to an end — distinct from a `CorrectiveAction`, which prevents its recurrence. Displayed under the bold label **Resolution**.

## lesson

```yaml
role: text
merge: append
```

A one-sentence statement of the transferable lesson, rendered emphasized (`*lesson*`). Phrased generally: it must remain true of the next incident that teaches it.

## severity

```yaml
role: meta
merge: immutable
```

The incident's severity: `SEV1`/`SEV2`/`SEV3`.

## startedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which impact began, whether or not anyone knew.

## detectedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which the incident became known. Required on every `Incident`: it is the temporal anchor every incident query starts from.

## acknowledgedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which a responder took ownership of the incident.

## mitigatedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which customer impact ended, whether or not the underlying condition was fixed.

## resolvedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which the system was returned to its intended state.

## closedAt

```yaml
role: meta
merge: immutable
```

ISO-8601 timestamp at which the incident was formally closed.

## causeType

```yaml
role: meta
merge: union
```

The nature of the condition — `technical`/`process`/`organizational`/`human`/`external` — intrinsic to it, unlike the role it played in any one incident, which the edge carries.

## actionStatus

```yaml
role: meta
merge: lastWriteWin
```

The action's state of completion: `proposed`/`accepted`/`inProgress`/`done`/`dropped`. The one legitimately mutable predicate in this profile. Deliberately not named `status`: an earlier CORE draft used that name for a `Reference`'s reading state before retiring it in CORE v0.12 without reinstating it. Kept distinct here anyway — predicates are registered globally under one name.

## due

```yaml
role: meta
merge: lastWriteWin
```

ISO-8601 date the action is expected to complete. Mutable: dates are re-committed.

## derivedFrom

```yaml
role: edge
merge: union
aligned: "prov:wasDerivedFrom"
```

The provenance edge to the postmortem(s) a node was distilled from. Carried by every domain node of this profile — `Incident`, `Cause`, `ContributingFactor`, `CorrectiveAction`, `Learning` — to every `Source` that asserts it. Identical to DOMAIN-ARTICLE's predicate of the same name.

## concerns

```yaml
role: edge
merge: union
aligned: "schema:about"
```

The entities a condition involves or a lesson generalizes to. Identical to DOMAIN-ARTICLE's predicate of the same name.

## addresses

```yaml
role: edge
merge: union
aligned: "arc:addresses"
```

What a corrective action tackles — a `Cause`, a `ContributingFactor`, or the `Incident` as a whole. Displayed under the bold label **Addresses**. Identical to DOMAIN-ARTICLE's predicate of the same name (there, Hypothesis → Aporia); this profile extends the targets it applies to, not its meaning.

## addressedBy

```yaml
role: edge
merge: union
aligned: "arc:addressedBy"
```

The inverse of `addresses`, displayed under the bold label **Addressed by**. The canonical direction is `addresses`, asserted by the action; where both are asserted they MUST be consistent.

## overview

```yaml
role: text
merge: append
```

A short paragraph of context. Identical to DOMAIN-ARTICLE's predicate of the same name.

# Class

## Incident

```yaml
```

The operational situation a postmortem analyzes — a recognized disruption with a lifecycle, not a single event.

**Requires**
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[severity]]
- required:: [[detectedAt]]
- required:: [[impact]]
- required:: [[affects]]

**Optional**
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

## Cause

```yaml
```

A condition that made an incident possible or initiated it, stated as a recurring condition rather than a dated occurrence.

**Requires**
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[causeType]]

**Optional**
- optional:: [[contributingFactor]]
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[tags]]

## ContributingFactor

```yaml
```

A condition that increased an incident's probability, severity, or duration without causing or initiating it. Bound to a [[Cause]] by [[contributingFactor]].

**Requires**
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[causeType]]

**Optional**
- optional:: [[concerns]]
- optional:: [[addressedBy]]
- optional:: [[tags]]

## CorrectiveAction

```yaml
```

A remedy a postmortem commits to, addressing an identified cause or contributing factor.

**Requires**
- required:: [[derivedFrom]]
- required:: [[text]]
- required:: [[addresses]]
- required:: [[actionStatus]]

**Optional**
- optional:: [[owner]]
- optional:: [[due]]
- optional:: [[tags]]

## Learning

```yaml
```

The transferable knowledge a postmortem produced, generalized beyond the incident that occasioned it.

**Requires**
- required:: [[derivedFrom]]
- required:: [[lesson]]
- required:: [[concerns]]

**Optional**
- optional:: [[overview]]
- optional:: [[citesAsEvidence]]
- optional:: [[tags]]

## Source

```yaml
```

**Optional**
- optional:: [[analyzes]]
