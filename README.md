# Odoo 18 Docker Compose Deployment

[English](#english) | [中文](#中文)

---

## English

### Overview

Production-ready Docker Compose setup for **Odoo 18 Community Edition** with **PostgreSQL 16** and **pgvector** extension for AI capabilities.

### Features

- Odoo 18 Community Edition
- PostgreSQL 16 with pgvector extension (for embeddings/AI)
- Docker volumes for data persistence
- Custom addons support
- Configurable via environment variables

### Architecture

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

### File Structure

```
.
├── docker-compose.yml      # Main Docker Compose file
├── .env.example            # Example environment variables
├── .env                    # Your environment variables (create from .env.example)
├── .gitignore              # Git ignore rules
├── postgres/
│   └── Dockerfile          # PostgreSQL 16 + pgvector image
├── addons/                 # Custom Odoo modules directory
│   └── .gitkeep
├── config/
│   └── odoo.conf           # Odoo configuration file
└── README.md               # This file
```

### Prerequisites

- Docker Engine 20.10+
- Docker Compose v2.0+
- At least 4GB RAM recommended
- At least 10GB disk space

### Quick Start

#### 1. Clone the repository

```bash
git clone <repository-url>
cd podman_docker_app
```

#### 2. Configure environment

```bash
cp .env.example .env
# Edit .env and set a secure password
nano .env
```

#### 3. Start services

```bash
docker compose up -d
```

#### 4. Access Odoo

Open browser: `http://localhost:18069`

- Create a new database
- Default master password: `admin` (change in `config/odoo.conf`)

### Commands

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services |
| `docker compose down` | Stop all services |
| `docker compose logs -f` | View logs |
| `docker compose logs -f web` | View Odoo logs |
| `docker compose logs -f db` | View PostgreSQL logs |
| `docker compose restart` | Restart all services |
| `docker compose pull` | Update images |

### Enable pgvector

Connect to PostgreSQL and enable the extension:

```bash
docker compose exec db psql -U odoo -d your_database_name -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### Custom Modules

Place custom Odoo modules in the `./addons/` directory. They will be automatically available in Odoo.

### Data Persistence

Data is stored in Docker volumes:

| Volume | Purpose |
|--------|---------|
| `odoo18-db-data` | PostgreSQL database |
| `odoo18-web-data` | Odoo filestore |

### Backup

#### Database backup

```bash
docker compose exec db pg_dump -U odoo your_database_name > backup_$(date +%Y%m%d).sql
```

#### Full backup (including filestore)

```bash
# Stop services first for consistent backup
docker compose down

# Backup volumes
docker run --rm -v odoo18-db-data:/data -v $(pwd):/backup alpine tar czf /backup/db-backup.tar.gz /data
docker run --rm -v odoo18-web-data:/data -v $(pwd):/backup alpine tar czf /backup/web-backup.tar.gz /data

# Restart services
docker compose up -d
```

### Restore

#### Database restore

```bash
docker compose exec -T db psql -U odoo your_database_name < backup.sql
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `POSTGRES_USER` | PostgreSQL username | `odoo` |
| `POSTGRES_PASSWORD` | PostgreSQL password | **required** |
| `POSTGRES_DB` | PostgreSQL database | `postgres` |
| `ODOO_PORT` | Exposed Odoo port | `18069` |

### Troubleshooting

#### Container won't start

```bash
# Check logs
docker compose logs

# Check container status
docker compose ps
```

#### Database connection error

Ensure PostgreSQL is fully started before Odoo:

```bash
docker compose restart web
```

#### Permission issues with addons

```bash
sudo chown -R 101:101 ./addons
```

---

## 中文

### 概述

用於 **Odoo 18 社區版** 的生產級 Docker Compose 部署方案，包含 **PostgreSQL 16** 和 **pgvector** 擴展以支援 AI 功能。

### 功能特點

- Odoo 18 社區版
- PostgreSQL 16 含 pgvector 擴展（用於嵌入向量/AI）
- Docker volumes 資料持久化
- 支援自定義模組
- 環境變數配置

### 系統架構

```
┌─────────────────────────────────────────────────────┐
│                      主機                            │
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

### 檔案結構

```
.
├── docker-compose.yml      # 主要 Docker Compose 檔案
├── .env.example            # 環境變數範例
├── .env                    # 您的環境變數（從 .env.example 複製）
├── .gitignore              # Git 忽略規則
├── postgres/
│   └── Dockerfile          # PostgreSQL 16 + pgvector 映像檔
├── addons/                 # 自定義 Odoo 模組目錄
│   └── .gitkeep
├── config/
│   └── odoo.conf           # Odoo 配置檔案
└── README.md               # 本檔案
```

### 系統需求

- Docker Engine 20.10+
- Docker Compose v2.0+
- 建議至少 4GB RAM
- 建議至少 10GB 硬碟空間

### 快速部署

#### 1. 複製專案

```bash
git clone <repository-url>
cd podman_docker_app
```

#### 2. 配置環境變數

```bash
cp .env.example .env
# 編輯 .env 並設定安全密碼
nano .env
```

#### 3. 啟動服務

```bash
docker compose up -d
```

#### 4. 存取 Odoo

開啟瀏覽器：`http://localhost:18069`

- 建立新資料庫
- 預設主密碼：`admin`（可在 `config/odoo.conf` 中修改）

### 常用指令

| 指令 | 說明 |
|------|------|
| `docker compose up -d` | 啟動所有服務 |
| `docker compose down` | 停止所有服務 |
| `docker compose logs -f` | 查看日誌 |
| `docker compose logs -f web` | 查看 Odoo 日誌 |
| `docker compose logs -f db` | 查看 PostgreSQL 日誌 |
| `docker compose restart` | 重啟所有服務 |
| `docker compose pull` | 更新映像檔 |

### 啟用 pgvector

連接 PostgreSQL 並啟用擴展：

```bash
docker compose exec db psql -U odoo -d your_database_name -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### 自定義模組

將自定義 Odoo 模組放入 `./addons/` 目錄，模組將自動在 Odoo 中可用。

### 資料持久化

資料儲存在 Docker volumes 中：

| Volume | 用途 |
|--------|------|
| `odoo18-db-data` | PostgreSQL 資料庫 |
| `odoo18-web-data` | Odoo 檔案儲存 |

### 備份

#### 資料庫備份

```bash
docker compose exec db pg_dump -U odoo your_database_name > backup_$(date +%Y%m%d).sql
```

#### 完整備份（包含檔案儲存）

```bash
# 先停止服務以確保備份一致性
docker compose down

# 備份 volumes
docker run --rm -v odoo18-db-data:/data -v $(pwd):/backup alpine tar czf /backup/db-backup.tar.gz /data
docker run --rm -v odoo18-web-data:/data -v $(pwd):/backup alpine tar czf /backup/web-backup.tar.gz /data

# 重啟服務
docker compose up -d
```

### 還原

#### 資料庫還原

```bash
docker compose exec -T db psql -U odoo your_database_name < backup.sql
```

### 環境變數

| 變數 | 說明 | 預設值 |
|------|------|--------|
| `POSTGRES_USER` | PostgreSQL 使用者名稱 | `odoo` |
| `POSTGRES_PASSWORD` | PostgreSQL 密碼 | **必填** |
| `POSTGRES_DB` | PostgreSQL 資料庫 | `postgres` |
| `ODOO_PORT` | Odoo 對外連接埠 | `18069` |

### 疑難排解

#### 容器無法啟動

```bash
# 查看日誌
docker compose logs

# 查看容器狀態
docker compose ps
```

#### 資料庫連線錯誤

確保 PostgreSQL 在 Odoo 之前完全啟動：

```bash
docker compose restart web
```

#### addons 權限問題

```bash
sudo chown -R 101:101 ./addons
```

---

## AI Deployment Guide | AI 部署指南

### For AI Assistants

This section provides structured information for AI assistants to quickly deploy this Odoo setup.

#### Quick Deploy Steps

1. **Ensure files exist:**
   - `docker-compose.yml` - Main orchestration file
   - `postgres/Dockerfile` - PostgreSQL 16 + pgvector build
   - `.env` - Environment configuration
   - `config/odoo.conf` - Odoo settings
   - `addons/` - Custom modules directory

2. **Verify .env contains:**
   ```
   POSTGRES_USER=odoo
   POSTGRES_PASSWORD=<secure_password>
   POSTGRES_DB=postgres
   ODOO_PORT=18069
   ```

3. **Deploy command:**
   ```bash
   docker compose up -d
   ```

4. **Verify deployment:**
   ```bash
   docker compose ps
   # Expected: 2 containers running (odoo18-web, odoo18-db)
   ```

5. **Test access:**
   ```bash
   curl -I http://localhost:18069
   # Expected: HTTP 303 redirect to /web
   ```

#### Key Technical Details

| Component | Image/Version | Port |
|-----------|---------------|------|
| Odoo | `odoo:18` | 18069→8069 |
| PostgreSQL | `postgres:16` + pgvector v0.7.4 | internal only |

| Volume | Mount Point |
|--------|-------------|
| `odoo18-db-data` | `/var/lib/postgresql/data` |
| `odoo18-web-data` | `/var/lib/odoo` |
| `./addons` (bind) | `/mnt/extra-addons` |
| `./config` (bind) | `/etc/odoo` |

#### Network

- Internal network: `odoo18-network`
- PostgreSQL accessible at hostname `db` within network

---

**Version:** 1.0.0
**Last Updated:** 2026-02-17
