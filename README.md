# Configuration Repository (config-repo)

Central externalized configuration source consumed by the Spring Cloud Config Server.

## 🏗️ **Configuration Architecture - Industry Standard Pattern**

This config repository implements a **hybrid configuration strategy** that separates concerns:

### **1. Infrastructure Bootstrap (.env files)**

- **Purpose**: Container startup and infrastructure initialization
- **Location**: `environments/.env.*` (Docker Compose, Makefiles)
- **Contains**: Docker image versions, bootstrap credentials, network config
- **Used by**: Docker infrastructure before application starts

### **2. Application Configuration (this repository)**

- **Purpose**: Business logic, application behavior, service integration
- **Location**: `config-repo/` (Spring Cloud Config)
- **Contains**: OAuth settings, connection pools, retry policies, service discovery
- **Used by**: Spring applications at runtime

### **3. Runtime Secrets (HashiCorp Vault)**

- **Purpose**: Production credentials, dynamic secrets, access control
- **Location**: Vault secret engine (`secret/database/*`, `secret/rabbitmq/*`)
- **Contains**: Production passwords, certificates, API keys
- **Used by**: Spring Cloud Vault at runtime

## 🔐 **Vault Secret Integration**

Application configuration references Vault secrets using Spring placeholders:

```yaml
# Infrastructure service names (hardcoded)
spring:
  datasource:
    url: jdbc:postgresql://postgres:5432/microservices_db
    username: ${database.username}  # ← From Vault: secret/database/username
    password: ${database.password}  # ← From Vault: secret/database/password
  
  rabbitmq:
    host: rabbitmq  # Infrastructure service name
    username: ${rabbitmq.username}  # ← From Vault: secret/rabbitmq/username
    password: ${rabbitmq.password}  # ← From Vault: secret/rabbitmq/password
```

## 📁 **Directory Structure**

```text
config-repo/
  application.yml                # Global defaults (all services / profiles)
  application-<profile>.yml      # Profile-wide overrides (e.g. dev, prod, local)
  application-performance.yml    # Performance optimizations (caching, resilience, monitoring)
  <service-name>/application.yml # Service-specific overrides
  <service-name>/application-<profile>.yml (optional)
```

Profiles merge in this order (last wins):

1. application.yml
2. service base file
3. application-&lt;profile&gt;.yml
4. service application-&lt;profile&gt;.yml
5. Vault secrets interpolation (if enabled)

## 🚀 **Usage Modes**

| Mode | When | Setup |
|------|------|-------|
| Native (filesystem) | Fast local dev / workshops / offline | Run `make local` or `make start PROFILE=dev NATIVE=yes` |
| Git remote | Shared team config, audit/history | Set env vars then `make start PROFILE=dev` |

## ⚙️ **Switching Between Native and Git**

You can select native vs Git when starting (interactive prompt for dev/prod) or force it:

```bash
# Force native (filesystem)
make start PROFILE=dev NATIVE=yes

# Force Git
make start PROFILE=dev NATIVE=no

# Via start.sh
./start.sh dev --native
./start.sh dev --no-native
```

## 🌐 **Environment Variables (Git Mode)**

| Variable | Purpose | Example |
|----------|---------|---------|
| CONFIG_REPO_URI | Git URL (HTTPS or SSH) | https://github.com/org/micro-config.git |
| GIT_USERNAME | Optional username (HTTPS private repos) | myuser |
| GIT_TOKEN | Personal access token / password | ghp_xxx |

If using SSH + deploy key, typical Java system properties or global Git config handle key usage; no username/token needed.

## 🔧 **Making It a Separate Repository**

1. Create a new bare repo (e.g. micro-config).
2. Move (or copy) existing contents of `config-repo/` there.
3. Update submodule reference (optional):

```bash
git submodule add <git-url> config-repo
```

4. Commit and push.
5. Export CONFIG_REPO_URI to point at remote.

## ✏️ **Adding/Changing Properties**

1. Edit the appropriate file (avoid duplicating defaults—prefer minimal overrides).
2. Commit & push (Git mode) or just save (native mode).
3. Trigger refresh (if using Spring Cloud Bus) or restart affected service; otherwise config is applied at bootstrap only.

## 🔒 **Vault Integration**

When Vault is enabled (dev/prod profiles default), placeholders like `${database.username}` are resolved automatically by Spring Cloud Vault and not stored here in plain text.

**Vault Secret Paths:**

- `secret/database/username` → `${database.username}`
- `secret/database/password` → `${database.password}`
- `secret/rabbitmq/username` → `${rabbitmq.username}`
- `secret/rabbitmq/password` → `${rabbitmq.password}`
- `secret/redis/password` → `${redis.password}`

## ⚡ **Performance Profile**

The `performance` profile (`application-performance.yml`) provides optimized settings for production workloads:

**Included optimizations:**

- **Caching**: Caffeine cache configuration for better response times
- **Connection Pools**: Optimized Hikari and Lettuce pool settings
- **Service Discovery**: Faster Consul health check intervals for quicker failover
- **Monitoring**: Enhanced metrics and health check configurations

## 🏷️ **Profile Strategy**

| Profile | Configuration Source | Vault | Use Case |
|---------|---------------------|-------|----------|
| `local,native` | Filesystem config | ❌ Disabled | Local development |
| `dev,native,vault` | Filesystem config | ✅ Enabled | Development with Vault |
| `dev,vault` | Git repository | ✅ Enabled | Development environment |
| `prod,vault,performance` | Git repository | ✅ Enabled | Production deployment |

## 🧹 **Configuration Best Practices**

### **✅ DO:**

- Keep service-specific configs minimal (override only what's needed)
- Use Vault for all sensitive data in non-local environments
- Reference infrastructure services by name (not IP/port)
- Document configuration changes in commit messages

### **❌ DON'T:**

- Duplicate global settings in service-specific files
- Hardcode infrastructure credentials in application config
- Mix infrastructure bootstrap with application configuration
- Store production secrets in configuration files

## 🔍 **Troubleshooting**

### **Config not updating:**

- Restart service (config applied at bootstrap only without refresh)
- Check Spring Cloud Config Server logs
- Verify profile activation and config server URI

### **Vault secrets not resolving:**

- Ensure Vault is enabled in environment (`VAULT_ENABLED=true`)
- Check Vault authentication and secret paths
- Verify Spring Cloud Vault configuration

### **Service can't connect to infrastructure:**

- Verify Docker network connectivity
- Check infrastructure service names match configuration
- Ensure infrastructure started before application services
- **Resilience4j**: Circuit breakers, retries, and timeouts for fault tolerance  
- **Monitoring**: Enhanced metrics with Prometheus integration and percentiles
- **Consul**: Optimized health check intervals for faster service discovery
- **Logging**: Reduced verbosity for better performance

**Usage:**
```bash
# Enable performance optimizations
make start-performance PROFILE=prod

# Or combine with other profiles
SPRING_PROFILES_ACTIVE=prod,performance make start

# In docker-compose override
environment:
  - SPRING_PROFILES_ACTIVE=prod,performance
```

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

