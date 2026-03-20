# Odoo 18 — Kubernetes (K3s) Deployment

Kubernetes manifests for deploying **Odoo 18** with **PostgreSQL 16** on K3s.

## Quick Start

```bash
git clone -b k3s https://github.com/WOOWTECH/Woow_odoo_docker_compose_all.git
cd Woow_odoo_docker_compose_all
kubectl apply -k .
```

## Manifests

| File | Description |
|------|-------------|
| `namespace.yaml` | Namespace definition |
| `secret.yaml` | Secrets (credentials) |
| `configmap.yaml` | Configuration data |
| `pvc.yaml` | Persistent Volume Claims |
| `postgres-statefulset.yaml` | PostgreSQL StatefulSet |
| `postgres-service.yaml` | PostgreSQL Service |
| `odoo-deployment.yaml` | Odoo Deployment |
| `odoo-service.yaml` | Odoo Service |
| `kustomization.yaml` | Kustomize configuration |

## Other Deployment Options

| Platform | Branch | Description |
|----------|--------|-------------|
| Docker / Podman | [`podman`](../../tree/podman) | Docker Compose deployment |
| **Kubernetes (K3s)** | [`k3s`](../../tree/k3s) | ← You are here |
| Home Assistant | [`ha`](../../tree/ha) | HA add-on with one-click install |
