---
name: transfer
description: Export or migrate vault knowledge to a destination outside the working vault — another vault, a portable markdown bundle, or a shareable selection. Read-only on the source vault; it copies, never moves or mutates the originals. Carries each concept's MOC context and resolves its `[[links]]` so the export isn't a pile of orphaned notes. Trigger on phrasings like "export my notes on X", "share my vault knowledge", "bundle these concepts", "migrate notes to another vault", "give me a portable copy of my X notes", "hand off my notes on Y". Refuses to export hollow or draft-only notes without flagging them.
---

# Transfer

Knowledge that only lives in one vault is hard to share or move. `transfer` produces a portable copy of a selected slice of the vault — concepts plus the MOC context and links that make them legible — without disturbing the source. It is a read-then-write-elsewhere skill: it reads the working vault and writes only to the chosen destination.

It is plumbing, deliberately scoped. It does not assess understanding (`review`), reorganize the source (`restructure`), or teach (`learn`). But it keeps the project's posture: it won't quietly export notes that are empty, draft-only, or hollow, because a bundle of stubs misrepresents what the user actually knows. It flags those and lets the user decide.

## Step 0: Resolve vault path

1. Read `~/.claude/vault-config.json` for `vault_path` (the source).
2. If missing, stop: _"No initialized vault found. Run `init-vault` first."_

## Step 1: Resolve the selection

Determine what to export, from the user's request:

- **By topic/concept:** named concepts. Grep title and alias matches. Confirm the exact notes resolved: _"Matched [[Random Forest]], [[Bagging]], [[Boosting]]. Export these three?"_
- **By domain:** everything under a `domain:` (e.g. `ai/ml`). List the concepts and MOCs in scope.
- **By MOC subtree:** a MOC and all concepts reachable beneath it.

If the selection is empty or ambiguous, ask once, then proceed. Never export the whole vault unless the user explicitly asks — and never include `personal/` (off-limits, even on explicit "everything").

## Step 2: Resolve the destination and format

Ask if not given:

- **Another vault:** an absolute path to a different initialized vault. Files land in the matching folders there.
- **Portable bundle:** a directory of self-contained markdown, suitable for sharing or archiving.

Confirm the destination path explicitly before writing. If it's another vault, check for `title:` collisions there (concept titles are globally unique within a vault) and surface any clash: _"Destination already has a `Random Forest` concept. Skip, rename, or overwrite?"_ Never silently overwrite.

## Step 3: Gather context, don't orphan

A concept torn out of the vault loses its meaning if its links and parents are stripped. For each selected concept:

1. Collect its `parent:` MOC chain up to the root, so the export carries its place in the tree.
2. Resolve inline `[[links]]` in the body. For each link, either include the target in the export (if in scope) or, if out of scope, note it so the user knows the bundle references a concept it doesn't contain: _"`Random Forest` links to [[Cross-Entropy]], which isn't in your selection. Include it, or leave it as an external reference?"_
3. Include the relevant MOCs so the exported tree is navigable.

The goal: the export reads as a coherent mini-vault, not a heap of disconnected files.

## Step 4: Flag hollow notes before exporting

Before writing, scan the selection for notes that would misrepresent the user's knowledge:

- Empty or near-empty bodies (a title and frontmatter, no real content).
- `status: draft` notes — unfinished by the user's own marking.
- `confidence: learning` concepts — the user has flagged these as not-yet-solid.

Do not silently drop or silently include them. List them: _"3 of these are `draft` or near-empty: [[X]], [[Y]], [[Z]]. Include them as-is, exclude them, or pause to finish them first?"_ The user decides. This keeps an export honest about what's actually understood.

## Step 5: Write to the destination and verify

1. Copy the resolved files (concepts, MOCs, in-scope link targets) to the destination, preserving folder structure.
2. **Source is read-only.** Originals are never moved, edited, or deleted. `transfer` copies.
3. If exporting to another vault, register any new domain slugs in *that* vault's `_claude/domains.md` and ensure the MOC chain connects to its `concepts/_moc.md`.
4. Verify: every exported concept is reachable in the destination tree; every in-scope `[[link]]` resolves; report what was written, what was skipped, and which links point outside the bundle.

## Hard rules

- **Source vault is read-only.** This skill copies outward. It never moves, edits, or deletes a source note. (Moving *within* a vault is `restructure`; that's a different skill.)
- **Never export `personal/`.** Off-limits even on "export everything."
- **Never silently overwrite at the destination.** Surface title collisions; let the user choose.
- **Never export hollow notes silently.** Flag empty, `draft`, or `learning` notes and let the user decide. An export should reflect real understanding, not stubs.
- **Never orphan exported concepts.** Carry the MOC chain and flag out-of-scope links rather than emitting disconnected files.
- **Never confirm the destination implicitly.** State the exact path and wait, since writing outside the vault is outward-facing and harder to undo.
- **Never alter the source's `status:` or `confidence:`.** Those axes belong to `transfer`'s siblings, not here.
