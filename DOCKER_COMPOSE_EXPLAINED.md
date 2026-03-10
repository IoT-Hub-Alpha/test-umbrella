# Docker Compose Configuration Explained

This guide explains every setting in the `docker-compose.yml` file.

## Table of Contents

1. [Overview](#overview)
2. [Services Configuration](#services-configuration)
3. [Network Configuration](#network-configuration)
4. [Reference Guide](#reference-guide)
5. [Common Modifications](#common-modifications)

---

## Overview

Docker Compose simplifies running multiple containers. Our `docker-compose.yml` defines:
- 2 microservices (FastAPI and Django)
- 1 custom network for inter-service communication
- Environment configuration for each service

**File Structure:**
```yaml
services:        # All containers
  - fastapi-microservice
  - django-microservice

networks:        # Custom networks
  - microservices-network
```

---

## Services Configuration

### `services`

Top-level key that defines all Docker containers to run.

```yaml
services:
  fastapi-microservice:
    # Configuration here
  django-microservice:
    # Configuration here
```

---

### `build`

Specifies how to build the Docker image.

```yaml
build:
  context: ./test-fastapi-microservice      # Build context directory
  dockerfile: Dockerfile                     # Dockerfile name
```

**What it does:**
- `context`: Directory containing the Dockerfile and source code
- `dockerfile`: Name of the Dockerfile (defaults to `Dockerfile`)

**Equivalent Docker Command:**
```bash
docker build -f ./test-fastapi-microservice/Dockerfile ./test-fastapi-microservice
```

**When Docker Compose uses this:**
```bash
docker compose up --build        # Rebuilds images
docker compose up               # Uses existing images (faster)
```

---

### `container_name`

Assigns a permanent name to the running container.

```yaml
container_name: test-fastapi-microservice
```

**Why it matters:**
- Easy to reference: `docker compose exec test-fastapi-microservice bash`
- Readable in `docker ps` output
- Must be unique (can't have duplicate names)

**Without this:**
```bash
# Docker auto-generates names like:
test-umbrella_fastapi-microservice_1
test-umbrella_fastapi-microservice_2  # If scaled
```

**Example usage:**
```bash
docker compose exec test-fastapi-microservice bash
docker compose logs test-fastapi-microservice
docker compose restart test-fastapi-microservice
```

---

### `ports`

Maps ports from container to host machine.

```yaml
ports:
  - "8101:8101"     # host_port:container_port
```

**Format:** `"host_port:container_port"`

**In this project:**
| Service | Mapping | Access URL |
|---------|---------|-----------|
| FastAPI | `8101:8101` | http://localhost:8101 |
| Django | `8100:8100` | http://localhost:8100 |

**How it works:**
```
User's Machine          Docker Container
localhost:8101    -->   (FastAPI container)
                        listening on :8101
```

**Different port example:**
```yaml
ports:
  - "3000:8101"     # Host port 3000 → Container port 8101
                    # Access at http://localhost:3000
```

**Without port mapping:**
```yaml
# No ports section = container is only accessible from other containers
# Not accessible from host machine
```

**Multiple ports:**
```yaml
ports:
  - "8101:8101"     # HTTP
  - "5432:5432"     # Database
  - "9000:9000"     # Another service
```

---

### `env_file`

Loads environment variables from a file into the container.

```yaml
env_file:
  - .env
```

**What happens:**
```bash
# .env file contents:
FASTAPI_HOST=fastapi-microservice
DJANGO_HOST=django-microservice

# Inside container, these variables are available:
$ echo $FASTAPI_HOST
fastapi-microservice
```

**Current .env in this project:**
```bash
# Microservices Configuration
FASTAPI_HOST=fastapi-microservice    # Docker service name
DJANGO_HOST=django-microservice      # Docker service name
```

**Use cases:**
- Configuration that varies per environment
- Service discovery hostnames
- API keys and secrets (though not recommended)

**Multiple env files:**
```yaml
env_file:
  - .env
  - .env.local      # Local overrides
```

---

### `environment`

Sets environment variables directly in the compose file.

```yaml
environment:
  - PYTHONUNBUFFERED=1
```

**Why PYTHONUNBUFFERED=1?**
- Python buffers output by default
- `PYTHONUNBUFFERED=1` forces immediate output
- **Critical for Docker logs** - without it, logs won't appear in real-time

**Other common Python variables:**
```yaml
environment:
  - PYTHONUNBUFFERED=1              # Real-time output
  - PYTHONDONTWRITEBYTECODE=1       # Don't create .pyc files
  - DEBUG=True                       # Custom application variable
  - FLASK_ENV=development            # Flask-specific
```

**Difference between `env_file` and `environment`:**

| Aspect | `env_file` | `environment` |
|--------|-----------|--------------|
| Source | From file (`.env`) | Direct in compose |
| Flexibility | Easy to change per environment | Fixed in compose |
| Use case | Config per environment | Standard/fixed settings |
| Sensitive data | Possible (not recommended) | Not recommended |

**Example combining both:**
```yaml
env_file:
  - .env                          # Load from file
environment:
  - PYTHONUNBUFFERED=1            # Override/add specific vars
  - DEBUG=${DEBUG:-False}         # Use env var with default
```

---

### `networks`

Connects the container to a custom network.

```yaml
networks:
  - microservices-network
```

**What it does:**
- Places container on `microservices-network` bridge network
- Enables DNS service discovery
- **Allows containers to communicate by service name**

**How service discovery works:**

```python
# Inside FastAPI container
import requests

# Without networks: Can't reach Django
# With networks: Can reach Django by name!
response = requests.get("http://django-microservice:8100/api/ping")

# Docker DNS automatically resolves:
# "django-microservice" → Django container's IP address
```

**Network isolation:**
```yaml
# Only containers on microservices-network can talk to each other
# External containers can't access them

services:
  fastapi:
    networks:
      - microservices-network  # Can reach django

  django:
    networks:
      - microservices-network  # Can reach fastapi

  external-app:
    networks:
      - other-network          # Can't reach fastapi or django
```

**Practical example from this project:**
```python
# In FastAPI code
django_host = os.getenv("DJANGO_HOST", "127.0.0.1")  # "django-microservice"
url = f"http://{django_host}:8100/api/ping"

# Inside Docker: Works! DNS resolves "django-microservice"
# Outside Docker (local dev): Falls back to "127.0.0.1"
```

---

### `restart`

Defines the restart policy.

```yaml
restart: unless-stopped
```

**Available policies:**

| Policy | Behavior |
|--------|----------|
| `no` | Don't restart automatically |
| `always` | Always restart if container stops |
| `unless-stopped` | **Restart unless explicitly stopped** |
| `on-failure` | Only restart if exit code is non-zero |
| `on-failure:5` | Restart max 5 times on failure |

**What `unless-stopped` does:**

```bash
# Scenario 1: Container crashes
$ docker compose logs
[ERROR] Application crashed
$ docker compose ps
# Container automatically restarts

# Scenario 2: Explicitly stopped
$ docker compose down
$ docker compose ps
# Container stays stopped (respects the stop)

# Scenario 3: Host reboots
$ sudo reboot
$ docker compose ps
# Container automatically restarts (unless-stopped policy)
```

**Without restart policy:**
```yaml
restart: no
# If container crashes, it stays stopped
# You must manually restart: docker compose restart fastapi-microservice
```

---

## Network Configuration

### Top-level `networks`

Defines custom networks for the project.

```yaml
networks:
  microservices-network:
    driver: bridge
```

**Network drivers:**

| Driver | Use Case |
|--------|----------|
| `bridge` | **Default, best for most cases** |
| `host` | Uses host machine network (no isolation) |
| `overlay` | For Docker Swarm multi-host |
| `none` | No network access |

**Bridge network benefits:**
```yaml
driver: bridge
# - Containers can communicate by service name
# - Isolated from other networks
# - Good for local development
```

**How containers join the network:**
```yaml
services:
  fastapi:
    networks:
      - microservices-network    # Joins this network

  django:
    networks:
      - microservices-network    # Joins same network

  # Now fastapi and django can communicate:
  # fastapi → http://django-microservice:8100
```

---

## Reference Guide

### Complete Service Definition

```yaml
services:
  fastapi-microservice:
    # BUILD: How to create the image
    build:
      context: ./test-fastapi-microservice
      dockerfile: Dockerfile

    # NAME: Easy reference to this container
    container_name: test-fastapi-microservice

    # PORTS: Host machine access
    ports:
      - "8101:8101"              # host:container

    # ENV FROM FILE: Variables from .env
    env_file:
      - .env

    # ENV DIRECT: Variables set directly
    environment:
      - PYTHONUNBUFFERED=1

    # NETWORK: Which network to join
    networks:
      - microservices-network

    # RESTART: Auto-restart policy
    restart: unless-stopped
```

### Minimal Configuration

```yaml
services:
  myservice:
    build:
      context: ./myservice
    ports:
      - "8000:8000"
    networks:
      - default      # Built-in default network
```

### Full-Featured Configuration

```yaml
services:
  myservice:
    build:
      context: ./myservice
      dockerfile: Dockerfile
      args:
        BUILD_ENV: production

    container_name: my-service

    ports:
      - "8000:8000"

    env_file:
      - .env

    environment:
      - DEBUG=False
      - LOG_LEVEL=INFO

    networks:
      - mynetwork

    restart: unless-stopped

    # VOLUMES: Mount directories
    volumes:
      - ./myservice:/app
      - db_data:/data/db

    # DEPENDENCIES: Wait for other services
    depends_on:
      - database

    # HEALTH: Container health monitoring
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

    # RESOURCES: CPU/Memory limits
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

volumes:
  db_data:

networks:
  mynetwork:
    driver: bridge
```

---

## Common Modifications

### Adding Health Checks

```yaml
services:
  fastapi-microservice:
    # ... existing config ...
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8101/health"]
      interval: 30s           # Check every 30 seconds
      timeout: 10s            # Wait 10s for response
      retries: 3              # Fail after 3 failed checks
      start_period: 40s       # Wait before first check
```

### Adding Dependencies

```yaml
services:
  fastapi-microservice:
    # ... existing config ...
    depends_on:
      django-microservice:
        condition: service_started  # Wait for Django to start
```

### Adding Volumes (Persistent Data)

```yaml
services:
  django-microservice:
    # ... existing config ...
    volumes:
      - ./test-django-microservice:/app           # Mount code
      - django_db:/data/db                        # Named volume
      - ./config.yaml:/app/config.yaml:ro         # Read-only file

volumes:
  django_db:                 # Define named volumes
```

### Adding Resource Limits

```yaml
services:
  fastapi-microservice:
    # ... existing config ...
    deploy:
      resources:
        limits:
          cpus: '0.5'        # Max 50% of 1 CPU
          memory: 512M       # Max 512MB RAM
        reservations:
          cpus: '0.25'       # Reserve 25% CPU
          memory: 256M       # Reserve 256MB RAM
```

### Adding Multiple Networks

```yaml
services:
  fastapi-microservice:
    networks:
      - microservices-network    # Can reach other microservices
      - frontend-network         # Can reach frontend containers
      - monitoring-network       # Can reach monitoring containers

networks:
  microservices-network:
    driver: bridge
  frontend-network:
    driver: bridge
  monitoring-network:
    driver: bridge
```

---

## Debugging Tips

### View Service Configuration

```bash
# See what Docker Compose will run
docker compose config

# See just one service
docker compose config --services
```

### Check Environment Variables

```bash
# What env vars does a service have?
docker compose exec fastapi-microservice env

# Check specific variable
docker compose exec fastapi-microservice echo $PYTHONUNBUFFERED
```

### Test Network Connectivity

```bash
# From fastapi, can you reach django?
docker compose exec fastapi-microservice ping django-microservice

# Curl from one service to another
docker compose exec fastapi-microservice curl http://django-microservice:8100/api/ping
```

### Inspect Actual Container

```bash
# What port is actually being used?
docker compose port fastapi-microservice 8101

# What network is it on?
docker inspect test-fastapi-microservice | grep -i network

# Full container details
docker compose exec fastapi-microservice whoami
```

---

## Summary Table

| Setting | Purpose | Example |
|---------|---------|---------|
| `build` | How to build image | `context: ./test-fastapi` |
| `container_name` | Container name | `test-fastapi-microservice` |
| `ports` | Port mapping | `"8101:8101"` |
| `env_file` | Load env from file | `.env` |
| `environment` | Set env variables | `PYTHONUNBUFFERED=1` |
| `networks` | Join network | `microservices-network` |
| `restart` | Auto-restart policy | `unless-stopped` |
| `volumes` | Mount directories | `./app:/app` |
| `depends_on` | Service dependencies | `database` |
| `healthcheck` | Health monitoring | HTTP checks |
| `deploy.resources` | Resource limits | CPU/Memory |