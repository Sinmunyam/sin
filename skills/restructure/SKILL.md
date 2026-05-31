---
name: restructure
description: Reorganize an existing vault's MOC tree and concept placement as it grows — split an overgrown domain into sub-MOCs, re-home a concept that landed in the wrong domain, merge thin MOCs, or rebalance the parent chain. Operates on structure only; never changes a note's meaning, `confidence:`, or `status:`. Trigger on phrasings like "my vault is getting messy", "this domain has too many notes", "reorganize my MOCs", "this concept is in the wrong place", "split X into sub-topics", "merge these MOCs", "rebalance my vault". Proposes the full move plan and waits for approval before touching anything.
---

# Restructure

The knowledge map drifts as it grows. A domain that started with three concepts now has thirty and needs sub-MOCs. A concept got filed under `ai` when it really belongs in `ai/ml`. Two MOCs cover the same ground. `restructure` fixes the *shape* of the vault without touching the *content* of any note. It is surgery on `parent:`, `domain:`, MOC membership, and file location — nothing else.

It never decides whether the user understands a topic (`review`), never teaches (`learn`), never files new ideas (`add-new-concepts`). If a reorganization reveals a *missing* concept or MOC, it creates the MOC scaffold but routes new *concepts* to `add-new-concepts`.

## Step 0: Resolve vault path

1. Read `~/.claude/vault-config.json` for `vault_path`.
2. If missing, stop: _"No initialized vault found. Run `init-vault` first."_
3. Read `_claude/vault-spec.md` to refresh the MOC and linking rules. The "Granularity" and "Linking" sections govern this skill.

## Step 1: Diagnose before proposing

Survey the current structure. Do not move anything yet.

- Inventory MOCs and their child counts: `find <vault>/concepts -name "_moc.md"`, and for each, count the concepts that name it as `parent:`.
- Flag structural smells against the spec:
  - **Overgrown MOC:** a domain with many concepts and no sub-structure (spec: "Subfolder only at 5+ notes"). Candidate for a sub-MOC split.
  - **Misfiled concept:** `domain:` or `parent:` that doesn't match where the note actually sits, or a note whose topic clearly belongs elsewhere.
  - **Thin MOC:** a MOC with one child that could fold into its parent.
  - **Topic-shaped concept:** a "concept" that's really a MOC in disguise (spec: `Loss Functions` is a MOC, `Cross-Entropy` is a concept). Flag for promotion to MOC.
  - **Broken chain:** a concept whose `parent:` MOC doesn't exist, or a MOC not reachable from `concepts/_moc.md`.

Report the diagnosis plainly. If the vault is structurally healthy, say so and stop — do not invent reorganization to justify the invocation.

## Step 2: Propose the full move plan (and wait)

Present the complete set of moves as one reviewable plan **before touching any file**. For each move, state the before and after:

- _"Move `concepts/ai/backprop.md`: `domain: ai` → `ai/dl`, `parent: [[ai]]` → `[[dl]]`. Create `concepts/ai/dl/_moc.md` (new sub-MOC under [[ai]])."_
- _"Promote `Loss Functions` from concept to MOC; re-home its 4 current siblings under it."_
- _"Merge thin MOC [[stats-basics]] into [[statistics]]; re-point its 2 children."_

Wait for explicit approval. Restructuring touches many files and breaks links if done wrong — this is exactly the kind of hard-to-reverse, multi-file change that needs a confirmed plan, not silent execution. If the user approves only part, execute only that part.

## Step 3: Execute, preserving every link

Apply the approved moves. The invariant: **no concept loses its place in the tree and no `[[link]]` dangles.**

For each move:
1. If moving a file, move it and update its `domain:` and `parent:` frontmatter to match the new location.
2. Create any new `_moc.md` from `<vault>/templates/moc.md`, filling `title:` (title-cased segment), `type: moc`, `parent:` (next-up MOC), `domain:` (path to this segment). MOCs are always `status: verified` per the spec.
3. Update MOC membership: remove the concept's wikilink from the old MOC, add it to the new deepest MOC. Each MOC lists only its direct children.
4. For a promoted concept→MOC, preserve its body content; re-point its former siblings' `parent:`.
5. For a merged MOC, re-point all children to the surviving MOC, then delete the empty one.
6. **Find and fix inbound links.** Grep the whole vault for `[[Old Title]]` (and `parent: [[old-slug]]`) and update them so nothing dangles. A title change is not done until every inbound reference is repointed.
7. Register or retire domain slugs in `<vault>/_claude/domains.md`.

## Step 4: Verify the tree is whole

1. Every moved/created concept is reachable from `concepts/_moc.md` by walking `parent:` links.
2. No MOC lists a child that no longer exists; no child names a `parent:` MOC that's missing.
3. Grep for the old titles/slugs — zero remaining inbound references.
4. Report: files moved, MOCs created/merged/deleted, links repointed.

## Hard rules

- **Never change a note's meaning.** This skill edits `parent:`, `domain:`, location, and MOC membership only. Bodies are touched solely to repoint inbound `[[links]]`, never to rewrite content.
- **Never touch `confidence:` or `status:`.** Structure is orthogonal to understanding and lifecycle. Those are `review` and `transfer`.
- **Never execute before the user approves the move plan.** Multi-file structural changes are hard to reverse; present the full plan first.
- **Never leave a dangling link.** A rename or move is incomplete until every inbound reference across the vault is repointed.
- **Never orphan a concept.** Every note must still hang off the MOC tree after the move.
- **Never create new concepts here.** If restructuring surfaces a missing idea, route to `add-new-concepts`. This skill creates MOC scaffolding only.
- **Never edit `personal/`.** Off-limits.
- **Never invent reorganization.** If the vault is structurally sound, say so and stop.
