# PIES Studio License Administration Portal

## Overview

The `pies-studio-license-web` image serves as the administrator-facing frontend UI for managing PIES Studio licenses in offline environments. It is an Angular application bundled and served using NGINX.

---

## Image Details

* **Docker Image Name:** `piesio/pies-studio-license-web`
* **Tag:** `offline`
* **Base Image:** `nginx:alpine`
* **Port:** 80 (default)
* **Bundled Framework:** Angular (via `ng build`)

---

## Configuration

### Config File

The application loads runtime configuration from a JSON file typically mounted to:

```
/usr/share/nginx/html/browser/assets/env/config.json
```

### Example `config.json`

```json
{
  "hostUrl": "http://localhost:4100",
  "backendUrl": "http://localhost:9070",
  "authUrl": "http://localhost:9070/auth/login",
  "authRedirectUrl": "http://localhost:4100/auth/login"
}
```

> **Note:** Replace `localhost` with the actual IP or domain name applicable to your setup.

---

## Docker Volume Mount

In your `docker-compose.yml`, make sure you mount the config file correctly:

```yaml
volumes:
  - ./config/license/config.web.json:/usr/share/nginx/html/browser/assets/env/config.json:ro
```

This ensures the container serves the runtime configuration file correctly at application startup.

---

## Environment Variables

This container does **not** require any custom environment variables.

---

## Customization Points

There are no customization hooks or runtime logic available for this image beyond the `config.json` file.

---

## Exposed Port

| Port | Description                           |
| ---- | ------------------------------------- |
| 80   | NGINX server serving Angular frontend |

---

## Usage

This image is typically included in the Docker Compose setup of PIES Studio. It depends on the `pies-studio-license-server` for backend API functionality.

---

## Support

If you face any issues using this image, please contact `support@pies.io` with relevant logs and configuration context.
