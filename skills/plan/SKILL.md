---
name: plan
description: Interview the user about a feature or change until every decision is settled, then write a short spec with ordered vertical-slice tasks to docs/specs/.
disable-model-invocation: true
argument-hint: [what you want to build]
---

# Plan: $ARGUMENTS

Goal: no ambiguity left before any code is written, and a spec small enough to read in two minutes.

## 1. Explore first

Read the code the change will touch, the repo `CLAUDE.md`, and any related spec in `docs/specs/`. Answer every question you can from the code yourself; only ask the user what the code can't tell you.

## 2. Interview

- Ask **one question at a time**. Lead with your recommended answer so the user can accept it in a word, and add a one-line reason.
- Walk the decision tree: scope and non-goals, inputs and outputs, data model changes, edge cases and error handling, performance or volume, security and personal data, how we'll know it works.
- When an answer opens new branches, follow them before moving on.
- Push back on scope creep and on solutions more complex than the problem needs. Propose the simplest version that works.
- Stop when you can state every task without an open question.

## 3. Write the spec

Write `docs/specs/<yyyy-mm-dd>-<slug>.md`:

```markdown
# <Feature>

## Goal
<2-3 sentences: the problem and the outcome.>

## Decisions
- <each settled decision, one line, with the reason when it isn't obvious>

## Out of scope
- <explicit non-goals>

## Tasks
- [ ] 1. <vertical slice: a thin end-to-end change that can be tested on its own>
  - Test seam: <public interface where the behaviour is checked>
- [ ] 2. <next slice> (after 1)
```

Rules for tasks:
- Each task is a **vertical slice** through the stack (route + controller + UI, or loader + transform + output), never "all backend, then all frontend".
- Order them so each one leaves the app working.
- Each names the test seam: the public function, endpoint or component where behaviour is verified.

## 4. Hand off

Show the spec path and the task list. Suggest `/cedric:implement <spec path>` as the next step.
