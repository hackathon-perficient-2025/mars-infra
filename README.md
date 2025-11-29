# Mars Base Cargo Control - Infrastructure

Docker Compose configuration for the complete Mars Base Cargo Control system.

## Architecture

This repository contains the infrastructure setup for running the entire Mars Base Cargo Control application stack using Docker Compose.

### Services

1. **MongoDB** (Port 27017)

   - Database: `mars_cargo`
   - User: `marsadmin`
   - Password: `marspassword123`
   - Persistent volumes for data and configuration
   - Health checks enabled

2. **Backend API** (Port 3000)

   - Node.js/TypeScript Express application
   - Real-time updates via Socket.IO
   - REST API with Swagger documentation
   - Connects to MongoDB for data persistence

3. **Frontend** (Port 8080)
   - React application served via Nginx
   - Built with Vite and TypeScript
   - Mars-themed UI with ShadcnUI components

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 2GB of free RAM
- Ports 27017, 3000, and 8080 available

## Quick Start

### 1. Start All Services

```bash
docker-compose up -d
```

This will:

- Pull the MongoDB image
- Build the backend Docker image
- Build the frontend Docker image
- Create a Docker network for service communication
- Start all services with health checks

### 2. Check Service Status

```bash
docker-compose ps
```

All services should show as "healthy" after initialization.

### 3. Access the Application

- **Frontend UI**: http://localhost:8080
- **Backend API**: http://localhost:3000/api
- **API Documentation**: http://localhost:3000/api-docs
- **MongoDB**: mongodb://marsadmin:marspassword123@localhost:27017/mars_cargo?authSource=admin

### 4. View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mongodb
```

### 5. Stop All Services

```bash
docker-compose down
```

### 6. Stop and Remove All Data

```bash
docker-compose down -v
```

**Warning**: This will delete all MongoDB data!

## Development Workflow

### Rebuild Services After Code Changes

```bash
# Rebuild and restart backend
docker-compose up -d --build backend

# Rebuild and restart frontend
docker-compose up -d --build frontend

# Rebuild all services
docker-compose up -d --build
```

### Execute Commands in Containers

```bash
# Access backend shell
docker-compose exec backend sh

# Access MongoDB shell
docker-compose exec mongodb mongosh -u marsadmin -p marspassword123 --authenticationDatabase admin mars_cargo

# View backend environment variables
docker-compose exec backend env
```

## Database Management

### Backup MongoDB Data

```bash
docker-compose exec mongodb mongodump \
  --username=marsadmin \
  --password=marspassword123 \
  --authenticationDatabase=admin \
  --db=mars_cargo \
  --out=/data/backup
```

### Restore MongoDB Data

```bash
docker-compose exec mongodb mongorestore \
  --username=marsadmin \
  --password=marspassword123 \
  --authenticationDatabase=admin \
  --db=mars_cargo \
  /data/backup/mars_cargo
```

### Connect to MongoDB from Host

```bash
mongosh "mongodb://marsadmin:marspassword123@localhost:27017/mars_cargo?authSource=admin"
```

## Environment Variables

### Backend Environment Variables

You can customize the backend by modifying the `docker-compose.yaml` file:

```yaml
environment:
  NODE_ENV: production
  PORT: 3000
  MONGODB_URI: mongodb://marsadmin:marspassword123@mongodb:27017/mars_cargo?authSource=admin
  CORS_ORIGIN: http://localhost:8080
  RESOURCE_UPDATE_INTERVAL: 5000 # 5 seconds
  ALERT_CHECK_INTERVAL: 10000 # 10 seconds
```

## Networking

All services communicate through a dedicated Docker network (`mars-network`):

- Services can reference each other by service name
- Backend connects to MongoDB using `mongodb:27017`
- Frontend makes API calls to the backend (configured in build-time environment variables)

## Volumes

### Persistent Data

```bash
# List volumes
docker volume ls | grep mars

# Inspect volume
docker volume inspect mars-mongodb-data

# Remove volumes (WARNING: Data loss!)
docker volume rm mars-mongodb-data mars-mongodb-config
```

### Volume Locations

- `mars-mongodb-data`: MongoDB database files
- `mars-mongodb-config`: MongoDB configuration files

## Troubleshooting

### Service Won't Start

```bash
# Check logs for errors
docker-compose logs backend
docker-compose logs mongodb

# Check service health
docker-compose ps
```

### Port Already in Use

```bash
# Find process using port 3000
lsof -i :3000

# Kill the process or change the port in docker-compose.yaml
```

### MongoDB Connection Issues

```bash
# Verify MongoDB is healthy
docker-compose exec mongodb mongosh -u marsadmin -p marspassword123 --authenticationDatabase admin --eval "db.adminCommand('ping')"

# Check MongoDB logs
docker-compose logs mongodb
```

### Frontend Can't Connect to Backend

- Ensure CORS_ORIGIN in backend matches frontend URL
- Check that backend service is healthy
- Verify network connectivity: `docker network inspect mars-network`

## Production Deployment

### Security Considerations

For production deployment:

1. **Change MongoDB credentials** in docker-compose.yaml
2. **Use environment files** instead of hardcoded values
3. **Enable SSL/TLS** for MongoDB connections
4. **Add authentication** to the backend API
5. **Use secrets management** (Docker Secrets, Vault, etc.)
6. **Set up reverse proxy** (Nginx, Traefik) for HTTPS
7. **Configure firewall rules** to restrict access

### Example Production Setup

```yaml
# docker-compose.prod.yaml
services:
  mongodb:
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
  backend:
    environment:
      MONGODB_URI: ${MONGODB_URI}
      CORS_ORIGIN: ${FRONTEND_URL}
```

Create a `.env` file:

```env
MONGO_USER=production_user
MONGO_PASSWORD=strong_random_password
MONGODB_URI=mongodb://production_user:strong_random_password@mongodb:27017/mars_cargo?authSource=admin
FRONTEND_URL=https://mars.yourdomain.com
```

Run with:

```bash
docker-compose -f docker-compose.prod.yaml up -d
```

## Monitoring

### Health Checks

All services have health checks configured:

```bash
# Check health status
docker-compose ps

# View health check logs
docker inspect mars-backend --format='{{json .State.Health}}' | jq
```

### Resource Usage

```bash
# View resource consumption
docker stats mars-backend mars-frontend mars-mongodb

# Limit resource usage (add to docker-compose.yaml)
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
```

## CI/CD

### GitHub Actions

This repository includes a CI/CD workflow that validates the docker-compose configuration:

- **Validates YAML syntax** - Ensures docker-compose.yaml is valid
- **Checks for warnings** - Detects deprecated syntax
- **Verifies files** - Ensures required documentation exists

**Note:** The workflow does NOT build Docker images since the backend and frontend source code is in separate repositories (`mars-backend` and `mars-frontend`).

See `.github/workflows/README.md` for details.

### Deployment

For deployment guides, see `DEPLOYMENT.md`.

## License

Copyright PERFICIENT - All rights reserved

## Support

For issues and questions:

- Check logs: `docker-compose logs -f`
- Verify health: `docker-compose ps`
- Review MongoDB status: `docker-compose exec mongodb mongosh --eval "db.adminCommand('ping')"`
