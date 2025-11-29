# Infrastructure CI/CD Workflows

## Workflows

### `ci.yml` - Docker Compose Validation

Validates the docker-compose configuration without building (since frontend and backend are in separate repositories).

**Jobs:**

1. **Validate**

   - Checks docker-compose.yaml syntax
   - Validates service configurations
   - Ensures compose file is valid
   - Checks for deprecated syntax warnings

2. **Lint**
   - Verifies required files exist (README, docker-compose.yaml)
   - Checks for documentation files
   - Ensures repository structure is correct

**Note:** This workflow does NOT build the services since `mars-backend` and `mars-frontend` are in separate repositories. Use this for configuration validation only.

## Usage

### Validate Locally

```bash
docker compose config
```

### Check for Warnings

```bash
docker compose config 2>&1 | grep -i "warning"
```

## Important Note

This repository contains only the infrastructure configuration (docker-compose.yaml).

**To run the full stack:**

1. Clone all three repositories side by side:
   ```
   parent-folder/
   ├── mars-backend/
   ├── mars-frontend/
   └── mars-infra/
   ```
2. From `mars-infra`, run: `docker compose up`

The docker-compose.yaml references `../mars-backend` and `../mars-frontend` for builds.

### Build & Start Services (Local Development)

```bash
# Ensure you have all three repos cloned side by side
docker compose up --build
```

### Stop Services

```bash
docker compose down -v
```

## Services

The docker-compose.yaml orchestrates:

- **MongoDB** - Database with authentication (built-in image)
- **Backend API** - Express.js server (references ../mars-backend)
- **Frontend** - React/Vite application (references ../mars-frontend)

## CI/CD Validation Only

The GitHub Actions workflow validates the docker-compose configuration syntax but does NOT build the services since this repository doesn't contain the source code.

**What the CI validates:**

- YAML syntax is correct
- Service definitions are valid
- No deprecated syntax
- Required files are present

**What the CI does NOT do:**

- Build Docker images (requires separate repositories)
- Start services (not possible in isolated repo)
- Run integration tests (handled by individual repos)

## Local Testing

For full integration testing, ensure all repos are cloned:

All services include health checks:

- MongoDB: mongosh ping
- Backend: HTTP health endpoint
- Frontend: HTTP availability check

## Environment

See `docker-compose.yaml` for service configurations and environment variables.
