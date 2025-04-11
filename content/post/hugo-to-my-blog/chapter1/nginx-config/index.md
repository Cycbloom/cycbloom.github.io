---
title: "Nginx 配置与优化：反向代理与性能调优"
description: 🌐 Nginx 服务器配置、反向代理设置与性能优化实践
date: 2025-04-10T23:15:22+08:00
slug: hugo-to-my-blog/chapter1/nginx-config
series:
  - 自主开发全栈博客系统全过程
---

## 🎯 目标

在本篇文章中，我们将完成：

- Nginx 基础配置
- 反向代理设置
- 静态资源优化
- 性能调优

## 🌐 Nginx 基础配置

### 1. 主配置文件结构

```text
/etc/nginx/
├── nginx.conf            # 主配置文件（定义全局设置）
└── conf.d/               # 子配置目录
    └── default.conf      # 默认子配置（通常被你的 tacticore.conf 覆盖）
```

### 2. 项目专用配置

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

## 🔄 反向代理配置

### 1. 基本代理设置

```nginx
location /api/ {
    proxy_pass http://backend:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### 2. 代理优化配置

```nginx
# 缓冲区设置
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;

# 超时设置
proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;
```

## 📦 静态资源优化

### 1. Gzip 压缩

```nginx
# 启用 Gzip
gzip on;
gzip_vary on;
gzip_min_length 10240;
gzip_proxied expired no-cache no-store private auth;
gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml application/javascript;
gzip_disable "MSIE [1-6]\.";
```

### 2. 缓存控制

```nginx
# 静态资源缓存
location /static/ {
    expires 30d;
    add_header Cache-Control "public, no-transform";
}

# 禁用特定文件缓存
location ~* \.(html|htm)$ {
    expires -1;
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

## ⚡ 性能优化

### 1. 工作进程配置

```nginx
# nginx.conf
worker_processes auto;  # 自动根据 CPU 核心数分配
worker_rlimit_nofile 65535;  # 文件描述符限制

events {
    worker_connections 1024;
    multi_accept on;
    use epoll;  # Linux 系统使用 epoll
}
```

### 2. 缓冲区优化

```nginx
# 客户端缓冲区
client_body_buffer_size 10K;
client_header_buffer_size 1k;
client_max_body_size 8m;
large_client_header_buffers 2 1k;

# 文件缓存
open_file_cache max=1000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;
```

## 🔒 安全配置

### 1. 基本安全头

```nginx
# 安全相关响应头
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### 2. SSL 配置（生产环境）

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/example.com/privkey.pem;

    # SSL 配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

## 🔍 调试与监控

### 1. 访问日志格式

```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;
error_log /var/log/nginx/error.log warn;
```

### 2. 状态监控

```nginx
# 启用状态页面
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}
```

## 📝 总结

通过本文，我们完成了：

1. Nginx 基础配置
2. 反向代理设置
3. 静态资源优化
4. 性能调优
5. 安全配置
6. 监控设置

这些配置为我们的应用提供了：

- 高性能的静态资源服务
- 安全的反向代理
- 优化的缓存策略
- 完善的监控机制

在下一篇文章中，我们将详细介绍环境变量与配置管理。
