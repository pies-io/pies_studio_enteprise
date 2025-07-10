# PIES Studio License Server (Offline Edition)

The PIES Studio License Server is a standalone authentication and license management backend service designed to run in offline or air-gapped environments. It handles license decryption, user management, authentication, and SSO logic.

---

## Image Overview

- **Base Image**: `debian:buster`
- **Default Port**: `9070`
- **Mode**: Offline-only
- **License File**: Must be mounted via volume
- **Vault Integration**: Not required
- **Dependencies**: MongoDB and Redis

---

## Configuration

The License Server reads its configuration from a `config.json` file that must be mounted into the container at runtime.

### Sample `config.json`

```json
{
  "host": "mongodb://<USERNAME>:<PASSWORD>@<MONGO_HOST>:<PORT>/",
  "db": "pies_license_offline_db",
  "redis": {
    "addr": "<REDIS_HOST>:6379",
    "password": ""
  },
  "mail": {
    "host": "smtp.office365.com",
    "port": 587,
    "username": "noreply_studio@pies.io",
    "password": ""
  },
  "deployer": {
    "secret": "<DEPLOYER_SECRET>"
  },
  "pies_url": "http://<PIES_STUDIO_WEB>:4200",
  "pies_license_url": "http://<LICENSE_ADMIN>:4100",
  "pies_billing_url": "",
  "pies_deployer_url": "http://<DEPLOYER>:9071",
  "pies_ai_url": {
    "ws": "ws://<AI_SERVICE>:9076/ai/ws",
    "http": "http://<AI_SERVICE>:9075/ai"
  },
  "pies_auth_url": "http://<LICENSE_SERVER>:9070/auth/login/success",
  "secret": "<JWT_SECRET>"
}
```

> Replace placeholders like `<MONGO_HOST>`, `<REDIS_HOST>`, `<DEPLOYER_SECRET>` with actual values as per your deployment.

---

## Environment Variables

| Variable              | Description                                      | Required |
|-----------------------|--------------------------------------------------|----------|
| `ENVIRONMENT`         | Should be set to `offline`                       | Yes      |
| `CONFIG_PATH`         | Path to the mounted config file inside container | Yes      |
| `REDIRECT_URI_STUDIO` | Redirect URI for Studio login                    | Yes      |
| `REDIRECT_URI_ADMIN`  | Redirect URI for Admin login                     | Yes      |
| `ORG_DOMAIN`          | Domain name used for SSO                         | Yes      |

---

## Volume Mounts

| Host Path                                | Container Path                     | Purpose                                  |
|------------------------------------------|------------------------------------|------------------------------------------|
| `./config/license/config.json`           | `/bin/app/config/config.json`      | Main configuration file                  |
| `./config/license/pies_studio.license`   | `/bin/app/pies_studio.license`     | Encrypted license file (must be writable) |

---

## Running the Container

Example using `docker run`:

```bash
docker run -d \
  --name pies-studio-license-server \
  -p 9070:9070 \
  -e ENVIRONMENT=offline \
  -e CONFIG_PATH=/bin/app/config/config.json \
  -e REDIRECT_URI_STUDIO=http://localhost:4200/login \
  -e REDIRECT_URI_ADMIN=http://localhost:4100/auth/login \
  -e ORG_DOMAIN=localhost \
  -v $(pwd)/config/license/config.json:/bin/app/config/config.json:ro \
  -v $(pwd)/config/license/pies_studio.license:/bin/app/pies_studio.license:rw \
  piesio/pies-studio-license-server:offline
```

---

## Health & Dependencies

- Requires MongoDB with authentication enabled
- Requires Redis (password optional)
- Container will fail to start if the license file is missing or invalid
- Logs are output directly to stdout/stderr

---

## Support

For configuration help, licensing issues, or enterprise onboarding, please contact `support@pies.io`.
