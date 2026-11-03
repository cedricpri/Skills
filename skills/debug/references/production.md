# Production-only, post-deploy and intermittent bugs

When it works locally (or in tests) but not in production, the difference between the two environments is the bug until proven otherwise. List the differences and check them one by one.

## 1. Is production running the code you think it is?
- Compare the deployed commit with the branch you are reading (`git log -1` on the server, a version endpoint, the deploy log). Production running an older commit is common.
- PHP: OPcache with `validate_timestamps=0` keeps serving old code until PHP-FPM is reloaded.
- Laravel caches: config, routes, views, events (`php artisan optimize:clear` clears them; `php artisan optimize` rebuilds). With cached config, `env()` called outside `config/` files returns `null`.
- Queue workers are long-running processes that keep the old code in memory until restarted (`php artisan queue:restart`, or restart the Supervisor or Docker service).
- Frontend: the browser, a CDN or a service worker serving an old bundle; an old tab still open calling a changed API.
- Docker: the container running an old image, a volume masking files from the image.

## 2. Deploy-time races
- Requests hitting the app while files, caches or migrations are half updated (cache deleted between a check and a read, code deployed before the migration ran or after). Look at the timestamp of the error vs the deploy time.
- Maintenance mode not used around migrations; a migration that locks a big table.

## 3. Environment and configuration
- `.env` differences: `APP_ENV`, `APP_DEBUG`, `APP_URL`, cache, session and queue drivers, mail, third-party keys.
- PHP, Node or Python versions and extensions (`php -m`), `memory_limit`, `max_execution_time`, upload limits (`upload_max_filesize`, `post_max_size`, nginx `client_max_body_size`).
- Database: MySQL version and `sql_mode`, collation, timezone, data volume (a query fine on 100 rows times out on 1M).
- File permissions and ownership (`storage/`, `bootstrap/cache/`), disk full, case-sensitive filesystem.
- Reverse proxy: trusted proxies (wrong scheme or IP, redirect loops, mixed content), headers stripped, timeouts at nginx or Cloudflare shorter than the app's.

## 4. Data only production has
- Get the exact record, payload or file that fails. Most "can't reproduce" bugs are a data shape the local seed never has: nulls, empty strings, very long values, unusual characters, old records from before a migration, duplicates.
- Ask me to run a read-only query that shows the shape of the failing data. Give me the exact query.

## 5. Intermittent failures
- Run the failing test or request many times in a loop to measure how often it fails.
- Look for: concurrency (two requests or jobs on the same record), ordering without `ORDER BY`, time-dependent logic (midnight, month end, DST, timezones, tests using "now"), caches expiring, rate limits and timeouts on external services, retries that duplicate work.
- Correlate failures with time, user, record, server or worker in the logs.

## What to ask me for
Everything in one message, with exact commands:
- The full error and stack trace from `storage/logs/laravel.log` (or the relevant log) around the time of the failure.
- The deployed commit, and when the last deploy happened.
- The output of the specific `php artisan`, `docker`, `mysql` or `php -m` commands that would confirm or rule out your top hypotheses.
- Read-only commands only, unless I agree otherwise.
