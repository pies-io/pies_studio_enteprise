# PIES Studio Code Generation Engine

This container provides the backend service responsible for code generation in the PIES Studio platform. It generates production-ready backend and frontend application code based on user-defined models and templates. This engine integrates with Vault for secret management and supports Docker-based previews of generated applications.

---

## Image Details

* **Image Name:** `piesio/pies-studio-codegen`
* **Tag:** `offline`
* **Base Image:** Multi-stage Go builder with `golang:1.22` and `alpine` runtime
* **Port:** `9090` (HTTP API for code generation)

---

## Configuration File

This container expects a `config.json` file mounted at runtime. This file contains environment-specific configurations.

### Sample `config.json`

```json
{
  "project_path": "./generated/",
  "sub_domain": "",
  "vault": {
    "url": "http://host.docker.internal:8200",
    "mount": "app/secrets",
    "init_response_path": "../vault/config/init-response.json"
  },
  "redis": {
    "addr": "host.docker.internal:6379",
    "password": ""
  },
  "url": {
    "ui": "http://host.docker.internal:9010",
    "backend": "http://host.docker.internal:9020",
    "core": "http://host.docker.internal:8080"
  },
  "db": {
    "host": "host.docker.internal:3306",
    "name": "pies_preview_db",
    "username": "root",
    "password": "ajsfa8osfy90a8"
  }
}
```

> Replace all hardcoded URLs, credentials, and database information with values suitable for your own environment before deploying.

---

## Volume Mounts

| Host Path                                  | Container Path                     | Description                                         |
| ------------------------------------------ | ---------------------------------- | --------------------------------------------------- |
| `./generated/`                             | `/loki/generated`                  | Output directory for generated application code     |
| `./certs/client/`                          | `/certs/client`                    | TLS certificates for Docker-in-Docker communication |
| `./config/codegen/config.json`             | `/loki/config/config.json`         | Main configuration file for the Codegen engine      |
| `./config/vault/config/init-response.json` | `/vault/config/init-response.json` | Vault unseal response, used to auto-unseal Vault    |

---

## Environment Variables

| Variable            | Description                                           |
| ------------------- | ----------------------------------------------------- |
| `CONFIG_PATH`       | Path to the main configuration file (`config.json`)   |
| `ENVIRONMENT`       | Environment name (e.g., `offline`, `prod`)            |
| `MODE`              | Should be set to `listen` for normal server mode      |
| `DOCKER_HOST`       | URL to Docker daemon                                  |
| `DOCKER_CERT_PATH`  | Path to TLS client certificates                       |
| `DOCKER_TLS_VERIFY` | Should be `enable` to enforce Docker TLS verification |

---

## Docker Compose Example

```yaml
pies-studio-codegen:
  image: piesio/pies-studio-codegen:offline
  container_name: pies-studio-codegen
  ports:
    - "9090:9090"
  environment:
    - CONFIG_PATH=/loki/config/config.json
    - ENVIRONMENT=offline
    - MODE=listen
    - DOCKER_HOST=tcp://docker:2376
    - DOCKER_CERT_PATH=/certs/client/
    - DOCKER_TLS_VERIFY=enable
  volumes:
    - ./certs/client:/certs/client
    - ./generated:/loki/generated
    - ./config/codegen/config.json:/loki/config/config.json:ro
    - ./config/vault/config/init-response.json:/vault/config/init-response.json
  depends_on:
    - pies-studio-vault
    - mysql
    - redis
    - docker
```

---

## Notes

* This service depends on Docker-in-Docker (`pies-studio-dind`) for creating preview containers.
* Vault must be configured and initialized correctly. The `init-response.json` file should contain unseal keys and token.
* Generated applications are placed inside the `generated` directory, mounted from the host.

---

For support, refer to the main deployment guide or contact the PIES onboarding team.
