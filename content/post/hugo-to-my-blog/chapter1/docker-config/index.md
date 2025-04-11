---
title: "Docker 环境配置：容器化部署实践"
description: 🐳 使用 Docker 实现前后端服务的容器化部署
date: 2025-04-10T23:15:22+08:00
slug: hugo-to-my-blog/chapter1/docker-config
series:
  - 自主开发全栈博客系统全过程
---

## 🎯 目标

在本篇文章中，我们将完成：

- 前端 Docker 配置
- 后端 Docker 配置
- MongoDB 初始化配置
- Nginx 反向代理配置
- Docker Compose 服务编排

## 📂 项目结构

```text
TactiCore/
├── infra/
│   ├── docker/
│   │   ├── nginx/
│   │   │   ├── nginx.conf          # Nginx 主配置
│   │   │   └── tacticore.conf      # 项目专用配置
│   │   └── mongo/
│   │       └── init-mongo.js       # MongoDB 初始化脚本
├── docker-compose.yml               # 主编排文件
├── frontend/
│   ├── Dockerfile.prod             # 前端生产环境配置
│   └── Dockerfile.dev              # 前端开发环境配置
└── backend/
    ├── Dockerfile.prod             # 后端生产环境配置
    └── Dockerfile.dev              # 后端开发环境配置
```

## 🐳 Docker 配置详解

### 1. 前端 Docker 配置

#### 生产环境 (frontend/Dockerfile.prod)

```dockerfile
# 构建阶段
FROM node:22-alpine AS builder
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

# 生产阶段
FROM nginx:alpine AS prod
COPY --from=builder /app/dist /usr/share/nginx/html
COPY infra/docker/nginx/tacticore.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### 开发环境 (frontend/Dockerfile.dev)

```dockerfile
FROM node:22-alpine AS dev
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

### 2. 后端 Docker 配置

#### 生产环境 (backend/Dockerfile.prod)

```dockerfile
# 构建阶段
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:22-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --production
CMD ["node", "dist/main.js"]
```

#### 开发环境 (backend/Dockerfile.dev)

```dockerfile
FROM node:22-alpine AS development
WORKDIR /app
RUN npm install -g nodemon
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "run", "start:dev:docker"]
```

### 3. MongoDB 初始化配置

```javascript
// infra/docker/mongo/init-mongo.js
db = db.getSiblingDB("tacticore");

db.createUser({
  user: "tacticore",
  pwd: "securepassword123",
  roles: [
    { role: "readWrite", db: "tacticore" },
    { role: "clusterMonitor", db: "admin" },
  ],
});

// 创建初始集合
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["username", "email"],
      properties: {
        username: { bsonType: "string" },
        email: { bsonType: "string" },
      },
    },
  },
});

// 创建索引
db.users.createIndex({ email: 1 }, { unique: true });
```

### 4. Nginx 配置

```nginx
# infra/docker/nginx/tacticore.conf
server {
    listen 80;
    server_name localhost;

    # 前端路由处理
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    # 后端代理
    location /api/ {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 5. Docker Compose 配置

```yaml
# docker-compose.yml
services:
  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile.prod
      target: prod
    ports:
      - "8080:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    env_file:
      - ./backend/.env
    environment:
      - NODE_ENV=production
    volumes:
      - ./backend/.env:/app/.env
    depends_on:
      - mongo
    ports:
      - "3000:3000"

  mongo:
    image: mongo:6
    volumes:
      - mongo_data:/data/db
      - ./infra/docker/mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=rootpassword123
    ports:
      - "27017:27017"

volumes:
  mongo_data:
```

## 🚀 启动服务

1. 构建并启动所有服务：

```bash
docker-compose up --build
```

2. 访问服务：

- 前端：http://localhost:8080
- 后端健康检查：http://localhost:3000/health

## 🔍 验证部署

1. 检查容器状态：

```bash
docker-compose ps
```

2. 查看服务日志：

```bash
docker-compose logs -f
```

## 📝 总结

通过本文，我们完成了：

1. 前后端服务的 Docker 配置
2. MongoDB 数据库初始化
3. Nginx 反向代理设置
4. Docker Compose 服务编排

在下一篇文章中，我们将详细介绍如何优化 Docker 开发环境，实现热重载等功能。
