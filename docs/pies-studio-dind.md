# PIES Studio - Docker In Docker (DIND) Image

This container enables a Docker-in-Docker environment specifically designed for the PIES Studio offline preview system. It works alongside the PIES Studio Code Generation Engine to support application builds and previews in isolated environments.

## Overview

* **Image Name:** `piesio/pies-studio-dind:offline`
* **Base Image:** docker\:dind
* **Purpose:** Runs a privileged Docker daemon that listens for container deployment and preview requests initiated by the Codegen service.
* **Network Mode:** Must be on the same Docker network as Codegen.

## Configuration

The container reads its configuration from a JSON file provided via volume mount.

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
    "password": "<YOUR_DATABASE_PASSWORD>"
  }
}
```

> Replace `host.docker.internal` and passwords with values specific to your environment.

## Volume Mounts

| Container Path                     | Description                                            |
| ---------------------------------- | ------------------------------------------------------ |
| `/certs/client`                    | TLS certificates used for secure Docker communication  |
| `/loki/generated`                  | Folder where generated projects are mounted and built  |
| `/loki/config/config.json`         | Configuration file for this container (read-only)      |
| `/vault/config/init-response.json` | Vault auto-unseal file shared with Codegen (read-only) |

## Environment Variables

| Variable             | Description                                                                        |
| -------------------- | ---------------------------------------------------------------------------------- |
| `ENVIRONMENT`        | Set to `offline` for offline deployments                                           |
| `CONFIG_PATH`        | Path to the config file inside the container, typically `/loki/config/config.json` |
| `DOCKER_TLS_CERTDIR` | Must be set to `/certs/` to allow TLS-based Docker communication                   |
| `DOCKER_TLS_VERIFY`  | Enables Docker TLS verification, should be `enable`                                |
| `MODE`               | Should be set to `proxy` to expose Docker as a proxy endpoint                      |

## Ports

| Port   | Purpose                        |
| ------ | ------------------------------ |
| `9010` | UI Preview Container Port      |
| `9020` | Backend Preview Container Port |

## Example Docker Compose Entry

```yaml
docker:
  image: piesio/pies-studio-dind:offline
  container_name: docker
  privileged: true
  environment:
    - CONFIG_PATH=/loki/config/config.json
    - ENVIRONMENT=offline
    - DOCKER_TLS_CERTDIR=/certs/
    - DOCKER_TLS_VERIFY=enable
    - MODE=proxy
  ports:
    - "9010:9010"
    - "9020:9020"
  volumes:
    - ./certs/client:/certs/client
    - ./generated:/loki/generated
    - ./config/codegen/config.json:/loki/config/config.json:ro
    - ./config/vault/config/init-response.json:/vault/config/init-response.json:ro
  networks:
    - pies-network
```

## Notes

* This container must run in privileged mode to allow nested Docker builds.
* Should be deployed alongside the Codegen container in the same network.
* Vault must be configured and initialized prior to starting this container.

---

For more help, reach out to the PIES onboarding team or visit the support documentation.
