# Configuration Repository (config-repo)

Central externalized configuration source consumed by the Spring Cloud Config Server.
Can be:
- A Git repository (recommended for team / CI usage)
- A git submodule checked out locally (current template setup)
- A plain local directory ("native" mode) for rapid iteration / air‑gapped demos

## Directory Structure
```
config-repo/
  application.yml                # Global defaults (all services / profiles)
  application-<profile>.yml      # Profile-wide overrides (e.g. dev, prod, local)
  <service-name>/application.yml # Service-specific overrides
  <service-name>/application-<profile>.yml (optional)
```
Profiles merge in this order (last wins):
1. application.yml
2. service base file
3. application-<profile>.yml
4. service application-<profile>.yml
5. Vault secrets interpolation (if enabled)

## Usage Modes
| Mode | When | Setup |
|------|------|-------|
| Native (filesystem) | Fast local dev / workshops / offline | Run `make local` or `make start PROFILE=dev NATIVE=yes` |
| Git remote | Shared team config, audit/history | Set env vars then `make start PROFILE=dev` |

## Switching Between Native and Git
You can select native vs Git when starting (interactive prompt for dev/prod) or force it:
```
# Force native (filesystem)
make start PROFILE=dev NATIVE=yes

# Force Git
make start PROFILE=dev NATIVE=no

# Via start.sh
a./start.sh dev --native
b./start.sh dev --no-native
```

## Environment Variables (Git Mode)
| Variable | Purpose | Example |
|----------|---------|---------|
| CONFIG_REPO_URI | Git URL (HTTPS or SSH) | https://github.com/org/micro-config.git |
| GIT_USERNAME | Optional username (HTTPS private repos) | myuser |
| GIT_TOKEN | Personal access token / password | ghp_xxx |

If using SSH + deploy key, typical Java system properties or global Git config handle key usage; no username/token needed.

## Making It a Separate Repository
1. Create a new bare repo (e.g. micro-config).
2. Move (or copy) existing contents of `config-repo/` there.
3. Update submodule reference (optional):
```
git submodule add <git-url> config-repo
```
4. Commit and push.
5. Export CONFIG_REPO_URI to point at remote.

## Adding/Changing Properties
1. Edit the appropriate file (avoid duplicating defaults—prefer minimal overrides).
2. Commit & push (Git mode) or just save (native mode).
3. Trigger refresh (if using Spring Cloud Bus) or restart affected service; otherwise config is applied at bootstrap only.

## Vault Integration
When Vault is enabled (dev/prod profiles default), placeholders like `${vault.secret.microservices.jwt.secret}` are resolved automatically by Spring Cloud and not stored here in plain text.

## Recommendations
- Keep sensitive values only in Vault; never commit real secrets.
- Prefer small, focused service overrides; share common settings in root application.yml.
- Keep profile files lean—only what differs (e.g., URLs, feature flags, log levels).

## Troubleshooting
| Issue | Cause | Action |
|-------|-------|--------|
| 404 from config server for service | Name mismatch | Match `spring.application.name` in service |
| Stale values after edit | Service not restarted / no refresh | Restart or add bus refresh flow |
| Git auth failures | Missing/invalid credentials | Set CONFIG_REPO_URI/GIT_USERNAME/GIT_TOKEN |
| Vault placeholders unresolved | Vault not running / init skipped | Run `make vault-init` |

## See Also
- config-server/README.md (backend details & endpoints)
- Root README (profiles & startup flows)

