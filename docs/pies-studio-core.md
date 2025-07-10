# PIES Studio Core - Docker Image Documentation

## Overview

This document provides all necessary information for configuring and running the `pies-studio-core` Docker image as part of the PIES Studio offline deployment stack. This image represents the core backend server of the PIES Studio platform.

---

## Image Details

* **Image Name:** `piesio/pies-studio-core:offline`
* **Purpose:** Backend server that powers core functionalities such as user logic, workflows, screen generation, deployment coordination, and integrations with licensing, codegen, and frontend systems.
* **Port Exposed:** `8080` (HTTP API), `9081` (WebSocket server)

---

## Volume Mounts

| Mount Path in Container     | Purpose                                                             |
| --------------------------- | ------------------------------------------------------------------- |
| `/dist/config/config.json`  | Injects the runtime configuration file for the core backend         |
| `/dist/assets` *(optional)* | Optional directory used for hosting static assets or logs if needed |

---

## Configuration File (config.json)

Place this JSON file in the host path mapped to `/dist/config/config.json`.

```json
{
  "host": "mongodb://<USERNAME>:<PASSWORD>@<MONGO_HOST>:<PORT>/",
  "db": "pies_db",
  "domain": "",
  "log_snapshot_path": "./logs",
  "piesUrl": "http://<STUDIO_WEB_HOST>:4200",
  "core_url": "http://<CORE_HOST>:8080",
  "license_url": "http://<LICENSE_SERVER_HOST>:9070",
  "loki_web": "http://<PREVIEW_HOST>:4300",
  "loki_ws": "ws://<CODEGEN_HOST>:9090",
  "loki_http": "http://<CODEGEN_HOST>:9090",
  "agbi_url": "http://<AI_SERVER_HOST>:9070",
  "registration_url": "http://<LICENSE_WEB_HOST>:4100",
  "deployment": {
    "ws": "ws://<DEPLOYER_HOST>:9072/deployer/socket",
    "http": "http://<DEPLOYER_HOST>:9071/deployer/api"
  },
  "mail": {
    "host": "smtp.office365.com",
    "port": 587,
    "username": "<SMTP_USERNAME>",
    "password": "<SMTP_PASSWORD>"
  },
  "default_smtp": {
    "name": "PIES Studio Default Server",
    "host": "smtp.office365.com",
    "port": 587,
    "username": "<SMTP_USERNAME>",
    "password": "<SMTP_PASSWORD>"
  },
  "license": {
    "secret": "<LICENSE_SECRET>" #Same as the one mentioned in the pies-license-server config file.
  }
}
```

> **Note:** Replace all placeholder values (e.g., `<MONGO_HOST>`, `<SMTP_USERNAME>`) with actual values for your environment.

---

## Environment Variables

| Variable      | Description                                                             |
| ------------- | ----------------------------------------------------------------------- |
| `ENVIRONMENT` | Environment name (e.g., `offline`)                                      |
| `CONFIG_PATH` | Path to config file inside container (e.g., `/dist/config/config.json`) |

---

## Example Docker Run Command

```bash
docker run -d \
  --name pies-studio-core \
  -p 8080:8080 \
  -p 9081:9081 \
  -e ENVIRONMENT=offline \
  -e CONFIG_PATH=/dist/config/config.json \
  -v $(pwd)/config/core/config.json:/dist/config/config.json:ro \
  piesio/pies-studio-core:offline
```

---

## Notes

* The application binary is built using Go and compiled with security flags in a multi-stage build.
* Git is required in the container as part of runtime logic.
* SMTP credentials are necessary for features like user invites or system notifications.

---

## Support

For further questions or support, reach out to `support@pies.io` with container logs and environment context.
