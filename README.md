# Sin

A set of Claude Code skills that ground code generation in the user's own Obsidian knowledge vault. The premise: don't let the model write code for topics the user doesn't understand. The vault is the user's knowledge map; skills read it before they write.

## Why

LLMs happily generate code the user can't explain, debug, or maintain. `sin` makes that harder. Every code-generating action passes through a gate: the topic must already exist in the vault as a concept the user has filed with their own paraphrase. If it doesn't, the user is routed to `/learn` (a Socratic loop that never hands over the obvious answer, or a bridge-mode analogy to something they already know) and then files the concept before code is written.

## Skills

All skills live in `skills/<skill-name>/SKILL.md`.

| Skill | Purpose |
|---|---|
| `init-vault` | One-shot scaffold for a new Obsidian vault. Creates folder structure, frontmatter templates, root MOC, and writes `~/.claude/vault-config.json`. |
| `add-new-concepts` | Explicit write path into the vault. Files a new concept, creates any missing MOCs / sub-MOCs along the parent chain, and enforces cross-domain inline `[[links]]`. |
| `learn` | The understanding front-door, two modes. **Bridge mode** (read-only) builds one analogy from notes the user already understands — triggered when they're stuck mid-question or ask to map an idea to something familiar. **Loop mode** runs a from-scratch Socratic teaching loop that never hands over the obvious answer, then routes to `add-new-concepts` once understanding is shown. |
| `plan` | Grounded project planning. Reads the vault to find foundation/gaps for a stated goal; refuses to write a plan until the user has (or files) the required knowledge. |
| `generate` | Code generation. Stops unless the topic is grounded in the vault. Never expands scope silently. |
| `review` | Recall audit. Hides a filed concept, makes the user explain it cold, then reveals and updates `confidence:` on how recall actually went. The one skill that catches decayed understanding before `plan`/`generate` build on it. Confidence can go down. |
| `restructure` | Structural surgery on an existing vault — split overgrown MOCs, re-home misfiled concepts, merge thin MOCs, fix broken chains. Edits `parent:`/`domain:`/location only; never changes a note's meaning, `confidence:`, or `status:`. Proposes the full move plan and waits. |
| `transfer` | Export/migrate a slice of the vault to another vault or a portable bundle. Read-only on the source; carries MOC context and resolves links so the export isn't orphaned. Flags hollow/draft notes rather than exporting stubs silently. |

## Vault structure

Defined in `vault-architecture.md` and mirrored inside each vault at `_claude/vault-spec.md`.

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

Five note types — `concept`, `research`, `project`, `review`, `moc` — each with a defined frontmatter schema and lifecycle (see `vault-architecture.md`).

## Install

### 1. Install Claude Code (if you haven't already)

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. Install the skills

**Recommended — one-liner via the `skills` CLI:**

```bash
npx skills add Sinmunyam/sin
```

This clones the repo and drops each folder under `skills/` into `~/.claude/skills/`. Works on macOS, Linux, and Windows (PowerShell).

Install a single skill instead of all of them:

```bash
npx skills add Sinmunyam/sin/generate
```

**Manual alternative — git clone + copy:**

```bash
git clone https://github.com/Sinmunyam/sin.git
# macOS / Linux
cp -r sin/skills/* ~/.claude/skills/
# Windows (PowerShell)
Copy-Item -Recurse sin\skills\* $env:USERPROFILE\.claude\skills\
```

### 3. Initialize your vault

Once the skills are in place, run Claude Code and invoke the first skill:

```
> Use init-vault to set up my vault at ~/my-vault
```

This scaffolds your Obsidian vault structure and writes `~/.claude/vault-config.json`.

## Getting started

1. Run `init-vault` and point it at a directory. It scaffolds the structure and writes `~/.claude/vault-config.json`.
2. Use `add-new-concepts` to file a new concept you already understand. It asks you to paraphrase the idea in one sentence (one beat of friction, not a quiz), creates any missing MOCs along the parent chain, and enforces cross-domain links.
3. When you want to understand a new topic, use `/learn <topic>`. Bridge mode gives one analogy to something already in your vault (read-only, never files); loop mode walks you through a brand-new topic from scratch with questions, not answers. If it clicks, loop mode offers to file it via `add-new-concepts` — and either way, filing is *your* call.
4. Use `plan` to scope new work — it will refuse if your vault doesn't show the foundation.
5. Use `generate` to write code — it will refuse if the topic isn't grounded.
6. Use `review` to re-prove a concept later. It hides the note, asks you to explain it cold, and adjusts `confidence:` on how you actually did — downgrading when recall slips. As the vault grows, `restructure` rebalances the MOC tree and `transfer` exports a slice elsewhere.

## Cold-start flow (empty vault, brand-new topic)

A genuinely new topic with nothing related in your vault has two paths:

1. **Guided:** run `/learn <topic>` in loop mode. It teaches from scratch through questions rather than definitions, makes you prove understanding before advancing, and offers to file the concept via `add-new-concepts` once you own it.
2. **External:** learn it yourself — official docs, a textbook, a course (bridge mode can suggest sources when it finds no vault anchor) — then run `add-new-concepts` to file the concept in your own words.

Either way, `plan` and `generate` stay gated on the filed concept. The bar is "you've internalized this enough to write a one-sentence definition," not "the model gave you the answer." `/learn` loop mode is built to honor that bar, not bypass it — it withholds the obvious answer precisely so the understanding is yours.

## Known gaps (deferred)

- **No `status:` write path.** `review` now drives the `confidence:` axis (`learning → solid → fluent`, and back down on a failed recall). But transitioning `status:` (`draft → verified → outdated`) still has no dedicated skill — edit that frontmatter directly for now.
- **Concept-only writes.** `add-new-concepts` files concepts. Research, project, and review notes have schemas but no dedicated write skill yet — create them by copying from `templates/` and editing.

## Layout

```
skills/              # skill definitions (SKILL.md per skill)
vault-template/      # canonical templates init-vault copies into new vaults
vault-architecture.md   # the vault spec
README.md
LICENSE
```

## Conventions

- One global vault per user, located via `~/.claude/vault-config.json`.
- Concept titles are globally unique. Cross-domain inline `[[links]]` are required on every concept.
- `status` describes the note; `confidence` describes the user — never collapse them.
- `personal/` is off-limits without explicit user permission.
- `/learn` bridge mode is read-only. All vault writes go through `add-new-concepts`.
