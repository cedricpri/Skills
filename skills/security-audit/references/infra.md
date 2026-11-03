# Self-hosted server, Docker, nginx and Cloudflare checklist

## Exposure
- List what is actually reachable from the internet and from the local network: published Docker ports, nginx server blocks, Cloudflare tunnel ingress rules, router port forwards.
- Docker `ports: "3306:3306"` binds to all interfaces (`0.0.0.0`). Bind internal services to `127.0.0.1:` or don't publish them, and use the Docker network instead.
- Docker's published ports bypass UFW / firewalld rules on the host (Docker writes its own iptables rules). Don't rely on the host firewall for published ports.
- Databases, Redis, phpMyAdmin, Adminer, Mailpit, Horizon, Telescope, Portainer, dashboards and dev tools not reachable publicly. Dev compose files never used in production.
- phpMyAdmin with `auth_type = config` (auto-login) gives anyone who reaches it full database access.

## Cloudflare tunnel and proxy
- Tunnel ingress rules: only the intended hostnames and services; the catch-all rule returns 404 (`http_status:404`) rather than routing to a service.
- Services behind Cloudflare Access: the origin validates the Access JWT (`Cf-Access-Jwt-Assertion` header) or is only reachable through the tunnel, so Access can't be bypassed by reaching the origin directly.
- Origin not reachable directly by IP if everything is supposed to go through Cloudflare.
- SSL mode Full (strict) when the origin has a certificate; no Flexible SSL for authenticated apps.
- Real client IP restored at the origin (trusted proxies, `CF-Connecting-IP`) so rate limiting and logs work.

## nginx
- A default server block that rejects unknown hostnames instead of serving an app (catch-all risk).
- Hidden and sensitive files denied: `.env`, `.git`, `composer.json`, `storage/`, backups (`*.sql`, `*.zip`, `*.bak`).
- PHP executed only for `index.php` (Laravel), not for files in upload directories.
- `autoindex off`; `server_tokens off`; sensible `client_max_body_size` and timeouts.
- TLS: modern protocols only, HSTS when HTTPS is stable.

## Containers
- Secrets in `docker-compose.yml`, Dockerfiles, image layers or committed `.env` files. Use env files ignored by git or Docker secrets.
- Containers running as root without need; `privileged: true`; the Docker socket (`/var/run/docker.sock`) mounted into containers (equivalent to root on the host).
- Images pinned to versions and updated regularly; base images still maintained.
- Volumes exposing more of the host than needed.

## Host
- SSH: key-only auth, no root login, non-default user; fail2ban or equivalent.
- Automatic security updates for the OS; supported OS version.
- Backups exist, are encrypted, stored off the machine, and a restore has been tested.
- Default passwords changed on every service (database root, admin panels, router).

## What to ask for when it isn't in the repo
- `docker ps --format "{{.Names}} {{.Ports}}"` and the compose files used in production
- The cloudflared config (`config.yml` ingress rules, with credentials removed) or the tunnel's public hostnames
- `sudo ss -tlnp` to see listening ports on the host
- The nginx site configs
