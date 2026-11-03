---
name: security-audit
description: Thorough security audit of a codebase, a folder, a self-hosted server setup (Docker, nginx, Cloudflare tunnel) or dependencies - automated scans, attack surface mapping, stack-specific manual review, and only verified, exploitable findings reported by severity before anything is fixed. Use when I ask for a security review, audit, vulnerability check or whether something is exposed.
argument-hint: [folder | "server" | "deps", default: whole repo]
---

# Security audit: $ARGUMENTS

Find what an attacker could actually do, prove it, and report it. Fix nothing until I pick what to fix.

Read the references that match what is in scope:
- Laravel / PHP: [references/laravel.md](references/laravel.md)
- Vue / frontend: [references/vue.md](references/vue.md)
- Python / AWS / data: [references/python-aws.md](references/python-aws.md)
- Self-hosted server, Docker, nginx, Cloudflare, databases: [references/infra.md](references/infra.md)

## 1. Threat model

In a few lines: what the system does, who uses it (anonymous visitors, registered users, premium users, admins), what is valuable (accounts, personal data, payment status, the server itself), and how it is exposed (public internet, Cloudflare tunnel, internal network, AWS). This decides what matters most.

## 2. Automated scans

Run what fits the stack. If a tool is missing, ask before installing it (as a dev dependency, or run it once with `npx`, `pipx run` or Docker).
- PHP: `composer audit`
- JavaScript: `npm audit --omit=dev` (and without `--omit=dev` for build-time risks, reported separately)
- Python: `pip-audit`, `bandit -r <src>`
- Secrets in files and git history: `gitleaks git` (history) and `gitleaks dir` (working tree) for gitleaks 8.19+; older versions use `gitleaks detect`. Check `gitleaks --help` first.
- Runtime versions: PHP, Node, Python, MySQL, nginx versions against their supported and patched releases. Look up current advisories online, don't rely on memory.

For each dependency finding, check whether the vulnerable code path is actually used before rating it.

## 3. Map the attack surface

List every entry point in scope, and for each: who can reach it, what authentication it requires, what input it takes.
- HTTP routes (`php artisan route:list`), API endpoints, webhooks, OAuth callbacks
- File uploads and imports (CSV, PDF, images)
- Forms without authentication: login, registration, password reset, contact
- Queue jobs, scheduled commands and CLI scripts that process external data
- Lambda handlers, S3 event triggers, Streamlit pages
- Exposed services and ports (see infra reference)

## 4. Manual review

Follow untrusted input from each entry point to every place it is used (query, shell, file path, HTML, redirect, deserialization, outgoing request, LLM prompt). Then go through the reference checklists for the stack. Pay particular attention to:
- **Authorization**: for every endpoint that takes an id, can user A read or change user B's record? Are premium or admin features enforced in the backend?
- **Authentication flows**: password reset, email verification, social login, session lifetime, remember-me, logout.
- **Personal data**: what is collected, where it is stored, logged, exported, sent to third parties, and who can see it.
- **Secrets**: in code, config, git history, Docker files, frontend bundles, CI logs.

## 5. Verify

- For each candidate, trace the full flow and confirm it is reachable by the attacker in the threat model and exploitable with real inputs.
- Where it is safe, prove it locally: a feature test or request that shows the hole (for example, user A fetching user B's record). Never test against production without my explicit approval.
- Things you can't verify from here (server config you can't see, cloud settings) go under "Needs checking" with the exact command or setting for me to look at.
- Drop anything that is theoretical with no realistic path.

## 6. Report

Start with a summary: the number of findings per severity and the one thing to fix first.

For each verified finding:
- **Severity** (Critical / High / Medium / Low), based on impact and how easy it is to exploit in this threat model
- `file:line` (or config file and setting), and the flow from entry point to sink
- **Exploit scenario**: who does what, and what they get, in one or two sentences
- **Fix**: concrete, minimal, in the style of the codebase
- **Proof**: how it was verified

Then: dependency findings with the fixed version and whether the vulnerable code is used, the "Needs checking" list, and what was checked and found fine (one line).

## 7. Fix (only after I choose)

For each finding I pick, in order of severity:
1. Write a test that demonstrates the hole (fails before the fix) where practical.
2. Apply the minimal fix.
3. Run the test and the full verification from the global instructions.
4. Check that the fix doesn't break the legitimate use (authorized users still have access).
Server and config changes: give me the exact change and the command to apply and check it, unless the files are in the repo.
