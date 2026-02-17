# Odoo 18 Docker Compose Production Setup Design

## Overview

Production-ready Docker Compose setup for Odoo 18 Community Edition with PostgreSQL 16 and pgvector extension for future AI capabilities.

## Requirements

| Requirement | Decision |
|-------------|----------|
| Odoo Version | 18 Community Edition |
| Database | PostgreSQL 16 |
| pgvector | Enabled for future AI/embedding use |
| Port | 18069 |
| Data Persistence | Docker volumes (database) + Bind mounts (addons/config) |
| Setup Complexity | Basic (minimal config) |

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Host Machine                      │
│                                                      │
│  ┌─────────────┐         ┌─────────────────────┐    │
│  │   Odoo 18   │────────▶│  PostgreSQL 16      │    │
│  │  :18069     │         │  + pgvector         │    │
│  └─────────────┘         └─────────────────────┘    │
│        │                          │                  │
│        ▼                          ▼                  │
│  ┌─────────────┐         ┌─────────────────────┐    │
│  │ ./addons    │         │ odoo-db-data volume │    │
│  │ ./config    │         └─────────────────────┘    │
│  └─────────────┘                                    │
└─────────────────────────────────────────────────────┘
```

## Components

### 1. PostgreSQL 16 with pgvector

- Custom Dockerfile extending `postgres:16`
- pgvector extension pre-installed
- Data persisted in Docker volume `odoo-db-data`

### 2. Odoo 18 Community

- Official `odoo:18` image
- Port mapped to `18069`
- Custom addons via bind mount `./addons`
- Configuration via bind mount `./config`
- Odoo web data in volume `odoo-web-data`

## File Structure

```
.
├── docker-compose.yml      # Main compose file
├── .env                    # Environment variables
├── .env.example            # Example env file for documentation
├── postgres/
│   └── Dockerfile          # PostgreSQL 16 + pgvector
├── addons/                 # Custom Odoo modules
│   └── .gitkeep
├── config/
│   └── odoo.conf           # Odoo configuration
└── README.md               # Bilingual deployment guide
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| POSTGRES_USER | Database username | odoo |
| POSTGRES_PASSWORD | Database password | (required) |
| POSTGRES_DB | Database name | postgres |
| ODOO_PORT | Exposed Odoo port | 18069 |

## Security Considerations

- PostgreSQL not exposed externally (internal network only)
- Strong password required via `.env` file
- `.env` file excluded from git via `.gitignore`

## Deployment Steps

1. Clone repository
2. Copy `.env.example` to `.env` and set password
3. Run `docker compose up -d`
4. Access Odoo at `http://localhost:18069`

---

**Status:** Approved for implementation
**Date:** 2026-02-17
