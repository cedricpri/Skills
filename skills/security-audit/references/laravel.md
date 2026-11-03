# Laravel / PHP security checklist

## Authorization and access control
- Every route that loads a record by id checks ownership: policy, `authorize()`, `can` middleware, or a query scoped to `auth()->user()`. Route model binding alone does not.
- Scoped bindings for nested resources (`/users/{user}/trips/{trip}` checks the trip belongs to the user).
- Admin routes behind an admin check in middleware, not only hidden links.
- Premium / paid features enforced in the backend.
- API tokens (Sanctum, Passport) with the right abilities and expiry; tokens revoked on logout or password change.

## Mass assignment
- `$guarded = []` or very wide `$fillable` including `is_admin`, `role`, `user_id`, `plan`, `upgraded_at`, `email_verified_at`.
- `$request->all()` or `$request->input()` passed to `create`, `update`, `fill`, `forceFill`.

## Injection
- Raw SQL with input: `DB::raw`, `whereRaw`, `selectRaw`, `orderByRaw`, `havingRaw`, `DB::statement` without bindings.
- `orderBy` / `groupBy` with a user-chosen column name (column names can't be bound: use an allowlist).
- Shell: `exec`, `shell_exec`, `system`, `passthru`, `proc_open`, `Process` with user input.
- File paths built from input: `Storage::get($request->path)`, `response()->download()`, includes. Check for `../` traversal.
- `unserialize()` on untrusted data.
- Outgoing requests to user-supplied URLs (SSRF): webhooks, image fetch by URL, link previews. Block internal addresses.

## XSS and output
- `{!! !!}` with any user-controlled data; HTML built in PHP and echoed raw.
- User data in `<script>` blocks, inline event handlers, `href` (`javascript:` URLs).
- Markdown or rich text rendered without sanitization.

## Authentication
- Rate limiting on login, registration, password reset, email verification resend, 2FA (`throttle` middleware or `RateLimiter`).
- Password reset tokens expiring and single use; no account enumeration through different messages for unknown emails (judge the trade-off).
- Social login: email from the provider trusted only if verified; account linking can't take over an existing account.
- Session regenerated on login (`$request->session()->regenerate()`), invalidated on logout.
- Session and cookie config: `secure`, `http_only`, `same_site`, session lifetime.

## CSRF
- Routes excluded from CSRF in middleware or `bootstrap/app.php`: each exclusion justified (webhooks with signature verification).
- State-changing actions on `GET` routes.

## Uploads
- Validation of type (by content, `mimes` / `mimetypes`) and size.
- Stored outside `public/` or with random names; never executed (no `.php` reachable under the web root).
- Images re-encoded or at least not served with a user-controlled content type; SVG uploads can contain scripts.
- Imported CSV and Excel: formula injection (`=`, `+`, `-`, `@` at the start of a cell) when exported again.

## Configuration and secrets
- `APP_DEBUG=false` in production (debug pages leak env and stack traces).
- `APP_KEY` never committed or shared: anyone with it can forge encrypted cookies and signed URLs.
- `.env` not in git, not reachable over HTTP; no secrets in `config/` defaults.
- CORS (`config/cors.php`): no `*` with credentials; allowed origins explicit.
- Trusted proxies configured correctly (otherwise IP-based rate limiting and HTTPS detection break).
- Security headers: HSTS, `X-Content-Type-Options`, `X-Frame-Options` or CSP `frame-ancestors`, a CSP if practical.
- Signed URLs (`URL::signedRoute`) for sensitive links (unsubscribe, download, email actions), verified with the `signed` middleware.

## Logging and personal data
- Passwords, tokens and personal data not logged (request logging, exception context, Telescope in production).
- Exports and API responses not including more fields than needed (hidden attributes, Resources).
- Telescope, Horizon, Debugbar, phpMyAdmin, log viewers not publicly reachable in production.

## Dependencies and runtime
- `composer audit` findings; abandoned packages.
- PHP version still supported and patched.
