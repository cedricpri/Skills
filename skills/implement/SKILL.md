---
name: implement
description: Implement a spec from docs/specs/ (or a described change) task by task, test-first, verifying after each task and reviewing the full diff at the end.
disable-model-invocation: true
argument-hint: [spec path or task description]
---

# Implement: $ARGUMENTS

Working, verified code that fits the codebase. Nothing is "done" until it has been run.

## Before writing code

1. Read the spec (or the task description), the repo `CLAUDE.md`, and all the code each task touches, plus the closest existing feature that does something similar. Follow its patterns.
2. If there is no spec and the change is more than a small fix, suggest `/cedric:plan` first.
3. Run the verification from the global instructions to get a baseline. If it is already red, tell me before starting, so new failures can be told apart from old ones.
4. List the tests you will write (one line each: the seam, the cases) and wait for me to confirm.
5. If the code you will change has no tests, add characterization tests at those seams first, capturing the current behaviour.
6. When a library or API is involved, check its docs for the installed version before using it.

## For each task, in order

1. Write the first failing test for the task. Run it and see it fail for the right reason.
2. Write the minimum code to make it pass, following the coding standards. Run it.
3. Repeat for the remaining cases of the task (edge cases, errors, permissions).
4. Complete the slice: migration, model casts and fillable, validation, authorization, UI states (loading, empty, error), translations in every locale, as the task needs.
5. Run the verification from the global instructions. For UI work, open the page in a browser at mobile and desktop widths and check it.
6. Re-read your diff for the task: debug code left behind, unused imports, duplication, anything beyond the task.
7. Tick the task in the spec.

If a task turns out to need a decision the spec doesn't cover, stop and ask instead of choosing silently.

## Test rules

- Test behaviour through public interfaces (endpoints, commands, public functions, components), never private methods or internals.
- Mock only what crosses a system boundary (HTTP, AWS, mail, the clock). Use real objects and a test database.
- Expected values come from a literal or the spec, never recomputed the way the code computes them.
- Cover the unhappy paths: invalid input, unauthorized user, missing related data, empty lists.
- Test names describe behaviour in the project's vocabulary: `test_export_includes_only_filtered_trips`.

## When done

1. Run `/cedric:review` on the full diff and fix what it finds.
2. Run the full verification one last time.
3. Report: tasks completed, tests added, verification results (and what couldn't run here, with the command for me), migrations to run, anything deliberately left out. Propose a commit message; don't commit unless asked.
