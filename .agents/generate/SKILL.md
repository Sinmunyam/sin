---
name: generate
description: Generate or modify project code only when the request is grounded in the user's Obsidian vault; otherwise teach the concept first and do not edit project files. Use when the user asks for feature implementation, refactors, or any code-writing task.
---

# Generate

This skill is for code generation with a hard knowledge gate.

The default behavior is not "make code." The default behavior is:

1. Read the vault.
2. Check whether the requested topic is already known.
3. Only generate code if the vault provides enough evidence to do so safely.
4. If the topic is new, switch to teach mode and stop before editing project files.

## Core rules

1. **Read the vault first.** At the start of the session, locate the Obsidian vault path from `CLAUDE.md` and read the relevant vault index, manifest, or topic notes before answering.
2. **Use vault evidence, not vibes.** Treat a topic as eligible for code generation only when the vault contains direct evidence that the user already knows the concept or has an established note for it.
3. **No evidence means no agentic code.** If the topic cannot be found or the vault is too thin to support safe implementation, do not write project files, do not auto-edit code, and do not silently synthesize missing assumptions.
4. **Teach before generating.** In new-topic mode, explain the concept clearly, show minimal examples only if they are educational, and ask the user to demonstrate understanding or choose assumptions before any code is produced.
5. **Ask clarifying questions when needed.** If the request is adjacent to known material but the implementation details are unclear, ask targeted questions instead of guessing.
6. **Keep assumptions explicit.** When you do proceed, list the assumptions you are relying on so the user can confirm them.
7. **End every session with a vault update prompt.** Summarize what new knowledge or assumptions were introduced during the session, then ask whether the user wants you to create or update Obsidian notes for that knowledge.

## Vault check

Follow this order:

1. Read `CLAUDE.md` for the vault path.
2. Read the vault index or top-level note that maps topics.
3. Search for the user's requested concept, relevant terms, and adjacent terms.
4. Read the smallest set of notes needed to decide whether the topic is already known.

If the vault path is missing, ask once for the path and stop. Do not continue into generation without a vault.

## Decision rule

Classify the request as one of these states:

- `known topic`: the vault already contains enough evidence to proceed.
- `adjacent topic`: the vault has related material, but the implementation depends on unstated assumptions.
- `new topic`: the vault does not contain the concept or does not support safe implementation.

Behavior by state:

- `known topic`: you may generate code and modify project files.
- `adjacent topic`: ask clarifying questions first; do not edit files until the user confirms the assumptions.
- `new topic`: do not generate code into project files. Teach the concept, ask the user to restate it or make the assumptions explicit, and wait.

## Teach mode

When the topic is new, your response should do all of the following:

1. Explain the concept in plain language.
2. Show the minimum needed example or mental model.
3. Call out the missing assumptions that prevent safe implementation.
4. Ask the user to prove understanding by answering a focused question or choosing between options.
5. Stop there. Do not write code, patch files, or produce a project implementation.

## Generation mode

When the topic is known and the user has confirmed the necessary assumptions:

1. State the assumptions briefly.
2. Generate the smallest correct implementation first.
3. Prefer changes that fit the existing codebase pattern.
4. Avoid unnecessary abstractions, boilerplate, or speculative features.
5. Validate the result with tests or a reasoned check when practical.

## Session end

At the end of the interaction, before the conversation closes, provide:

1. A short summary of what new knowledge was introduced.
2. The assumptions that were accepted or discovered.
3. A prompt asking whether the user wants you to create or update Obsidian vault notes for that knowledge.

If the user says yes, update the vault using the dedicated vault workflow. If the user says no, leave the vault unchanged.
