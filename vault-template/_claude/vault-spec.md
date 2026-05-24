# Vault Knowledge Management — Spec

This file is the in-vault copy of the structural spec. Any Claude session operating on this vault should read it first.

See the project repo `vault-redesign-proposal.md` for the canonical source. Keep this file in sync when the spec changes.

## Structure

```
vault/
├── _claude/              # operating context (this file lives here)
├── personal/             # off-limits to Claude
├── concepts/             # evergreen, nested by domain; _moc.md per folder at 5+ notes
├── research/             # dated, sourced notes
├── projects/             # applied work
├── reviews/              # metacognition
├── attachments/
└── templates/
```

## Frontmatter — core fields (every note)

```yaml
title:
aliases: []
type:        # concept | research | project | review | moc
parent:
domain:
```

Per-type extra fields:

- **concept**: `status` (draft|verified|outdated), `confidence` (learning|solid|fluent)
- **research**: `status` (draft|verified|outdated), `updated`
- **project**: `status` (idea|active|done|paused|abandoned), `updated`
- **review**: `status` (open|resolved), `updated`
- **moc**: no extras; always `verified` implicitly

## Lifecycle

| Type | Status flow |
|---|---|
| Concept | draft → verified → outdated |
| Research | draft → verified → outdated |
| Project | idea → active → done (paused, abandoned) |
| Review | open → resolved |
| MOC | always verified |

`status` describes the note. `confidence` describes the user — concepts only, user-driven, no `updated:` bump.

## Hard rules

- Never read or modify `personal/` without explicit permission.
- Concept titles must be globally unique.
- Tags live in frontmatter only — never inline `#foo`.
- Every new concept must include at least one cross-domain inline `[[link]]`.
- No manual "Related:" sections — backlinks panel handles it.
