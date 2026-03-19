# Woow Odoo 18 - Home Assistant Add-on

Odoo 18 社區版 ERP 系統，內建 PostgreSQL 16 資料庫與 Nginx 反向代理，作為 Home Assistant 附加元件獨立運行。

## 目錄

- [功能特色](#功能特色)
- [系統架構](#系統架構)
- [安裝方式](#安裝方式)
- [設定說明](#設定說明)
- [Cloudflare Tunnel 設定](#cloudflare-tunnel-設定)
- [資料目錄結構](#資料目錄結構)
- [WOOW Dashboard 模組](#woow-dashboard-模組)
- [資料庫管理](#資料庫管理)
- [效能調校](#效能調校)
- [故障排除](#故障排除)
- [常見問題](#常見問題)

## 功能特色

- **Odoo 18 社區版**：完整的 ERP 系統，支援 CRM、銷售、庫存、會計等模組
- **內建 PostgreSQL 16**：無需另外安裝資料庫（不含 pgvector，適合邊緣主機）
- **內建 Nginx 反向代理**：支援 WebSocket/長輪詢代理，靜態檔案快取
- **HTTP Only**：區網僅使用 HTTP，對外 HTTPS 透過 Cloudflare Tunnel
- **s6-overlay 行程管理**：自動啟動/監控 PostgreSQL、Odoo、Nginx 三個服務
- **自動初始化**：首次啟動自動建立資料庫和 Odoo 資料目錄
- **繁體中文支援**：完整的繁體中文介面翻譯

## 系統架構

```
┌────────────────────────────────────────────────────┐
│              Home Assistant Add-on                  │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │   s6-overlay 行程管理                          │  │
│  │                                                │  │
│  │   ┌─────────────────────────────────────┐     │  │
│  │   │  Nginx (Port 8069)                   │     │  │
│  │   │  ├── /websocket    → Odoo:8072      │     │  │
│  │   │  ├── /longpolling  → Odoo:8072      │     │  │
│  │   │  ├── /web/static/  → 快取           │     │  │
│  │   │  └── /             → Odoo:8069      │     │  │
│  │   └─────────────────────────────────────┘     │  │
│  │                     ↕                          │  │
│  │   ┌─────────────────────────────────────┐     │  │
│  │   │  Odoo 18 (HTTP:8069 + WS:8072)     │     │  │
│  │   │  ├── workers × N                     │     │  │
│  │   │  └── gevent worker (WebSocket)       │     │  │
│  │   └─────────────────────────────────────┘     │  │
│  │                     ↕                          │  │
│  │   ┌─────────────────────────────────────┐     │  │
│  │   │  PostgreSQL 16 (localhost:5432)      │     │  │
│  │   │  └── 資料庫: odoo                    │     │  │
│  │   └─────────────────────────────────────┘     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  持久化資料：                                       │
│  ├── /data/postgres/  → PostgreSQL 資料檔案        │
│  ├── /data/odoo/      → Odoo 檔案儲存/工作階段     │
│  └── /data/logs/      → 日誌檔案                   │
└────────────────────────────────────────────────────┘
```

## 安裝方式

### 1. 新增 Add-on 儲存庫

在 Home Assistant 中：

1. 前往 **設定** → **附加元件** → **附加元件商店**
2. 點選右上角 **⋮** → **儲存庫**
3. 新增儲存庫 URL：
   ```
   https://github.com/WOOWTECH/Woow_odoo_docker_compose_all
   ```
4. 點選 **新增** → **關閉**

### 2. 安裝 Add-on

1. 在附加元件商店中搜尋 **Woow Odoo 18**
2. 點選 **安裝**
3. 等待安裝完成（首次安裝需下載 Odoo + PostgreSQL 映像，可能需要較長時間）

### 3. 初始設定

1. 前往 **設定** 分頁
2. 修改 `db_password`（建議更改預設密碼）
3. 設定 `TZ` 為您的時區（預設已是 `Asia/Taipei`）
4. 根據需要調整 `odoo_workers`（建議值：CPU 核心數 × 2 + 1）
5. 點選 **儲存**

### 4. 啟動

1. 點選 **啟動**
2. 等待首次初始化完成（PostgreSQL 初始化 + Odoo 啟動）
3. 點選 **開啟網頁介面** 或瀏覽 `http://<HA_IP>:8069`

### 5. 首次使用

1. 首次存取時會看到 Odoo 資料庫管理頁面
2. 輸入主密碼（如有設定 `admin_password`）
3. 填寫公司資訊，建立新資料庫
4. 選擇要安裝的 Odoo 模組

## 設定說明

### 基本設定

| 設定項目 | 預設值 | 說明 |
|---------|--------|------|
| `TZ` | `Asia/Taipei` | 時區設定 |
| `db_password` | `odoo` | PostgreSQL 資料庫密碼 |
| `admin_password` | _(空)_ | Odoo 主密碼（留空停用資料庫管理） |
| `odoo_db_name` | `odoo` | 資料庫名稱 |
| `odoo_without_demo` | `true` | 不載入示範資料 |

### 效能設定

| 設定項目 | 預設值 | 說明 |
|---------|--------|------|
| `odoo_workers` | `2` | 工作行程數（0=單執行緒） |
| `odoo_max_cron_threads` | `1` | 排程執行緒數 |
| `odoo_limit_memory_hard` | _(空)_ | 記憶體硬限制（位元組） |
| `odoo_limit_memory_soft` | _(空)_ | 記憶體軟限制（位元組） |
| `odoo_limit_time_cpu` | _(空)_ | CPU 時間限制（秒） |
| `odoo_limit_time_real` | _(空)_ | 實際時間限制（秒） |

### 進階設定

| 設定項目 | 預設值 | 說明 |
|---------|--------|------|
| `odoo_log_level` | `info` | 日誌等級（debug/info/warn/error/critical） |
| `odoo_proxy_mode` | `true` | 代理模式（處理 X-Forwarded 標頭） |
| `odoo_extra_addons` | _(空)_ | 額外模組路徑（逗號分隔） |
| `env_vars` | `[]` | 額外環境變數 |

### 連接埠設定

| 連接埠 | 用途 | 預設映射 |
|--------|------|---------|
| 8069/tcp | Odoo 網頁介面 | 8069 |
| 8072/tcp | WebSocket/長輪詢 | 停用 |
| 5432/tcp | PostgreSQL 資料庫 | 停用 |

> **注意**：WebSocket 和 PostgreSQL 連接埠預設不對外映射。Nginx 已在內部處理 WebSocket 代理。如需直接連線資料庫，可在網路設定中啟用 5432 連接埠。

## Cloudflare Tunnel 設定

### 為什麼用 Cloudflare Tunnel？

此 Add-on 設計為區網使用 HTTP，外部 HTTPS 存取透過 Cloudflare Tunnel：
- 無需管理 SSL 憑證
- 無需開放路由器連接埠
- 自動獲得 DDoS 防護
- 零信任存取控制

### 設定步驟

1. **安裝 Cloudflare Tunnel Add-on**
   - 在 HA 附加元件商店搜尋 **Cloudflared**
   - 或新增儲存庫：`https://github.com/brenner-tobias/ha-addons`

2. **建立 Tunnel**
   ```yaml
   # Cloudflared 附加元件設定
   external_hostname: odoo.yourdomain.com
   tunnel_name: ha-tunnel
   additional_hosts:
     - hostname: odoo.yourdomain.com
       service: http://localhost:8069
   ```

3. **DNS 設定**
   - 在 Cloudflare Dashboard 確認 `odoo.yourdomain.com` 的 CNAME 記錄指向 Tunnel

4. **驗證**
   - 透過 `https://odoo.yourdomain.com` 存取 Odoo
   - 確認 WebSocket 連線正常（即時通訊功能）

### Odoo 設定建議

透過 Cloudflare Tunnel 存取時：
- 確保 `odoo_proxy_mode` 設為 `true`
- Odoo 將正確處理 `X-Forwarded-Proto: https` 標頭

## 資料目錄結構

```
/data/
├── postgres/           # PostgreSQL 資料檔案
│   ├── PG_VERSION      # PostgreSQL 版本標記
│   ├── base/           # 系統資料庫
│   ├── global/         # 全域設定
│   └── pg_wal/         # WAL 日誌
├── odoo/               # Odoo 資料
│   ├── filestore/      # 附件/文件儲存
│   ├── sessions/       # 使用者工作階段
│   └── addons/         # 下載的模組
├── logs/
│   ├── postgres/       # PostgreSQL 日誌
│   ├── nginx/          # Nginx 日誌
│   └── odoo/           # Odoo 日誌
└── options.json        # Add-on 設定檔
```

## WOOW Dashboard 模組

此 Add-on 可搭配 [WOOW Dashboard](https://github.com/WOOWTECH/odoo-addons) 模組使用，提供 Home Assistant 整合功能：

- **HA 實體管理**：在 Odoo 中管理 Home Assistant 實體
- **即時狀態**：透過 WebSocket 即時同步 HA 設備狀態
- **儀表板**：自訂 HA 設備監控儀表板
- **權限控制**：兩層權限（使用者/管理員）
- **Portal 分享**：與外部使用者分享實體控制

### 安裝 WOOW Dashboard

1. 將 `odoo_ha_addon` 模組放入額外模組路徑
2. 在 `odoo_extra_addons` 中設定路徑
3. 在 Odoo 中啟用開發者模式
4. 前往 **應用程式** → 更新模組清單 → 搜尋 **WOOW Dashboard** → 安裝

## 資料庫管理

### 備份資料庫

```bash
# 透過 Odoo Web 管理介面
# http://<HA_IP>:8069/web/database/manager

# 或透過 pg_dump（需啟用 5432 連接埠）
pg_dump -h <HA_IP> -p 5432 -U odoo -F c -f odoo_backup.dump odoo
```

### 還原資料庫

```bash
# 透過 Odoo Web 管理介面還原 .zip 備份

# 或透過 pg_restore（需啟用 5432 連接埠）
pg_restore -h <HA_IP> -p 5432 -U odoo -d odoo -c odoo_backup.dump
```

### 重設資料庫

如需完全重設，停止 Add-on 後刪除 `/data/postgres/` 目錄，重新啟動即可自動初始化新的資料庫。

## 效能調校

### 依機器規格調整

| 設備 | 建議 Workers | 記憶體建議 |
|------|------------|-----------|
| Raspberry Pi 4 (4GB) | 0-1 | limit_memory_hard: 1GB |
| Intel N100 (8GB) | 2-3 | limit_memory_hard: 2GB |
| x86 (16GB+) | 4-6 | limit_memory_hard: 2.5GB |

### Workers 說明

- `workers = 0`：單執行緒模式，適合低資源設備
- `workers = 2`：基本多工，適合一般使用
- `workers > 2`：高並發，每個 worker 約佔 150-300MB 記憶體

### 記憶體計算

```
總記憶體需求 ≈ PostgreSQL(~200MB) + Nginx(~20MB) + Odoo(workers × 300MB) + 系統(~200MB)
```

## 故障排除

### Odoo 無法啟動

1. 檢查日誌：附加元件 → 日誌 分頁
2. 確認 PostgreSQL 已正常啟動
3. 確認 `db_password` 設定正確
4. 嘗試減少 `odoo_workers` 值

### 資料庫連線失敗

1. 確認 `/data/postgres/` 目錄存在且有正確權限
2. 停止並重新啟動 Add-on
3. 如問題持續，刪除 `/data/postgres/` 重新初始化

### WebSocket 連線問題

1. 確認 `odoo_workers` > 0（WebSocket 需要 gevent worker）
2. 檢查 Nginx 設定是否正確生成
3. 確認 8072 連接埠在容器內可用

### 首次啟動很慢

正常現象。首次啟動需要：
1. 初始化 PostgreSQL 資料庫
2. 建立 Odoo 資料庫結構
3. 安裝基礎模組

這可能需要數分鐘，取決於設備效能。

## 常見問題

### Q: 為什麼不包含 pgvector？

因為 HAOS 通常運行在邊緣主機（如 Raspberry Pi、Intel N100 迷你主機），這些設備的運算資源有限。pgvector 主要用於 AI 向量搜尋等進階功能，在邊緣主機上增加不必要的記憶體消耗。如需 pgvector，建議使用 Docker Compose 或 K3s 部署方式。

### Q: 可以使用外部 PostgreSQL 嗎？

目前此 Add-on 內建 PostgreSQL，如需連線外部資料庫，需要自行修改 Odoo 設定。

### Q: 資料會保留嗎？

會。所有資料儲存在 `/data/` 目錄下，Add-on 更新或重啟不會影響資料。

### Q: 支援哪些架構？

- `amd64`（x86_64）：Intel/AMD 處理器
- `aarch64`（ARM64）：Raspberry Pi 4/5、Apple Silicon 等

### Q: 如何安裝自訂 Odoo 模組？

1. 將模組放入 HA 的 `addon_configs/` 或 `share/` 目錄
2. 在 `odoo_extra_addons` 中設定對應路徑
3. 在 Odoo 中更新模組清單並安裝

### Q: PostgreSQL 版本為什麼是 16？

與 [Docker Compose 版本](https://github.com/WOOWTECH/Woow_odoo_docker_compose_all) 和 [K3s 版本](https://github.com/WOOWTECH/Woow_odoo_docker_compose_all/tree/k3s) 保持一致，均使用 PostgreSQL 16。差異僅在於 HA 版不包含 pgvector 擴充。

---

**維護者**：[WOOWTECH](https://github.com/WOOWTECH)
**授權**：LGPL-3.0
**來源**：基於 [Odoo 18.0 官方 Docker 映像](https://hub.docker.com/_/odoo)
