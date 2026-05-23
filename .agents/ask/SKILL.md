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

At the start of every response in this loop, restate the current struggle count silently in your reasoning so you do not lose track: e.g. `[struggle count: 1/3]`. Do not show this to the user.

After posing the question, evaluate the user's response against one of three outcomes:

### If user shows struggle or misunderstanding

- **Interrogate further**: ask _"What specifically are you stuck on?"_ or _"Which part is unclear — [option A] or [option B]?"_
- Do **not** give the answer directly.
- Increment the struggle counter by 1 and loop back to Step 3.

### If user gives a partial or almost-correct answer

- Acknowledge what they got right warmly: _"You're close — you've got [correct part] right."_
- Give a small nudge toward the missing piece without revealing the full answer.
- Do **not** increment the struggle counter — this is progress, not failure.
- Ask one more focused follow-up question and loop back to Step 3.

### If struggle counter reaches 3

- Stop the loop immediately.
- Do **not** write to the Vault.
- Suggest external resources relevant to the topic. Pick the most appropriate source based on the topic type:
  - Programming language or web API → MDN, official language docs
  - Framework or library → that project's official docs
  - CS concept → Wikipedia, a reputable textbook, or a known course (e.g. CS50, MIT OpenCourseWare)
- Example: _"It might help to read up on this before we continue. A good starting point for [topic] is [specific resource]. Come back once you've had a look."_
- End the session.

### If user shows real understanding

- Reset the struggle counter to 0.
- Confirm with a short affirmation: _"Exactly — you've got it."_
- Proceed to Step 4.

---

## Step 4 — Update / Create File in Vault

Once the user demonstrates real understanding:

1. Infer a reasonable Vault path from the topic (e.g. `Programming/JavaScript/closures.md`). Only ask the user if the topic is genuinely ambiguous.
2. **Create** a new `.md` note if no file exists, or **update** an existing one if a related file was found earlier.
3. Structure the note with:
   - **Topic title** as the H1 heading
   - **Summary** — concise definition in the user's own framing
   - **Key concepts** — bullet list of what was discussed
   - **Examples** — the analogies or hints given during the session
   - **Links** — `[[wikilinks]]` to related Vault notes if applicable

---

## Step 5 — Optionally Generate Code

After the Vault file is written, ask:

> _"Do you want me to generate code for this now?"_

- **Yes** → Generate clean, correct implementation code.
- **No** → End the session. Confirm the Vault note was saved.

If the user was classified as **Know**, skip redundant explanation and proceed to the Gen Code prompt directly.

---

## Rules

- **Never generate code before confirming understanding** for Clueless or Know-Some users.
- **Never give the answer directly** — always guide through questions first.
- **One question at a time** — do not overwhelm with multiple questions at once.
- **Interrogate, don't lecture** — the user's explanation is the proof of understanding, not your explanation.
- **Partial answers are progress** — acknowledge them warmly and nudge, do not penalize.
- **After 3 failed attempts, exit the loop** — suggest a specific resource matched to the topic type and end the session without writing to the Vault.
- **Restate struggle count internally each turn** — do not lose track of where the user is in the loop.
- Vault notes must be written in Markdown and use `[[wikilinks]]` for internal links.
