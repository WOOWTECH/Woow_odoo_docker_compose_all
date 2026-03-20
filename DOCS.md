# Woow Odoo 18 - Home Assistant Add-on

## Overview

This add-on provides a complete Odoo 18 ERP system running as a single Home Assistant add-on container with built-in PostgreSQL 16 database and Nginx reverse proxy.

## Architecture

```
┌─────────────────────────────────────────┐
│        Home Assistant Add-on            │
│  ┌───────────────────────────────────┐  │
│  │   Nginx (Port 8069)               │  │
│  │   ├── /websocket → Odoo:8072     │  │
│  │   ├── /longpolling → Odoo:8072   │  │
│  │   ├── /web/static/ → cached      │  │
│  │   └── / → Odoo:8069              │  │
│  ├───────────────────────────────────┤  │
│  │   Odoo 18 (Ports 8069 + 8072)    │  │
│  ├───────────────────────────────────┤  │
│  │   PostgreSQL 16 (localhost:5432)  │  │
│  └───────────────────────────────────┘  │
│  Data: /data/odoo, /data/postgres       │
└─────────────────────────────────────────┘
```

## Configuration

See the **Configuration** tab in the add-on settings for all available options.

## Persistence

All data is stored under `/data/`:
- `/data/postgres/` - PostgreSQL database files
- `/data/odoo/` - Odoo filestore, sessions
- `/data/logs/` - Log files

## External HTTPS

Use [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) for secure external access.
