# test-umbrella

Umbrella repository for running test-fastapi-microservice and test-django-microservice together using Docker Compose.

> **📚 Want to add a new microservice?** See [HOW_ADD_MICROSERVICE.md](./HOW_ADD_MICROSERVICE.md) for detailed instructions.

## Quick Start

### Clone with submodules
```bash
git clone --recursive https://github.com/IoT-Hub-Alpha/test-umbrella.git
cd test-umbrella
```

### Run with Docker Compose
```bash
docker-compose up --build
```

This will:
- Build both microservices from their Dockerfiles
- Start the FastAPI service on `http://localhost:8101`
- Start the Django service on `http://localhost:8100`
- Create a shared Docker network allowing both services to communicate

### Stop the services
```bash
docker-compose down
```

## Services

- **FastAPI Microservice**: Port 8101 (Uvicorn server)
- **Django Microservice**: Port 8100 (Django development server)

## Service Communication

Both services are on the same Docker network (`microservices-network`). They can communicate using service names:
- FastAPI: `http://fastapi-microservice:8101`
- Django: `http://django-microservice:8100`

### Inter-service Communication

The Django microservice communicates with the FastAPI microservice using an environment variable:

- **In Docker Compose**: Uses `FASTAPI_HOST=fastapi-microservice` from `.env` file
- **Running locally**: Uses default `127.0.0.1` (localhost)

You can override the `FASTAPI_HOST` in the `.env` file if needed.

## Git Workflow with Submodules

### Understanding the Repository Structure

This is an **umbrella repository** with two git submodules:
- `test-fastapi-microservice/` - FastAPI microservice repository
- `test-django-microservice/` - Django microservice repository

Each is a **separate git repository** with its own:
- `.git` folder
- branches
- commits
- remote tracking

Git commands only affect the repository in your **current directory**.

### Switching Between Repositories

Simply change directories:

```bash
# Go to umbrella repo
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella
git status  # Shows umbrella repo status

# Go to FastAPI microservice
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-fastapi-microservice
git status  # Shows FastAPI repo status

# Go to Django microservice
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-django-microservice
git status  # Shows Django repo status
```

### Pulling Changes

**Pull from umbrella repo:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella
git pull origin main
```

**Pull from FastAPI microservice:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-fastapi-microservice
git pull origin main
```

**Pull from Django microservice:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-django-microservice
git pull origin main
```

**Pull latest submodule commits into umbrella:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella
git submodule update --remote
```

### Pushing Changes

**Push from FastAPI microservice:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-fastapi-microservice
git add .
git commit -m "Your commit message"
git push origin main
```

**Push from Django microservice:**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella/test-django-microservice
git add .
git commit -m "Your commit message"
git push origin main
```

**Push umbrella changes (after updating submodule references):**
```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella
git add .
git commit -m "Update submodule references"
git push origin main
```

### Committing Submodule Changes

When you update a microservice and push it, the umbrella repo needs to be updated with the new submodule commit reference:

```bash
# After pushing FastAPI changes
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella
git add test-fastapi-microservice
git commit -m "Update FastAPI microservice to latest commit"
git push origin main
```

## Docker Compose Commands

### Start Services
```bash
# Build and start all services
docker compose up --build

# Start in background
docker compose up -d --build

# Start without rebuilding
docker compose up -d
```

### View Logs
```bash
# View logs from all services
docker compose logs

# Follow logs (real-time)
docker compose logs -f

# Follow logs from specific service
docker compose logs -f fastapi-microservice
docker compose logs -f django-microservice
```

### Stop Services
```bash
# Stop all services
docker compose down

# Stop and remove volumes (clean up data)
docker compose down -v
```

### Service Status
```bash
# Check running services
docker compose ps

# Check service details
docker compose ps -a
```

### Execute Commands in Container
```bash
# Run command in FastAPI service
docker compose exec fastapi-microservice bash

# Run command in Django service
docker compose exec django-microservice bash

# Run Django migrations example
docker compose exec django-microservice python manage.py migrate
```

### Rebuild Services
```bash
# Rebuild all services
docker compose build

# Rebuild specific service
docker compose build fastapi-microservice

# Rebuild without cache
docker compose build --no-cache
```

## Configuration

The `.env` file controls inter-service communication settings:
```bash
# Microservices Configuration
FASTAPI_HOST=fastapi-microservice  # Docker service name
DJANGO_HOST=django-microservice    # Docker service name
```

For local development (outside Docker), change these to:
```bash
FASTAPI_HOST=127.0.0.1
DJANGO_HOST=127.0.0.1
```

## Typical Workflow

### Development Cycle
```bash
# 1. Clone with submodules
git clone --recursive https://github.com/IoT-Hub-Alpha/test-umbrella.git
cd test-umbrella

# 2. Start services
docker compose up -d --build

# 3. Make changes in a microservice
cd test-fastapi-microservice
# ... edit files ...

# 4. Rebuild and test
docker compose down
docker compose up -d --build
docker compose logs -f fastapi-microservice

# 5. Commit and push microservice changes
git add .
git commit -m "Update feature X"
git push origin main

# 6. Update umbrella submodule reference
cd ..
git add test-fastapi-microservice
git commit -m "Update FastAPI to latest"
git push origin main

# 7. Stop services when done
docker compose down
```

## Troubleshooting

**Detached HEAD state in submodule:**
```bash
cd test-fastapi-microservice
git checkout main
git pull origin main
```

**Submodule not synced:**
```bash
git submodule sync
git submodule update --init --recursive
```

**Force rebuild services:**
```bash
docker compose down -v
docker compose build --no-cache
docker compose up -d
```

## Documentation

This repository includes comprehensive guides:

- **[README.md](./README.md)** (this file)
  - Overview and quick start
  - Git workflows with submodules
  - Docker Compose commands
  - Configuration and typical workflow

- **[DOCKER_COMPOSE_EXPLAINED.md](./DOCKER_COMPOSE_EXPLAINED.md)** - Detailed reference for every docker-compose.yml setting
  - Services configuration (build, ports, networks, environment)
  - Network configuration and service discovery
  - Common modifications and examples
  - Debugging tips
  - Complete reference guide

- **[HOW_ADD_MICROSERVICE.md](./HOW_ADD_MICROSERVICE.md)** - Step-by-step guide to add a new microservice
  - Adding git submodules
  - Docker Compose configuration
  - Environment variable setup
  - Code examples for different frameworks
  - Port allocation
  - Troubleshooting

### Quick Documentation Links

| Question | Document |
|----------|----------|
| How do I get started? | [README.md](./README.md) - Quick Start section |
| What does each docker-compose setting do? | [DOCKER_COMPOSE_EXPLAINED.md](./DOCKER_COMPOSE_EXPLAINED.md) |
| How do I add a new microservice? | [HOW_ADD_MICROSERVICE.md](./HOW_ADD_MICROSERVICE.md) |
| How do I push/pull changes? | [README.md](./README.md) - Git Workflow section |
| How do I run the services? | [README.md](./README.md) - Docker Compose Commands section |

For detailed information, refer to the appropriate guide above. 
