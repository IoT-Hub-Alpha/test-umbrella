# How to Add a New Microservice to the Umbrella

This guide explains how to add a new microservice to the test-umbrella repository.

## Prerequisites

- A new GitHub repository for your microservice (e.g., `https://github.com/IoT-Hub-Alpha/test-new-microservice`)
- A `Dockerfile` in the root of your microservice repository
- Access to the test-umbrella repository
- Git and Docker installed locally

## Step-by-Step Guide

### Step 1: Clone the Umbrella Repository (if needed)

```bash
cd ~/SOFTSERVE/MICROSERVICES
git clone --recursive https://github.com/IoT-Hub-Alpha/test-umbrella.git
cd test-umbrella
```

### Step 2: Add the Microservice as a Git Submodule

Add your new microservice repository as a submodule to the umbrella:

```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella

# Add the new microservice
git submodule add https://github.com/IoT-Hub-Alpha/test-new-microservice test-new-microservice

# Initialize and update all submodules
git submodule update --init --recursive
```

**Replace `test-new-microservice` with your actual repository name and URL.**

### Step 3: Update docker-compose.yml

Edit `docker-compose.yml` and add your new service. Choose an available port number:

```yaml
services:
  fastapi-microservice:
    # ... existing FastAPI config ...

  django-microservice:
    # ... existing Django config ...

  new-microservice:
    build:
      context: ./test-new-microservice
      dockerfile: Dockerfile
    container_name: test-new-microservice
    ports:
      - "8102:8102"  # Choose an unused port
    env_file:
      - .env
    environment:
      - PYTHONUNBUFFERED=1
    networks:
      - microservices-network
    restart: unless-stopped
```

**Port Number Guidelines:**
- FastAPI: 8101
- Django: 8100
- New services: 8102, 8103, 8104, etc.

### Step 4: Update .env File

Add environment variables for your new service to enable inter-service communication:

```bash
# Microservices Configuration
FASTAPI_HOST=fastapi-microservice
DJANGO_HOST=django-microservice
NEW_HOST=new-microservice  # Add this line
```

Replace `NEW_HOST` and `new-microservice` with appropriate names for your service.

### Step 5: Update Your Microservice Code

In your microservice code, use environment variables to discover other services:

**For Python (FastAPI):**
```python
import os

# Get host from environment (defaults to localhost for standalone development)
fastapi_host = os.getenv("FASTAPI_HOST", "127.0.0.1")
django_host = os.getenv("DJANGO_HOST", "127.0.0.1")
new_host = os.getenv("NEW_HOST", "127.0.0.1")

# Use in requests
import httpx

async def call_fastapi():
    async with httpx.AsyncClient() as client:
        response = await client.get(f"http://{fastapi_host}:8101/api/ping")
        return response.json()
```

**For Python (Django):**
```python
import os
import requests

# Get host from environment
fastapi_host = os.getenv("FASTAPI_HOST", "127.0.0.1")

# Use in requests
response = requests.get(f"http://{fastapi_host}:8101/api/ping")
```

**For Node.js/Express:**
```javascript
const fastapi_host = process.env.FASTAPI_HOST || '127.0.0.1';

// Use in requests
const fetch = require('node-fetch');
fetch(`http://${fastapi_host}:8101/api/ping`)
  .then(res => res.json())
  .then(data => console.log(data));
```

### Step 6: Update README.md

Add documentation for the new service to the main README:

**Add to "## Services" section:**
```markdown
## Services

- **FastAPI Microservice**: Port 8101 (Uvicorn server)
- **Django Microservice**: Port 8100 (Django development server)
- **New Microservice**: Port 8102 (Your framework/language here)
```

**Add to "## Service Communication" section:**
```markdown
Both services are on the same Docker network (`microservices-network`). They can communicate using service names:
- FastAPI: `http://fastapi-microservice:8101`
- Django: `http://django-microservice:8100`
- New: `http://new-microservice:8102`
```

### Step 7: Verify the Dockerfile

Make sure your microservice has a `Dockerfile` in the root directory:

```dockerfile
FROM python:3.13-slim  # or your base image

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8102  # Your service port

CMD ["python", "app.py"]  # Your startup command
```

### Step 8: Commit Changes to Umbrella Repository

Commit all changes to the umbrella repository:

```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella

# Stage all changes
git add .gitmodules test-new-microservice docker-compose.yml .env README.md

# Commit with descriptive message
git commit -m "Add test-new-microservice to umbrella"

# Push to remote
git push origin main
```

### Step 9: Test the Setup

Verify everything works:

```bash
# Navigate to umbrella directory
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella

# Stop any running containers
docker compose down

# Build and start all services
docker compose up -d --build

# Check all services are running
docker compose ps

# View logs for all services
docker compose logs

# View logs for specific service
docker compose logs -f new-microservice

# Test the service
curl http://localhost:8102/  # Replace with your service endpoint
```

### Step 10: Document Your Service

Create a README in your microservice directory with:
- Service description
- API endpoints
- Environment variables
- How to run locally
- Dependencies

Example: `test-new-microservice/README.md`

```markdown
# Test New Microservice

Brief description of what this service does.

## API Endpoints

- `GET /health` - Health check
- `GET /api/status` - Service status

## Environment Variables

- `FASTAPI_HOST` - FastAPI service hostname (default: 127.0.0.1)
- `DJANGO_HOST` - Django service hostname (default: 127.0.0.1)

## Running Locally

```bash
pip install -r requirements.txt
python app.py
```

## Docker

```bash
docker build -t test-new-microservice .
docker run -p 8102:8102 test-new-microservice
```
```

## Port Allocation

Keep track of which ports are used:

| Service | Port | Framework |
|---------|------|-----------|
| FastAPI | 8101 | FastAPI (Python) |
| Django | 8100 | Django (Python) |
| [New Service] | 8102 | [Your Framework] |

Add new ports in order (8102, 8103, 8104, etc.)

## Troubleshooting

### Port Already in Use

If a port is already in use:
```bash
# Find which process is using the port
lsof -i :8102

# Kill the process
kill -9 <PID>
```

### Docker Build Fails

```bash
# Build with no cache
docker compose build --no-cache new-microservice

# Check logs
docker compose logs new-microservice
```

### Submodule Not Updating

```bash
# Sync submodules
git submodule sync

# Update to latest
git submodule update --init --recursive
```

### Service Can't Connect to Other Services

1. Verify the service name in environment variable matches docker-compose.yml
2. Check if all services are running: `docker compose ps`
3. Verify network: `docker network ls`
4. Check service logs: `docker compose logs <service-name>`

## Removing a Microservice

If you need to remove a service:

```bash
cd ~/SOFTSERVE/MICROSERVICES/test-umbrella

# Remove from docker-compose.yml (manual edit)

# Remove from .env file (manual edit)

# Remove the submodule
git submodule deinit -f test-new-microservice
git rm --cached test-new-microservice
rm -rf .git/modules/test-new-microservice

# Commit changes
git add .gitmodules docker-compose.yml .env
git commit -m "Remove test-new-microservice"
git push origin main
```

## Best Practices

1. **Service Naming**: Use consistent naming like `test-<framework>-microservice`
2. **Port Numbers**: Use sequential ports starting from 8100+
3. **Environment Variables**: Always support service discovery via env vars
4. **Documentation**: Keep README updated for each service
5. **Health Checks**: Add a `/health` endpoint for monitoring
6. **Error Handling**: Handle connection errors gracefully when calling other services
7. **Logging**: Use PYTHONUNBUFFERED=1 for real-time log visibility
8. **Networking**: All services use the same `microservices-network` bridge network

## Quick Reference

```bash
# Add submodule
git submodule add <repo-url> <directory-name>

# Clone with all submodules
git clone --recursive <umbrella-repo-url>

# Update submodules
git submodule update --remote

# Start all services
docker compose up -d --build

# Stop all services
docker compose down

# View logs
docker compose logs -f <service-name>

# Execute command in service
docker compose exec <service-name> bash
```