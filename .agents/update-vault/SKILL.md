---
name: update-vault
description: Add new notes (concept, research, project, review) to the user's Obsidian vault and properly link them to MOCs and related notes. Also handles note lifecycle transitions (status, confidence). Use when the user explicitly wants to file new knowledge or change a note's metadata. Do NOT use to teach — that's `ask`.
---

# Update Vault

Explicit write-path for the user's Obsidian vault. Two modes:

- **Mode A — Add note:** create a new concept / research / project / review, link it into the MOC hierarchy, propose cross-domain links.
- **Mode B — Lifecycle update:** transition a note's `status` or `confidence` per the spec.

This skill is **not** a teaching skill. If the user is asking "what is X" or "explain Y", route to `ask`. This skill assumes the user already understands the content they want filed.

---

## Step 0 — Resolve vault path

1. Read `~/.claude/vault-config.json` → `vault_path`.
2. If missing or points to a directory without `_claude/vault-spec.md` → stop:
   - _"No initialized vault found. Run `init-vault` first."_
3. Read `_claude/vault-spec.md` to refresh the schema and lifecycle rules in context.

---

## Step 1 — Classify intent

From the user's prompt, classify into one mode:

| Signal in prompt                                                | Mode |
|-----------------------------------------------------------------|------|
| "add a concept", "file this", "make a note for", "new research" | A    |
| "mark X outdated", "I'm fluent in", "resolve the review on"     | B    |

If ambiguous, ask one question to disambiguate. Do not guess.

---

## Mode A — Add note

### A1. Discovery (mandatory, never skip)

Search the vault before writing anything. In order:

1. `find <vault>/concepts <vault>/research -iname "*<term>*"` — filename match
2. `grep -ril "^title:.*<term>" <vault>/concepts <vault>/research` — title match
3. `grep -ril "^aliases:.*<term>" <vault>/concepts <vault>/research` — alias match

**If a match exists** → stop. Show the user the existing note path and ask:

- _"This already exists at `<path>`. Did you mean to update it instead?"_

Do not silently overwrite.

### A2. Infer path and type

- Type comes from the user's request (concept | research | project | review). If unclear, ask once.
- Domain comes from the user or is inferred from the topic. Check `_claude/domains.md` for existing domain slugs and reuse one if it fits.
- Path: `<type>s/<domain>/<kebab-title>.md` (e.g. `concepts/ai/ml/random-forest.md`).
- Only ask the user about the path if genuinely ambiguous.

### A3. Draft from template

Copy `<vault>/templates/<type>.md` to the inferred path. Fill the frontmatter per the spec in `_claude/vault-spec.md`:

- `title:` — globally unique for concepts. If a duplicate exists, abort.
- `aliases:` — ask the user for any.
- `parent:` — the nearest MOC slug, e.g. `[[ml]]`.
- `domain:` — slash-separated, e.g. `ai/ml`.
- `status:` — initial per type (`draft` for concept/research, `idea` for project, `open` for review).
- `confidence:` (concepts only) — ask the user; default `learning`.

### A4. MOC linking

1. Walk up the `parent:` chain from the new note's domain.
2. For every missing MOC along the chain, create it from `templates/moc.md` with `status: verified`.
3. Append a wikilink to the new note from the nearest folder's `_moc.md`.
4. If the domain is new (not in `_claude/domains.md`), append it there.

### A5. Cross-domain link proposal

For concepts, the spec **requires** at least one cross-domain inline `[[link]]`.

1. Extract key terms from the note's body (or the user's draft).
2. Grep the vault for titles/aliases matching those terms.
3. Propose 2–4 candidate links to the user, e.g.:
   - _"I found these possibly related notes — confirm which to link: [[Bagging]], [[Decision Tree]], [[Bias-Variance Tradeoff]]."_
4. Insert only the links the user approves. Never auto-insert.
5. If the user approves zero and the note is a concept → abort and ask them to either name a related concept to create or pick from the candidates. Do not write a concept with no cross-domain links.

### A6. Write and verify

1. Write the file.
2. Re-run the discovery search (A1) to confirm the new note is findable.
3. Confirm the MOC update lists the new note.
4. Report path + links created to the user.

---

## Mode B — Lifecycle update

### B1. Find the note

Same discovery search as A1. If no match, stop and tell the user. Do not create.

### B2. Validate the transition

Look up the note's `type:` and check the requested transition against this table:

| Type     | Allowed transitions                                                  |
|----------|----------------------------------------------------------------------|
| concept  | `draft → verified`, `verified → outdated`                            |
| research | `draft → verified`, `verified → outdated`                            |
| project  | `idea → active`, `active → done/paused/abandoned`, `paused → active` |
| review   | `open → resolved`                                                    |

`confidence` (concept only) may move freely between `learning | solid | fluent` on explicit user request.

Reject illegal transitions. Tell the user the current status and the legal next states.

### B3. Apply the change

1. Edit the frontmatter field.
2. **`updated:` rule:**
   - Bump to today's date on any `status` transition.
   - Do **not** bump on a `confidence` change.
3. If transitioning to `outdated`:
   - Ask the user for the superseding note title.
   - Insert `Superseded by [[<New Title>]].` as the first body line of the outdated note.
   - If the superseding note doesn't exist yet, offer to create it via Mode A.

### B4. Verify

Re-read the file and confirm the frontmatter change took effect. Report the new state.

---

## Hard rules

- Never edit anything under `personal/`.
- Never run before `~/.claude/vault-config.json` exists — direct the user to `init-vault`.
- Never create a duplicate concept title — abort with the existing path.
- Never modify `confidence:` without an explicit user request naming the new level.
- Never auto-insert `[[wikilinks]]` the user has not approved.
- Never write a concept with zero cross-domain inline links.
- Never silently overwrite an existing note — always discover first, ask before modifying.
- Never use this skill to teach. If the user is in learning mode, route to `ask`.
- Never bump `updated:` on confidence changes.
