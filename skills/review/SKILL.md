---
name: review
description: In-depth code review of a diff, a branch, a pull request, a folder or the whole codebase - correctness bugs, data integrity, edge cases, breaking changes, security, performance, UI states, i18n, tests and coding standards - with every finding verified before it is reported. Use when I ask for a review, a code review, to find bugs, or to check a change before merging.
argument-hint: [nothing = current changes | ref or branch | PR number | path | "all"]
---

# Review: $ARGUMENTS

Find the problems that would hurt in production, prove each one, and report them clearly. Don't edit code unless I ask.

## 1. Scope

- **Nothing given**: uncommitted changes plus the commits on the current branch that are not on the default branch (`main`, `master` or `development`: check which one the repo uses).
- **Ref or branch**: `git diff <ref>...HEAD` plus uncommitted changes.
- **PR number**: the PR diff, its description and existing review comments (GitHub tools).
- **Path or "all"**: every file in scope, not a diff. Map the structure first (entry points, models, services, components, jobs, tests), then review in order of risk: authentication and permissions, payments and upgrades, anything that writes or deletes data, imports and exports, background jobs, then the rest. Read each file in full.

## 2. Understand the intent

- Find what the change is supposed to do: the spec in `docs/specs/`, the PR description, the task I described, the commit messages.
- For a codebase review, find what the app does and its main flows from the routes, pages and README.
- Write the intent down in one or two sentences. Missing behaviour is a finding too.

## 3. Run the checks

Run the verification from the global instructions. Every failure is a finding. Note what couldn't run here.

## 4. Read the code, then hunt

Read every changed line **and its context**: the whole function, its callers (search for every usage), the code it calls, related tests, the migration next to the model change, the component next to the API change. Then make one pass per axis below, and use the checklist for the stack:
- Laravel / PHP: [references/laravel.md](references/laravel.md)
- Vue / frontend: [references/vue.md](references/vue.md)
- Python: [references/python.md](references/python.md)

**Correctness**
- Does it do what the intent says, completely? Every case of the spec, every branch of the UI.
- Edge cases: null, empty, zero, one, many, duplicates, very long values, unusual characters, first and last items, boundaries (midnight, month end, DST, timezones, leap years).
- Error handling: what happens when the API, database, file or external service fails or returns something unexpected? Errors swallowed? User left with a spinner forever?
- State and concurrency: double submits, two users on the same record, jobs running twice or out of order, caches serving stale data.

**Data integrity**
- Migrations: safe on existing production data (nullable or default for new columns, backfill for existing rows, data loss on dropped or renamed columns, long locks on big tables), and a working `down()` if the repo uses them.
- Writes that should be in a transaction, validation that the database doesn't enforce, orphaned records on delete.

**Breaking changes**
- Changed signatures, routes, request or response shapes, database columns, config keys, events, translation keys, CSV formats. Find every caller and consumer, including the frontend, jobs, commands, tests, and other apps or mobile clients using the API.

**Security**
- User input reaching queries, shell, file paths, HTML or redirects; missing authorization (can user A see or change user B's data?); secrets in code; personal data in logs or responses. For a deep pass, use `/cedric:security-audit`.

**Performance**
- N+1 queries, queries in loops, missing indexes for new queries, loading whole tables, unpaginated lists, heavy work on every request, large payloads, expensive watchers or computed properties.

**UI and UX** (if there is a frontend)
- Loading, empty and error states; mobile layout; disabled states during submit; feedback after actions; accessibility basics (labels, focus, contrast).
- Every new user-facing string translated in every locale the app has.

**Tests**
- Is the new behaviour tested? Would each test fail if the behaviour broke? Tests that only mock and assert the mock are not tests.
- Expected values from literals or the spec, not recomputed like the code does.

**Standards and simplicity**
- The global and repo `CLAUDE.md` rules: double quotes, imports at the top, helpers after the method that calls them in call order, minimal code.
- Overengineering: abstractions, options or layers the task doesn't need. Duplicated logic that already exists elsewhere in the codebase (search for it). Dead code, debug output and commented-out code left behind.
- Names that don't match the codebase's existing vocabulary.

## 5. Verify every finding

- Re-read the code for each candidate and trace the actual flow. Confirm that the problem is reachable with real inputs.
- When it is cheap, prove it: a failing test, a tinker or REPL snippet, a request. Mark proven findings as such.
- Drop anything you can't confirm, or list it under "Questions" with what would settle it. No speculative findings in the main list.

## 6. Report

Start with a one-line verdict (ready to merge / fix first / needs rework).

Then group by severity:
- **Must fix**: bugs, data loss, security holes, broken behaviour, failing checks.
- **Should fix**: likely problems, missing tests for risky code, performance issues, missing UI states or translations.
- **Nit**: style, naming, small simplifications.

For each: `file:line`, the problem in one sentence, the concrete scenario where it breaks, and the fix. Proven findings say how they were proven.

End with **Questions** (unconfirmed items) and what you checked that was fine, in one line. Offer to fix the findings; don't edit before I say so.
