# PIES Studio Web (Offline Edition)

The **PIES Studio Web** image provides the frontend interface for the PIES Studio no-code platform. It is a fully static Angular application served via **NGINX**, designed to work in offline or air-gapped environments.

---

## Image Summary

- **Base Image:** `nginx:alpine`
- **Frontend Framework:** Angular (compiled using `ng build`)
- **Port Exposed:** `80`
- **Volume Mount (for runtime config):** `/usr/share/nginx/html/assets/env/config.json`
- **Runtime Config Format:** JSON
- **Environment Variables:** None

---

## Configuration

This image expects a `config.json` file to be mounted at:

```
/usr/share/nginx/html/assets/env/config.json
```

The configuration file should contain URLs for backend communication. Below is a sample structure:

```json
{
  "baseUrl": "http://<YOUR_CORE_BACKEND_URL>/core",
  "baseWebSocketUrl": "ws://<YOUR_CORE_BACKEND_URL>:9081/socket",
  "previewUrl": "http://<YOUR_PREVIEW_CLIENT_URL>",
  "agbiUrl": "ws://<YOUR_AI_SERVICE_URL>",
  "licenseServerUrl": "http://<YOUR_LICENSE_SERVER_URL>",
  "licenseClientUrl": "http://<YOUR_LICENSE_WEB_CLIENT_URL>"
}
```

Replace the placeholder URLs with actual values for your deployment environment.

---

## Volume Mounts

| Host Path                          | Container Path                                             | Purpose                                   |
|-----------------------------------|------------------------------------------------------------|-------------------------------------------|
| `./config/web/config.json`        | `/usr/share/nginx/html/assets/env/config.json`            | Runtime config for frontend API endpoints |

---

## Quick Start (Standalone)

```bash
docker run -d \
  --name pies-studio-web \
  -p 4200:80 \
  -v $(pwd)/config/web/config.json:/usr/share/nginx/html/assets/env/config.json:ro \
  piesio/pies-studio-web:offline
```

---

## Docker Compose Reference

Example service snippet:

```yaml
pies-studio-web:
  image: piesio/pies-studio-web:offline
  container_name: pies-studio-web
  ports:
    - "4200:80"
  volumes:
    - ./config/web/config.json:/usr/share/nginx/html/assets/env/config.json:ro
```

---

## Notes

- This container does not require any environment variables.
- SSL termination should be handled externally (e.g., via a reverse proxy or gateway).
- For customization or branding, refer to the PIES Studio Admin configuration interface.

---

## Build Details

This image is built using a two-stage Dockerfile:

1. **Stage 1:** Uses `node:lts` to compile the Angular application with `npm run <env>`.
2. **Stage 2:** Serves the app using `nginx:alpine`.

---

## Support

If you're encountering issues with this image or need deployment assistance, please contact your onboarding specialist or reach out to:

Email: `support@pies.io`
