# Infrastructure

VPS infrastructure setup for hosting applications with Traefik reverse proxy and private Docker registry.

## Services

- **Traefik** - Reverse proxy with automatic SSL/HTTPS via Let's Encrypt
- **Docker Registry** - Private container registry with basic authentication

## Prerequisites

- VPS with Docker and Docker Compose installed
- Domain names pointing to your VPS IP:
  - `registry.noelvega.dev` → Registry

## Setup

1. **Clone this repository on your VPS:**
   ```bash
   git clone <infrastructure-repo-url>
   cd infrastructure
   ```

2. **Create environment file:**
   ```bash
   cp .env.example .env
   # Edit .env with your actual values
   nano .env
   ```

3. **Start infrastructure services:**
   ```bash
   docker-compose up -d
   ```

4. **Verify services are running:**
   ```bash
   docker-compose ps
   docker-compose logs -f
   ```

## Using the Registry

### Login to Registry

From your local machine or CI/CD:

```bash
docker login registry.noelvega.dev
# Enter username and password from .env file
```

### Push Images

```bash
# Tag your image
docker tag my-app:latest registry.noelvega.dev/my-app:latest

# Push to registry
docker push registry.noelvega.dev/my-app:latest
```

### Pull Images

```bash
docker pull registry.noelvega.dev/my-app:latest
```

## Deploying Applications

Applications can connect to the Traefik network to get automatic SSL:

```yaml
services:
  my-app:
    image: my-app:latest
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.my-app.rule=Host(`my-app.noelvega.dev`)"
      - "traefik.http.routers.my-app.entrypoints=websecure"
      - "traefik.http.routers.my-app.tls.certresolver=letsencrypt"

networks:
  traefik:
    external: true
    name: traefik
```

## Maintenance

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f traefik
docker-compose logs -f registry
```

### Restart Services

```bash
docker-compose restart
```

### Update Services

```bash
docker-compose pull
docker-compose up -d
```

### Stop Services

```bash
docker-compose down
```

## Security Notes

- Registry uses basic auth (username/password)
- All traffic is encrypted via HTTPS
- Docker socket is mounted read-only
- Credentials stored in `.env` (never commit this file!)
