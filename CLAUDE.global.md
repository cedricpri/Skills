# Global instructions

## Coding standards
- I mostly write Python, Vue.js and Laravel.
- Always use double quotes.
- Keep code minimal and well written. Don't overengineer simple features: no abstraction, config option or helper until it is needed twice.
- Put all imports at the top of the file.
- When a main method calls helper methods, place the helpers after it, in the order they are called.
- Follow the existing conventions of the file and repo over generic best practice.

## Working rules
- What you implement must actually work and must not break anything. Verify it (see below) before saying it is done.
- If you don't know how a library, API or tool behaves, search the docs online first. Never guess.
- Distinguish what you verified from what you suspect. Never present a guess as a finding.
- Don't commit, push or delete branches unless I ask.
- When something is ambiguous and expensive to get wrong, ask one question before starting.
- I often read on my phone: keep reports short and scannable, with the conclusion first.

## Verifying changes
- Find the checks the repo really uses, in this order: the repo `CLAUDE.md`, CI workflows (`.github/workflows/`), `Makefile`, `composer.json` / `package.json` scripts, `pyproject.toml`.
- Run the tests closest to the change first, then everything CI runs: tests, linters, static analysis, and the frontend build (`npm run build` catches template and import errors tests miss).
- For UI changes, open the page in a browser (Playwright) at mobile and desktop widths and look at it.
- If something can't run here (no database, Docker, Cypress, credentials), say so and give me the exact command to run.
- Say what you ran and the result.
