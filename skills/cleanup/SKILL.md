---
name: cleanup
description: Find and remove dead code, leftovers of removed features, duplication and needless complexity in a folder or the whole codebase without changing behaviour - tool findings verified one by one against every way code can be referenced, proposed in a table, then applied in small verified batches. Use when I ask to clean up, simplify, remove dead or unused code, or remove a feature.
argument-hint: [folder, module, or feature to remove; default: whole repo]
---

# Cleanup: $ARGUMENTS

Less code, same behaviour. Every removal is proven unused, every step is verified, and nothing is applied before I approve it.

## 1. Safety net

- Run the verification from the global instructions. If it is red, stop and report: don't clean on a red baseline.
- Check that the scope is covered by tests. Where it isn't, propose characterization tests at its public seams (endpoints, commands, public functions, pages) that capture current behaviour, and add them after I agree.

## 2. Collect candidates

Run the tools that fit the stack, limited to the scope (ask before installing any):
- Python: `vulture` (start at `--min-confidence 80`), `deptry` for unused and missing dependencies, `ruff check --select F401,F841,ERA001`.
- PHP / Laravel: PHPStan or Larastan if configured; `composer-unused` for dependencies.
- Vue / JS: `knip` for unused files, exports and dependencies; ESLint `no-unused-vars`.

Then read the code yourself. Tools miss most of what matters:
- **Dead code**: unused classes, methods, functions, components, props, emits, routes, controllers, views, jobs, listeners, commands, config keys, `.env` variables, translation keys, CSS classes, images and other assets, database columns and tables no code reads.
- **Leftovers of removed or finished features**: feature flags that are always on or off, settings nobody can change any more, branches for old data formats after the migration ran, compatibility code for versions no longer supported.
- **Duplication**: the same logic in several places (search for similar names and similar code), copy-pasted components, helpers that reimplement framework or library functions.
- **Needless complexity**: interfaces with one implementation, wrappers that only forward, layers that add nothing, options nobody uses, deep nesting that early returns would flatten, overly clever code.
- **Debris**: commented-out code, debug output, `TODO`s for things already done, stale comments that contradict the code.

**Removing a feature** (when `$ARGUMENTS` names one): find every trace of it across the whole stack: routes, controllers, models and casts, migrations (a new migration to drop columns, after I confirm), policies, jobs, events, scheduled commands, config, `.env.example`, Blade and Vue components, store state, API fields, translations in every locale, CSS, tests, factories, seeders, docs. Then decide with me what happens to existing data.

## 3. Verify every candidate

Tools give false positives. Before calling anything unused, search for every way it can be referenced:
- **Laravel**: string references in routes, Blade (`@include`, `<x-component>`, `@livewire`), `config/`, service providers, `$listen` and event discovery, `routes/console.php` and the scheduler, queued job class names stored in the `jobs` / `failed_jobs` tables, policies auto-discovery, `app("...")` and container bindings, `__("key")` and `trans_choice` for translations, migrations, tests, the frontend calling the route by URL.
- **Vue / JS**: template usage, global or auto-registered components (`app.component`, auto-import plugins), router lazy imports, `defineExpose`, dynamic `<component :is>`, string-built class names, translation keys built dynamically, files referenced from `index.html` or the build config.
- **Python**: entry points in `pyproject.toml`, CLI and scheduler configs, `__all__`, pipeline or DAG configs naming functions by string, `getattr` and plugin registries, pytest fixtures and `conftest.py`, notebooks, Lambda and SageMaker handler strings in infrastructure config, Streamlit pages discovered by folder.
- **Everything**: other repos, apps or mobile clients that call the API; external cron jobs; webhooks registered with third parties; database rows referencing the code (class names in polymorphic relations or job payloads).

Classify each one:
- **Certain**: no reference anywhere, including dynamic ones.
- **Likely**: only possible dynamic or external references; say which.
- **Keep**: actually used; say where.

## 4. Propose

Show tables, grouped: dead code, feature leftovers, duplication, simplifications, dependencies.
- For removals: item, `file:line`, classification, evidence, action.
- For simplifications: location, one-line before/after, why it is equivalent.
- Total estimate of lines and files removed.

Wait for me to choose. Default to only **Certain** items unless I say otherwise.

## 5. Apply in small batches

One batch per module or logical group:
1. Apply the changes.
2. Re-search for references to everything removed (imports, strings, translations, tests).
3. Run the verification from the global instructions, plus the build for frontend changes.
4. If anything fails, revert the whole batch and report why, instead of patching around it.

Order: code first, then translations and assets, then database columns (with a migration, after I confirm), and dependencies last (remove, reinstall, verify again).

## 6. Report

Lines removed, files deleted, dependencies dropped, simplifications made, items kept or skipped with the reason, and anything that needs a decision from me (data to drop, external consumers to check). Propose a commit message per logical group; don't commit unless asked.
