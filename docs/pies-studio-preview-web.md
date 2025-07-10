# PIES Studio Preview Client

This image hosts the preview interface of PIES Studio, allowing users to view and interact with live previews of their applications in offline environments.

---

## Image Details

* **Image Name**: `piesio/pies-studio-preview:offline`
* **Purpose**: Frontend client for the preview experience
* **Base Image**: `nginx:alpine`
* **Exposed Port**: `80`
* **Default Host Port**: `4300`

---

## Configuration File

The `config.json` file must be mounted to provide runtime configuration for the frontend. The following is a sample config:

```json
{
  "baseUrl": "http://localhost:8080/core",
  "baseWsUrl": "ws://localhost:9081/socket",
  "licenseServerUrl": "http://localhost:9070",
  "previewServerUrl": "http://localhost:9010"
}
```

> Replace the URLs above with appropriate values from your deployment environment.

---

## Mount Paths

`config.json` must be mounted at:

```
/usr/share/nginx/html/assets/env/config.json
```

> Ensure the config file path is accurate for the application to load correctly.

---

## Docker Example

Here is an example service entry from a Docker Compose setup:

```yaml
pies-studio-preview:
  image: piesio/pies-studio-preview:offline
  container_name: pies-studio-preview
  ports:
    - "4300:80"
  volumes:
    - ./config/preview/config.json:/usr/share/nginx/html/assets/env/config.json:ro
  networks:
    - pies-network
  depends_on:
    - pies-studio-core
    - pies-studio-codegen
````

---

## Environment Variables

This image does **not** require any environment variables.

---

## Customization

This image does not support further runtime customization. Configuration is handled entirely through the `config.json`.

---

## Notes

* Make sure the mounted config.json file contains valid JSON and accessible network URLs.
* No authentication or secrets are handled by this container.
* If you use an orchestration system (e.g., Kubernetes or Nomad), replicate the mount path and networking as needed.

---
