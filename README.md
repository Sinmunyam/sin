# sin

A set of Claude Code skills that ground code generation in the user's own Obsidian knowledge vault. The premise: don't let the model write code for topics the user doesn't understand. The vault is the user's knowledge map; skills read it before they write.

## Why

LLMs happily generate code the user can't explain, debug, or maintain. `sin` makes that harder. Every code-generating action passes through a gate: either the topic exists in the vault (the user has captured their understanding), or the user must learn it first via a Socratic teaching loop.

## Skills

All skills live in `.agents/<skill-name>/SKILL.md`.

| Skill | Purpose |
|---|---|
| `init-vault` | One-shot scaffold for a new Obsidian vault. Creates folder structure, frontmatter templates, root MOC, and writes `~/.claude/vault-config.json`. |
| `update-vault` | Explicit write path into the vault. Mode A adds new notes (with MOC linking and cross-domain link enforcement). Mode B applies validated lifecycle transitions (`status`, `confidence`). |
| `ask` | Socratic teaching loop. Triggered when a topic isn't in the vault. Assesses knowledge level, interrogates the user with one focused question at a time, and on confirmed understanding hands the distilled content to `update-vault`. |
| `plan` | Grounded project planning. Reads the vault to find foundation/gaps for a stated goal; refuses to write a plan until the user has (or learns) the required knowledge. |
| `generate` | Code generation. Stops unless the topic is grounded in the vault (or just confirmed via `ask`). Never expands scope silently. |

## Vault structure

Defined in `vault-redesign-proposal.md` and mirrored inside each vault at `_claude/vault-spec.md`.

```
vault/
├── _claude/        # operating context (vault-spec, domains registry, session drafts)
├── personal/       # off-limits to Claude
├── concepts/       # evergreen, nested by domain
├── research/       # dated, sourced notes
├── projects/       # applied work
├── reviews/        # metacognition
├── attachments/
└── templates/      # per-type frontmatter scaffolds
```

Five note types — `concept`, `research`, `project`, `review`, `moc` — each with a defined frontmatter schema and lifecycle (see `vault-redesign-proposal.md`).

## Getting started

1. Run `init-vault` and point it at a directory. It scaffolds the structure and writes `~/.claude/vault-config.json`.
2. Use `ask` for any topic you want to learn. On confirmed understanding the note is filed automatically via `update-vault`.
3. Use `update-vault` directly to add notes you already understand, or to transition lifecycle metadata.
4. Use `plan` to scope new work — it will refuse if your vault doesn't show the foundation.
5. Use `generate` to write code — it will refuse if the topic isn't grounded.

## Layout

```
.agents/             # skill definitions (SKILL.md per skill)
vault-template/      # canonical templates init-vault copies into new vaults
vault-redesign-proposal.md   # the vault spec
README.md
LICENSE
```

## Conventions

- One global vault per user, located via `~/.claude/vault-config.json`.
- Concept titles are globally unique. Cross-domain inline `[[links]]` are required on every concept.
- `status` describes the note; `confidence` describes the user — never collapse them.
- `personal/` is off-limits without explicit user permission.
- `ask` writes only to `_claude/drafts/`. All real vault writes go through `update-vault`.
