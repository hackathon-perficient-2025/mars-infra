# Infrastructure CI/CD Workflows

## 📋 Workflows

### `ci.yml` - Docker Compose Validation

Validates and tests the docker-compose configuration.

**Jobs:**

1. **Validate**

   - Checks docker-compose.yaml syntax
   - Validates service configurations
   - Ensures compose file is valid

2. **Test Build**
   - Builds all services (backend, frontend, MongoDB)
   - Starts the full stack
   - Waits for services to be healthy
   - Checks service status
   - Shows logs on failure
   - Cleans up containers and volumes

## 🚀 Usage

### Validate Locally

```bash
docker compose config
```

### Build & Start Services

```bash
docker compose up --build
```

### Stop Services

```bash
docker compose down -v
```

## 📦 Services

The docker-compose.yaml orchestrates:

- **MongoDB** - Database with authentication
- **Backend API** - Express.js server
- **Frontend** - React/Vite application

## ✅ Health Checks

All services include health checks:

- MongoDB: mongosh ping
- Backend: HTTP health endpoint
- Frontend: HTTP availability check

## 🔧 Environment

See `docker-compose.yaml` for service configurations and environment variables.
