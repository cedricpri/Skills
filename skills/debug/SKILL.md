---
name: debug
description: Thorough root-cause debugging for errors, stack traces, failing tests, wrong outputs, UI glitches, bad data, production-only failures and slow pages. Reproduce, trace, prove the cause with evidence, fix it at the root, lock it with a regression test and sweep for the same bug elsewhere. Use whenever something doesn't behave as expected, including when I paste an error or a log.
argument-hint: [error, symptom or failing test]
---

# Debug: $ARGUMENTS

Find the real cause before touching code, prove it with evidence, and fix it so it can't come back. No shotgun changes, no "this should fix it".

Read the reference for the stack involved before step 3:
- Laravel / PHP: [references/laravel.md](references/laravel.md)
- Vue / frontend: [references/vue.md](references/vue.md)
- Python / AWS / data: [references/python.md](references/python.md)
- Works locally but fails in production, after a deploy, or only sometimes: [references/production.md](references/production.md)
- Slow page, query, job or script: [Performance](#performance) below

## 1. Collect the evidence

- Read the whole error: exception class, message, and the full stack trace. Find the **first frame in application code** (not vendor or framework code). That is where to start reading, not necessarily where the cause is.
- Write down the exact symptom: input, expected result, actual result, where it happens (local, production, which browser or device, which user or record).
- Find out when it started and what changed since: `git log --since`, recent merges and deploys, dependency updates (`composer.lock`, `package-lock.json`, `requirements*.txt` diffs), config or environment changes, new data.
- Look at the logs around the time of the failure: `storage/logs/laravel.log`, the browser console and network tab, queue worker output, CloudWatch for Lambda.
- If evidence is missing and I have it (production logs, the exact record, the browser), ask for it with the **exact command or click path** to get it. Ask for everything you need in one message.

## 2. Reproduce

- Get the bug to fail on demand, in the fastest loop possible, in this order of preference: a failing automated test, a script or tinker/REPL snippet, a single command, a browser script (Playwright).
- Use the real failing input: the same record, payload, file or request. A reproduction with made-up data that "should" be equivalent proves nothing.
- If it only fails in production or intermittently, follow [references/production.md](references/production.md) before concluding it can't be reproduced.
- If you truly can't reproduce it, say so. Keep going only with hypotheses you can test against the evidence you have, and label every conclusion as unconfirmed.

## 3. Trace and narrow down

- Follow the data from the entry point (route, command, job, event handler, component) to the failure. Read every function on that path in full, including the code that produced the bad value, not just the line that crashed on it.
- At each step, check the actual value against what it should be (logging, `dd()`, a breakpoint, a test assertion). Find the first place where reality diverges from expectations.
- Shrink the reproduction: smallest input, fewest steps, narrowest code path. For data bugs, the smallest set of rows that still triggers it.
- If it used to work, find the breaking commit with `git bisect` and the reproduction as the test.
- Read `git log -p` and `git blame` on the lines involved: the reason for the current code is often in the commit that introduced it.

## 4. Hypothesize and prove

- List every plausible cause, not just the obvious one, ranked by likelihood. Always consider: bad or unexpected data, environment and configuration, caches and stale builds, timezones and dates, ordering and concurrency, a dependency or browser behaving differently than assumed.
- For each, write the observation that would confirm or rule it out, then make that observation. Change one thing at a time.
- When a hypothesis depends on how a library, framework or API behaves, check its docs, source or changelog for the installed version. Don't rely on memory.
- Keep asking "why" until you reach a cause that, once fixed, makes the bug impossible, not just this occurrence. "The value was null" is a symptom; "the import skips airports without an ICAO code, so the relation is null" is a cause.
- State the root cause with its chain of evidence before writing the fix. If there are several causes, deal with each one.

## 5. Fix the root cause

- Fix the cause, at the layer where it lives. No silent `try/except` or `?? ""`, no special case for the one bad input, no retry that hides a bug, unless that really is the correct behaviour (and say why).
- Check every caller and every other path that uses the changed code, so the fix doesn't break them.
- If bad data was already written (wrong rows, broken files, wrong cached values), say how much is affected, with the query that counts it, and propose a repair (migration, artisan command, script). Don't touch real data without my approval.
- Keep the fix minimal and in the style of the surrounding code.

## 6. Prove the fix

- Turn the reproduction into a regression test at the right level (unit for logic, feature/HTTP test for endpoints, component or E2E test for UI).
- Show that it fails without the fix and passes with it: stash or revert the fix, run the test, restore the fix, run it again.
- Re-run the original reproduction with the real input.
- Remove every temporary log, `dd()`, `console.log`, print and debug flag.
- Run the full verification from the global instructions.

## 7. Sweep

- Search the codebase for the same pattern (same function misuse, same missing null check, same wrong assumption) and list every other occurrence with `file:line`. Fix them only if I agree.
- Note anything else suspicious you saw on the way, separately from the bug.

## Performance

Same discipline, with numbers instead of stack traces:
1. Measure first, on realistic data volume: response time, query count and time (`DB::enableQueryLog()`, Debugbar or Telescope if installed), `EXPLAIN` on slow queries, browser Performance and Network tabs, Python `cProfile` or timers.
2. Find the dominant cost. Don't optimize anything that isn't on the measured hot path.
3. Typical culprits: N+1 queries, missing indexes, loading whole tables or all columns, work done per row that could be done once, unpaginated lists, heavy computation on every request that could be cached or precomputed, large JS bundles, rendering thousands of DOM nodes, repeated API calls.
4. Fix, re-measure with the same method, and report before/after numbers.

## Report

1. **Root cause**: one or two sentences, plus the evidence that proves it.
2. **Fix**: what changed and why at that layer.
3. **Proof**: the regression test (fails before, passes after) and the verification results.
4. **Data impact and repair**, if any.
5. **Same pattern elsewhere**: `file:line` list.
6. **Anything left unconfirmed**, and what would confirm it.
