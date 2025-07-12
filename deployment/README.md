# Afilmory Docker 部署指南

这个目录包含了使用 Docker 和 Docker Compose 部署 Afilmory 照片库应用所需的配置文件。

## 📁 目录结构

```
deployment/
├── docker-compose.yml       # Docker Compose 主配置文件
├── .env.example            # 环境变量示例文件
├── .env.production         # 生产环境配置模板
├── .env.local              # 本地存储模式示例
├── config.json             # 应用配置文件
├── builder.config.json     # 构建器配置文件
├── README.md              # 本文档
├── photos/                 # 本地图片目录 (本地存储模式)
└── data/                   # 数据持久化目录 (运行时创建)
    ├── postgres/           # 数据库文件
    ├── thumbnails/         # 缩略图文件
    └── manifest/           # 清单文件
```

## 🚀 快速开始

### 1. 准备配置文件

```bash
# 复制环境变量文件
cp .env.example .env

# 复制配置文件 (如果不存在)
cp config.example.json config.json
cp builder.config.example.json builder.config.json

# 编辑配置文件，填入实际值
nano .env
nano config.json
nano builder.config.json
```

### 2. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f afilmory
```

### 3. 访问应用

- **Web界面**: http://localhost:3000
- **数据库**: localhost:5432

## ⚙️ 服务配置

### 服务组件

- **afilmory**: 主应用 (Next.js + React)
- **postgres**: PostgreSQL 数据库

## 🔧 配置说明

### 配置文件

应用使用以下配置文件：

1. **config.json**: 应用基本配置
   - 网站名称、标题、描述
   - 作者信息、头像
   - 主题色彩配置
   - 地图组件配置

2. **builder.config.json**: 构建器配置
   - 存储提供商设置
   - S3存储桶配置
   - 存储路径前缀

### 环境变量

| 变量名 | 说明 | 必需 |
|--------|------|------|
| `POSTGRES_PASSWORD` | PostgreSQL 数据库密码 | ✅ |
| `USE_LOCAL_STORAGE` | 使用本地存储模式 (true/false) | ❌ |
| `PHOTOS_PATH` | 本地photos目录路径 | ❌ |
| `S3_ACCESS_KEY_ID` | S3 访问密钥 ID | ❌ |
| `S3_SECRET_ACCESS_KEY` | S3 访问密钥 | ❌ |
| `S3_BUCKET_NAME` | S3 存储桶名称 | ❌ |
| `S3_REGION` | S3 区域 | ❌ |
| `S3_ENDPOINT` | S3 端点 URL | ❌ |
| `S3_CUSTOM_DOMAIN` | 自定义 CDN 域名 | ❌ |
| `GIT_TOKEN` | GitHub Personal Access Token | ❌ |

### 存储配置

应用支持两种存储模式：

#### 1. 本地存储模式
```bash
# 在 .env 文件中设置
USE_LOCAL_STORAGE=true
PHOTOS_PATH=./photos  # 或使用绝对路径如 /path/to/your/photos
```

- 将本地photos目录映射到容器中
- 适合小规模部署或测试环境
- 图片文件直接存储在本地文件系统

#### 2. S3云存储模式 (默认)
```bash
# 在 .env 文件中设置
USE_LOCAL_STORAGE=false
S3_ACCESS_KEY_ID=your_access_key
S3_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET_NAME=your-bucket
```

支持的S3兼容存储服务：
- **AWS S3**: 使用默认配置
- **阿里云OSS**: 设置自定义 `S3_ENDPOINT`
- **腾讯云COS**: 设置自定义 `S3_ENDPOINT`
- **其他S3兼容存储**: 设置自定义 `S3_ENDPOINT`

## 📋 常用命令

### 服务管理

```bash
# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 重启特定服务
docker-compose restart afilmory

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f [service_name]

# 进入容器
docker-compose exec afilmory bash
```

### 数据库操作

```bash
# 连接到数据库
docker-compose exec postgres psql -U afilmory -d afilmory

# 运行数据库迁移 (如果需要)
docker-compose exec afilmory pnpm --filter @afilmory/ssr db:migrate

# 生成数据库迁移
docker-compose exec afilmory pnpm --filter @afilmory/ssr db:generate
```

### 应用操作

```bash
# 构建照片清单
docker-compose exec afilmory pnpm run build:manifest

# 查看应用配置
docker-compose exec afilmory pnpm --filter @afilmory/builder cli --config
```

## 🔄 更新应用

```bash
# 拉取最新镜像
docker-compose pull

# 重启服务
docker-compose up -d

# 清理旧镜像
docker image prune
```

## 💡 使用提示

### 本地存储模式
1. **创建photos目录**: 在deployment目录下创建photos文件夹，并将图片放入其中
2. **设置环境变量**: 在.env文件中设置 `USE_LOCAL_STORAGE=true`
3. **目录结构**: 
   ```
   deployment/
   ├── photos/           # 你的图片文件
   │   ├── IMG_001.jpg
   │   ├── IMG_002.png
   │   └── ...
   ├── data/             # 自动生成的数据目录
   │   ├── thumbnails/   # 缩略图文件
   │   └── manifest/     # 清单文件
   ├── docker-compose.yml
   └── .env
   ```
4. **构建清单**: 启动后运行 `docker-compose exec afilmory pnpm run build:manifest`

### S3存储模式
1. **环境变量**: 确保填写正确的S3存储配置信息
2. **存储桶权限**: 确保S3存储桶有正确的访问权限
3. **构建清单**: 启动后运行 `docker-compose exec afilmory pnpm run build:manifest`

### 通用提示
1. **首次部署**: 
   ```bash
   # 创建必要目录
   mkdir -p photos data/postgres data/thumbnails data/manifest
   
   # 复制配置文件
   cp .env.local .env  # 本地存储模式
   cp config.example.json config.json
   cp builder.config.example.json builder.config.json
   
   # 编辑配置文件根据需要调整
   
   # 启动服务
   docker-compose up -d
   ```

2. **数据持久化**: 
   - 所有数据都保存在本地 `./data/` 目录中
   - 重新部署会保留所有生成的文件
   - 可以直接备份整个deployment目录

3. **目录权限**: 确保Docker有权限访问photos目录和创建data目录

---

**注意**: 请根据实际需求调整配置文件中的参数设置。