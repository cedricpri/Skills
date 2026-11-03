# Global instructions

## Coding standards
- I mostly write Python, Vue.js and Laravel.
- Always use double quotes.
- Keep code minimal and well written. Don't overengineer simple features: no abstraction, config option or helper until it is needed twice.
- Put all imports at the top of the file.
- When a main method calls helper methods, place the helpers after it, in the order they are called.
- Follow the existing conventions of the file and repo over generic best practice.

## Working rules
- What you implement must actually work and must not break anything. After every change, run the repo's check command (see the repo's CLAUDE.md). If there is none, run the relevant linters and tests you can find, and say what you ran.
- If you don't know how a library, API or tool behaves, search the docs online first. Never guess.
- Don't commit, push or delete branches unless I ask.
- When something is ambiguous and expensive to get wrong, ask one question before starting.

## Delegation
- Keep planning, design decisions, debugging analysis and reviews in the main conversation.
- Delegate implementation to the `cedric:implementer` subagent (Sonnet) whenever the change is more than a few lines and the approach is agreed. Give it a self-contained brief: goal, files, test seams, constraints, and the check command.
- After it returns, verify its work yourself: read the diff and run the check command before reporting success.
- Run independent tasks in parallel only when they touch different files.
