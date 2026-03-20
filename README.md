# Woow Odoo 18 — Home Assistant Add-on

Odoo 18 ERP with built-in PostgreSQL 16 and Nginx reverse proxy, packaged as a Home Assistant add-on.

## Installation

### One-Click Install

[![Open your Home Assistant instance and show the dashboard of an add-on.](https://my.home-assistant.io/badges/supervisor_addon.svg)](https://my.home-assistant.io/redirect/supervisor_addon/?addon=woow-odoo&repository_url=https%3A%2F%2Fgithub.com%2FWOOWTECH%2FWoow_odoo_docker_compose_all)

### Manual Installation

1. Add this repository to your Home Assistant:

   [![Add repository to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FWOOWTECH%2FWoow_odoo_docker_compose_all)

2. Navigate to **Settings → Add-ons → Add-on Store**
3. Find **"Woow Odoo 18"** and click **INSTALL**
4. Configure the add-on options
5. Start the add-on

## Configuration

| Option | Description | Default |
|--------|-------------|---------|
| `TZ` | Timezone | `Asia/Taipei` |
| `db_password` | PostgreSQL password | `odoo` |
| `odoo_workers` | Number of workers | `2` |
| `odoo_log_level` | Log level | `info` |
| `odoo_proxy_mode` | Enable proxy mode | `true` |
| `odoo_db_name` | Database name | `odoo` |

## Ports

| Port | Description |
|------|-------------|
| 8069 | Odoo Web Interface |
| 8072 | Odoo WebSocket/Longpolling |
| 5432 | PostgreSQL Database |

## Other Deployment Options

| Platform | Branch | Description |
|----------|--------|-------------|
| Docker / Podman | [`podman`](../../tree/podman) | Docker Compose deployment |
| Kubernetes (K3s) | [`k3s`](../../tree/k3s) | K8s manifests with Kustomize |
| **Home Assistant** | [`ha`](../../tree/ha) | ← You are here |
