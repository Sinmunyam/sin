# Sin

A set of Claude Code skills that ground code generation in the user's own Obsidian knowledge vault. The premise: don't let the model write code for topics the user doesn't understand. The vault is the user's knowledge map; skills read it before they write.

## Why

LLMs happily generate code the user can't explain, debug, or maintain. `sin` makes that harder. Every code-generating action passes through a gate: the topic must already exist in the vault as a concept the user has filed with their own paraphrase. If it doesn't, the user is routed to learn (externally or via `/explain` analogies) and then file the concept before code is written.

## Skills

All skills live in `skills/<skill-name>/SKILL.md`.

| Skill | Purpose |
|---|---|
| `init-vault` | One-shot scaffold for a new Obsidian vault. Creates folder structure, frontmatter templates, root MOC, and writes `~/.claude/vault-config.json`. |
| `add-new-concepts` | Explicit write path into the vault. Files a new concept, creates any missing MOCs / sub-MOCs along the parent chain, and enforces cross-domain inline `[[links]]`. |
| `explain` | On-demand analogy helper. Read-only. Triggered when the user is visibly stuck answering a clarifying question from another skill, or manually via `/explain <topic>`. Builds an analogy from notes the user already understands; never writes. |
| `plan` | Grounded project planning. Reads the vault to find foundation/gaps for a stated goal; refuses to write a plan until the user has (or files) the required knowledge. |
| `generate` | Code generation. Stops unless the topic is grounded in the vault. Never expands scope silently. |

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
npx skills add A-SKE22-Team/sin
```

This clones the repo and drops each folder under `skills/` into `~/.claude/skills/`. Works on macOS, Linux, and Windows (PowerShell).

Install a single skill instead of all of them:

```bash
npx skills add A-SKE22-Team/sin/generate
```

**Manual alternative — git clone + copy:**

```bash
git clone https://github.com/A-SKE22-Team/sin.git
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
3. When you're stuck mid-question, or you want to bridge an unfamiliar idea to something already in your vault, use `/explain <topic>`. It only reads, never files. If the topic clicks afterward, it's *your* call to file it via `add-new-concepts`.
4. Use `plan` to scope new work — it will refuse if your vault doesn't show the foundation.
5. Use `generate` to write code — it will refuse if the topic isn't grounded.

## Cold-start flow (empty vault, brand-new topic)

By design, none of these skills will teach you a topic from scratch. If a topic is genuinely new and there's nothing related in your vault to anchor an analogy:

1. Learn the topic externally — official docs, a textbook, a course. `/explain` can suggest sources when it can't find a vault anchor.
2. Return and run `add-new-concepts` to file the concept in your own words. The paraphrase step exists for exactly this moment.
3. Then `plan` or `generate` will accept it.

This is intentional: the gate is "you've internalized this enough to write a one-sentence definition," not "the model explained it to you."

## Known gaps (deferred)

- **No lifecycle write path.** Transitioning `status` (`draft → verified → outdated`) or `confidence` (`learning → solid → fluent`) isn't covered by any current skill. Edit the frontmatter directly for now.
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
- `explain` is read-only. All vault writes go through `add-new-concepts`.
