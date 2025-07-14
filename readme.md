# PIES Studio - Enterprise License Installation Guide

This guide walks you through setting up the full PIES Studio platform in offline mode using Docker Compose. It is designed for enterprise customers who have received access to licensed containers and configuration files.

---

## **Table of Contents**

- [Table of Contents](#table-of-contents)
- [1. Prerequisites](#key-1-prerequisites)
- [2. Getting Started](#key-2-getting-started)
  - [Step 1: Install Docker and Docker Compose](#step-1-install-docker-and-docker-compose)
  - [Step 2: Prepare Required Files](#step-2-prepare-required-files)
    - [Recommended Approach](#recommended-approach)
    - [Cloning the Repository](#cloning-the-repository)
  - [Copying the License File](#copying-the-license-file)
  - [Preparing Your Deployment Environments](#preparing-your-deployment-environments)
    - [Step 1: Create a Service Account](#step-1-create-a-service-account)
    - [Step 2: Assign Required Roles](#step-2-assign-required-roles)
    - [Step 3: Generate a Service Account Key](#step-3-generate-a-service-account-key)
  - [Setting up your SMTP servers](#setting-up-your-smtp-servers)
  - [Modifying the Configuration Files](#modifying-the-configuration-files)
    - [Manual Approach](#manual-approach)
  - [Step 3: Start the System](#step-3-start-the-system)
- [3. Docker Setup](#key-3-docker-setup)

  - [Manual Setup](#manual-setup)- [PIES Studio Images](#pies-studio-images)
  - [Start and Stop Scripts](#start-and-stop-scripts)
- [4. Post-Setup Verification](#key-4-post-setup-verification)
- [5. Running on Other Platforms (Kubernetes, Nomad, etc.)](#key-5-running-on-other-platforms-kubernetes-nomad-etc)
- [6. Individual Installation](#key-6-individual-installation)
- [7. Support](#key-7-support)

---

## **1. Prerequisites**

Before you begin, ensure the following are installed on your system:

- Docker (v20+ recommended)
- Docker Compose (v1.29+ or Docker Compose Plugin for v2)
- Linux/Unix environment (Ubuntu/RHEL with ARM64 architecture recommended)
- A valid offline license file `pies_studio.license`

If you’re using a VM or air-gapped machine, ensure Docker can access the required ports.

---

## **2. Getting Started**

### **Step 1: Install Docker and Docker Compose**

Refer to the official [Docker installation docs](https://docs.docker.com/engine/install/) and [Compose plugin guide](https://docs.docker.com/compose/install/).

### **Step 2: Prepare Required Files**

#### Recommended Approach

The most straightforward way to set up the required directory structure is to clone our PIES Studio Enterprise repository from GitHub directly. The link to the repository can be found below.

<https://github.com/pies-io/pies_studio_enteprise>

#### **Cloning the Repository**

Run the below command to clone the .git repository, this should provide you all the necessary files and folders along with the `docker-compose.yml` file.

```bash
git clone https://github.com/pies-io/pies_studio_enteprise
```

### **Copying the License File**

Place the `pies_studio.license` file that was provided to you inside the `config/license` folder.

### **Preparing Your Deployment Environments**

The PIES Studio Deployment Engine requires credentials to interact with cloud platforms such as Google Cloud. This section outlines how to prepare your deployment environment using a Service Account approach for secure access to Google Cloud resources.

---

#### **Step 1: Create a Service Account**

1. Go to the [Google Cloud Console | <https://console.cloud.google.com/>].
2. Navigate to *IAM & Admin* → *Service Accounts*.
3. Click on *Create Service Account*.
4. Provide a name and description for the service account (e.g., pies-studio-deployer).
5. Click *Create and Continue*.

---

#### **Step 2: Assign Required Roles**

Assign the following roles to the service account to enable deployment functionality:

- *Kubernetes Engine Admin* – To create and manage GKE clusters.
- *Artifact Registry Administrator* – To create repositories and manage pushed images.
- *Compute Network Admin* – To manage VPC and networking during cluster provisioning.
- *Service Account User* – To allow the use of this service account within GKE clusters.
- *Storage Admin* – To allow storage access where required by Kubernetes or Artifact Registry.

> [!NOTE]
> Note: These are minimum required roles. Additional roles may be necessary depending on your organization’s security policies.

---

#### **Step 3: Generate a Service Account Key**

1. After creating the service account, go to the *Keys* tab.
2. Click *Add Key* → *Create new key*.
3. Choose *JSON* and click *Create*.
4. Download and securely store the generated key file (e.g., service\_account\_config.json).
5. This downloaded JSON key file should be mounted as a volume to the `pies-studio-deployer` image as mentioned in the below configuration.

### **Setting up your SMTP servers**

In order to receive emails from the platform, you need to configure details for an SMTP server inside the following configuration files -

1. `config/core/config.json`
2. `config/license/config.json`

### **Modifying the Configuration Files**

> [!IMPORTANT]
> Each microservice container reads its environment-specific configuration from mounted JSON files located in the `config`folder of the repository.
>
> Before starting, make sure that each configuration file contains the correct URLs. Replace any instances of `http://localhost` with your server's hostname or IP address.

|  File Path                                    |  Purpose                                                                                            |
|:----------------------------------------------|:----------------------------------------------------------------------------------------------------|
| `config/license/config.json`                  | Configuration for the License Server                                                                |
| `config/license/config.web.json`              | Configuration for the License Administrator portal                                                  |
| `config/core/config.json`                     | Configuration for the PIES Studio Core backend                                                      |
| `config/web/config.json`                      | Runtime config for the Studio frontend                                                              |
| `config/preview/config.json`                  | Runtime config for the Preview frontend                                                             |
| `config/codegen/config.json`                  | Runtime config for the code generation engine                                                       |
| `config/deployer/config.json`                 | Deployment engine service configuration                                                             |
| `config/deployer/service_account_config.json` | Cloud credentials for deployment (Service account JSON key file)                                    |
| `vault/config/init-response.json`             | Unsealing data for Vault, used by Codegen. Automatically generated, should not be written manually. |
| `pies_studio.license`                         | Your encrypted license key (do not modify)                                                          |

> [!NOTE]
> Detailed information regarding each configuration file can be found on the ‘Repository Overview’ section for the image on DockerHub.  
> The links to the DockerHub repositories can be found in the sections that follow.
>
> For platform-specific deployments (Kubernetes, Nomad, etc.), ensure these configuration files are:
>
> - Mounted into containers using ConfigMaps or host volumes
> - Set to read-only where applicable
> - Maintained securely, especially pies\_studio.license

<details>
<summary>Advanced</summary>

#### Manual Approach

If you wish to set up manually, create the below folder structure on your system.

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
|   |-- deployer/
|   |   |-- config.json
|   |   |-- service_account_config.json
|   |-- vault/config/init-response.json
|-- certs/
|   |-- client/
|-- db/
    |-- mongo/
|-- generated/
```

> [!IMPORTANT]
> While this guide references the directory structure below for Docker Compose, it is not required to follow this exact layout. You may use a custom setup, but ensure that the paths to the config.json files are correctly mapped to the appropriate containers.

</details>

### **Step 3: Start the System**

Navigate to your base folder and run the provided start.sh script:

```java
./start.sh
```

The first time this command runs, it will download all PIES Studio artifacts, create the required networks and volumes, and start all services.

To view running containers:

```java
docker ps
```

To view logs for a container:

```java
docker logs -f <container_name>
```

To shut down all services and clean up generated files, use the stop.sh script:

```java
./stop.sh
```

---

## **3. Docker Setup**

A `docker-compose.yml` file will already be present in the cloned repository with all the necessary configurations. No additional setup steps are necessary if you follow the recommended approach.

If you wish to set-up the `docker-compose` file manually, please expand the section below.

<details>
<summary>Advanced</summary>

#### Manual Setup

Below is the complete Docker Compose configuration for running PIES Studio in offline mode:

```java
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
  # Application Deployment Engine
  # -----------------------------------------------
  # Packages generate code into docker artifacts and deploys them over cloud environments
  pies-studio-deployer:
    restart: always
    image: piesio/pies-studio-deployer:offline
    container_name: pies-studio-deployer
    ports:
      - "9071:9071"
      - "9072:9072"
    environment:
      - CONFIG_PATH=/deployer/config/config.json
      - CLOUD_CONFIG_PATH=/deployer/config/config.cloud.json
      - ENVIRONMENT=offline
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client/
      - DOCKER_TLS_VERIFY=enable
    volumes:
      - ./certs/client:/certs/client
      - ./generated:/deployer/generated
      - ./config/deployer/config.json:/deployer/config/config.json:ro
      - ./config/deployer/service_account_config.json:/deployer/config/config.cloud.json:ro
    networks:
      - pies-network
    depends_on:
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

</details>

### **PIES Studio Images**

The following Docker images are used in this deployment. All are hosted under the piesio organization on DockerHub:

|                              |                                                |               |                                                                               |
|:-----------------------------|:-----------------------------------------------|:--------------|:------------------------------------------------------------------------------|
| Service Name                 | Description                                    | Port(s)       | Image URL                                                                     |
| `pies-studio-web`            | No-code studio frontend                        | 4200          | <https://hub.docker.com/repository/docker/piesio/pies-studio-web/>            |
| `pies-studio-core`           | Backend API server                             | 8080, 9081    | <https://hub.docker.com/repository/docker/piesio/pies-studio-core/>           |
| `pies-studio-license-server` | License and Auth management server             | 9070          | <https://hub.docker.com/repository/docker/piesio/pies-studio-license-server/> |
| `pies-studio-license-web`    | Admin portal for licenses and SSO              | 4100          | <https://hub.docker.com/repository/docker/piesio/pies-studio-license-web/>    |
| `pies-studio-codegen`        | Code generation engine                         | 9090          | <https://hub.docker.com/repository/docker/piesio/pies-studio-codegen/>        |
| `pies-studio-deployer`       | Application deployment engine                  | 9071, 9072    | <https://hub.docker.com/repository/docker/piesio/pies-studio-deployer/>       |
| `docker`                     | Docker-in-Docker service for preview execution | 9010, 9020    | <https://hub.docker.com/repository/docker/piesio/pies-studio-dind/>           |
| `pies-studio-preview`        | Frontend client for previewing deployed apps   | 4300          | <https://hub.docker.com/repository/docker/piesio/pies-studio-preview/>        |
| `pies-studio-ai`             | Backend AI assistant service                   | 9075, 9076    | <https://hub.docker.com/repository/docker/piesio/pies-studio-ai/>             |
| `pies-studio-vault`          | Modified Vault image for secure secret storage | 8200          | <https://hub.docker.com/repository/docker/piesio/pies-studio-vault/>          |
| `mongo`                      | MongoDB database                               | 28018 (27017) | <https://hub.docker.com/_/mongo>                                              |
| `mysql`                      | MySQL database for codegen                     | 3306          | <https://hub.docker.com/_/mysql>                                              |
| `redis`                      | Caching and OTP queue                          | 6379          | <https://hub.docker.com/_/redis>                                              |

The setup also includes official images for:

- MySQL 8.0
- MongoDB
- Redis

### **Start and Stop Scripts**

**start.sh**

```java
docker compose up -d
```

**stop.sh**

```java
docker compose down
rm -rf certs/client/*
rm -rf generated/*
```

---

## **4. Post-Setup Verification**

After the services are up:

- The platform administrator user should receive an email to set-up their password. This is the same email which was used for the license registration.
- Click on the link in the email to set-up your password.
- Once done, visit <http://localhost:4100> and log-in with your credentials to access the License Admin portal.
- Visit <http://localhost:4200> to access the PIES Studio portal. You can use the same credentials.
- Check logs of the codegen container to verify Vault unsealing.
- Use docker exec -it mongo mongosh to verify DB connection if needed.

If any containers fail, inspect logs using docker logs or use docker-compose down and retry after fixing config issues.

---

## 5. **Running on Other Platforms (Kubernetes, Nomad, etc.)**

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

## 6. Individual Installation

For details regarding installing each of the images individually, please refer to the ‘Repository Overview’ section on the DockerHub website for each of the images.

---

## **7. Support**

For licensing issues or deployment support, please contact your onboarding specialist or reach out to [support@pies.io](mailto:support@pies.io).

Please include the following when raising an issue:

- Docker Compose logs (docker-compose logs)
- Environment details (OS, Docker version, etc.)

---
