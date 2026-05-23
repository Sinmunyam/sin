---
name: generate
description: Generate or modify project code after the topic has been grounded in the user's Obsidian vault and understanding has been confirmed. Use when the user asks for feature implementation, refactors, or any code-writing task.
---

# Generate

This skill is responsible for code generation only.

The default behavior is NOT "make code." The default behavior is:

1. Accept the topic only after `ask` has confirmed understanding or a related vault note already exists.
2. Generate code or stop without editing files if the user has not confirmed the assumptions needed for implementation.

---

## Generation Mode

### Pre-generation checklist (state these before writing any code)

1. List the assumptions you are relying on.
2. Confirm which vault notes or prior decisions are supporting this implementation.
3. State the scope: which files will be touched, what will be added or changed.

### Generation rules

- Generate the smallest correct implementation first. Do not over-engineer.
- Match the existing codebase's patterns and conventions.
- Avoid speculative features, unnecessary abstractions, or boilerplate not required by the task.
- Validate the result with tests or a reasoned correctness check where practical.
- Do not silently expand scope. If you identify something adjacent that "should probably also be done," flag it and ask — do not do it.
- If the assumptions needed for implementation are not confirmed, stop and ask a focused question instead of guessing.

---

## Hard rules (never violated)

- Never bypass `ask` when the topic is new or ungrounded.
- Never generate project code unless the topic has been confirmed or already exists in the vault.
- Never treat user agreement ("sounds good," "ok," "yes") as a passed gate question.
- Never expand scope silently.
