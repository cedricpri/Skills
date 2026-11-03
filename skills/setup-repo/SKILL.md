---
name: setup-repo
description: Prepare the current repo for the other skills - detect the stack, add a single check command, configure linters, dead-code and security tools, and add a repo section to CLAUDE.md. Run once per repo.
disable-model-invocation: true
---

# Set up this repo

The other skills rely on one **check command** that runs lint, static analysis and tests, and on the tool configuration this skill adds. Explore, propose, confirm, then write.

## 1. Explore

Read what exists; don't assume:
- Stack: `pyproject.toml`, `requirements*.txt`, `composer.json`, `package.json`, `vite.config.*`, notebooks (`*.ipynb`).
- Existing tooling: ruff, mypy, pytest, PHPStan/Larastan, Rector, Pint, PHPUnit/Pest, ESLint, Prettier, knip, vitest, pre-commit, Makefile, composer or npm scripts.
- `CLAUDE.md` at the repo root, and whether it already has a `## Repo tooling` section.
- Test coverage at a glance: does a test suite exist and does it pass?

## 2. Propose

Present what you found and what you would add, per stack. Default tools:

| Purpose | Python | Laravel | Vue |
|---|---|---|---|
| Lint + format | ruff (`quote-style = "double"`) | Pint | ESLint + eslint-plugin-vue, Prettier |
| Static analysis | mypy or pyright (only if the code is already typed) | Larastan | vue-tsc (only if TypeScript) |
| Dead code | vulture, deptry | Larastan | knip |
| Security | bandit, pip-audit | composer audit | npm audit |
| Tests | pytest | PHPUnit or Pest (whichever exists) | vitest (only if tests exist) |

Rules:
- Reuse tools already in the repo; never add a second tool for the same job.
- Start strict tools at a level the codebase passes today (e.g. Larastan level 5, a vulture whitelist or `min_confidence = 80`) so the check is green from day one. Note the stricter target level in the CLAUDE.md section.
- Look up the current install and config syntax of each tool online before writing config.

Then propose one check command, using whatever the repo already uses (`make check`, `composer check`, `npm run check`, or a `scripts/check.sh`). It must fail on any error and run fast enough to use after every change. Dead-code and security tools go in a separate `audit` command, not in `check`.

## 3. Confirm, then write

Show the diff of every file you will create or change and wait for approval. Then:
1. Install dev dependencies and write the config.
2. Run `check` and `audit` once. Fix only configuration problems; report code findings without fixing them.
3. Add or update this section in the repo `CLAUDE.md` (never create `AGENTS.md`):

```markdown
## Repo tooling
- Check (run after every change): `<command>`
- Audit (dead code + security): `<command>`
- Stack: <short summary>
- Specs live in `docs/specs/`.
```

## 4. Report

List what was installed, the two commands, and the current audit findings count per tool, so the user can decide when to run `/cedric:cleanup` or `/cedric:security-audit`.
