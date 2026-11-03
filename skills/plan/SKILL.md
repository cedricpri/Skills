---
name: plan
description: Interview me about a feature or change until every decision is settled, then write a short spec with ordered vertical-slice tasks to docs/specs/.
disable-model-invocation: true
argument-hint: [what you want to build]
---

# Plan: $ARGUMENTS

Goal: no open decision left before any code is written, and a spec I can read in two minutes.

## 1. Explore first

- Read the repo `CLAUDE.md`, related specs in `docs/specs/`, and every part of the code the change will touch: routes, controllers, models and migrations, jobs, components, stores, translations, tests.
- Find the existing patterns this feature should follow (a similar feature already in the app) and reuse them.
- Find what the change affects beyond the obvious: existing data, other pages showing the same data, exports and imports, API consumers, premium/free tiers, emails and notifications, every locale.
- When a library or API is involved, check its current docs online.
- Answer every question you can from the code yourself. Only ask me what the code can't tell you.

## 2. Interview

- Ask **one question at a time**. Lead with your recommended answer and a one-line reason, so I can accept it in a word.
- Walk the decision tree, skipping what doesn't apply:
  - Scope: what is in, what is explicitly out, the smallest version that is useful.
  - Users and permissions: who can use it, free vs premium, admins.
  - Data: new fields or tables, what happens to existing rows (defaults, backfill), validation rules, deletion.
  - Behaviour: inputs, outputs, every edge case (empty, null, duplicates, timezones, very large volumes).
  - UI: where it lives, mobile layout, loading, empty and error states, wording in every locale.
  - Errors: what the user sees when something fails, what is logged.
  - Performance: expected data volume, pagination, caching, background jobs.
  - Security and personal data: who can see what, what is stored or sent to third parties.
  - Rollout: migration on production data, feature flag if needed, deploy order.
  - Done: how we'll know it works (tests and a manual check).
- When an answer opens new branches, follow them before moving on.
- Push back on scope creep and on solutions more complex than the problem. Propose the simplest version that works.
- Stop when you can describe every task without an open question.

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
  - Test: <public interface where the behaviour is checked, and the key cases>
- [ ] 2. <next slice> (after 1)

## Manual check
- <what to click through in the browser, at mobile and desktop width>
```

Rules for tasks:
- Each task is a **vertical slice** through the stack (migration + model + endpoint + UI + translations, or loader + transform + output), never "all backend, then all frontend".
- Ordered so each one leaves the app working and deployable.
- Each names its test: the endpoint, public function, command or component where the behaviour is verified, and the important cases.

## 4. Hand off

Show the spec path and the task list. Suggest `/cedric:implement <spec path>` as the next step.
