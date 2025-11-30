# 1Panel 部署 Dify 完整指南

本文档详细介绍如何通过 1Panel 面板在服务器上部署 Dify 开源 LLM 应用开发平台。

## 目录

- [1. 环境要求](#1-环境要求)
- [2. 1Panel 简介与安装](#2-1panel-简介与安装)
- [3. 部署前准备](#3-部署前准备)
- [4. 通过 1Panel 部署 Dify](#4-通过-1panel-部署-dify)
- [5. 关键配置说明](#5-关键配置说明)
- [6. 配置域名和 SSL 证书](#6-配置域名和-ssl-证书)
- [7. 访问和初始化 Dify](#7-访问和初始化-dify)
- [8. 常见问题与故障排除](#8-常见问题与故障排除)
- [9. 升级与维护](#9-升级与维护)

---

## 1. 环境要求

### 1.1 硬件要求

| 资源 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 2 核 | 4 核及以上 |
| 内存 | 4 GB | 8 GB 及以上 |
| 硬盘 | 40 GB SSD | 100 GB SSD 及以上 |

### 1.2 软件要求

- **操作系统**: CentOS 7.x / Ubuntu 18.04+ / Debian 10+
- **1Panel 版本**: 1.9.0 及以上
- **Docker**: 20.10+ (1Panel 自带安装)
- **Docker Compose**: v2.0+ (1Panel 自带安装)

### 1.3 网络要求

- 服务器需能访问互联网（用于拉取 Docker 镜像）
- 需开放以下端口：
  - `80` - HTTP 访问
  - `443` - HTTPS 访问
  - `5003` - 插件调试端口（可选）

---

## 2. 1Panel 简介与安装

### 2.1 什么是 1Panel

[1Panel](https://1panel.cn/) 是一款现代化的 Linux 服务器运维管理面板，它基于 Go 语言开发，提供了 Docker 可视化管理、应用商店、网站管理等功能，非常适合用来部署和管理 Dify。

### 2.2 安装 1Panel

如果您还未安装 1Panel，请使用以下命令安装：

```bash
# 使用官方安装脚本
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

安装完成后，记录面板的访问地址、用户名和密码。

### 2.3 验证 Docker 环境

登录 1Panel 面板后，进入 **容器 > 配置** 确认 Docker 已正常运行。如果未安装，可以通过 1Panel 的 **应用商店 > 运行环境** 安装 Docker。

---

## 3. 部署前准备

### 3.1 创建部署目录

在 1Panel 中，进入 **主机 > 文件管理**，创建 Dify 的部署目录：

```bash
/opt/dify
```

### 3.2 下载 Dify Docker 配置文件

通过 1Panel 的 **终端** 功能执行以下命令：

```bash
# 进入部署目录
cd /opt/dify

# 下载 Docker 部署文件
git clone https://github.com/langgenius/dify.git --depth=1

# 进入 docker 目录
cd dify/docker

# 复制环境配置文件
cp .env.example .env
```

如果服务器无法访问 GitHub，可以手动上传 `docker` 目录下的文件。

---

## 4. 通过 1Panel 部署 Dify

### 4.1 方式一：使用 Docker Compose 文件部署（推荐）

#### 步骤 1：配置环境变量

在 1Panel 中编辑 `/opt/dify/dify/docker/.env` 文件，修改以下关键配置：

```bash
# ⚠️ 重要安全配置 - 必须修改！
# 生成安全密钥：使用命令 openssl rand -base64 42
# 此密钥用于加密会话和敏感数据，使用弱密钥将导致严重安全风险
SECRET_KEY=<使用 openssl rand -base64 42 生成的密钥>

# 设置初始管理员密码（可选，不超过30字符）
INIT_PASSWORD=<设置一个强密码>

# 如果使用域名访问，配置以下URL
CONSOLE_API_URL=https://your-domain.com
CONSOLE_WEB_URL=https://your-domain.com
SERVICE_API_URL=https://your-domain.com
APP_WEB_URL=https://your-domain.com
FILES_URL=https://your-domain.com

# ⚠️ 数据库配置 - 生产环境务必修改默认密码！
# 使用强密码，包含大小写字母、数字和特殊字符
DB_PASSWORD=<设置一个强密码>
REDIS_PASSWORD=<设置一个强密码>

# 向量数据库配置（默认使用 weaviate）
VECTOR_STORE=weaviate

# 对于中国用户，可配置 pip 镜像加速
PIP_MIRROR_URL=https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 步骤 2：使用 1Panel 创建 Compose 项目

1. 进入 **1Panel > 容器 > Compose 模板**
2. 点击 **创建 Compose 模板**
3. 填写名称：`dify`
4. 内容选择 **从路径获取**，路径填写：`/opt/dify/dify/docker/docker-compose.yaml`
5. 点击 **确认** 保存

#### 步骤 3：创建并启动服务

1. 进入 **容器 > Compose**
2. 点击 **创建 Compose**
3. 选择模板：选择刚创建的 `dify` 模板
4. 路径：`/opt/dify/dify/docker`
5. 环境变量文件：勾选 **使用 .env 文件**
6. 点击 **确认** 开始部署

等待所有容器启动完成（约 3-10 分钟，取决于网络速度）。

### 4.2 方式二：命令行部署

通过 1Panel 的 **终端** 功能执行：

```bash
cd /opt/dify/dify/docker

# 启动所有服务
docker compose up -d

# 查看服务状态
docker compose ps
```

### 4.3 验证部署状态

部署完成后，应该看到以下容器正在运行：

| 容器名称 | 说明 |
|----------|------|
| docker-api-1 | API 服务 |
| docker-web-1 | Web 前端服务 |
| docker-worker-1 | 异步任务处理 |
| docker-worker_beat-1 | 定时任务调度 |
| docker-nginx-1 | 反向代理 |
| docker-db_postgres-1 | PostgreSQL 数据库 |
| docker-redis-1 | Redis 缓存 |
| docker-sandbox-1 | 代码执行沙箱 |
| docker-ssrf_proxy-1 | SSRF 代理保护 |
| docker-weaviate-1 | 向量数据库 |
| docker-plugin_daemon-1 | 插件服务 |

---

## 5. 关键配置说明

### 5.1 环境变量分类

#### 核心配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `SECRET_KEY` | 加密密钥，用于会话和敏感数据加密 | 需要修改 |
| `INIT_PASSWORD` | 初始管理员密码 | 空 |
| `DEPLOY_ENV` | 部署环境 (PRODUCTION/TESTING) | PRODUCTION |
| `LOG_LEVEL` | 日志级别 | INFO |

#### 数据库配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `DB_TYPE` | 数据库类型 | postgresql |
| `DB_USERNAME` | 数据库用户名 | postgres |
| `DB_PASSWORD` | 数据库密码 | difyai123456 |
| `DB_HOST` | 数据库主机 | db_postgres |
| `DB_PORT` | 数据库端口 | 5432 |
| `DB_DATABASE` | 数据库名称 | dify |

#### Redis 配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `REDIS_HOST` | Redis 主机 | redis |
| `REDIS_PORT` | Redis 端口 | 6379 |
| `REDIS_PASSWORD` | Redis 密码 | difyai123456 |

#### 向量数据库配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `VECTOR_STORE` | 向量数据库类型 | weaviate |
| `WEAVIATE_ENDPOINT` | Weaviate 地址 | http://weaviate:8080 |
| `WEAVIATE_API_KEY` | Weaviate API Key | WVF5YThaHlkYwhGUSmCRgsX3tD5ngdN8pkih |

支持的向量数据库：`weaviate`, `qdrant`, `milvus`, `pgvector`, `chroma`, `opensearch`, `elasticsearch` 等。

#### 文件存储配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `STORAGE_TYPE` | 存储类型 | opendal |
| `OPENDAL_SCHEME` | OpenDAL 方案 | fs (本地文件系统) |
| `OPENDAL_FS_ROOT` | 本地存储路径 | storage |

支持的存储类型：本地存储、S3、Azure Blob、阿里云 OSS、腾讯云 COS 等。

#### Nginx 配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `NGINX_PORT` | HTTP 端口 | 80 |
| `NGINX_SSL_PORT` | HTTPS 端口 | 443 |
| `NGINX_HTTPS_ENABLED` | 启用 HTTPS | false |
| `EXPOSE_NGINX_PORT` | 对外暴露的 HTTP 端口 | 80 |
| `EXPOSE_NGINX_SSL_PORT` | 对外暴露的 HTTPS 端口 | 443 |

### 5.2 切换向量数据库

如需使用其他向量数据库，修改 `.env` 文件：

**使用 Qdrant：**
```bash
VECTOR_STORE=qdrant
QDRANT_URL=http://qdrant:6333
QDRANT_API_KEY=difyai123456
```

**使用 pgvector：**
```bash
VECTOR_STORE=pgvector
PGVECTOR_HOST=pgvector
PGVECTOR_PORT=5432
PGVECTOR_USER=postgres
PGVECTOR_PASSWORD=difyai123456
PGVECTOR_DATABASE=dify
```

修改后需重启服务：
```bash
docker compose down
docker compose up -d
```

### 5.3 配置对象存储（可选）

**使用阿里云 OSS：**
```bash
STORAGE_TYPE=aliyun-oss
ALIYUN_OSS_BUCKET_NAME=your-bucket-name
ALIYUN_OSS_ACCESS_KEY=your-access-key
ALIYUN_OSS_SECRET_KEY=your-secret-key
ALIYUN_OSS_ENDPOINT=https://oss-cn-hangzhou.aliyuncs.com
ALIYUN_OSS_REGION=cn-hangzhou
```

**使用腾讯云 COS：**
```bash
STORAGE_TYPE=tencent-cos
TENCENT_COS_BUCKET_NAME=your-bucket-name
TENCENT_COS_SECRET_KEY=your-secret-key
TENCENT_COS_SECRET_ID=your-secret-id
TENCENT_COS_REGION=ap-guangzhou
```

---

## 6. 配置域名和 SSL 证书

### 6.1 通过 1Panel 配置反向代理

1. 进入 **1Panel > 网站 > 网站**
2. 点击 **创建网站**
3. 选择 **反向代理**
4. 填写配置：
   - 主域名：`your-domain.com`
   - 代理地址：`http://127.0.0.1:80`（Dify 的 Nginx 端口）
5. 点击 **确认**

### 6.2 申请 SSL 证书

1. 进入刚创建的网站配置
2. 点击 **HTTPS**
3. 选择 **申请证书**
4. 推荐使用 **Let's Encrypt** 免费证书
5. 填写邮箱，选择验证方式
6. 点击 **申请**

### 6.3 更新 Dify 环境变量

申请证书成功后，更新 `.env` 文件中的 URL 配置：

```bash
CONSOLE_API_URL=https://your-domain.com
CONSOLE_WEB_URL=https://your-domain.com
SERVICE_API_URL=https://your-domain.com
APP_API_URL=https://your-domain.com
APP_WEB_URL=https://your-domain.com
FILES_URL=https://your-domain.com
```

然后重启服务：
```bash
docker compose restart
```

---

## 7. 访问和初始化 Dify

### 7.1 访问 Dify

部署完成后，通过以下地址访问：

- 未配置域名：`http://服务器IP/install`
- 已配置域名：`https://your-domain.com/install`

### 7.2 初始化设置

首次访问时需要完成初始化：

1. 设置管理员邮箱和密码
2. 配置系统语言
3. 完成初始化

### 7.3 配置模型提供商

登录后，进入 **设置 > 模型提供商** 配置 AI 模型：

- **OpenAI**: 输入 API Key
- **Azure OpenAI**: 配置终结点和密钥
- **Anthropic (Claude)**: 输入 API Key
- **国内模型**: 支持通义千问、智谱、百川等

---

## 8. 常见问题与故障排除

### 8.1 容器启动失败

**问题**：某些容器无法启动

**解决方案**：
```bash
# 查看日志
docker compose logs -f <container_name>

# 重启特定服务
docker compose restart <service_name>

# 完全重建
docker compose down
docker compose up -d
```

### 8.2 数据库连接失败

**问题**：API 服务无法连接数据库

**解决方案**：
1. 确认数据库容器正常运行
2. 检查 `.env` 文件中的数据库配置
3. 查看数据库日志：`docker compose logs db_postgres`

### 8.3 访问返回 502 错误

**问题**：通过域名访问返回 502

**解决方案**：
1. 确认所有容器都已启动
2. 检查 API 服务日志：`docker compose logs api`
3. 确认反向代理配置正确

### 8.4 文件上传失败

**问题**：无法上传文件

**解决方案**：
1. 检查存储目录权限：`chmod -R 777 volumes/app/storage`
2. 确认 `FILES_URL` 配置正确
3. 检查 `UPLOAD_FILE_SIZE_LIMIT` 设置

### 8.5 模型调用超时

**问题**：AI 模型调用超时

**解决方案**：
1. 检查网络连接（国内服务器可能无法直接访问 OpenAI）
2. 配置代理或使用国内模型
3. 调整超时设置：`GUNICORN_TIMEOUT=360`

### 8.6 内存不足

**问题**：服务频繁重启

**解决方案**：
1. 增加服务器内存
2. 减少 Worker 数量：`CELERY_WORKER_AMOUNT=1`
3. 配置数据库参数限制内存使用

### 8.7 镜像拉取缓慢

**问题**：Docker 镜像下载很慢

**解决方案**：
在 1Panel 中配置 Docker 镜像加速：

1. 进入 **容器 > 配置**
2. 在 **镜像加速** 中添加加速地址：
   ```
   https://registry.docker-cn.com
   https://docker.mirrors.ustc.edu.cn
   ```
3. 保存并重启 Docker

---

## 9. 升级与维护

### 9.1 升级 Dify

```bash
cd /opt/dify/dify/docker

# 拉取最新代码
git pull origin main

# 拉取最新镜像
docker compose pull

# 重启服务
docker compose up -d
```

### 9.2 数据备份

#### 备份数据库
```bash
# PostgreSQL 备份
docker compose exec db_postgres pg_dump -U postgres dify > backup_$(date +%Y%m%d).sql
```

#### 备份配置和文件
```bash
# 备份整个 docker 目录
tar -czvf dify_backup_$(date +%Y%m%d).tar.gz /opt/dify/dify/docker/volumes
```

### 9.3 日志管理

查看各服务日志：
```bash
# 查看 API 日志
docker compose logs -f api

# 查看 Worker 日志
docker compose logs -f worker

# 查看所有日志
docker compose logs -f
```

### 9.4 性能优化

**增加 Worker 数量**（高并发场景）：
```bash
# .env
SERVER_WORKER_AMOUNT=4
CELERY_WORKER_AMOUNT=4
```

**数据库优化**：
```bash
# .env
POSTGRES_MAX_CONNECTIONS=200
POSTGRES_SHARED_BUFFERS=256MB
POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
```

---

## 附录

### A. 完整服务端口参考

| 服务 | 内部端口 | 对外端口 | 说明 |
|------|----------|----------|------|
| Nginx | 80 | 80 | HTTP |
| Nginx | 443 | 443 | HTTPS |
| API | 5001 | - | API 服务 |
| Web | 3000 | - | Web 前端 |
| PostgreSQL | 5432 | - | 数据库 |
| Redis | 6379 | - | 缓存 |
| Weaviate | 8080 | - | 向量数据库 |
| Plugin Daemon | 5002 | 5002 | 插件服务 |
| Plugin Debug | 5003 | 5003 | 插件调试 |
| Sandbox | 8194 | - | 代码沙箱 |

### B. 相关链接

- [Dify 官方文档](https://docs.dify.ai)
- [Dify GitHub 仓库](https://github.com/langgenius/dify)
- [1Panel 官方文档](https://1panel.cn/docs/)
- [Docker 官方文档](https://docs.docker.com/)

### C. 技术支持

如遇到问题，可以通过以下方式获取帮助：

- [Dify GitHub Discussions](https://github.com/langgenius/dify/discussions)
- [Dify Discord 社区](https://discord.gg/FngNHpbcY7)
- [1Panel 社区论坛](https://bbs.fit2cloud.com/)

---

*本文档最后更新时间：2024年11月*
