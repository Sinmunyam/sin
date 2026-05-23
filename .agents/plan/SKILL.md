---
name: plan
description: Help the user plan a project through Socratic questioning grounded in their Obsidian vault. Reads the vault to compare stated intent against existing knowledge, asks "what" and "why" before suggesting anything, and prompts the user to think rather than handing them a plan. Use when the user wants to plan a project, brainstorm an approach, scope new work, or figure out where to start on something — even if they don't use the word "plan". Trigger on phrases like "I want to build", "I'm thinking of starting", "how should I approach", "help me figure out", or "I don't know where to begin".
---

# Plan

A Socratic planning skill. The goal is **not** to produce a polished plan on the first turn — it is to help the user develop systematic design and thinking patterns by forcing them to articulate intent, then grounding suggestions in what their Obsidian vault shows they already know (and don't know).

## Core principles

1. **Extract before you ask.** Read what the user has already told you. If their message already answers what, why, and scope — proceed directly to reading the vault. Only ask for what is genuinely missing.
2. **Read the vault before responding.** Before reflecting anything back, search the vault. Compare what it shows about their skills/knowledge to what they're proposing.
3. **Suggest, don't prescribe.** Offer prompts and tradeoffs — not a finished plan. Let the user close the gaps themselves.
4. **The final plan reflects the user's words.** When you do write the plan, it follows what *they* decided. Flag your additions as `**Suggestion:**`.

---

## Step 1 — Locate the Obsidian vault

Find the vault path in this order:

1. Check the project's `CLAUDE.md` (and any parent `CLAUDE.md`) for a vault path — look for keys like `obsidian_vault`, `vault_path`, or a heading mentioning Obsidian.
2. Check `~/.claude/CLAUDE.md` for a global vault setting.
3. Check additional working directories for an Obsidian-looking folder (contains `.obsidian/`).
4. If still not found, ask the user once, then suggest they add it to `CLAUDE.md` so future runs skip this step.

**If the vault is inaccessible or missing:** continue with the skill using only the conversation. Skip vault references in your reflection. Note briefly: *"I couldn't read your vault, so I'm working from what you've told me."*

---

## Step 2 — Extract intent from what they've already said

Before asking anything, check whether the user's message already contains answers to these three questions:

- **What** — the concrete outcome (not the method)
- **Why** — the underlying motivation (learning, shipping, exploring, fixing)
- **Scope / timebox** — a weekend, a quarter, open-ended

If all three are answered or clearly implied, skip to Step 3.

If one or two are missing, ask only for what you don't have — in a single message. Do not ask all three if only one is missing. Do not batch questions mechanically; weave them naturally into the conversation.

---

## Step 3 — Read the vault

With the user's stated goal in hand, search the vault for relevant notes:

- Use `Glob` on the vault directory for filenames matching keywords from their answer.
- Use `Grep` inside the vault for topic keywords, tech names, and concepts they mentioned.
- Read the 3–6 most relevant notes. Don't dump everything.

You are looking for:
- **Prior knowledge** — what the user has already learned or written about this area.
- **Gaps** — concepts adjacent to their goal that the vault does *not* cover.
- **Past projects / patterns** — how they've approached similar work before.
- **Stated preferences or principles** — design philosophies they've recorded.

---

## Step 4 — Reflect back, then prompt (do not plan yet)

Respond with:

1. A short summary (2–4 sentences) of what the vault shows about their current understanding of this area.
2. **2–3 open questions** that highlight the highest-stakes gaps or unstated assumptions — not a quiz, not low-signal observations. Prioritize questions where the answer would materially change the plan. Examples:
   - "Your vault has notes on X but not Y — have you thought about how Y fits in?"
   - "You mentioned wanting Z, but your earlier notes on similar projects pushed back on Z. What changed?"
   - "What does 'done' look like here — is this something you'll ship, or something you want to understand?"

Then **stop and wait**. Do not produce a plan in this turn.

---

## Step 5 — Write the plan

Once the user has responded to your prompts, write the plan. Don't wait for perfect answers — if they've engaged with your questions at all, that's enough to proceed.

Rules:
- Use their words, their structure, their priorities.
- Each step is an action phrased as an imperative ("Write...", "Sketch...", "Test...").
- Inline suggestions get `**Suggestion:**` so they're visually distinct from the user's decisions.
- Keep it tight. 5–8 steps they will execute beats 20 they won't.
- No timelines, risk registers, or ceremony unless they asked.

If their response reveals new significant gaps, you may ask one more round of targeted questions before writing the plan. Use judgment — don't loop indefinitely.

---

## What this skill never does

- Never produce a plan before Steps 2–4 are complete.
- Never ask for information the user already gave you.
- Never recommend something without checking the vault (or noting that you couldn't).
- Never override the user's stated direction — surface tradeoffs, then let them choose.
- Never write the plan to a file unless the user asks.