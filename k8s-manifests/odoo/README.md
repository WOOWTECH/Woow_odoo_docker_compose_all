# Odoo 18 - K3s/Kubernetes 部署指南

[English](#english) | [中文](#中文)

---

## English

### Overview

Full-featured open-source ERP (Enterprise Resource Planning) and business management suite. Odoo 18 provides integrated modules for CRM, Accounting, Inventory, Human Resources, Project Management, Website Builder, eCommerce, and more. This deployment includes PostgreSQL with pgvector for AI-powered features and a dedicated volume for custom addons.

> **GitHub Repo (Podman/Docker):** [Woow_odoo_docker_compose_all](https://github.com/WOOWTECH/Woow_odoo_docker_compose_all)

### Architecture

```
                         ┌─────────────────────────────────────────────────┐
                         │                K3s / Kubernetes                 │
                         │                                                 │
  ┌───────────┐          │  ┌─────────────────────────────────────────┐    │
  │  Browser   │  :18069  │  │          Namespace: odoo                │    │
  │           ├──────────►│  │                                         │    │
  └───────────┘  NodePort │  │  ┌───────────┐    ┌─────────────────┐  │    │
                         │  │  │  Service   │    │   Deployment    │  │    │
                         │  │  │ odoo       ├───►│   odoo          │  │    │
                         │  │  │ :8069      │    │  (odoo:18)      │  │    │
                         │  │  └───────────┘    │                 │  │    │
                         │  │                    │ [PVC: 10Gi      │  │    │
                         │  │                    │  web-data]      │  │    │
                         │  │                    │ [PVC: 5Gi       │  │    │
                         │  │                    │  addons]        │  │    │
                         │  │                    └────────┬────────┘  │    │
                         │  │                             │           │    │
                         │  │                             ▼           │    │
                         │  │                    ┌─────────────────┐  │    │
                         │  │                    │  StatefulSet    │  │    │
                         │  │                    │  db (pgvector/  │  │    │
                         │  │                    │  pgvector:pg16) │  │    │
                         │  │                    │  :5432          │  │    │
                         │  │                    │ [PVC: 10Gi]     │  │    │
                         │  │                    └─────────────────┘  │    │
                         │  │                                         │    │
                         │  └─────────────────────────────────────────┘    │
                         └─────────────────────────────────────────────────┘

  Port Mappings:
    External :18069  ──►  Service :8069  ──►  Pod odoo :8069
    Internal :5432   ──►  Pod db (PostgreSQL + pgvector) :5432

  Volume Mounts (Odoo Pod):
    /var/lib/odoo       ──►  odoo-web-data PVC (filestore, sessions)
    /mnt/extra-addons   ──►  odoo-addons PVC (custom modules)
    /etc/odoo/odoo.conf ──►  ConfigMap (odoo.conf)
```

### Features

- Full ERP suite: CRM, Accounting, Inventory, HR, Project Management, and more
- Website Builder and eCommerce integrated
- PostgreSQL with pgvector for AI-powered vector search features
- Custom addons volume for community/third-party modules
- Configurable via `odoo.conf` mounted from ConfigMap
- Init container ensures database is ready before Odoo starts
- Dedicated PVC for filestore (attachments, session data)

### Quick Start

```bash
# 1. Update secrets before deploying
nano k8s-manifests/odoo/secret.yaml

# 2. Deploy all Odoo components
kubectl apply -k k8s-manifests/odoo/

# 3. Verify pods are running
kubectl -n odoo get pods

# 4. Watch Odoo startup logs
kubectl -n odoo logs deploy/odoo -f
```

### Configuration

#### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `HOST` | PostgreSQL hostname | `db` | Yes |
| `PORT` | PostgreSQL port | `5432` | Yes |

#### Odoo Configuration File (`odoo.conf`)

The Odoo configuration is mounted from the ConfigMap at `/etc/odoo/odoo.conf`:

| Setting | Description | Default |
|---------|-------------|---------|
| `addons_path` | Path to custom addons | `/mnt/extra-addons` |
| `admin_passwd` | Database management admin password | `admin` |
| `log_level` | Logging verbosity | `info` |
| `workers` | Number of worker processes (0 = multi-threading) | `0` |
| `max_cron_threads` | Maximum cron worker threads | `1` |
| `limit_memory_hard` | Hard memory limit per worker (bytes) | `2684354560` (2.5GB) |
| `limit_memory_soft` | Soft memory limit per worker (bytes) | `2147483648` (2GB) |
| `limit_time_cpu` | CPU time limit per request (seconds) | `600` |
| `limit_time_real` | Real time limit per request (seconds) | `1200` |
| `proxy_mode` | Enable if behind a reverse proxy | `False` |

#### Secrets

Edit `secret.yaml` before deploying:

| Secret Key | Description | Default (change me!) |
|------------|-------------|----------------------|
| `POSTGRES_USER` | PostgreSQL username | `odoo` |
| `POSTGRES_PASSWORD` | PostgreSQL password | `changeme` |
| `POSTGRES_DB` | PostgreSQL database name | `postgres` |

```bash
nano k8s-manifests/odoo/secret.yaml
```

### Accessing the Service

| Endpoint | URL | Protocol |
|----------|-----|----------|
| Odoo Web UI | `http://<node-ip>:18069` | HTTP (NodePort) |
| Internal (cluster) | `http://odoo.odoo.svc.cluster.local:8069` | HTTP |

On first access, Odoo presents the database manager where you can create a new database and install modules.

### Data Persistence

| PVC Name | Mount Path | Size | Purpose |
|----------|------------|------|---------|
| `odoo-db-data` | `/var/lib/postgresql/data` | 10Gi | PostgreSQL database files |
| `odoo-web-data` | `/var/lib/odoo` | 10Gi | Odoo filestore (attachments, session data) |
| `odoo-addons` | `/mnt/extra-addons` | 5Gi | Custom/community Odoo modules |
| `odoo-config` | ConfigMap mount | 100Mi | Configuration file storage |

All PVCs use the `local-path` storage class (k3s default).

### Backup & Restore

#### Backup

```bash
# 1. Backup PostgreSQL database
kubectl -n odoo exec sts/db -- pg_dump -U odoo postgres > odoo-db-backup.sql

# 2. Backup Odoo filestore
kubectl -n odoo exec deploy/odoo -- tar czf /tmp/odoo-filestore.tar.gz /var/lib/odoo
kubectl -n odoo cp odoo/<odoo-pod>:/tmp/odoo-filestore.tar.gz ./odoo-filestore.tar.gz

# 3. Backup custom addons
kubectl -n odoo exec deploy/odoo -- tar czf /tmp/extra-addons.tar.gz /mnt/extra-addons
kubectl -n odoo cp odoo/<odoo-pod>:/tmp/extra-addons.tar.gz ./extra-addons.tar.gz
```

#### Restore

```bash
# 1. Restore PostgreSQL database
kubectl -n odoo exec -i sts/db -- psql -U odoo postgres < odoo-db-backup.sql

# 2. Restore Odoo filestore
kubectl -n odoo cp ./odoo-filestore.tar.gz odoo/<odoo-pod>:/tmp/odoo-filestore.tar.gz
kubectl -n odoo exec deploy/odoo -- tar xzf /tmp/odoo-filestore.tar.gz -C /

# 3. Restore custom addons
kubectl -n odoo cp ./extra-addons.tar.gz odoo/<odoo-pod>:/tmp/extra-addons.tar.gz
kubectl -n odoo exec deploy/odoo -- tar xzf /tmp/extra-addons.tar.gz -C /

# 4. Restart Odoo
kubectl -n odoo rollout restart deploy/odoo
```

### Useful Commands

```bash
# Check all resources in the namespace
kubectl -n odoo get all

# View real-time Odoo logs
kubectl -n odoo logs deploy/odoo -f

# Restart Odoo
kubectl -n odoo rollout restart deploy/odoo

# Check PostgreSQL status
kubectl -n odoo exec sts/db -- pg_isready -U odoo

# List custom addons
kubectl -n odoo exec deploy/odoo -- ls -la /mnt/extra-addons/

# Access PostgreSQL shell
kubectl -n odoo exec -it sts/db -- psql -U odoo postgres

# Delete and redeploy
kubectl delete -k k8s-manifests/odoo/
kubectl apply -k k8s-manifests/odoo/
```

### Troubleshooting

#### Odoo stuck waiting for database

The Odoo pod has an init container (`wait-for-db`) that waits for PostgreSQL. Check:

```bash
kubectl -n odoo get pods -l component=database
kubectl -n odoo logs sts/db
```

#### Cannot install modules / "Module not found"

Verify custom addons are in the correct path:

```bash
kubectl -n odoo exec deploy/odoo -- ls -la /mnt/extra-addons/
```

After adding new addons, update the module list in Odoo UI: Settings > Apps > Update Apps List.

#### Slow performance / high memory usage

Increase `workers` in `configmap.yaml` for production use. When using workers > 0, also set `proxy_mode = True` if behind a reverse proxy:

```yaml
workers = 4
proxy_mode = True
```

#### Database management password

The default database management password is `admin` (set in `odoo.conf`). Change this in `configmap.yaml` for production environments.

#### Pod keeps restarting (OOMKilled)

Increase the memory limits in `odoo-deployment.yaml`:

```yaml
resources:
  limits:
    memory: 4Gi  # Increase from 2Gi
```

### File Structure

```
k8s-manifests/odoo/
├── kustomization.yaml          # Kustomize entry point
├── namespace.yaml              # Namespace: odoo
├── configmap.yaml              # Odoo configuration (odoo.conf)
├── secret.yaml                 # PostgreSQL credentials
├── postgres-statefulset.yaml   # PostgreSQL 16 + pgvector StatefulSet
├── postgres-service.yaml       # ClusterIP service for PostgreSQL
├── odoo-deployment.yaml        # Odoo 18 Deployment with init container
├── odoo-service.yaml           # NodePort service (18069)
├── pvc.yaml                    # PVCs for Odoo data and addons
└── README.md                   # This file
```

---

## 中文

### 概述

功能完整的開源 ERP（企業資源規劃）與商業管理套件。Odoo 18 提供整合的 CRM、會計、庫存、人力資源、專案管理、網站建置器、電子商務等模組。此部署包含具有 pgvector 的 PostgreSQL 以支援 AI 功能，以及專用的自訂附加模組磁碟區。

> **GitHub 儲存庫 (Podman/Docker):** [Woow_odoo_docker_compose_all](https://github.com/WOOWTECH/Woow_odoo_docker_compose_all)

### 架構圖

```
                         ┌─────────────────────────────────────────────────┐
                         │                K3s / Kubernetes                 │
                         │                                                 │
  ┌───────────┐          │  ┌─────────────────────────────────────────┐    │
  │   瀏覽器   │  :18069  │  │          命名空間: odoo                 │    │
  │           ├──────────►│  │                                         │    │
  └───────────┘  NodePort │  │  ┌───────────┐    ┌─────────────────┐  │    │
                         │  │  │  Service   │    │   Deployment    │  │    │
                         │  │  │ odoo       ├───►│   odoo          │  │    │
                         │  │  │ :8069      │    │  (odoo:18)      │  │    │
                         │  │  └───────────┘    │                 │  │    │
                         │  │                    │ [PVC: 10Gi      │  │    │
                         │  │                    │  web-data]      │  │    │
                         │  │                    │ [PVC: 5Gi       │  │    │
                         │  │                    │  addons]        │  │    │
                         │  │                    └────────┬────────┘  │    │
                         │  │                             │           │    │
                         │  │                             ▼           │    │
                         │  │                    ┌─────────────────┐  │    │
                         │  │                    │  StatefulSet    │  │    │
                         │  │                    │  db (pgvector/  │  │    │
                         │  │                    │  pgvector:pg16) │  │    │
                         │  │                    │  :5432          │  │    │
                         │  │                    │ [PVC: 10Gi]     │  │    │
                         │  │                    └─────────────────┘  │    │
                         │  │                                         │    │
                         │  └─────────────────────────────────────────┘    │
                         └─────────────────────────────────────────────────┘

  連接埠對應:
    外部 :18069  ──►  Service :8069  ──►  Pod odoo :8069
    內部 :5432   ──►  Pod db (PostgreSQL + pgvector) :5432

  磁碟區掛載（Odoo Pod）:
    /var/lib/odoo       ──►  odoo-web-data PVC（檔案儲存、工作階段）
    /mnt/extra-addons   ──►  odoo-addons PVC（自訂模組）
    /etc/odoo/odoo.conf ──►  ConfigMap (odoo.conf)
```

### 功能特色

- 完整 ERP 套件：CRM、會計、庫存、人力資源、專案管理等
- 整合網站建置器與電子商務功能
- PostgreSQL 搭配 pgvector 支援 AI 向量搜尋功能
- 自訂附加模組磁碟區用於社群/第三方模組
- 透過 ConfigMap 掛載的 `odoo.conf` 進行設定
- Init 容器確保資料庫在 Odoo 啟動前就緒
- 專用 PVC 儲存附件與工作階段資料

### 快速開始

```bash
# 1. 部署前更新密鑰設定
nano k8s-manifests/odoo/secret.yaml

# 2. 部署所有 Odoo 元件
kubectl apply -k k8s-manifests/odoo/

# 3. 確認 Pod 正常運行
kubectl -n odoo get pods

# 4. 監看 Odoo 啟動日誌
kubectl -n odoo logs deploy/odoo -f
```

### 設定

#### 環境變數

| 變數 | 說明 | 預設值 | 必填 |
|------|------|--------|------|
| `HOST` | PostgreSQL 主機名稱 | `db` | 是 |
| `PORT` | PostgreSQL 連接埠 | `5432` | 是 |

#### Odoo 設定檔（`odoo.conf`）

Odoo 設定透過 ConfigMap 掛載於 `/etc/odoo/odoo.conf`：

| 設定 | 說明 | 預設值 |
|------|------|--------|
| `addons_path` | 自訂附加模組路徑 | `/mnt/extra-addons` |
| `admin_passwd` | 資料庫管理員密碼 | `admin` |
| `log_level` | 日誌詳細程度 | `info` |
| `workers` | 工作行程數（0 = 多執行緒模式） | `0` |
| `max_cron_threads` | 最大 cron 工作執行緒數 | `1` |
| `limit_memory_hard` | 每個工作行程的硬記憶體限制（位元組） | `2684354560`（2.5GB） |
| `limit_memory_soft` | 每個工作行程的軟記憶體限制（位元組） | `2147483648`（2GB） |
| `limit_time_cpu` | 每個請求的 CPU 時間限制（秒） | `600` |
| `limit_time_real` | 每個請求的實際時間限制（秒） | `1200` |
| `proxy_mode` | 反向代理後方時啟用 | `False` |

#### 密鑰設定

部署前請編輯 `secret.yaml`：

| 密鑰名稱 | 說明 | 預設值（請更改！） |
|----------|------|-------------------|
| `POSTGRES_USER` | PostgreSQL 使用者名稱 | `odoo` |
| `POSTGRES_PASSWORD` | PostgreSQL 密碼 | `changeme` |
| `POSTGRES_DB` | PostgreSQL 資料庫名稱 | `postgres` |

```bash
nano k8s-manifests/odoo/secret.yaml
```

### 存取服務

| 端點 | URL | 協定 |
|------|-----|------|
| Odoo 網頁介面 | `http://<節點IP>:18069` | HTTP (NodePort) |
| 內部（叢集） | `http://odoo.odoo.svc.cluster.local:8069` | HTTP |

首次存取時，Odoo 會顯示資料庫管理員頁面，您可以在此建立新資料庫並安裝模組。

### 資料持久化

| PVC 名稱 | 掛載路徑 | 大小 | 用途 |
|----------|----------|------|------|
| `odoo-db-data` | `/var/lib/postgresql/data` | 10Gi | PostgreSQL 資料庫檔案 |
| `odoo-web-data` | `/var/lib/odoo` | 10Gi | Odoo 檔案儲存（附件、工作階段資料） |
| `odoo-addons` | `/mnt/extra-addons` | 5Gi | 自訂/社群 Odoo 模組 |
| `odoo-config` | ConfigMap 掛載 | 100Mi | 設定檔儲存 |

所有 PVC 使用 `local-path` 儲存類別（k3s 預設）。

### 備份與還原

#### 備份

```bash
# 1. 備份 PostgreSQL 資料庫
kubectl -n odoo exec sts/db -- pg_dump -U odoo postgres > odoo-db-backup.sql

# 2. 備份 Odoo 檔案儲存
kubectl -n odoo exec deploy/odoo -- tar czf /tmp/odoo-filestore.tar.gz /var/lib/odoo
kubectl -n odoo cp odoo/<odoo-pod>:/tmp/odoo-filestore.tar.gz ./odoo-filestore.tar.gz

# 3. 備份自訂附加模組
kubectl -n odoo exec deploy/odoo -- tar czf /tmp/extra-addons.tar.gz /mnt/extra-addons
kubectl -n odoo cp odoo/<odoo-pod>:/tmp/extra-addons.tar.gz ./extra-addons.tar.gz
```

#### 還原

```bash
# 1. 還原 PostgreSQL 資料庫
kubectl -n odoo exec -i sts/db -- psql -U odoo postgres < odoo-db-backup.sql

# 2. 還原 Odoo 檔案儲存
kubectl -n odoo cp ./odoo-filestore.tar.gz odoo/<odoo-pod>:/tmp/odoo-filestore.tar.gz
kubectl -n odoo exec deploy/odoo -- tar xzf /tmp/odoo-filestore.tar.gz -C /

# 3. 還原自訂附加模組
kubectl -n odoo cp ./extra-addons.tar.gz odoo/<odoo-pod>:/tmp/extra-addons.tar.gz
kubectl -n odoo exec deploy/odoo -- tar xzf /tmp/extra-addons.tar.gz -C /

# 4. 重啟 Odoo
kubectl -n odoo rollout restart deploy/odoo
```

### 實用指令

```bash
# 檢視命名空間中的所有資源
kubectl -n odoo get all

# 即時檢視 Odoo 日誌
kubectl -n odoo logs deploy/odoo -f

# 重啟 Odoo
kubectl -n odoo rollout restart deploy/odoo

# 檢查 PostgreSQL 狀態
kubectl -n odoo exec sts/db -- pg_isready -U odoo

# 列出自訂附加模組
kubectl -n odoo exec deploy/odoo -- ls -la /mnt/extra-addons/

# 存取 PostgreSQL 命令列
kubectl -n odoo exec -it sts/db -- psql -U odoo postgres

# 刪除並重新部署
kubectl delete -k k8s-manifests/odoo/
kubectl apply -k k8s-manifests/odoo/
```

### 疑難排解

#### Odoo 卡在等待資料庫

Odoo Pod 具有一個 init 容器（`wait-for-db`）會等待 PostgreSQL 就緒。請檢查：

```bash
kubectl -n odoo get pods -l component=database
kubectl -n odoo logs sts/db
```

#### 無法安裝模組 / 「Module not found」

確認自訂附加模組位於正確路徑：

```bash
kubectl -n odoo exec deploy/odoo -- ls -la /mnt/extra-addons/
```

新增附加模組後，在 Odoo 介面更新模組清單：設定 > 應用程式 > 更新應用程式清單。

#### 效能緩慢 / 記憶體使用率高

在 `configmap.yaml` 中增加 `workers` 數值以用於生產環境。當 workers > 0 時，若位於反向代理後方，也請設定 `proxy_mode = True`：

```yaml
workers = 4
proxy_mode = True
```

#### 資料庫管理密碼

預設的資料庫管理密碼為 `admin`（設定於 `odoo.conf` 中）。在生產環境中請於 `configmap.yaml` 中更改。

#### Pod 持續重啟（OOMKilled）

在 `odoo-deployment.yaml` 中增加記憶體限制：

```yaml
resources:
  limits:
    memory: 4Gi  # 從 2Gi 增加
```

### 檔案結構

```
k8s-manifests/odoo/
├── kustomization.yaml          # Kustomize 進入點
├── namespace.yaml              # 命名空間: odoo
├── configmap.yaml              # Odoo 設定（odoo.conf）
├── secret.yaml                 # PostgreSQL 帳號密碼
├── postgres-statefulset.yaml   # PostgreSQL 16 + pgvector StatefulSet
├── postgres-service.yaml       # PostgreSQL ClusterIP 服務
├── odoo-deployment.yaml        # Odoo 18 Deployment（含 init 容器）
├── odoo-service.yaml           # NodePort 服務（18069）
├── pvc.yaml                    # Odoo 資料與附加模組的 PVC
└── README.md                   # 本文件
```
