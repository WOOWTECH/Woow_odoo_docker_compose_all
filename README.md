# Woow Odoo 18

Production-ready **Odoo 18 Community Edition** with **PostgreSQL 16** and **pgvector** extension for AI capabilities, packaged for multiple deployment platforms.

## Deployment Options

| Platform | Branch | Description |
|----------|--------|-------------|
| Docker / Podman | [`podman`](../../tree/podman) | Docker Compose deployment with PostgreSQL 16 |
| Kubernetes (K3s) | [`k3s`](../../tree/k3s) | K8s manifests with Kustomize |
| Home Assistant | [`ha`](../../tree/ha) | HA add-on with one-click install |

## Quick Start

### Docker / Podman

```bash
git clone -b podman https://github.com/WOOWTECH/Woow_odoo_docker_compose_all.git
cd Woow_odoo_docker_compose_all
cp .env.example .env
docker compose up -d
```

### Kubernetes (K3s)

```bash
git clone -b k3s https://github.com/WOOWTECH/Woow_odoo_docker_compose_all.git
cd Woow_odoo_docker_compose_all
kubectl apply -k .
```

### Home Assistant

[![Open your Home Assistant instance and show the dashboard of an add-on.](https://my.home-assistant.io/badges/supervisor_addon.svg)](https://my.home-assistant.io/redirect/supervisor_addon/?addon=woow-odoo&repository_url=https%3A%2F%2Fgithub.com%2FWOOWTECH%2FWoow_odoo_docker_compose_all)

## Features

- Odoo 18 Community Edition
- PostgreSQL 16 with pgvector (AI/embeddings)
- Nginx reverse proxy
- s6-overlay process supervision
- Multi-architecture support (amd64, aarch64)

## License

See individual branch documentation for details.
