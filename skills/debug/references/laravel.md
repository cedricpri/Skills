# Laravel / PHP bug patterns

Check the ones that match the symptom. Confirm against the installed Laravel and PHP versions (`composer show laravel/framework`, `php -v`).

## Errors and what they usually hide
- **`Undefined array key` / `Undefined index`**: an array shape assumption that is false for some data (optional field, empty API response, a lookup map missing a key). Find which input produces the missing key, don't just add `?? null`.
- **`Attempt to read property "x" on null`**: a relation or `first()` that returns null for some records (deleted parent, missing foreign key, record filtered by a global scope or soft delete). Count how many records are in that state.
- **`Call to a member function on null`** in Blade or a resource: same as above, often a relation that is optional in the data but required in the code.
- **`ModelNotFoundException` / 404**: route model binding scoped by a global scope, soft deletes, or the wrong key (`getRouteKeyName`).
- **`MassAssignmentException` or silently missing fields**: attribute not in `$fillable`. Laravel silently drops non-fillable attributes unless `Model::preventSilentlyDiscardingAttributes()` is on.
- **`419 Page Expired`**: CSRF token or session problem (session domain, cookie settings, cached page holding an old token).
- **`Class not found` after a deploy**: stale `composer dump-autoload`, case-sensitive filename on Linux vs macOS/Windows.

## Silent wrong results
- **Dates and timezones**: `app.timezone` vs the database timezone vs the user's timezone; dates stored in UTC and compared to local dates; `whereDate` on a UTC column; `Carbon` is mutable, so `$date->addDay()` changes the original (use `copy()` or `CarbonImmutable`); day/month order when parsing strings; DST edges.
- **Flights, stays and trips across midnight or timezones**: duration and ordering computed in local time of different airports. Check both departure and arrival timezones.
- **Eloquent casts**: decimals returned as strings, booleans as `0`/`1`, JSON columns not cast, enum casts failing on unexpected values.
- **Queries**: `orWhere` without grouping in a closure (breaks the other conditions and scopes), `whereIn` with an empty array, `distinct` with `orderBy` on another column, `groupBy` with MySQL `ONLY_FULL_GROUP_BY`, `count()` on a query with `groupBy`, collation and case sensitivity differences between SQLite tests and MySQL production.
- **Collections vs queries**: filtering a collection that was already limited or paginated; `->first()` without `orderBy` is non-deterministic.
- **Relations**: eager loading with constraints that don't apply where you think, `withCount` vs loaded relation count, `hasManyThrough` keys.
- **Caching**: `Cache::remember` keys that don't include every parameter (user, locale, filters), stale cached values after a data change, cache not cleared on update.
- **Accessors and appends**: an accessor that runs a query per model, or `$appends` adding values to every serialized model.

## Side effects and async
- **Queues**: a job that works synchronously but fails on the queue (serialized model deleted or changed before the job runs, closure state, `afterCommit` missing so the job runs before the transaction commits).
- **Events and observers**: observers not firing on mass updates (`Model::where()->update()` skips them), events fired inside transactions that later roll back.
- **Transactions**: partial writes when an exception happens outside `DB::transaction`, nested transactions with savepoints.
- **Scheduler**: commands that overlap (`withoutOverlapping`), run in the server timezone, or run on several servers.

## Tests
- Tests passing on SQLite but failing on MySQL (or the reverse): JSON functions, string comparison, date functions, foreign key enforcement.
- `RefreshDatabase` vs `DatabaseTransactions` behaviour, factories producing data that production never has (or never producing what production has).
- Time-dependent tests: freeze time with `$this->travelTo()` / `Carbon::setTestNow()`.

## Tools
- `php artisan tinker` to reproduce with real models.
- `DB::enableQueryLog()` / `DB::getQueryLog()`, or `->toRawSql()` (Laravel 10.15+) to see the actual SQL.
- `php artisan route:list`, `php artisan about`, `php artisan config:show <file>` (if `php artisan list` has it) to check what the app really sees.
- `Model::shouldBeStrict()` in local to surface lazy loading, missing attributes and discarded attributes.
