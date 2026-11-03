---
name: cleanup
description: Find dead code and simplify a part of the codebase without changing behaviour - tool-driven findings, verified one by one, removed in small checked steps.
disable-model-invocation: true
argument-hint: [folder or module, default: whole repo]
---

# Cleanup: $ARGUMENTS

Behaviour must not change. Every step is small, checked, and reversible.

## 1. Safety net

- Run the repo check command. If it fails, stop and report: don't clean on a red baseline.
- Check that the scope has tests. If it doesn't, propose characterization tests at its public seams and add them (after confirmation) before touching anything.

## 2. Collect findings

Run the repo audit command, or the tools for the stack directly, limited to the scope:
- Python: `vulture`, `deptry`, `ruff check --select F401,F841,ERA001`
- Laravel: Larastan at the repo level, plus a search for unused routes, views, jobs, listeners, config keys
- Vue: `knip` (unused files, exports, dependencies), unused components and props

Also read the code yourself for: duplicated logic, needless abstractions (interfaces with one implementation, wrappers that only forward), deep nesting, dead branches and feature flags, commented-out code.

## 3. Verify every dead-code finding

Tools give false positives. Before calling anything dead, search for every way it can be used:
- **Laravel:** string references in routes, Blade, `config/`, service providers, `$listen` / events, scheduled commands, queued jobs, policies, `app("...")` / container bindings, migrations, tests.
- **Vue:** template usage, globally or auto-registered components, router lazy imports, `defineExpose`, dynamic `<component :is>`.
- **Python:** entry points in `pyproject.toml`, CLI and scheduler configs, `__all__`, pipeline or DAG configs naming functions by string, notebooks, pytest fixtures, `getattr` / plugin registries, SageMaker or Lambda handlers.

Classify each finding: **Certain** (no reference anywhere), **Likely** (only dynamic references possible), **Keep** (actually used).

## 4. Propose

Show a table: item, location, classification, evidence, proposed action. Group simplifications separately with a one-line before/after. Wait for the user to choose what to apply.

## 5. Apply in small steps

Delegate the approved changes to the `cedric:implementer` subagent, one batch per file or module. Each brief lists the exact items to remove or simplify, with your evidence, and states that behaviour must not change.

After each batch:
1. Read the diff and run the check command yourself.
2. If anything fails, revert that batch and report it.

Remove unused dependencies last, then reinstall and run the check again.

## 6. Report

Lines removed, files deleted, dependencies dropped, simplifications made, and items skipped with the reason. Propose a commit message per logical group; don't commit unless asked.
