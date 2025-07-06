# 🗂️ Configuration Repository

> [⬅️ Back to Main README](../README.md)

This directory is a **Git submodule** that contains all centralized and externalized configurations for the microservices ecosystem. It serves as the backend for the **Spring Cloud Config Server**.

---

## 🚨 SECURITY WARNING

**⚠️ IMPORTANT: Before deploying to production, you MUST configure the following environment variables:**

This repository contains placeholder values (`changeme`) for sensitive configuration. These are **NOT SECURE** and must be replaced with proper values in production:

### Required Environment Variables:
```bash
# Auth Service
export AUTH_SERVICE_USERNAME="your-secure-username"
export AUTH_SERVICE_PASSWORD="your-secure-password"
export OAUTH2_CLIENT_SECRET="your-secure-client-secret"
export OAUTH2_ISSUER_URI="https://your-domain.com/auth"

# Config Server (if security enabled)
export CONFIG_SERVER_USERNAME="config-admin"
export CONFIG_SERVER_PASSWORD="your-secure-config-password"
export CONFIG_SECURITY_ENABLED="true"

# Production URLs
export EUREKA_DEFAULT_ZONE="https://discovery.your-domain.com/eureka/"
export CORS_ALLOWED_ORIGINS="https://your-frontend-domain.com"
```

### Security Checklist:
- [ ] Replace all `changeme` values with strong, unique passwords
- [ ] Use environment variables for all sensitive values
- [ ] Enable HTTPS in production
- [ ] Configure proper CORS origins
- [ ] Enable authentication on config server
- [ ] Consider using Spring Cloud Config encryption (see below)

---

## 📋 Table of Contents

- [🏛️ How It Works](#-how-it-works)
- [🚀 Getting Started](#-getting-started)
- [📁 Repository Structure](#-repository-structure)
- [🔒 Spring Cloud Config Encryption](#-spring-cloud-config-encryption)
- [📝 Best Practices](#-best-practices)

---

## 🏛️ How It Works

This repository follows the **externalized configuration pattern**, which is a best practice for cloud-native applications.

-   **Single Source of Truth:** All configurations for all services and all environments are stored here.
-   **Git-Based:** Using Git provides version control, history, and auditing for all configuration changes.
-   **Dynamic Serving:** The **Config Server** reads from this repository and serves the appropriate configuration files to each service upon startup.

---

## 🚀 Getting Started

This repository is included as a Git submodule.

-   If you cloned the parent repository with `git clone --recurse-submodules`, this directory will be populated automatically.
-   If you cloned without that flag, you must initialize it manually:
    ```shell
    git submodule update --init --recursive
    ```

---

## 📁 Repository Structure

The repository is structured to support multiple services and environments.

```
config-repo/
├── application.yml                 # 🌐 Global defaults for ALL services
├── application-dev.yml             # 🔧 Global development overrides
├── application-prod.yml            # 🏭 Global production overrides
├── api-gateway/
│   ├── application.yml             # 🚪 Gateway-specific configuration
│   └── ...
├── auth-service/
│   ├── application.yml             # 🔐 Auth service config
│   └── ...
└── ... (other services)
```

## 🔒 Spring Cloud Config Encryption

For production environments, consider encrypting sensitive values using Spring Cloud Config's built-in encryption features:

### Setup Encryption Key

1. **Generate a symmetric encryption key:**
   ```bash
   # Generate a random 256-bit key
   openssl rand -hex 32
   ```

2. **Configure the config server with encryption key:**
   ```yaml
   # config-server/src/main/resources/application.yml
   encrypt:
     key: ${ENCRYPT_KEY:your-generated-key-here}
   ```

3. **Set environment variable:**
   ```bash
   export ENCRYPT_KEY="your-generated-256-bit-key"
   ```

### Encrypt Sensitive Values

1. **Use the config server's encrypt endpoint:**
   ```bash
   # Encrypt a password
   curl -X POST http://localhost:8888/encrypt \
     -H "Content-Type: text/plain" \
     -d "my-secret-password"
   
   # Returns: AQA...encrypted-value...==
   ```

2. **Use encrypted values in configuration:**
   ```yaml
   # Instead of:
   password: changeme
   
   # Use:
   password: '{cipher}AQA...encrypted-value...=='
   ```

### Example Encrypted Configuration

```yaml
spring:
  security:
    user:
      name: ${AUTH_SERVICE_USERNAME:user}
      password: '{cipher}AQB8n2gF7...encrypted-password...='
    oauth2:
      client:
        registration:
          oidc-client:
            client-secret: '{cipher}AQC2m9kL1...encrypted-secret...='
```

### Asymmetric Encryption (Recommended for Production)

For enhanced security, use RSA key pairs:

1. **Generate RSA key pair:**
   ```bash
   # Generate private key
   openssl genrsa -out config-server.pem 2048
   
   # Extract public key
   openssl rsa -in config-server.pem -pubout -out config-server.pub
   ```

2. **Configure config server:**
   ```yaml
   encrypt:
     key-store:
       location: classpath:config-server.jks
       password: ${KEYSTORE_PASSWORD}
       alias: config-server-key
       secret: ${KEY_SECRET}
   ```

### Benefits of Encryption

- **At-rest security**: Sensitive values are encrypted in Git
- **Automatic decryption**: Config server decrypts values before serving to clients
- **Key rotation**: Change encryption keys without updating all config files
- **Audit trail**: Git history shows when encrypted values changed (but not the actual values)

---

## 📝 Best Practices

-   **Keep Secrets Out:** Never commit sensitive data like passwords or API keys directly into this repository. Use environment variables in your deployment environment (e.g., Docker Compose) to provide them. The configuration files can reference these environment variables.
    *Example:* `password: ${DB_PASSWORD}`
-   **Use Global Files for Shared Config:** Define common properties (like Eureka's location) in the global `application.yml` to avoid repetition.
-   **Document Custom Properties:** If you add a new, non-obvious property, add a comment in the YAML file explaining what it does.

---

See the main [README](../README.md) for full system details.
