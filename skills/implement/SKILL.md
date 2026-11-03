---
name: implement
description: Implement a spec from docs/specs/ task by task, test-first at the agreed seams, running the repo check command after each task.
disable-model-invocation: true
argument-hint: [spec path or task description]
---

# Implement: $ARGUMENTS

## Before writing code

1. Read the spec (or the task description), the repo `CLAUDE.md`, and the code each task touches.
2. If no spec exists and the change is more than a small fix, suggest `/cedric:plan` first.
3. List the test seams you will use (one line each: what it catches, what it misses) and **wait for the user to confirm them**. No test at an unconfirmed seam.
4. If the code you will change has no tests, add characterization tests at those seams first, capturing current behaviour.

## For each task, in order

You orchestrate; the `cedric:implementer` subagent (Sonnet) writes the code.

1. Write a self-contained brief: the task goal, the decisions from the spec that apply to it, the files to touch, the confirmed test seams, what is out of scope, and the check command. The subagent sees nothing of this conversation, so include everything it needs.
2. Delegate it to `cedric:implementer`. Run tasks one after another; run two in parallel only when they touch different files and don't depend on each other.
3. When it returns, verify: read the diff, check it against the brief and the test rules below, and run the check command yourself.
4. If something is wrong or missing, send a corrective brief to the same subagent rather than fixing it yourself, unless it's a one-line fix.
5. If the subagent reports an open question, answer it from the spec, or ask the user.
6. Tick the task in the spec.

For a task of a few lines, do it directly instead of delegating.

## Test rules

- Test behaviour through public interfaces, never private methods or internals.
- Mock only what crosses a system boundary (HTTP, AWS, mail, clock). Prefer real objects and an in-memory or test database.
- Expected values come from a known-good literal or the spec, never recomputed the way the code computes them.
- Test names describe behaviour in the project's vocabulary: `test_export_includes_only_filtered_trips`.

## Code rules

Follow the global and repo `CLAUDE.md` standards. If a library's behaviour is uncertain, look it up before using it.

## When done

1. Run `/cedric:review` on the full diff and fix what it finds.
2. Report: tasks completed, tests added, check result, and anything deliberately left out. Propose a commit message; don't commit unless asked.
