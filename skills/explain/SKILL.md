---
name: explain
description: On-demand vault-grounded analogy helper. Read-only. Trigger when the user is visibly stuck answering a clarifying question from another skill (signals like "I don't know", "what do you mean", repeated wrong guesses), or when they manually invoke /explain on a topic, or when they say things like "compare X to something I already know", "use an analogy", "explain X in terms of Y", "I learn better with examples from what I've seen". Searches the vault for notes the user has already written and builds a bridge from them. Never writes to the vault, never lectures, one explanation per invocation.
---

# Explain

Read-only analogy helper. Explains an unfamiliar topic by analogy to something the user already understands in their vault. Not an entry point and not a teaching loop. One explanation per invocation, then hand control back.

## Trigger

Activate in only two cases:

1. **Stuck mid-question.** Another skill (`add-new-concepts`, `plan`, `generate`) asked a clarifying question and the user's response shows genuine stuckness: "I don't know", "what do you mean", repeated wrong guesses, visible frustration. Invoke `explain` once on the term they're stuck on, then return control to the caller.
2. **Manual invocation.** The user types `/explain <topic>` or says things like "explain X in terms of what I already know" or "compare X to something in my vault".

Do not activate:

- On every vault-miss. A missing note is not a stuck user.
- As a substitute for filing a note. `/explain` never writes.
- To answer a generic "what is X" question. If `~/.claude/vault-config.json` is missing, say so and stop.
- Pre-emptively, before a question has been asked or the user has signaled stuckness.

## Step 0: Resolve vault path

1. Read `~/.claude/vault-config.json` for `vault_path`.
2. If missing or invalid, stop: _"No initialized vault found. I need an initialized vault to ground analogies. Run `init-vault` first."_

## Step 1: Identify the target

- **Manual:** the topic is the argument the user passed. `/explain backpropagation` targets "backpropagation".
- **Mid-question:** the topic is whatever the user is currently stuck answering. Read recent turns and pick the noun phrase the question centers on.

If the topic is unclear, ask one short question: _"What specifically do you want me to explain?"_ Then proceed. Do not loop.

## Step 2: Find an anchor in the vault

Search the vault for notes that could serve as analogy anchors. Pull from any note type: concept, research, project, review. The goal is "something the user has touched and understood enough to write about", not type-purity.

1. Inventory: `find <vault>/concepts <vault>/research <vault>/projects <vault>/reviews -iname "*.md"`.
2. Rank candidates by:
   - **Structural match** is strongest. Both target and anchor share a shape. Both processes (`Backpropagation` with `Gradient Descent`), both aggregations (`Random Forest` with `Ensemble Methods`), both lookup-like (`Attention` with `Dictionary Lookup`).
   - **Confidence signal** breaks ties. For concept notes, prefer `confidence: solid` or `fluent`. `learning` is fine when nothing better exists.
   - **Recency** breaks further ties. Notes the user edited recently are fresher in their head.
3. Verify your top anchor on disk before quoting it. Run `grep -l "^title: <Anchor Title>" <vault>/concepts <vault>/research <vault>/projects <vault>/reviews -r` to confirm the title actually exists. Never reference an anchor you have not just verified this turn.

If no anchor can be found, do not invent one and do not switch into general-explainer mode. Route to external resources instead:

_"I couldn't find a related note in your vault to anchor an analogy. Try [specific external resource for the topic] first, then file the new understanding via `add-new-concepts`."_

Pick the resource by topic type:

- Programming language or web API: MDN, or the official language docs.
- Framework or library: that project's official docs.
- CS concept: Wikipedia, a known textbook, or a course like CS50 or MIT OCW.

## Step 3: Build the explanation

Produce one short explanation that does four things, in order:

1. **Name the anchor:** _"You already know [[Random Forest]]. Here's the parallel."_
2. **Map the concept onto the anchor part-by-part.** One or two concrete correspondences. Be specific, not abstract.
3. **Call out where the analogy breaks.** Every analogy is wrong somewhere. Saying so prevents the user from over-extending it.
4. **End with a check question:** _"Does that line up with how you'd describe [[anchor]]?"_

Keep the whole thing under ~150 words. The point is a fast bridge, not a lecture.

## Step 4: Hand back

- **Mid-question:** stop after the explanation. The caller resumes its question loop with the user's next response.
- **Manual:** stop after the explanation. An analogy is a bridge, not understanding. Filing a note is a separate, deliberate decision. If the user wants to capture the topic, they can invoke `add-new-concepts` themselves once they're confident they own the idea.

## Rules

- Never write to the vault. This skill only reads.
- Never invent vault knowledge. Every `[[link]]` you cite must exist. Grep before quoting.
- One analogy per invocation. If the user is still stuck after the explanation, return control. Do not spiral into a teaching loop.
- Always flag the limit of the analogy. Skipping this turns analogies into misconceptions.
- Do not bypass the calling skill's flow. When invoked mid-question, hand control back, do not take over.
- Do not lecture. If the user asked for an analogy, give an analogy, not a textbook section.
- If no vault anchor exists, do not switch into general-explainer mode. Route to external resources.
