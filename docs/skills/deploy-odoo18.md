# Skill: Deploy Odoo 18 Docker Compose

## Metadata

- **Name:** deploy-odoo18
- **Version:** 1.0.0
- **Description:** Deploy Odoo 18 Community Edition with PostgreSQL 16 + pgvector via Docker Compose
- **Trigger:** User asks to deploy Odoo 18, set up Odoo, or create Odoo Docker environment
- **Repository:** https://github.com/WOOWTECH/Woow_odoo_docker_compose_all

---

## Prerequisites Check

Before deploying, verify:

```bash
# Check Docker/Podman is available
docker --version || podman --version

# Check Docker Compose is available
docker compose version || podman-compose --version

# Check available RAM (need 4GB+)
free -h

# Check available disk (need 10GB+)
df -h .
```

---

## Required Files

The following files MUST exist in the project root. If any are missing, clone the repo first:

```bash
git clone https://github.com/WOOWTECH/Woow_odoo_docker_compose_all.git
cd Woow_odoo_docker_compose_all
```

### File Checklist

| File | Purpose | Required |
|------|---------|----------|
| `docker-compose.yml` | Service orchestration (Odoo 18 + PostgreSQL 16) | YES |
| `postgres/Dockerfile` | Custom PostgreSQL 16 image with pgvector v0.7.4 | YES |
| `.env` | Environment variables (passwords, ports) | YES |
| `.env.example` | Template for .env | YES (for reference) |
| `config/odoo.conf` | Odoo server configuration | YES |
| `addons/.gitkeep` | Custom modules directory | YES (directory) |
| `.gitignore` | Excludes .env from git | YES |

---

## File Contents Reference

### docker-compose.yml

```yaml
version: '3.8'
name: odoo18

services:
  db:
    build:
      context: ./postgres
      dockerfile: Dockerfile
    container_name: odoo18-db
    environment:
      - POSTGRES_USER=${POSTGRES_USER:-odoo}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:?POSTGRES_PASSWORD is required}
      - POSTGRES_DB=${POSTGRES_DB:-postgres}
    volumes:
      - odoo-db-data:/var/lib/postgresql/data
    networks:
      - odoo-network
    restart: unless-stopped

  web:
    image: odoo:18
    container_name: odoo18-web
    depends_on:
      - db
    ports:
      - "${ODOO_PORT:-18069}:8069"
    environment:
      - HOST=db
      - PORT=5432
      - USER=${POSTGRES_USER:-odoo}
      - PASSWORD=${POSTGRES_PASSWORD:?POSTGRES_PASSWORD is required}
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./addons:/mnt/extra-addons
      - ./config:/etc/odoo
    networks:
      - odoo-network
    restart: unless-stopped

volumes:
  odoo-db-data:
    name: odoo18-db-data
  odoo-web-data:
    name: odoo18-web-data

networks:
  odoo-network:
    name: odoo18-network
    driver: bridge
```

### postgres/Dockerfile

```dockerfile
FROM postgres:16

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    postgresql-server-dev-16 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN git clone --branch v0.7.4 https://github.com/pgvector/pgvector.git /tmp/pgvector \
    && cd /tmp/pgvector \
    && make \
    && make install \
    && rm -rf /tmp/pgvector

RUN apt-get update && apt-get remove -y \
    build-essential \
    git \
    postgresql-server-dev-16 \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*
```

### .env

```bash
POSTGRES_USER=odoo
POSTGRES_PASSWORD=<SET_A_SECURE_PASSWORD>
POSTGRES_DB=postgres
ODOO_PORT=18069
```

### config/odoo.conf

```ini
[options]
addons_path = /mnt/extra-addons
admin_passwd = admin
log_level = info
workers = 0
max_cron_threads = 1
limit_memory_hard = 2684354560
limit_memory_soft = 2147483648
limit_time_cpu = 600
limit_time_real = 1200
proxy_mode = False
```

---

## Deployment Steps

### Step 1: Configure Environment

```bash
cp .env.example .env
# Set a secure password - DO NOT use default in production
```

### Step 2: Start Services

```bash
# Docker Compose
docker compose up -d

# OR Podman Compose
podman-compose up -d
```

### Step 3: Verify Deployment

```bash
# Check containers are running
docker compose ps
# Expected: odoo18-web (Up), odoo18-db (Up)

# Test HTTP access
curl -I http://localhost:18069
# Expected: HTTP/1.1 303 SEE OTHER, Location: /odoo

# Verify pgvector is available
docker exec odoo18-db psql -U odoo -d postgres \
  -c "SELECT * FROM pg_available_extensions WHERE name = 'vector';"
# Expected: vector | 0.7.4
```

### Step 4: Access Odoo

- URL: `http://localhost:18069`
- First time: Create a new database via the database manager
- Master password: `admin` (from odoo.conf, change for production)

---

## Common Operations

### Enable pgvector on a database

```bash
docker exec odoo18-db psql -U odoo -d <DATABASE_NAME> \
  -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### Stop services

```bash
docker compose down
```

### View logs

```bash
docker compose logs -f        # All services
docker compose logs -f web    # Odoo only
docker compose logs -f db     # PostgreSQL only
```

### Backup database

```bash
docker exec odoo18-db pg_dump -U odoo <DATABASE_NAME> > backup_$(date +%Y%m%d).sql
```

### Restore database

```bash
docker exec -i odoo18-db psql -U odoo <DATABASE_NAME> < backup.sql
```

### Restart Odoo only

```bash
docker compose restart web
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Odoo can't connect to DB | `docker compose restart web` (DB may not be ready yet) |
| Permission error on addons | `sudo chown -R 101:101 ./addons` |
| Port 18069 already in use | Change `ODOO_PORT` in `.env` |
| pgvector not found | Rebuild DB image: `docker compose build db` |
| Container keeps restarting | Check logs: `docker compose logs db` or `docker compose logs web` |

---

## Architecture Summary

```
Host:18069 → odoo18-web (odoo:18) → odoo18-db (postgres:16+pgvector)
                │                         │
                ├─ /var/lib/odoo          ├─ /var/lib/postgresql/data
                │  (odoo18-web-data vol)  │  (odoo18-db-data vol)
                ├─ /mnt/extra-addons      │
                │  (./addons bind mount)  │
                └─ /etc/odoo              │
                   (./config bind mount)  │
                                          │
            Network: odoo18-network (bridge)
```

---

## Version History

| Date | Version | Change |
|------|---------|--------|
| 2026-02-17 | 1.0.0 | Initial deployment - Odoo 18 + PostgreSQL 16 + pgvector 0.7.4 |
