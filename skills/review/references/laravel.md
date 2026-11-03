# Laravel / PHP review checklist

## Controllers and requests
- Validation through a Form Request or `$request->validate()`, and only `validated()` data used for writes (never `$request->all()` into `create` / `update` / `fill`).
- Authorization on every action that reads or changes a record (policy, `authorize`, `can` middleware, or scoping the query to the current user). Route model binding alone doesn't check ownership.
- Correct HTTP status codes and consistent response shapes; API Resources used where the codebase uses them.
- Redirects back with errors and old input for web forms.

## Eloquent and queries
- N+1: relations used in loops, Blade or Resources without `with()` / `load()`; `withCount` instead of loading a relation just to count it.
- `orWhere` grouped in a closure so it doesn't escape other conditions or scopes.
- `first()` and `get()` with an explicit `orderBy` when the order matters; pagination or `chunk` / `lazy` for large sets.
- `whereIn` with a possibly empty array; `find()` returning null handled.
- Queries in accessors, `$appends` or Blade loops.
- Mass updates (`Model::where()->update()`) skip model events, observers and mutators: is that intended?
- New columns used in `where`, `orderBy` or joins have an index when the table is big.

## Migrations
- Safe on existing data: new columns nullable or with a default, backfill when needed, no data loss on rename or drop.
- Foreign keys with the right `onDelete` behaviour; unique constraints for things that must be unique.
- `down()` correct if the repo writes them.
- Model updated together with the migration: `$fillable`, `$casts`, relations, factories, seeders.

## Dates and timezones
- Stored in UTC, displayed in the right timezone; comparisons in a consistent timezone.
- `Carbon` mutation (`addDay()` changes the instance): `copy()` or `CarbonImmutable` where the original is reused.
- Durations and ordering across midnight and across timezones (departure vs arrival).

## Jobs, events, scheduler
- Jobs idempotent (safe to retry), with sensible `tries`, `backoff` and `timeout`; dispatched `afterCommit` when they depend on data written in a transaction.
- Serialized models in jobs that may be deleted before the job runs.
- Scheduled commands: `withoutOverlapping`, `onOneServer` if several servers, timezone.

## Config and environment
- `env()` only inside `config/` files (it returns null once config is cached).
- New config keys and `.env` variables documented in `.env.example`.
- Feature flags and premium/upgrade checks applied consistently in the backend, not only hidden in the UI.

## Blade and output
- `{{ }}` for user data; `{!! !!}` only for trusted, sanitized HTML.
- Translations: every new string through `__()` / `trans()` and present in every locale in `lang/`.

## Tests
- Feature tests for new endpoints, including the unauthorized and invalid-input cases.
- Factories creating realistic data, including the nullable and edge-case states production has.
- No reliance on the current date without freezing time.
