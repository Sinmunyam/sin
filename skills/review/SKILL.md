---
name: review
description: Recall audit. Re-proves that a concept the user already filed is still understood, and updates `confidence:` based on how the recall actually goes. The one skill that catches decayed or hollow understanding before `plan` or `generate` build on it. Hides the note, makes the user explain it cold, then reveals and compares. Trigger on phrasings like "quiz me on X", "do I still understand X", "review my concept on X", "test my recall", "what's due for review", "check if I still know X", "I want to make sure this stuck". Never teaches the topic — if recall fails badly, routes to `/learn`.
---

# Review

The mirror image of `learn`. `learn` builds understanding the user doesn't yet have; `review` audits understanding the user *claims* to have by having filed a concept. Filed once is not understood forever. A note marked `confidence: solid` six months ago may be hollow now, and `plan`/`generate` will happily build on it anyway. This skill is the only thing in the chain that re-tests recorded understanding and writes the result back to `confidence:`.

Hard discipline: **hide the note before asking.** Recall with the answer visible is recognition, not recall, and recognition is exactly the illusion of understanding this project exists to puncture.

## Step 0: Resolve vault path

1. Read `~/.claude/vault-config.json` for `vault_path`.
2. If missing, stop: _"No initialized vault found. Run `init-vault` first."_ There is nothing to review without filed concepts.

## Step 1: Pick the concept

- **Named:** the user passed a topic. Find it: `grep -ril "^title:.*<term>" <vault>/concepts` and alias match. If no concept exists, stop: _"I don't see a concept on [X] in your vault. If you want to learn it, run `/learn [X]`; if you understand it but haven't filed it, run `add-new-concepts`."_ Do not review a note that isn't there.
- **"Due" / unspecified:** the user said "what's due" or just "quiz me." Select a candidate by, in order: lowest `confidence:` (`learning` before `solid` before `fluent`), then stalest (oldest `updated:` if present, else file mtime). Propose one: _"Let's review [[X]] — it's marked `confidence: learning` and you filed it a while back. Ready?"_

Review one concept per invocation. Depth over breadth.

## Step 2: Hide the note and prompt cold recall

Do **not** show the note's body. Read it yourself (you need it for Step 3) but keep it hidden from the user. Then prompt:

_"Without looking at your note — explain [[X]] to me as if teaching it. What is it, why does it matter, and where would you use it?"_

If the concept has known cross-domain links, you may add one targeted probe: _"And how does it connect to [[related]]?"_ — but only after the open recall, never as a hint before it.

Wait for a real attempt. Silence is the user thinking. Do not fill it.

## Step 3: Reveal and compare

Now show the relevant parts of the note and compare honestly against what the user said:

1. **What they nailed:** name the points they recalled accurately.
2. **What they missed or distorted:** name the gaps specifically — a missing mechanism, a wrong cause, a forgotten link. Be precise, not gentle-to-the-point-of-useless. The whole value is an honest signal.
3. **One follow-up probe** on the biggest gap, giving them a chance to recover it themselves before you explain. Don't immediately teach the gap — that's `/learn`'s job.

## Step 4: Update confidence (the write)

Based on the recall — not on the user's self-assessment — propose a `confidence:` change and explain the basis:

- **Fluent:** recalled cold, accurately, including the *why* and the cross-links; could teach it. 
- **Solid:** got the core right, minor gaps, no fundamental errors.
- **Learning:** major gaps, wrong mechanism, or could only recognize once shown.

State the proposal and reasoning: _"You explained the mechanism but missed why it reduces variance, and couldn't place the [[Bagging]] link. That reads like `solid`, not `fluent`. I'll downgrade `confidence:` to `solid` — agree?"_

Rules for the write:
- **Confidence can go down.** A failed recall that lowers `confidence:` is the feature, not a failure. Downgrade without apology when the recall earns it.
- Get the user's nod before writing. `confidence:` is user-facing truth about themselves; don't change it unilaterally.
- Edit only the `confidence:` field. Per the spec, a `confidence:` change does **not** bump `updated:` (that field tracks edits to the note's content, not the user's recall).
- Never touch `status:` here — that's `transfer`'s axis. Keep the two axes separate.

## Step 5: Route on the gap (do not teach here)

- **Recall was strong (→ solid/fluent):** done. _"Solid. Come back to this one less often."_
- **Recall failed badly (→ learning, or wrong mechanism):** do not run a teaching loop here. Route: _"The mechanism slipped. Want to rebuild it properly? Run `/learn [X]` — and when it's back, `/review` it again to confirm."_
- **Optional metacognition:** if the user wants to capture *why* it slipped (a recurring confusion), suggest a `review`-type note via the normal note path. Don't auto-create it.

## Hard rules

- **Never show the note before the user attempts recall.** Recognition with the answer visible defeats the entire purpose.
- **Never grade on the user's self-report.** "Yeah I know this" is not recall. Grade on what they actually produced in Step 2.
- **Never let `confidence:` only go up.** Downgrading on a failed recall is the core value. Apply it honestly.
- **Never review a concept that doesn't exist.** Grep first; route to `learn` or `add-new-concepts` if absent.
- **Never bump `updated:` on a confidence change**, and never touch `status:`. The two axes stay separate (spec rule).
- **Never teach the topic in this skill.** Audit and route. Teaching is `/learn`.
- **Never review more than one concept per invocation.** Depth over breadth; a recall sweep becomes a recognition sweep.
- **Never write `confidence:` without the user's confirmation.**
