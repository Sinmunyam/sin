---
name: learn
description: The understanding front-door. Two modes. BRIDGE mode gives one vault-grounded analogy and hands back — use when the user is stuck mid-question in another skill, or says "compare X to something I know", "use an analogy", "explain X in terms of Y". LOOP mode runs a from-scratch Socratic teaching loop that never hands over the obvious answer first — use when the user says "teach me X", "I know nothing about X", "help me understand X from scratch", "where do I even start with X", "I keep hearing about X but don't get it". Loop mode uses bridge mode internally when the learner gets stuck, and on genuine understanding routes to `add-new-concepts` to file the concept.
---

# Learn

The single entry point for understanding a topic. It absorbs what used to be two skills: a one-shot analogy bridge and a from-scratch teaching loop. They are not separate doors — they are two modes of the same skill, chosen by intent in Step 1.

- **Bridge mode** — one vault-grounded analogy, then hand back. Read-only. Fast. For when the user is stuck on a term mid-question, or explicitly asks to map an idea to something they already know. This is the old `explain` behavior, preserved exactly.
- **Loop mode** — a Socratic loop that teaches a genuinely new topic from zero. This is the only place in the whole skill chain that teaches from scratch (`add-new-concepts`, `plan`, `generate` all assume understanding). Its discipline: **never give the obvious answer first.** When the learner gets stuck inside the loop, loop mode invokes *bridge mode* as a move, then continues.

The core discipline across both modes: **a definition handed over is a definition forgotten.** Make the user reach for the idea before you confirm it.

## Step 0: Resolve vault path (soft gate)

1. Read `~/.claude/vault-config.json` for `vault_path`.
2. If present, note it. Both modes search it — bridge mode for an analogy anchor, loop mode to connect new learning to what the user already knows and to route filing at the end.
3. If missing:
   - **Bridge mode** needs a vault anchor to function. With no vault, do not invent one and do not switch into general-explainer mode. Tell the user: _"I need an initialized vault to ground an analogy. Run `init-vault` first,"_ or point them at an external resource (see Bridge Step 2).
   - **Loop mode** may still run. Teaching from scratch is exactly the case where the user may not have a vault yet. Skip the vault search and, at the end, tell them that to file what they learned they'll need `init-vault` first.

## Step 1: Pick the mode

Read intent before doing anything else.

**Bridge mode** when:
- Another skill (`add-new-concepts`, `plan`, `generate`) asked a clarifying question and the user is genuinely stuck: "I don't know", "what do you mean", repeated wrong guesses, visible frustration.
- The user says "compare X to something I already know", "use an analogy", "explain X in terms of Y", or manually wants a single bridge.

**Loop mode** when:
- The user says "teach me X", "I know nothing about X", "help me understand X from scratch", "where do I even start with X", "I keep hearing about X but don't get it" — anything signalling they want to *build* understanding, not just get one analogy.

If it's ambiguous, ask one short question: _"Do you want a quick analogy to something you already know, or do you want to work through [X] from scratch?"_ Then proceed. Do not loop on this.

---

# BRIDGE MODE

One explanation per invocation. Read-only. Never writes to the vault. Then hand control back.

## Bridge Step 1: Identify the target

- **Manual / explicit:** the topic is what the user passed. "compare backpropagation to something I know" targets "backpropagation".
- **Mid-question:** the topic is whatever the user is currently stuck answering. Read recent turns and pick the noun phrase the question centers on.

If unclear, ask one short question, then proceed. Do not loop.

## Bridge Step 2: Find an anchor in the vault

Search for notes that could serve as analogy anchors, from any note type — concept, research, project, review. The goal is "something the user has touched and understood enough to write about", not type-purity.

1. Inventory: `find <vault>/concepts <vault>/research <vault>/projects <vault>/reviews -iname "*.md"`.
2. Rank candidates by:
   - **Structural match** is strongest. Both target and anchor share a shape — both processes (`Backpropagation` with `Gradient Descent`), both aggregations (`Random Forest` with `Ensemble Methods`), both lookup-like (`Attention` with `Dictionary Lookup`).
   - **Confidence signal** breaks ties. For concept notes, prefer `confidence: solid` or `fluent`. `learning` is fine when nothing better exists.
   - **Recency** breaks further ties.
3. Verify your top anchor on disk before quoting it: `grep -l "^title: <Anchor Title>" <vault>/concepts <vault>/research <vault>/projects <vault>/reviews -r`. Never reference an anchor you have not verified this turn.

If no anchor can be found, do not invent one and do not switch into general-explainer mode. Route to external resources by topic type:

_"I couldn't find a related note in your vault to anchor an analogy. Try [specific resource] first, then file the new understanding via `add-new-concepts`."_

- Programming language or web API: MDN, or the official language docs.
- Framework or library: that project's official docs.
- CS concept: Wikipedia, a known textbook, or a course like CS50 or MIT OCW.

## Bridge Step 3: Build the explanation

One short explanation, four parts, in order:

1. **Name the anchor:** _"You already know [[Random Forest]]. Here's the parallel."_
2. **Map the concept onto the anchor part-by-part.** One or two concrete correspondences. Specific, not abstract.
3. **Call out where the analogy breaks.** Every analogy is wrong somewhere. Saying so stops the user over-extending it.
4. **End with a check question:** _"Does that line up with how you'd describe [[anchor]]?"_

Keep the whole thing under ~150 words. A fast bridge, not a lecture.

## Bridge Step 4: Hand back

- **Mid-question:** stop after the explanation. The calling skill resumes its question loop with the user's next response. Do not take over its flow.
- **Manual:** stop after the explanation. An analogy is a bridge, not understanding. Filing is a separate, deliberate decision — the user can invoke `add-new-concepts` themselves once they own the idea.
- **Invoked from loop mode:** return to the loop's teaching step where you left off.

---

# LOOP MODE

A Socratic loop that teaches a new topic from the ground up without ever handing over the obvious answer first.

## Loop principles

1. **No answer before the attempt.** Lead with a question, not a definition. Withholding is the method.
2. **Start from what they already believe.** Surface the user's current model first, wrong parts included. You teach against their real starting point.
3. **One idea at a time.** Smallest meaningful steps. Check understanding after each before moving on.
4. **Productive struggle, not frustration.** Let them sit with a question. If genuinely stuck, hint or bridge — never the full answer.
5. **Understanding is shown, not claimed.** "Got it" is not proof. They explain it back, apply it, or predict an outcome.
6. **Teach, then route. Don't file mid-loop.** Filing is a deliberate handoff to `add-new-concepts` at the end.

## Loop Step 1: Pin the target and the destination

- **The topic:** _"What exactly do you want to understand — the whole of [X], or one part of it?"_
- **The destination:** _"When you understand [X], what should you be able to do or explain that you can't now?"_ This sets the exit bar for Loop Step 5 and stops the loop running forever.

One or two questions. Don't interrogate.

## Loop Step 2: Surface the starting model (mandatory)

Ask what the user already thinks is true. This is the heart of the mode — you teach against their real model.

_"Before I say anything: what's your current guess about how [X] works? Even a rough or wrong one is useful — it tells me where to start."_

Then, if a vault exists, search for related notes the user already wrote:

1. `find <vault>/concepts <vault>/research <vault>/projects -iname "*<term>*"`.
2. `grep -ril "<term>" <vault>/concepts` for related titles, aliases, or body mentions.

Note solid neighbors — they become bridge-mode anchors when the learner gets stuck, and the cross-domain links the eventual concept note will need. Never reference a note you have not verified this turn.

## Loop Step 3: Decompose into smallest meaningful steps

Break the topic into an ordered chain of sub-ideas, each a single thing the user can grasp and confirm. Cross out anything they already demonstrated in Loop Step 2. Plan 3 to 6 steps; more than that and the topic was too big — narrow it with the user. Keep the decomposition internal; reveal it one step at a time.

## Loop Step 4: The teaching loop (one step per pass)

For each sub-idea, run this loop. Do not break it. Leading with the definition is the single failure this mode exists to prevent.

1. **Pose, don't tell.** Open with a question, a small concrete scenario, or a prediction. _"If [setup], what do you think happens to [thing]? Why?"_ Never open with "X is defined as…".
2. **Let them attempt.** Wait for a real answer. Silence and thinking are the point.
3. **Respond to the actual answer:**
   - **Right:** confirm briefly, push one level deeper or advance. Don't over-praise.
   - **Partly right:** affirm the correct part by name, probe the gap with another question. Don't patch it for them.
   - **Wrong:** don't say "no" and supply the answer. Ask a question that exposes the contradiction, or give a counterexample to reconcile.
   - **Stuck (genuinely, not thinking):** give the smallest hint that unblocks. If the topic structurally resembles a vault note, **drop into bridge mode** for one analogy, then return here. Never resolve stuckness by handing over the full answer.
4. **Check before advancing.** Make the user show understanding: explain it back in their own words, apply it to a fresh case, or predict an outcome. Restating your words verbatim doesn't count. Advance only when the check passes.

Calibration: three instant correct answers in a row → they know more than assumed; skip ahead or raise difficulty. Stuck on the same step twice → the step is too big; split it.

## Loop Step 5: Confirm real understanding

Run a final check against the destination from Loop Step 1. Pose one synthesis task requiring the whole chain — a new scenario, a "what would break if…", or "explain [X] to someone who's never heard of it."

- **Passes:** they own it. Go to Loop Step 6.
- **Falls short:** name the specific gap and return to that step's loop. Don't paper over it. The destination is the bar, not turn count.

## Loop Step 6: Route to filing (do not file here)

_"You've got this now. Want to file it as a concept in your vault so it's linked to what you already know? I can hand off to `add-new-concepts`."_

- **Yes, vault exists:** invoke `add-new-concepts`. The paraphrase that skill asks for is already in hand — the user's Loop Step 5 synthesis. Pass the Loop Step 2 notes as candidate cross-domain links.
- **Yes, no vault config:** _"To file it you'll need a vault first. Run `init-vault`, then `add-new-concepts`."_
- **No:** stop. Understanding without a note is still a win. The user owns the decision to file.

---

# Hard rules (both modes)

- **Never lead with the answer in loop mode.** Every sub-idea opens with a question, scenario, or prediction.
- **Never skip Loop Step 2.** Teaching a blank slate that doesn't exist wastes the user's real (often wrong) starting model.
- **Never advance on a claim of understanding.** "Got it" / "makes sense" is not proof.
- **Never resolve stuckness with the full answer.** Hints, narrower questions, counterexamples, or a bridge-mode analogy — never the finished idea.
- **Never dump the whole topic at once.** One sub-idea per loop pass.
- **Bridge mode never writes to the vault.** It only reads. All vault writes go through `add-new-concepts`.
- **Never invent vault knowledge.** Every `[[link]]` or anchor you cite must exist. Grep before quoting, this turn.
- **Always flag where an analogy breaks.** An unqualified analogy becomes a misconception.
- **Never take over a calling skill's flow.** Invoked mid-question, give the bridge and hand control back.
- **Never run the loop forever.** The Loop Step 1 destination is the exit bar.
- **Bridge mode with no vault anchor routes to external resources** — it does not become a general explainer.
