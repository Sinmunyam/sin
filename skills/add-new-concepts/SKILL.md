---
name: add-new-concepts
description: File a new concept note in the user's Obsidian vault. Links it to related concepts and to the MOC tree, creating any missing parent or sub-MOCs along the chain. Trigger when the user says things like "file this", "add a concept for X", "save this to my vault", "I just learned about Y, log it", "note this down as a concept", even when they don't name a skill. Assumes the user already understands the topic and does not teach.
---

# Add New Concepts

Write-path for filing concept notes. The skill does discovery, a one-sentence paraphrase check, path inference, cross-domain link proposal, MOC and sub-MOC creation along the parent chain, template fill, and final write/verify.

Assumes the user already understands what they're filing. If they're shaky, route to `/learn` first — bridge mode for an analogy from something already in their vault, or loop mode to work through it from scratch.

## Step 0: Resolve vault path

1. Read `~/.claude/vault-config.json` for `vault_path`.
2. If missing, or if the path has no `_claude/vault-spec.md`, stop: _"No initialized vault found. Run `init-vault` first."_
3. Read `_claude/vault-spec.md` to refresh schema and lifecycle rules.

## Step 1: Discovery (mandatory)

Search before writing. In order:

1. `find <vault>/concepts -iname "*<term>*"` for filename match.
2. `grep -ril "^title:.*<term>" <vault>/concepts` for title match.
3. `grep -ril "^aliases:.*<term>" <vault>/concepts` for alias match.

If a match exists, stop and ask: _"This concept already exists at `<path>`. Do you want to add to it, or is this a different idea that needs a distinct title?"_ Never silently overwrite. Concept titles are globally unique.

## Step 2: One-beat paraphrase check

Ask once: _"In one sentence, what's the core idea of [topic]?"_

This is one beat of friction, not a knowledge gate or a teaching loop. It catches the case where the user can name a concept but hasn't internalized it. Use the answer verbatim as the opening line of the concept body in Step 6. It seeds the note in their voice and leaves a paper trail if the understanding turns out wrong later.

If the paraphrase is vague, evasive, or describes a different concept, flag it once: _"That sounds more like [Y] than [X]. Did you mean to file [Y]? Or want to run `/learn [X]` first?"_

Then accept whatever the user decides. Do not loop, grade, or refuse. The user owns their vault.

## Step 3: Infer path and domain

- Domain: ask, or infer from the topic. Reuse an existing slug from `<vault>/_claude/domains.md` when it fits.
- Path: `concepts/<domain>/<kebab-title>.md`. Example: `concepts/ai/ml/random-forest.md`.
- Ask about path or domain only when genuinely ambiguous.

## Step 4: Propose cross-domain links (before touching MOCs)

The vault spec requires every concept to have at least one cross-domain inline `[[link]]`. Settle this **before** any MOC writes, so a rejection here doesn't leave orphan MOC entries pointing at a not-yet-written note.

1. Extract key terms from the paraphrase and any draft content.
2. Grep the vault for matching titles and aliases in other domains.
3. Propose 2 to 4 candidates: _"I found these possibly related notes. Confirm which to link: [[Bagging]], [[Decision Tree]], [[Bias-Variance Tradeoff]]."_
4. Insert only approved links. Never auto-insert.
5. If the user approves zero, stop. Ask them to name a related concept to file first, or pick from the candidates. Do not proceed to MOC linking or write.

## Step 5: Walk the MOC chain and create what's missing

Every concept must hang off the MOC tree. Walk the parent chain from the new note's folder up to `concepts/_moc.md` and create any missing MOC along the way.

Example: for `concepts/ai/ml/random-forest.md`, the chain is `ml`, `ai`, root.

Procedure:

1. Walk up from the new note's folder to `concepts/_moc.md`.
2. For every segment whose `_moc.md` is missing, create it from `<vault>/templates/moc.md`:
   - `title:` title-cased from the segment (e.g. `ml` becomes `Machine Learning`).
   - `type: moc`
   - `status: verified`
   - `parent:` the next-up MOC slug, or the root MOC at the top of the chain.
   - `domain:` the path up to and including this segment.
   - Save as `concepts/<segment-path>/_moc.md`.
3. Append a wikilink to the new concept from the deepest folder's `_moc.md` only. Each MOC lists its direct children, so do not append the new concept to every MOC up the chain.
4. For each newly created sub-MOC, append its wikilink to its parent MOC.
5. Register the domain in `<vault>/_claude/domains.md` if the slug is new.

Report what was created: _"Created `concepts/ai/ml/_moc.md`. Linked new concept under it. Linked the new `ml` MOC under existing `ai` MOC."_

## Step 6: Draft and write the concept

Copy `<vault>/templates/concept.md` to the inferred path. Fill frontmatter per the spec:

- `title:` globally unique. Reject duplicates.
- `aliases:` ask the user.
- `type: concept`
- `parent:` nearest MOC slug, e.g. `[[ml]]`.
- `domain:` slash-separated, e.g. `ai/ml`.
- `status: draft`
- `confidence:` ask. Default `learning`.

Body: open with the user's paraphrase from Step 2, verbatim. Insert the approved cross-domain links from Step 4 inline. Leave the rest for the user to fill, or expand it together. Do not invent vault content the user did not say.

Write the file.

## Step 7: Verify

1. Re-run discovery (Step 1) to confirm the new note is findable.
2. Confirm each MOC along the parent chain lists the right child.
3. Report: created concept path, MOCs created or updated, cross-domain links inserted.

## Hard rules

- Never edit anything under `personal/`.
- Never run before `~/.claude/vault-config.json` exists. Direct the user to `init-vault`.
- Never create a duplicate concept title. Abort with the existing path.
- Never auto-insert `[[wikilinks]]` the user has not approved.
- Never write a concept with zero cross-domain inline links.
- Never silently overwrite an existing note. Discover first, ask before modifying.
- Never teach inside this skill. For analogy help or teaching, route to `/learn`.
- Never modify MOCs before cross-domain links are settled. Rejection at the link step must not leave orphan MOC entries.
