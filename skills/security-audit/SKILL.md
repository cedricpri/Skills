---
name: security-audit
description: Audit a repo or folder for security issues - dependency and secret scans plus a stack-specific manual review - and report verified findings by severity before fixing anything.
disable-model-invocation: true
argument-hint: [folder, default: whole repo]
---

# Security audit: $ARGUMENTS

## 1. Automated scans

Run what fits the stack (install as a dev dependency or run once if missing, after asking):
- Python: `bandit -r`, `pip-audit`
- Laravel: `composer audit`
- Vue: `npm audit --omit=dev`
- All: `gitleaks detect` (secrets in code and git history)

## 2. Map the attack surface

List the entry points in scope: HTTP routes and controllers, API endpoints, CLI commands, queue jobs, file uploads, webhooks, data loaders reading external files or S3. For each, note who can reach it and what input it takes. Follow untrusted input from each entry point to where it is used.

## 3. Manual review checklist

**Laravel**
- Mass assignment: `$guarded = []`, `$request->all()` passed to `create`/`update`/`fill`.
- Raw SQL with input: `DB::raw`, `whereRaw`, `selectRaw`, `orderBy` with a user-chosen column.
- Missing authorization: controllers without policies, gates or `authorize`, IDOR on route model binding (can user A load user B's record?).
- XSS: `{!! !!}` with user data. CSRF exceptions in middleware.
- Uploads: type and size validation, stored outside `public/`, no user-controlled path.
- Config: `APP_DEBUG=true` in production config, `.env` committed, permissive CORS, missing rate limiting on login and password reset.

**Vue**
- `v-html` with untrusted data. Tokens in `localStorage`. Secrets in `VITE_*` variables (they ship to the browser).

**Python / ML / AWS**
- Deserialization of untrusted files: `pickle`, `joblib.load`, `torch.load` without `weights_only`, `yaml.load` without `SafeLoader`.
- `subprocess` with `shell=True`, `eval`/`exec`, SQL built with f-strings.
- Hardcoded AWS keys or credentials; overly broad IAM policies or S3 bucket policies in IaC.
- Personal and health data: member IDs, names, national numbers, medical or reimbursement data in logs, exceptions, notebook outputs, test fixtures or LLM prompts sent to external services. Data copied to places with weaker access control.

**Self-hosted / Docker** (if present): secrets in `docker-compose.yml` or committed config, containers running as root without need, services exposed on `0.0.0.0` that should be internal.

## 4. Verify

For each candidate, trace the actual data flow and confirm it is reachable and exploitable. Drop anything you can't confirm, or list it separately as "needs a human look" with the reason.

## 5. Report

For each verified finding:
- Severity (Critical / High / Medium / Low) and confidence
- `file:line` and the vulnerable flow from entry point to sink
- A concrete exploit scenario in one or two sentences
- The fix

Add dependency findings with the fixed version. Don't fix anything until the user picks what to fix. Then delegate each finding to the `cedric:implementer` subagent with a brief containing the flow, the fix, and a test that proves the hole is closed where practical. Review each diff and run the check command yourself before moving to the next.
