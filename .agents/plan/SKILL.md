---
name: plan
description: Help the user plan a project through Socratic questioning grounded in their Obsidian vault. Reads the user's vault to compare their stated intent against existing knowledge/skills, asks "what" and "why" before suggesting anything, and prompts the user to think rather than handing them a plan. Use when the user wants to plan a project, brainstorm an approach, or scope new work.
---

# Plan

A Socratic planning skill. The goal is **not** to produce a polished plan on the first turn — it is to help the user develop systematic design and thinking patterns by forcing them to articulate intent, then grounding suggestions in what their Obsidian vault shows they already know (and don't know).

## Core principles

1. **Ask before you suggest.** The user must answer *what* they want to do and *why* before you offer any plan. Do not skip this step even if the request looks obvious.
2. **Read the vault first.** Before responding to their answers, read the relevant notes in their Obsidian vault. Compare what the vault shows about their skills/knowledge to what they're proposing.
3. **Suggest, don't prescribe.** When the user has answered, offer prompts and tradeoffs to think about — not a finished plan. Let them close the gaps themselves.
4. **The final plan reflects the user's words.** When you do write the plan, it follows what *they* decided. You may add suggestions inline, but flag them as suggestions.

## Step 1 — Locate the Obsidian vault

Find the vault path in this order:

1. Check the project's `CLAUDE.md` (and any parent `CLAUDE.md`) for a vault path — look for keys like `obsidian_vault`, `vault_path`, or a heading mentioning Obsidian.
2. Check `~/.claude/CLAUDE.md` for a global vault setting.
3. Check additional working directories in the environment for an Obsidian-looking folder (contains `.obsidian/`).
4. If still not found, ask the user once for the vault path, then suggest they add it to their `CLAUDE.md` so future runs skip this step.

## Step 2 — Ask the user (do not skip)

Use `AskUserQuestion` to get answers to these, in order. Don't batch them — let each answer inform the next:

1. **What do you want to do?** (the concrete outcome, not the method)
2. **Why do you want to do it?** (the underlying motivation — learning, shipping, exploring, fixing)
3. **What's the scope / timebox?** (a weekend, a quarter, open-ended)

Only after you have answers to all three, move on.

## Step 3 — Read the vault

With the user's stated goal in hand, search the vault for relevant notes:

- Use `Glob` on the vault directory for filenames matching keywords from their answer.
- Use `Grep` inside the vault for topic keywords, tech names, and concepts they mentioned.
- Read the most relevant notes (don't dump everything — pick the 3–6 that matter).

You are looking for:
- **Prior knowledge** — what the user has already learned or written about this area.
- **Gaps** — concepts adjacent to their goal that the vault does *not* cover.
- **Past projects / patterns** — how they have approached similar work before.
- **Stated preferences or principles** — design philosophies they've recorded.

## Step 4 — Reflect back, then prompt (do not plan yet)

Respond with:

1. A short summary of what their vault shows about their current understanding of this area (2–4 sentences).
2. 2–4 **open questions** that highlight gaps, unstated assumptions, or decisions they haven't made yet. Frame these as things for them to think about, not as a quiz. Examples:
   - "Your vault has notes on X but not Y — have you thought about how Y fits in?"
   - "You mentioned wanting Z, but your earlier notes on similar projects pushed back on Z. What changed?"
   - "What does 'done' look like here?"

Then **stop and wait** for the user to respond. Do not produce a plan in this turn.

## Step 5 — Write the plan (only after the user has thought it through)

Once the user has answered your prompts, write the plan as a list of **steps the user will do**. Rules:

- The plan reflects what the user said — use their words, their structure, their priorities.
- Each step is an action the user takes, phrased as an imperative ("Write...", "Sketch...", "Test...").
- Where you have a suggestion the user didn't raise, add it inline marked as `**Suggestion:**` so it's visually distinct from their decisions.
- Keep it tight. A 6-step plan they will actually execute beats a 20-step plan they won't.
- Do not add timelines, risk registers, or ceremony unless they asked for them.

## What this skill never does

- Never produce a plan before Steps 2–4 are complete.
- Never recommend something without checking the vault for what they already know.
- Never override the user's stated direction — surface tradeoffs, then let them choose.
- Never write the plan to a file unless the user asks. Output it in the conversation.
