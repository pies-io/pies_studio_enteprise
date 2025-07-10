# PIES Studio AI

This image hosts the AI backend engine for the PIES Studio platform. It is used to provide intelligent features and code generation support to the no-code editor in offline or air-gapped environments.

---

## Overview

The AI backend is a standalone service that exposes HTTP and WebSocket endpoints for interacting with AI-powered features (such as code block generation, design suggestions, etc.). It depends on the License Server for authentication and authorization.

---

## Ports

| Port | Purpose                                |
| ---- | -------------------------------------- |
| 9075 | HTTP API interface                     |
| 9076 | WebSocket endpoint for AI interactions |

These ports can be remapped externally as required.

---

## Environment Variables

| Variable   | Description                                                                       |
| ---------- | --------------------------------------------------------------------------------- |
| `AUTH_URL` | Required. Points to the license server endpoint that returns authentication keys. |

### Example:

```env
AUTH_URL=http://localhost:9070/auth/key
```

This must be a reachable endpoint from inside the container.

---

## Docker Usage

### Example Run Command

```bash
docker run -d \
  --name pies-studio-ai \
  -p 9075:9075 \
  -p 9076:9076 \
  -e AUTH_URL=http://localhost:9070/auth/key \
  piesio/pies-studio-ai:offline
```

### Docker Compose Reference

This image can be integrated into your Docker Compose setup using the following:

```yaml
pies-studio-ai:
  image: piesio/pies-studio-ai:offline
  container_name: pies-studio-ai
  ports:
    - "9075:9075"
    - "9076:9076"
  environment:
    - AUTH_URL=http://localhost:9070/auth/key
  networks:
    - pies-network
```

---

## Notes

* This service does not require any mounted volume or persistent storage.
* There is no config.json file for this service; configuration is via environment variables.
* The AI backend service must be started **after** the License Server is accessible.

---

## Support

If you encounter any issues during deployment or execution, please reach out to `support@pies.io` with the relevant logs and environment details.
