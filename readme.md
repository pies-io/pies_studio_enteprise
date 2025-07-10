
# PIES Studio - Offline Deployment Guide

A confluence version of this guide can also be found [here](https://pie.atlassian.net/wiki/x/AYAlV).

This guide walks you through setting up the full PIES Studio platform in offline mode using Docker Compose. It is designed for enterprise customers who have received access to licensed containers and configuration files.

---

## **Table of Contents**

- [Table of Contents](#table-of-contents)
- [1. Prerequisites](#key-1-prerequisites)
- [2. Directory Structure](#key-2-directory-structure)
- [3. Configuration Files](#key-3-configuration-files)
- [4. Getting Started](#key-4-getting-started)
  - [Step 1: Install Docker and Docker Compose](#step-1-install-docker-and-docker-compose)
  - [Step 2: Prepare Required Files](#step-2-prepare-required-files)
  - [Step 3: Start the System](#step-3-start-the-system)
- [5. Docker Compose Setup](#key-5-docker-compose-setup)
  - [PIES Studio Images](#pies-studio-images)
  - [Start and Stop Scripts](#start-and-stop-scripts)
- [6. Post-Setup Verification](#key-6-post-setup-verification)
- [7. Running on Other Platforms (Kubernetes, Nomad, etc.)](#key-7-running-on-other-platforms-kubernetes-nomad-etc)
- [8. Individual Installation](#key-8-individual-installation)
- [9. Support](#key-9-support)

---

## **1. Prerequisites**

Before you begin, ensure the following are installed on your system:

- Docker (v20+ recommended)
- Docker Compose (v1.29+ or Docker Compose Plugin for v2)
- Linux/Unix environment (Ubuntu/RHEL recommended)
- A valid offline license file pies\_studio.license

If you’re using a VM or air-gapped machine, ensure Docker can access the required ports.

---

## **2. Directory Structure**
> [!IMPORTANT]
> While this guide references the directory structure below for Docker Compose, it is not required to follow this exact layout. You may use a custom setup, but ensure that the paths to the config.json files are correctly mapped to the appropriate containers.

### Recommended Approach
The most straightforward way to set up the required directory structure is to clone this repository directly.

### Manual Approach
Create a directory structure as follows:

```java
/pies-studio/
|-- docker-compose.yml
|-- start.sh
|-- stop.sh
|-- config/
|   |-- license/
|   |   |-- config.json
|   |   |-- config.web.json
|   |   |-- pies_studio.license
|   |-- core/config.json
|   |-- web/config.json
|   |-- preview/config.json
|   |-- codegen/config.json
|   |-- vault/config/init-response.json
|-- certs/
|   |-- client/
|-- db/
    |-- mongo/
|-- generated/
```

---

## **3. Configuration Files**

Each microservice container reads its environment-specific configuration from mounted JSON files.

|  File Path                        |  Purpose                                                                                            |
|:----------------------------------|:----------------------------------------------------------------------------------------------------|
| `config/license/config.json`      | Configuration for the License Server                                                                |
| `config/core/config.json`         | Configuration for the PIES Studio Core backend                                                      |
| `config/web/config.json`          | Runtime config for the Studio frontend                                                              |
| `config/preview/config.json`      | Runtime config for the Preview frontend                                                             |
| `config/codegen/config.json`      | Runtime config for the code generation engine                                                       |
| `vault/config/init-response.json` | Unsealing data for Vault, used by Codegen. Automatically generated, should not be written manually. |
| `pies_studio.license`             | Your encrypted license key (do not modify)                                                          |

> [!IMPORTANT]
> Detailed information regarding each configuration file can be found on the ‘Repository Overview’ section for the image on DockerHub.  
> The links to the DockerHub repositories can be found in the sections that follow.  
>   
> For platform-specific deployments (Kubernetes, Nomad, etc.), ensure these configuration files are:
>
> - Mounted into containers using ConfigMaps or host volumes
> - Set to read-only where applicable
> - Maintained securely, especially pies\_studio.license

---

## **4. Getting Started**

### **Step 1: Install Docker and Docker Compose**

Refer to the official [Docker installation docs](https://docs.docker.com/engine/install/) and [Compose plugin guide](https://docs.docker.com/compose/install/).

### **Step 2: Prepare Required Files**

Place the license file and configuration JSONs in their respective directories as shown in the directory structure.

### **Step 3: Start the System**

Navigate to your base folder and run the provided start.sh script:

```shell
./start.sh
```

Docker will start all services and create the required networks and volumes.

To view running containers:

```shell
docker ps
```

To view logs for a container:

```shell
docker logs -f <container_name>
```

To shut down all services and clean up generated files, use the stop.sh script:

```shell
./stop.sh
```

---

## **5. Docker Compose Setup**

Below is the complete Docker Compose configuration for running PIES Studio in offline mode:

```docker
version: '3.7'

services:

  # -----------------------------------------------
  # License Server
  # -----------------------------------------------
  # Handles authentication, user, license, and SSO management.
  pies-studio-license-server:
    image: piesio/pies-studio-license-server:offline
    container_name: pies-studio-license-server
    ports:
      - "9070:9070"  # REST API exposed to internal services and web portals
    environment:
      - CONFIG_PATH=/bin/app/config/config.json  # Path to the license service configuration
      - ENVIRONMENT=offline
      - REDIRECT_URI_STUDIO=http://localhost:4200/login  # OAuth redirect
      - REDIRECT_URI_ADMIN=http://localhost:4100/auth/login
      - ORG_DOMAIN=localhost
    volumes:
      # Read-only config file used at startup
      - ./config/license/config.json:/bin/app/config/config.json:ro

      # Encrypted license file mounted read-write for runtime updates
      - ./config/license/pies_studio.license:/bin/app/pies_studio.license:rw
    networks:
      - pies-network
    depends_on:
      - redis
      - mongo

  # -----------------------------------------------
  # License Admin Portal
  # -----------------------------------------------
  # A static admin UI (Angular) to manage user access and licenses.
  pies-studio-license-web:
    image: piesio/pies-studio-license-web:offline
    container_name: pies-studio-license-web
    ports:
      - "4100:80"
    volumes:
      # Inject runtime config for the admin portal (config.web.json)
      - ./config/license/config.web.json:/usr/share/nginx/html/browser/assets/env/config.json:ro
    networks:
      - pies-network
    depends_on:
      - pies-studio-license-server

  # -----------------------------------------------
  # Core Backend Server
  # -----------------------------------------------
  # Hosts APIs for screens, workflows, apps, users and internal logic.
  pies-studio-core:
    image: piesio/pies-studio-core:offline
    container_name: pies-studio-core
    ports:
      - "8080:8080"   # Public API port
      - "9081:9081"   # Optional internal communication (e.g., pub/sub)
    environment:
      - CONFIG_PATH=/dist/config/config.json
      - ENVIRONMENT=offline
    volumes:
      # Backend configuration for database, license service, etc.
      - ./config/core/config.json:/dist/config/config.json:ro
    networks:
      - pies-network
    depends_on:
      - pies-studio-license-server
      - mongo

  # -----------------------------------------------
  # Studio Frontend Portal
  # -----------------------------------------------
  # The no-code interface for end users to build apps.
  pies-studio-web:
    image: piesio/pies-studio-web:offline
    container_name: pies-studio-web
    ports:
      - "4200:80"
    volumes:
      # Runtime environment config injected at container start
      - ./config/web/config.json:/usr/share/nginx/html/assets/env/config.json:ro
    networks:
      - pies-network
    depends_on:
      - pies-studio-core

  # -----------------------------------------------
  # Code Generation Engine
  # -----------------------------------------------
  # Converts app models into actual deployable code and artifacts.
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
      # Client certificates used to connect securely to Docker-in-Docker
      - ./certs/client:/certs/client

      # Shared folder for generated apps and temporary files
      - ./generated:/loki/generated

      # Codegen engine configuration file
      - ./config/codegen/config.json:/loki/config/config.json:ro

      # Vault unseal response for unlocking secrets on boot
      - ./config/vault/config/init-response.json:/vault/config/init-response.json
    networks:
      - pies-network
    depends_on:
      - pies-studio-vault
      - mysql
      - redis
      - docker

  # -----------------------------------------------
  # Docker-in-Docker Service
  # -----------------------------------------------
  # Allows PIES Studio to build and preview apps in isolation.
  docker:
    image: piesio/pies-studio-dind:offline
    container_name: docker
    environment:
      - CONFIG_PATH=/loki/config/config.json
      - ENVIRONMENT=offline
      - DOCKER_TLS_CERTDIR=/certs/
      - DOCKER_TLS_VERIFY=enable
      - MODE=proxy
    ports:
      - "9010:9010"  # HTTP port used by internal Docker proxy
      - "9020:9020"  # Optional gRPC or Docker events port
    volumes:
      - ./certs/client:/certs/client
      - ./generated:/loki/generated
      - ./config/codegen/config.json:/loki/config/config.json:ro
      - ./config/vault/config/init-response.json:/vault/config/init-response.json
    networks:
      - pies-network
    depends_on:
      - pies-studio-vault
    privileged: true  # Required for running Docker inside Docker

  # -----------------------------------------------
  # Preview Client
  # -----------------------------------------------
  # UI for previewing apps deployed by the codegen engine.
  pies-studio-preview:
    image: piesio/pies-studio-preview:offline
    container_name: pies-studio-preview
    ports:
      - "4300:80"
    volumes:
      # Preview environment configuration
      - ./config/preview/config.json:/usr/share/nginx/html/assets/env/config.json:ro
    networks:
      - pies-network
    depends_on:
      - pies-studio-core
      - pies-studio-codegen

  # -----------------------------------------------
  # AI Engine (Optional)
  # -----------------------------------------------
  # Handles AI-driven features (e.g., natural language generation).
  pies-studio-ai:
    image: piesio/pies-studio-ai:offline
    container_name: pies-studio-ai
    ports:
      - "9075:9075"
      - "9076:9076"
    environment:
      - AUTH_URL=http://host.docker.internal:9070/auth/key  # Replace if using custom host
    networks:
      - pies-network

  # -----------------------------------------------
  # Vault (Secrets Storage)
  # -----------------------------------------------
  # Used to store and retrieve sensitive secrets.
  pies-studio-vault:
    image: piesio/pies-studio-vault:offline
    container_name: pies-studio-vault
    ports:
      - "8200:8200"
    volumes:
      # Vault bootstrap config (sealed initially)
      - ./config/vault/config/:/vault/config/

      # Encrypted secrets and data
      - ./config/vault/data:/vault/data:rw
    networks:
      - pies-network

  # -----------------------------------------------
  # MySQL Database
  # -----------------------------------------------
  # Required for preview containers
  mysql:
    image: mysql:8.0
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=<YOUR_MYSQL_DB_PASSWORD>  # CHANGE THIS!
      - MYSQL_DATABASE=pies_preview_db
    networks:
      - pies-network

  # -----------------------------------------------
  # MongoDB (Primary Database)
  # -----------------------------------------------
  # Used by all core and license services for app/user/storage.
  mongo:
    image: mongo
    container_name: mongo
    ports:
      - "28018:27017"  # Maps internal port 27017 to external 28018
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=<YOUR_MONGO_DB_PASSWORD>  # CHANGE THIS!
    volumes:
      - ./db/mongo:/data/db
    networks:
      - pies-network

  # -----------------------------------------------
  # Redis (In-Memory Store)
  # -----------------------------------------------
  # Used for caching sessions, rate limits, and pub/sub.
  redis:
    image: redis
    container_name: redis
    ports:
      - "6379:6379"
    networks:
      - pies-network

# Shared network for all services
networks:
  pies-network:
    driver: bridge
```

### **PIES Studio Images**

The following Docker images are used in this deployment. All are hosted under the piesio organization on DockerHub:

|                              |                                                |               |                                                                                                   |
|:-----------------------------|:-----------------------------------------------|:--------------|:--------------------------------------------------------------------------------------------------|
| Service Name                 | Description                                    | Port(s)       | Image URL                                                                                         |
| `pies-studio-web`            | No-code studio frontend                        | 4200          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-web/general)              |
| `pies-studio-core`           | Backend API server                             | 8080, 9081    |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-core/general)             |
| `pies-studio-license-server` | License and Auth management server             | 9070          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-license-server/general)   |
| `pies-studio-license-web`    | Admin portal for licenses and SSO              | 4100          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-license-web/general)      |
| `pies-studio-codegen`        | Code generation engine                         | 9090          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-codegen/general)          |
| `docker`                     | Docker-in-Docker service for preview execution | 9010, 9020    |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-dind/general)             |
| `pies-studio-preview`        | Frontend client for previewing deployed apps   | 4300          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-preview/general)          |
| `pies-studio-ai`             | Backend AI assistant service                   | 9075, 9076    |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-ai/general)               |
| `pies-studio-vault`          | Modified Vault image for secure secret storage | 8200          |[Docker Hub](https://hub.docker.com/repository/docker/piesio/pies-studio-vault/general)            |
| `mongo`                      | MongoDB database                               | 28018 (27017) |                                                                                                   |
| `mysql`                      | MySQL database for codegen                     | 3306          |                                                                                                   |
| `redis`                      | Caching and OTP queue                          | 6379          |                                                                                                   |

The setup also includes official images for:

- MySQL 8.0
- MongoDB
- Redis

### **Start and Stop Scripts**

**start.sh**

```bash
docker compose up -d
```

**stop.sh**

```bash
docker compose down
rm -rf certs/client/*
rm -rf generated/*
```

---

## **6. Post-Setup Verification**

After the services are up:

- Visit <http://localhost:4200> to access the PIES Studio portal.
- Visit <http://localhost:4100> to access the License Admin portal.
- Check logs of the codegen container to verify Vault unsealing.
- Use docker exec -it mongo mongosh to verify DB connection if needed.

If any containers fail, inspect logs using docker logs or use docker-compose down and retry after fixing config issues.

---

## 7. **Running on Other Platforms (Kubernetes, Nomad, etc.)**

> [!IMPORTANT]
> While this guide focuses on Docker Compose, the PIES Studio Offline stack is fully containerised and can be deployed on any container orchestration platform, including Kubernetes, Nomad, Docker Swarm or custom container runtimes.

To do so:

- Use the full `docker-compose.yml` as a reference for service definitions, ports, inter-container networking, and volume mounts.
- Each container must have access to its corresponding `config.json` file through a volume mount or secret.
- The `pies_studio.license` file must be mounted read-only into the license server container.
- Vault must be unsealed via the shared volume between `pies-studio-vault` and `pies-studio-codegen` as specified in the compose file.
- Expose ports using Ingress (Kubernetes) or equivalent Service / Proxy mechanisms.
- Containers must run within the same network/namespace for internal communication.
- You may use environment-specific mechanisms to inject secrets (e.g., Kubernetes Secrets, AWS SSM, Nomad Vault integrations).

---

## 8. Individual Installation

For details regarding installing each of the images individually, please refer to the ‘Repository Overview’ section on the DockerHub website for each of the images.

---

## **9. Support**

For licensing issues or deployment support, please contact your onboarding specialist or reach out to [support@pies.io](mailto:support@pies.io).

Please include the following when raising an issue:

- Docker Compose logs (docker-compose logs)
- Environment details (OS, Docker version, etc.)

---
