---
name: debug
description: Disciplined debugging loop for bugs, failing tests, wrong outputs and performance regressions - reproduce, minimize, hypothesize, verify, fix the root cause, add a regression test. Use whenever something doesn't behave as expected.
---

# Debug

Never fix a bug you can't reproduce, and never guess a cause you haven't verified.

## 1. Reproduce

Build a fast, reliable way to see the bug fail: a failing test (preferred), a script, or a single command. Write down the exact expected vs actual result. If you can't reproduce it, gather what's missing (logs, input data, environment, versions) and ask; don't fix blind.

## 2. Minimize

Shrink the reproduction: smallest input, fewest steps, narrowest code path. For data bugs, find the smallest set of rows that triggers it. Use `git bisect` if it used to work.

## 3. Hypothesize

List 2-4 possible causes, ranked by likelihood, each with the observation that would confirm or rule it out. If a hypothesis depends on how a library behaves, check its docs or source instead of assuming.

## 4. Verify

Test the top hypothesis with targeted logging, assertions or a debugger. Move to the next one if it's ruled out. Don't change behaviour yet.

## 5. Fix the root cause

Fix the cause, not the symptom: no silent `try/except`, no special case for the one bad input unless that *is* the correct behaviour. Keep the fix minimal.

## 6. Lock it in

- Turn the reproduction into a regression test that fails without the fix and passes with it.
- Remove all temporary logging and debug code.
- Run the repo check command.
- Search for the same pattern elsewhere in the codebase and report other occurrences.

## Report

Root cause in one or two sentences, the fix, the regression test, and any other occurrences found.
