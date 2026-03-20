# Odoo 18 — Docker / Podman Deployment

Production-ready Docker Compose setup for **Odoo 18 Community Edition** with **PostgreSQL 16** and **pgvector** extension.

## Quick Start

```bash
git clone -b podman https://github.com/WOOWTECH/Woow_odoo_docker_compose_all.git
cd Woow_odoo_docker_compose_all
cp .env.example .env
# Edit .env with your settings
docker compose up -d
```

## Files

| File | Description |
|------|-------------|
| `docker-compose.yml` | Service definitions |
| `.env.example` | Environment variable template |
| `config/` | Odoo configuration files |
| `addons/` | Custom Odoo addons |
| `postgres/` | PostgreSQL data |
| `docs/` | Additional documentation |

## Other Deployment Options

| Platform | Branch | Description |
|----------|--------|-------------|
| **Docker / Podman** | [`podman`](../../tree/podman) | ← You are here |
| Kubernetes (K3s) | [`k3s`](../../tree/k3s) | K8s manifests with Kustomize |
| Home Assistant | [`ha`](../../tree/ha) | HA add-on with one-click install |
