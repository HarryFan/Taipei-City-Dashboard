# 台北城市儀表板 Docker 啟動指南

本文檔將指導您如何使用 Docker 啟動台北城市儀表板專案。

## 目錄

- [環境需求](#環境需求)
- [快速開始](#快速開始)
- [環境變數設定](#環境變數設定)
- [服務說明](#服務說明)
- [常用指令](#常用指令)
- [開發指南](#開發指南)
- [疑難排解](#疑難排解)
- [貢獻指南](#貢獻指南)

## 環境需求

- Docker 20.10.0 或更高版本
- Docker Compose 2.0.0 或更高版本
- Git (用於克隆專案)
- 至少 4GB 可用記憶體
- 至少 2 個 CPU 核心

## 快速開始

### 1. 克隆專案

```bash
git clone https://github.com/YourOrganization/Taipei-City-Dashboard.git
cd Taipei-City-Dashboard
```

### 2. 設定環境變數

```bash
# 複製環境變數範本
cp docker/.env.example .env

# 編輯環境變數 (根據需求修改)
nano .env
```

### 3. 建立 Docker 網路

```bash
docker network create --driver=bridge --subnet=192.168.128.0/24 --gateway=192.168.128.1 br_dashboard
```

### 4. 啟動資料庫服務

```bash
docker-compose -f docker/docker-compose-db.yaml up -d
```

### 5. 初始化資料庫

```bash
docker-compose -f docker/docker-compose-init.yaml up
```

### 6. 啟動應用程式

```bash
docker-compose -f docker/docker-compose.yaml up -d
```

## 環境變數設定

### 必要設定

| 變數名稱 | 預設值 | 說明 |
|---------|--------|------|
| `DB_DASHBOARD_PASSWORD` | - | 儀表板資料庫密碼 |
| `DB_MANAGER_PASSWORD` | - | 管理後台資料庫密碼 |
| `VITE_MAPBOXTOKEN` | - | Mapbox 存取權杖 |
| `JWT_SECRET` | - | JWT 密鑰 |
| `IDNO_SALT` | - | 身分證加密鹽值 |

### 完整環境變數說明

#### Docker 映像標籤
```env
# Nginx 映像標籤
NGINX_IMAGE_tag=

# Node.js 映像標籤 (預設: 21.6.0-alpine3.18)
NODE_IMAGE_TAG=21.6.0-alpine3.18

# Golang 映像標籤 (預設: 1.21.3-alpine3.18)
GOLANG_IMAGE_TAG=1.21.3-alpine3.18
```

#### 前端設定
```env
# API 基礎路徑 (預設: /api/dev)
VITE_API_URL=/api/dev

# 環境模式 (development/production)
NODE_ENV=development

# 應用程式標題
VITE_APP_TITLE=臺北城市儀表板

# 應用程式版本
VITE_APP_VERSION=2.0.0

# Mapbox 存取權杖 (必填)
VITE_MAPBOXTOKEN=your_token

# Mapbox 圖層 ID
VITE_MAPBOXTILE=your_tile
```

#### 後端伺服器設定
```env
# Gin 框架模式 (debug/release/test)
GIN_MODE=debug

# 監聽地址
GIN_DOMAIN=0.0.0.0

# 監聽端口
GIN_PORT=8080

# JWT 密鑰 (必填)
JWT_SECRET=your_jwt_secret

# 身分證加密鹽值 (必填)
IDNO_SALT=your_salt
```

#### 資料庫設定
```env
# ===== 儀表板資料庫 =====
DB_DASHBOARD_HOST=postgres-data
DB_DASHBOARD_USER=postgres
DB_DASHBOARD_PASSWORD=your_password  # 必填
DB_DASHBOARD_DBNAME=dashboard
DB_DASHBOARD_PORT=5432
DASHBOARD_SAMPLE_FILE=dashboard-demo.sql

# ===== 管理後台資料庫 =====
DB_MANAGER_HOST=postgres-manager
DB_MANAGER_USER=postgres
DB_MANAGER_PASSWORD=your_password  # 必填
DB_MANAGER_DBNAME=dashboardmanager
DB_MANAGER_PORT=5432
MANAGER_SAMPLE_FILE=dashboardmanager-demo.sql
```

#### Redis 設定
```env
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0
```

#### PgAdmin 設定
```env
PGADMIN_DEFAULT_EMAIL=your@email.com
PGADMIN_DEFAULT_PASSWORD=your_password
PGADMIN_LISTEN_PORT=80
```

#### 預設管理員帳號
```env
DASHBOARD_DEFAULT_USERNAME=admin888
DASHBOARD_DEFAULT_Email=admin@example.com
DASHBOARD_DEFAULT_PASSWORD=your_secure_password
```

## 服務說明

所有服務啟動後，可通過以下方式訪問：

| 服務 | 網址 | 預設帳號 |
|------|------|----------|
| 前端應用程式 | http://localhost:8080 | - |
| 後端 API | http://localhost:8088 | - |
| PgAdmin | http://localhost:8889 | admin888@kkk.com / 123123 |

## 常用指令

### 容器管理

```bash
# 查看所有容器狀態
docker ps -a

# 查看特定容器日誌
docker logs <container_name>

# 進入容器終端
docker exec -it <container_name> sh
```

### 服務控制

```bash
# 啟動所有服務
docker-compose -f docker/docker-compose-db.yaml up -d
docker-compose -f docker/docker-compose.yaml up -d

# 停止所有服務
docker-compose -f docker/docker-compose.yaml down
docker-compose -f docker/docker-compose-db.yaml down

# 清理所有容器和卷
docker-compose -f docker/docker-compose.yaml down -v
docker-compose -f docker/docker-compose-db.yaml down -v
```

## 開發指南

### 前端開發

```bash
# 重新建置前端容器
docker-compose -f docker/docker-compose.yaml build dashboard-fe

# 重啟前端服務
docker-compose -f docker/docker-compose.yaml up -d dashboard-fe
```

### 後端開發

```bash
# 重新建置後端容器
docker-compose -f docker/docker-compose.yaml build dashboard-be

# 重啟後端服務
docker-compose -f docker/docker-compose.yaml up -d dashboard-be
```

## 疑難排解

### PostgreSQL 容器無法啟動

1. 確認 `.env` 檔案中的密碼設定
2. 檢查 5432 連接埠是否被佔用
3. 檢查日誌：`docker logs postgres-data`

### 前端無法連接到後端

1. 確認 `VITE_API_URL` 設定正確
2. 檢查瀏覽器開發者工具中的網路請求
3. 確認後端服務正在運行：`docker ps | grep dashboard-be`

### 資料庫連接問題

1. 確保資料庫容器已啟動
2. 使用 PgAdmin 測試連接
3. 檢查後端日誌：`docker logs dashboard-be`

## 貢獻指南

我們歡迎並感謝您的貢獻！請遵循以下步驟：

1. Fork 專案到您的 GitHub 帳戶
2. 創建特性分支：`git checkout -b feature/AmazingFeature`
3. 提交您的更改：`git commit -m 'Add some AmazingFeature'`
4. 推送到分支：`git push origin feature/AmazingFeature`
5. 開啟 Pull Request

### 代碼風格

- 前端：遵循 Airbnb JavaScript 風格指南
- 後端：遵循 Go 官方代碼風格
- 提交訊息：使用約定式提交 (Conventional Commits)

### 問題回報

請在提交問題時提供：
1. 問題的詳細描述
2. 重現步驟
3. 預期行為與實際行為
4. 相關的日誌或截圖
