# Installation on Linux Using Docker

Request Example on [Treblle](https://platform.treblle.com/r?expires=1743002478&signature=ca5313796a31bab217dba4933ca[…]92q3yjfyrtn4cqv9egcra&requestId=01jq9d6f6vg9jbk9y2wgqm143z)

### Project Structure

```
kong-treblle-sandbox/
│
├── docker-compose.yml
├── kong.conf
├── Dockerfile
└── kong.yml
```

### Prerequisites

- Docker Desktop installed on your Mac
- Git installed
- A Treblle account with an API key and Project ID

## Configuration Files

### 1. Dockerfile

```dockerfile
FROM kong:3.4.0

USER root

# Install dependencies
RUN apk add --no-cache \
    git \
    unzip \
    curl \
    luarocks

# Install the Treblle plugin
RUN luarocks install --server=http://luarocks.org/manifests/treblle kong-plugin-treblle

# Copy Kong configuration
COPY kong.conf /etc/kong/kong.conf

USER kong
```

### 2. Kong Configuration (kong.conf)

```ini
database = on
pg_host = postgres
pg_port = 5432
pg_user = kong
pg_password = kong
pg_database = kong
plugins = bundled,treblle
```

### 3. Docker Compose (docker-compose.yml)

```yaml
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong
      POSTGRES_USER: kong
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 30s
      timeout: 30s
      retries: 3

  kong-migration:
    image: kong:3.4.0
    command: kong migrations bootstrap
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: postgres
      KONG_PG_PASSWORD: kong
      KONG_PG_USER: kong
    depends_on:
      postgres:
        condition: service_healthy

  kong:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: kong-treblle
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: postgres
      KONG_PG_PASSWORD: kong
      KONG_PG_USER: kong
      KONG_PLUGINS: bundled,treblle
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001, 0.0.0.0:8444 ssl
    ports:
      - "9000:8000"   # Proxy port
      - "9001:8001"   # Admin API port
      - "9443:8443"
      - "9444:8444"
    depends_on:
      postgres:
        condition: service_healthy
      kong-migration:
        condition: service_completed_successfully
    healthcheck:
      test: ["CMD", "kong", "health"]
      interval: 10s
      timeout: 10s
      retries: 10

volumes:
  postgres_data:
```

### 4. Declarative Configuration (kong.yml)

```yaml
_format_version: "3.0"

services:
 - name: test-service
   url: https://httpbin.org
   routes:
     - name: test-route
       paths:
         - /test
```

## Installation Steps

### 1. Prepare Project Directory

```bash
mkdir kong-treblle-sandbox
cd kong-treblle-sandbox
```

### 2. Initialize Docker Containers

```bash
docker-compose up -d
```

### 3. Create a Service in Kong

```bash
curl -i -X POST http://localhost:9001/services \
  --data name=httpbin-service \
  --data url=https://httpbin.org
```

### 4. Create Route

```bash
curl -i -X POST http://localhost:9001/services/httpbin-service/routes \
  --data "name=httpbin-route" \
  --data "paths[]=/test" \
  --data "methods[]=POST"
```

### 5. Enable Treblle Plugin

```bash
curl -i -X POST http://localhost:9001/services/httpbin-service/plugins \
  --data "name=treblle" \
  --data "config.api_key=YOUR_TREBLLE_API_KEY" \
  --data "config.project_id=YOUR_TREBLLE_PROJECT_ID" \
  --data "config.mask_keywords[]=Authorization" \
  --data "config.mask_keywords[]=API_Key" \
  --data "config.mask_keywords[]=Secure-Token"
```

## Troubleshooting

### Verification Commands

```bash
# Check Docker containers
docker-compose ps

# View Kong logs
docker-compose logs kong

# Test route
curl -X POST http://localhost:9000/test/post
```

## Cleanup

```bash
# Stop and remove containers
docker-compose down

# Remove volumes (optional)
docker-compose down -v
```
