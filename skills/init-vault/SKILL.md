---
name: init-vault
description: Scaffold a new Obsidian knowledge vault from the canonical template on first-time setup. Trigger on phrasings like "initialize my vault", "set up my Obsidian vault", "scaffold a vault", "I want to start tracking my knowledge", "create a vault at /path", or whenever the user is clearly starting fresh and the other vault skills (`add-new-concepts`, `explain`, `generate`, `plan`) would fail because `~/.claude/vault-config.json` does not exist yet. Creates the folder tree, frontmatter templates, root MOC, and the global config file. Refuses on second invocation against an already-initialized vault.
---

# Init Vault

One-shot scaffold for a new Obsidian vault that conforms to the project spec. Not for repairing or modifying existing vaults; use `add-new-concepts` to file notes.

## Trigger

Activate when the user asks to initialize, set up, create, or scaffold a vault, or runs the skill by name. Do not activate to add or edit notes.

## Step 0: Refuse if already initialized

1. Check `~/.claude/vault-config.json`.
2. If it exists and points to a directory containing `_claude/vault-spec.md`, refuse: _"Vault already initialized at `<path>`. Use `add-new-concepts` to add notes, or delete the config file to reinitialize."_
3. If it exists but points elsewhere, refuse and tell the user to edit or delete the config file.

## Step 1: Resolve vault path

1. If the user passed a path, use it. Otherwise ask once: _"Where should I create the vault? (absolute path)"_
2. Expand `~` to absolute.
3. If the target directory exists and is non-empty, refuse unless the user explicitly says in this turn that you may scaffold into it. Never delete existing files. Only add.

## Step 2: Create folder tree

Create under `<vault_path>/`:

```
_claude/
personal/
concepts/
research/
projects/
reviews/
attachments/
templates/
```

Add an empty `.gitkeep` to `personal/`, `research/`, `projects/`, `reviews/`, `attachments/` so they survive git init.

## Step 3: Copy template files

Copy verbatim from `vault-template/` in this repo (at `../../vault-template/` relative to this SKILL.md):

| Source                                  | Destination                |
| --------------------------------------- | -------------------------- |
| `vault-template/concept.md`             | `templates/concept.md`     |
| `vault-template/research.md`            | `templates/research.md`    |
| `vault-template/project.md`             | `templates/project.md`     |
| `vault-template/review.md`              | `templates/review.md`      |
| `vault-template/moc.md`                 | `templates/moc.md`         |
| `vault-template/_claude/vault-spec.md`  | `_claude/vault-spec.md`    |
| `vault-template/_claude/domains.md`     | `_claude/domains.md`       |
| `vault-template/concepts/_moc.md`       | `concepts/_moc.md`         |

Do not improvise frontmatter or content. Copy as-is.

## Step 4: Write global config

Create `~/.claude/vault-config.json`:

```json
{
  "vault_path": "<absolute-vault-path>",
  "initialized_at": "<YYYY-MM-DD>"
}
```

Create `~/.claude/` if it does not exist.

## Step 5: Self-check

Before reporting success, verify:

1. `ls <vault_path>` returns all 8 top-level folders.
2. `ls <vault_path>/templates` returns 5 template files.
3. `~/.claude/vault-config.json` is valid JSON with `vault_path` matching the actual path.
4. `<vault_path>/concepts/_moc.md` exists and has `type: moc` in its frontmatter.

If any check fails, report the failure rather than claiming success.

## Step 6: Report

Show the user:

- The vault path.
- The config file path.
- A one-line summary (folder count, template count).
- Next steps: _"You can now use `add-new-concepts` to file new notes, and `/explain` when you want an analogy bridge to something already in your vault."_

## Hard rules

- Never overwrite an existing file inside the vault, even when the user has given permission to scaffold into a non-empty directory.
- Never modify `personal/` beyond creating the empty folder.
- Never edit `~/.claude/vault-config.json` if it already exists and points to a valid initialized vault.
- Never improvise template content. Copy verbatim from `vault-template/`.
- Never run a second time on the same vault. Direct the user to `add-new-concepts`.
