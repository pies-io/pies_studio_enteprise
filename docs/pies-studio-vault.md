# PIES Studio Vault

This image provides a secure vault service for the PIES Studio offline deployment. It is based on a modified version of OpenBao and is used as the secure secrets manager for the entire platform.

## Overview

The Vault container plays a critical role in securely managing secrets during offline deployments of PIES Studio. It is designed to work seamlessly with the PIES Studio Code Generation Engine (`pies-studio-codegen`), which will automatically unseal the vault at runtime.

## Key Behavior

- On startup, this container automatically initializes and **generates a `init-response.json` file**.
- This file is stored at a user-mounted location (`/vault/config/init-response.json`).
- The `pies-studio-codegen` image reads this response and uses it to **auto-unseal the vault** securely without human intervention.

## Volume Mounts

The following volumes must be mounted to ensure correct behavior:

| Host Path                          | Container Path                 | Description                                      |
|-----------------------------------|--------------------------------|--------------------------------------------------|
| `./config/vault/config/`          | `/vault/config/`              | Directory where `init-response.json` will be written |
| `./config/vault/data/`            | `/vault/data/`                | Storage path for Vault data                     |

## Ports

| Port | Description                  |
|------|------------------------------|
| 8200 | Default HTTP API port for Vault |

## Configuration

No user-supplied configuration is needed.

The Vault is automatically initialized and sealed/unsealed based on internal PIES Studio logic. It does **not** require any environment variables or external configuration files.

## Dependencies

This container is typically used alongside:

- `pies-studio-codegen` (auto-unsealing)
- Any service needing secure secrets at runtime

## Usage

This container is meant to run as part of the PIES Studio Docker Compose stack or an equivalent Kubernetes setup.

For example, in `docker-compose.yml`:

```yaml
pies-studio-vault:
  image: piesio/pies-studio-vault:offline
  container_name: pies-studio-vault
  ports:
    - "8200:8200"
  networks:
    - pies-network
  volumes:
    - ./config/vault/config/:/vault/config/
    - ./config/vault/data:/vault/data:rw
```

## Notes

- Do **not** manually modify `init-response.json`. It is used by the system to handle Vault unsealing automatically.
- If you redeploy or recreate the container, ensure that `/vault/data` is persistent to retain existing secrets.

## License

This image is part of the PIES Studio Offline Licensed Distribution.