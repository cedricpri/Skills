---
name: review
description: Review a diff against the coding standards and the intended behaviour, run the checks, and report findings by severity without editing code.
disable-model-invocation: true
argument-hint: [base ref, default: uncommitted changes, then main]
---

# Review

## 1. Get the diff

- If `$ARGUMENTS` names a ref, review `git diff $ARGUMENTS...HEAD` plus uncommitted changes.
- Otherwise review uncommitted changes; if there are none, review the current branch against `main` (or `master`).
- Find the intent: the related spec in `docs/specs/`, the task the user described, or the commit messages.

## 2. Run the check command

Run the repo check command and include its failures as findings.

## 3. Review on two axes

**Correctness and intent**
- Does the diff do what the spec or task says, fully? Missing cases, wrong edge-case handling?
- Bugs: off-by-one, null/None handling, wrong comparisons, timezone and date issues, unhandled exceptions, race conditions, N+1 queries, mutations of shared state.
- Breaking changes: changed function signatures, routes, DB columns, API responses, config keys. Find every caller.
- Are the tests meaningful (they would fail if the behaviour broke)?

**Standards and simplicity**
- The global and repo `CLAUDE.md` rules: double quotes, imports at the top, helpers after the method that calls them, in call order.
- Overengineering: abstractions, options or layers the task doesn't need. Duplicated logic. Dead code left behind.
- Naming that doesn't match the codebase's existing vocabulary.
- Security: user input reaching queries, shell, file paths or HTML; secrets; personal data in logs.

## 4. Verify before reporting

For each finding, re-read the surrounding code and confirm it is real. Drop anything you can't confirm.

## 5. Report

Group by severity: **Must fix**, **Should fix**, **Nit**. For each: `file:line`, the problem in one sentence, and the suggested fix. End with a one-line verdict. Don't edit code unless the user asks.
