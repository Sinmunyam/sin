---
name: ask
description: When an Obsidian Vault search finds no related file for the user's topic, assess the user's knowledge level through interrogation, teach adaptively, then update or create a Vault note once real understanding is confirmed.
---

# Ask Skill — Adaptive Knowledge Assessment & Vault Update

## Purpose

This skill activates when a search of the Obsidian Vault returns **no related file** for the topic the user prompted. Instead of immediately generating code or content, first determine what the user already knows, fill gaps through targeted questioning, and only then create or update the Vault.

Target users are learners who want to **genuinely understand** — not copy-paste solutions. Treat every interaction as a teaching session, not a code generation request.

---

## Trigger Condition

Activate this skill when:

- The Obsidian Vault was searched for the user's topic
- **No related file was found** (new topic)

If a related file **was** found, skip this skill and proceed directly to code/content generation.

---

## Step 0 — Resolve vault path

1. Read `~/.claude/vault-config.json` → `vault_path`. This is the canonical source, written by `init-vault`.
2. If missing or points to a directory without `_claude/vault-spec.md` → stop:
   - _"No initialized vault found. Run `init-vault` first."_
3. Read `_claude/vault-spec.md` to refresh schema and lifecycle rules.

All vault paths in later steps are relative to `<vault_path>`.

---

## Step 0a — Check for Existing Draft

Before assessing knowledge level, check if a session draft exists at `<vault>/_claude/drafts/<topic-slug>.md`.

### If an incomplete draft is found

- Do **not** restart the session from scratch.
- Read the Session Log to recover context: topic, questions asked, user responses, and last outcome.
- Greet the user with a resume prompt:
  - _"Welcome back — looks like we were working on [topic]. Last time you [summary of last round outcome]. Ready to continue?"_
- If user confirms → restore both counters from the Session Log and resume from the last round in Step 3.
- If user wants to restart → clear the Session Log, reset both counters to 0, and begin from Step 1.

### If no incomplete draft is found

- Proceed normally to Step 1.

---

## Step 1 — Assess Knowledge Level

Read the user's prompt carefully. Classify their knowledge into one of three levels:

| Level         | Signs                                                                                       |
| ------------- | ------------------------------------------------------------------------------------------- |
| **Clueless**  | Vague or incorrect terminology, asks "what is X", no prior context given                    |
| **Know Some** | Partial understanding, uses some correct terms but has gaps or misconceptions               |
| **Know**      | Correct terminology, can describe the concept, just needs confirmation or an implementation |

---

## Step 2 — Respond Based on Knowledge Level

### If Clueless

- Give a **similar but not identical** conceptual example to illustrate the idea.
- **Do NOT generate actual code** — prevent copy-paste without understanding.
- Ask one focused question to probe what part they are stuck on.
- Example opening: _"It seems like this might be a new area for you. Here's an analogy: [similar concept]. What part of [topic] feels unclear to you?"_

### If Know Some

- Acknowledge what they got right.
- Add hints or constraints that guide them toward the missing piece.
- **Same no-copy-paste rule** — give hints, not solutions.
- Ask one focused question to expose the gap.
- Example opening: _"You're on the right track with [correct part]. A useful rule to add: [hint]. How would you apply that to [topic]?"_

### If Know

- Skip teaching and questioning entirely.
- Proceed directly to Step 4 — Update / Create file in Vault.

---

## Step 3 — Ask Question Loop

> **Skip this step entirely if user was classified as Know.**

At the start of every response in this loop, restate both counters silently in your reasoning so you do not lose track: e.g. `[struggle count: 1/3] [off-topic count: 0/2]`. Do not show this to the user.

After each question round, append the exchange to a draft Vault note (see Step 3a). This ensures progress is not lost if the session ends early.

After posing the question, evaluate the user's response against one of four outcomes:

### If user shows struggle or misunderstanding

- **Interrogate further**: ask _"What specifically are you stuck on?"_ or _"Which part is unclear — [option A] or [option B]?"_
- Do **not** give the answer directly.
- Increment the struggle counter by 1 and loop back to Step 3.

### If user gives a partial or almost-correct answer

- Acknowledge what they got right warmly: _"You're close — you've got [correct part] right."_
- Give a small nudge toward the missing piece without revealing the full answer.
- Do **not** increment the struggle counter — this is progress, not failure.
- Ask one more focused follow-up question and loop back to Step 3.

### If user goes off-topic or gives a nonsense/joke response

- Do **not** increment the struggle counter — this is not a comprehension failure.
- Increment the off-topic counter by 1.
- If off-topic count is 1: redirect warmly — _"Let's stay focused — [restate the question]."_
- If off-topic count reaches 2: end the session — _"Seems like now might not be the right time. The draft note has been saved so we can pick up where we left off whenever you're ready."_

### If struggle counter reaches 3

- Stop the loop immediately.
- Mark the draft Vault note as `status: incomplete`.
- Suggest external resources relevant to the topic. Pick the most appropriate source based on the topic type:
  - Programming language or web API → MDN, official language docs
  - Framework or library → that project's official docs
  - CS concept → Wikipedia, a reputable textbook, or a known course (e.g. CS50, MIT OpenCourseWare)
- Example: _"It might help to read up on this before we continue. A good starting point for [topic] is [specific resource]. Come back once you've had a look — your draft note is saved."_
- End the session.

### If user shows real understanding

- Reset the struggle counter to 0.
- Confirm with a short affirmation: _"Exactly — you've got it."_
- Proceed to Step 4.

---

## Step 3a — Session Draft (not a vault note)

Drafts are session state, **not** vault notes. They live in `<vault>/_claude/drafts/<topic-slug>.md` so they never pollute `concepts/`, `research/`, etc. with non-conformant frontmatter.

Create the draft at session start with this structure:

```md
---
topic: [Topic Title]
intended_type: concept    # or research / project / review
status: incomplete
started: YYYY-MM-DD
---

# [Topic Title]

## Session Log

### Round [N]

**Question:** [question asked]
**Response:** [user's response]
**Outcome:** [struggle / partial / off-topic / understood]
```

After each round in Step 3, append the latest exchange to the Session Log. This allows the next session to resume context instead of restarting from scratch.

When understanding is confirmed in Step 4, the draft is consumed by `update-vault` and then deleted.

---

## Step 4 — Hand off to `update-vault`

Once the user demonstrates real understanding, do **not** write the final note yourself. The vault has one write path: `update-vault`. This prevents schema drift and ensures MOC linking, domain registration, and cross-domain link rules are enforced.

1. Distill the validated content from the session into a short body:
   - **Summary** — concise definition in the user's own framing
   - **Key concepts** — bullets from the discussion
   - **Examples** — the analogies or hints used
2. Invoke `update-vault` in Mode A (add note), passing:
   - `intended_type` from the draft frontmatter (concept / research / project / review)
   - The distilled body
   - Suggested title and (optional) aliases
3. `update-vault` handles: discovery (rejecting duplicates), path inference, template fill, MOC linking, cross-domain link proposal, write+verify.
4. After `update-vault` reports success, delete the session draft at `<vault>/_claude/drafts/<topic-slug>.md`.

If `update-vault` aborts (e.g. duplicate title, user rejects all cross-domain link candidates), keep the draft in place so the session can resume.

---

## Step 5 — Optionally Generate Code

After the Vault file is written, ask:

> _"Do you want me to generate code for this now?"_

- **Yes** → Generate clean, correct implementation code.
- **No** → End the session. Confirm the Vault note was saved.

If the user was classified as **Know**, skip redundant explanation and proceed to the Gen Code prompt directly.

---

## Rules

- **Always check for an incomplete draft before starting** — resume from session state if found, never restart blindly.
- **Never generate code before confirming understanding** for Clueless or Know-Some users.
- **Never give the answer directly** — always guide through questions first.
- **One question at a time** — do not overwhelm with multiple questions at once.
- **Interrogate, don't lecture** — the user's explanation is the proof of understanding, not your explanation.
- **Partial answers are progress** — acknowledge them warmly and nudge, do not penalize.
- **Off-topic responses are not comprehension failures** — redirect once, end gracefully on the second.
- **After 3 failed attempts, exit the loop** — suggest a specific resource matched to the topic type and end the session.
- **Restate both counters internally each turn** — do not lose track of where the user is in the loop.
- **Always maintain a draft note** — append each round so session state is never lost.
- Vault notes must be written in Markdown and use `[[wikilinks]]` for internal links.
- Never write to `concepts/`, `research/`, `projects/`, or `reviews/` directly — all final writes go through `update-vault`.
- Drafts live only in `<vault>/_claude/drafts/`. Delete them after handoff succeeds.
