# 🗂️ Configuration Repository

> [⬅️ Back to Main README](../README.md)

This directory is a **Git submodule** that contains all centralized and externalized configurations for the microservices ecosystem. It serves as the backend for the **Spring Cloud Config Server**.

---

## 📋 Table of Contents

- [🏛️ How It Works](#-how-it-works)
- [🚀 Getting Started](#-getting-started)
- [📁 Repository Structure](#-repository-structure)
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

### Configuration Hierarchy

Spring Cloud Config resolves properties in a specific priority order (from lowest to highest):

1.  `application.yml`
2.  `application-{profile}.yml`
3.  `{service-name}/application.yml`
4.  `{service-name}/application-{profile}.yml` (**highest priority**)

This allows you to set global defaults and override them for specific services or environments as needed.

---

## 📝 Best Practices

-   **Keep Secrets Out:** Never commit sensitive data like passwords or API keys directly into this repository. Use environment variables in your deployment environment (e.g., Docker Compose) to provide them. The configuration files can reference these environment variables.
    *Example:* `password: ${DB_PASSWORD}`
-   **Use Global Files for Shared Config:** Define common properties (like Eureka's location) in the global `application.yml` to avoid repetition.
-   **Document Custom Properties:** If you add a new, non-obvious property, add a comment in the YAML file explaining what it does.

---

See the main [README](../README.md) for full system details.
