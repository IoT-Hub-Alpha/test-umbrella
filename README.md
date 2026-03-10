# test-umbrella

Umbrella repository for running test-fastapi-microservice and test-django-microservice together using Docker Compose.

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

## Updating submodules

To pull the latest changes from both microservice repositories:
```bash
git submodule update --remote
``` 
