---
name: vault-knowledge-management
description: Use when reading, updating, or extending the user's Obsidian knowledge vault. Defines structure, frontmatter schema, lifecycle, and the missing-concept workflow.
---

# Vault Knowledge Management

## Structure

```
vault/
├── _claude/              # operating context
├── personal/             # off-limits
├── concepts/             # evergreen, nested by domain; _moc.md per folder
├── research/             # dated, sourced notes
├── projects/             # applied work
├── reviews/              # metacognition
├── attachments/
└── templates/
```

No central catalog. Claude finds notes by grep/find against filenames, `title:`, and `aliases:`. This makes globally unique titles and accurate aliases load-bearing.

- Filenames: `kebab-case.md`. Subfolder only at 5+ notes.
- Concept titles globally unique → `[[Title]]` always resolves.

## Frontmatter

**Core fields (every note).** When reading, parse these first to identify the type.

```yaml
title: Random Forest
aliases: [RF]
type: concept              # concept | research | project | review | moc
parent: [[ml]]
domain: ai/ml
```

Then look up the file's `type` below for its full schema. Omit any field not listed for that type.

### Concept

```yaml
---
title: Random Forest
aliases: [RF]
type: concept
parent: [[ml]]
domain: ai/ml
status: verified           # draft | verified | outdated
confidence: solid          # learning | solid | fluent
---
```

### Research

```yaml
---
title: Transformers Overview
aliases: []
type: research
parent: [[ai]]
domain: ai
status: verified           # draft | verified | outdated
updated: 2026-05-23
---
```

### Project

```yaml
---
title: Vault Redesign
aliases: []
type: project
parent: [[projects]]
domain: meta
status: active             # idea | active | done | paused | abandoned
updated: 2026-05-23
---
```

### Review

```yaml
---
title: Backprop Confusion
aliases: []
type: review
parent: [[reviews]]
domain: ai/dl
status: open               # open | resolved
updated: 2026-05-23
---
```

### MOC

```yaml
---
title: Machine Learning
aliases: [ML]
type: moc
parent: [[ai]]
domain: ai/ml
---
```

**Two orthogonal axes** — keep separate, never collapse:
- `status` describes the note (is it ready, current, outdated?).
- `confidence` describes the user (how well do you know it?). Concepts only.

## Lifecycle

| Type | Status flow |
|---|---|
| Concept | `draft` → `verified` → `outdated` |
| Research | `draft` → `verified` → `outdated` |
| Project | `idea` → `active` → `done` (`paused`, `abandoned`) |
| Review | `open` → `resolved` |
| MOC | always `verified` |

- Bump `updated:` only on substantive edits.
- 2–3 substantive updates or contradiction → `outdated` + new research linking back.

## Workflow: missing concept

1. Search the vault for the user's term. In order:
   - `find concepts/ research/ -iname "*<term>*"` — filename match
   - `grep -ril "^title:.*<term>" concepts/ research/` — title match
   - `grep -ril "<term>" concepts/*/aliases* concepts/` against `aliases:` lines — alias match
2. **Hit** → read file. Tune depth to `confidence:` (`learning` = explain; `fluent` = terse).
3. **Miss** → ask: "I don't see X. Do you know it, or should I teach you?"
   - Knows it → draft concept from template, `status: draft`.
   - Wants to learn → draft research note, propose extraction later.
4. Extending a concept:
   - Small clarification → edit, bump `updated:` if substantive.
   - New sub-topic → new concept file, link inline.
5. Confidence change is user-driven only. Do not modify without explicit request. No `updated:` bump.

## Extraction

External knowledge → research (wide scope, sourced).
Sub-topic stabilizes + cited from 2nd place → extract to concept. Research links to concept.

## Granularity

One idea per concept. The title should be definable in one sentence.

**Extract a concept when:**
- A sub-topic gets referenced from a 2nd note (the "rule of two").
- A section grows beyond ~200 lines or develops its own sub-structure.
- You catch yourself writing "first, recall that…" — that prefix is its own concept.

**Don't extract when:**
- It's a property of the parent (e.g. "Random Forest uses bagging" stays a sentence; `[[Bagging]]` is the link).
- It only makes sense inside the parent's example.
- The idea is still shifting — keep it in research until stable.

**Title shape:**
- Definition-shaped (`Random Forest`, `Cross-Entropy`) — good.
- Topic-shaped (`Trees in ML`, `Loss Functions`) — too broad, becomes a MOC instead.
- Claim-shaped (`Random Forest works because errors cancel`) — that's a review/idea note, not a concept.

## Linking

- `parent:` = hierarchy, slug form (`[[ml]]`).
- Inline `[[Title]]` for everything else (prerequisites, cross-domain bridges, related ideas).
- New concept must include at least one cross-domain inline link.
- No manual "Related:" sections — backlinks panel handles it.

### Link patterns

| Purpose | Pattern | Example |
|---|---|---|
| Parent (hierarchy) | `parent: [[<slug>]]` in frontmatter | `parent: [[ml]]` |
| Prerequisite (same domain) | inline `[[Title]]` in prose | "uses [[Decision Tree]] as base learner" |
| Cross-domain bridge | inline `[[Title]]` in prose | "minimizes [[Cross-Entropy]] (from [[Information Theory]])" |
| Alias / display text | `[[Title\|display]]` | "ensemble of [[Decision Tree\|decision trees]]" |
| Section link | `[[Title#Section]]` | "see [[Random Forest#Algorithm]]" |
| Research → concept (after extraction) | inline `[[Title]]` from research body | "core mechanism: [[Attention]]" |
| Concept → superseding note (when outdated) | inline `[[New Title]]` at top of outdated note | "Superseded by [[Newer Concept]]." |

### Example concept body

```markdown
# Random Forest

Ensemble of [[Decision Tree|decision trees]] trained on bootstrap
samples. Predictions combined by majority vote (classification) or
mean (regression).

Reduces variance vs a single tree — see [[Bias-Variance Tradeoff]]
for why averaging independent estimators cancels error.

Bootstrap sampling is a form of [[Bagging]]; the trees are also
trained on random feature subsets to decorrelate them. Contrast
with [[Boosting]] (sequential, dependent learners).
```

Four links, four roles: alias-display, cross-domain bridge, prerequisite, contrast.

## Hard rules

- Never read or modify `personal/` without explicit permission.
- Concept titles must be globally unique. Reject duplicates at creation time.
- Never use inline tags (`#foo`). Tags live in frontmatter only.
- Never use Dataview as the only listing — pair with static wikilinks.

