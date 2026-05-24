---
name: init-vault
description: Scaffold a new Obsidian knowledge vault from the canonical template. Use only on first-time setup. Creates folder structure, frontmatter templates, root MOC, and writes ~/.claude/vault-config.json so other skills (ask, update-vault, generate) can find the vault.
---

# Init Vault

One-shot scaffold for a new Obsidian vault that conforms to the project's knowledge-management spec.

This skill is **not** for repairing or modifying existing vaults — use `update-vault` for that.

---

## Trigger

Activate when the user asks to:

- "Initialize my vault" / "set up a vault" / "create a vault" / "scaffold a vault".
- Or runs the skill explicitly by name.

Do **not** activate to add or edit notes — that's `update-vault` or `ask`.

---

## Step 0 — Refuse if already initialized

1. Check `~/.claude/vault-config.json`.
2. If it exists and points to a directory that contains `_claude/vault-spec.md` → refuse:
   - _"Vault already initialized at `<path>`. Use `update-vault` to modify it, or delete the config file to reinitialize."_
3. If it exists but points elsewhere → refuse and tell the user to edit or delete the config file.

---

## Step 1 — Resolve vault path

1. If the user passed a path, use it.
2. Else ask one question: _"Where should I create the vault? (absolute path)"_
3. Expand `~` and resolve to absolute.
4. If the target directory exists and is non-empty:
   - Refuse unless the user explicitly confirms overwriting (treat as `--force`).
   - Never delete existing files; only add to the directory.

---

## Step 2 — Create folder tree

Create the following directories under `<vault_path>/`:

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

Add an empty `.gitkeep` to `personal/`, `research/`, `projects/`, `reviews/`, `attachments/` so they survive git initialization.

---

## Step 3 — Copy template files

The canonical template lives in this repo at `vault-template/`. From the SKILL.md location at `.agents/init-vault/SKILL.md`, that resolves to `../../vault-template/`.

Copy the following files into the new vault, preserving structure:

| Source (in repo)                       | Destination (in vault)     |
|----------------------------------------|----------------------------|
| `vault-template/concept.md`            | `templates/concept.md`     |
| `vault-template/research.md`           | `templates/research.md`    |
| `vault-template/project.md`            | `templates/project.md`     |
| `vault-template/review.md`             | `templates/review.md`      |
| `vault-template/moc.md`                | `templates/moc.md`         |
| `vault-template/_claude/vault-spec.md` | `_claude/vault-spec.md`    |
| `vault-template/_claude/domains.md`    | `_claude/domains.md`       |
| `vault-template/concepts/_moc.md`      | `concepts/_moc.md`         |

Copy verbatim — do not improvise frontmatter or content.

---

## Step 4 — Write global config

Create `~/.claude/vault-config.json`:

```json
{
  "vault_path": "<absolute-vault-path>",
  "initialized_at": "<YYYY-MM-DD>"
}
```

If `~/.claude/` does not exist, create it.

---

## Step 5 — Self-check

Before reporting success, verify:

1. `ls <vault_path>` returns all 8 top-level folders.
2. `ls <vault_path>/templates` returns 5 template files.
3. `cat ~/.claude/vault-config.json` returns valid JSON with `vault_path` matching the actual path.
4. `<vault_path>/concepts/_moc.md` exists and has `type: moc` in its frontmatter.

If any check fails, report the failure rather than claiming success.

---

## Step 6 — Report

Show the user:

- The vault path
- The config file path
- A one-line summary of what was created (folder count, template count)
- Next steps: _"You can now use `update-vault` to add notes and `ask` to learn topics against this vault."_

---

## Hard rules

- Never overwrite an existing file inside the vault, even with `--force`. `--force` only allows scaffolding into a non-empty directory; it does not allow clobbering files.
- Never modify `personal/` beyond creating the empty folder.
- Never edit `~/.claude/vault-config.json` if it already exists and points to a valid initialized vault.
- Never improvise template content — copy verbatim from `vault-template/`.
- Never run a second time on the same vault. Direct the user to `update-vault`.
