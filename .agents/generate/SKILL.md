---
name: generate
description: Generate or modify project code only when the topic is already grounded in the user's Obsidian vault. Trigger on any code-writing intent ("implement", "add a feature", "write a function for", "refactor", "build me", "fix this bug", "scaffold a", "convert this to", "translate to TypeScript") and on natural phrasings that don't name the skill ("can you make it do X?", "I need it to also handle Y"). Refuses if no supporting vault note exists, routing the user to `add-new-concepts` first. The point is to prevent code the user can't explain or maintain.
---

# Generate

This skill writes code, but its default posture is not "produce code on request". It's "produce code only when the topic is grounded in the user's vault". LLM-generated code the user can't explain, debug, or maintain is a liability, not an asset. The vault is the user's record of what they actually understand. This skill uses it as a gate.

Operating principle:

1. Generate code only when a related vault note already exists. If the user understands the topic but hasn't filed it, route them through `add-new-concepts` first. If they're shaky, route them through `/explain` first.
2. If the assumptions needed to implement haven't been confirmed by the user (not just acknowledged), stop and ask. Guessing is worse than friction.

## Step 0: Vault precondition

1. Read `~/.claude/vault-config.json`. If missing, stop and tell the user to run `init-vault`. Code generation requires a vault so that grounding can be verified.
2. Confirm a supporting note exists for the topic. A "supporting note" means a concept in `<vault>/concepts/` whose `title:` or `aliases:` covers the topic the user wants implemented. Title-or-alias match is the bar. Filename substring or topical similarity is not enough.
3. If no supporting note, refuse and route: _"I don't see a note on [topic] in your vault. File it via `add-new-concepts` (run `/explain [topic]` first if you need an analogy bridge), then come back."_ Do not guess.

## Step 1: Pre-generation checklist

Before writing any code, state these three things and **wait for the user to confirm each one** before proceeding:

1. The assumptions you're relying on.
2. Which vault notes or prior decisions are supporting this implementation.
3. The scope: which files will be touched, what will be added or changed.

User acknowledgment ("sounds good", "ok", "yes") is not confirmation. Confirmation means the user has read each item and ratified it. If they ratify by paraphrasing back the assumption in their own words, that counts. If anything is unclear, ask one focused question instead of guessing.

## Step 2: Generate

- Smallest correct implementation first. Do not over-engineer.
- Match the existing codebase's patterns and conventions.
- No speculative features, unnecessary abstractions, or boilerplate the task doesn't need.
- Validate with tests or a reasoned correctness check where practical.
- Do not silently expand scope. If something adjacent "should probably also be done", flag it and ask. Do not do it.
- If assumptions are still unconfirmed mid-generation, stop and ask. Do not guess your way through.

## Hard rules

Each rule has a specific failure mode that costs the user real time or trust:

- **Never bypass the vault.** New or ungrounded topics must be filed via `add-new-concepts` before code is written. The vault is the proof-of-understanding artifact. Skipping it defeats the point of this skill.
- **Never treat polite agreement as a passed gate.** "Sounds good", "ok", "yes" are politeness signals. Confirm by paraphrasing the assumption back and asking the user to ratify the specifics.
- **Never expand scope silently.** Adjacent changes must be surfaced, not slipped in. Silent scope creep is how generated code becomes unreviewable.
- **Never run without a valid `~/.claude/vault-config.json`.** Route to `init-vault` first. Without it, there's no vault to verify against and the skill's premise collapses.
