---
title: "数据库初始化与管理：MongoDB 配置实践"
description: 🗄️ MongoDB 数据库初始化、用户管理与数据验证
date: 2025-04-11T18:43:55+08:00
hidden: false
comments: true
draft: true
slug: hugo-to-my-blog/chapter1/database-init
series:
  - 自主开发全栈博客系统全过程
---

## 🎯 目标

在本篇文章中，我们将完成：

- MongoDB 数据库初始化
- 用户权限管理
- 数据验证与索引创建
- 初始数据导入

## 🗄️ MongoDB 初始化配置

### 1. 初始化脚本

```javascript
// infra/docker/mongo/init-mongo.js
// ==================== 安全增强 ====================
// 1. 切换到目标数据库
db = db.getSiblingDB("tacticore");

// 2. 创建专用用户
db.createUser({
  user: "tacticore_user",
  pwd: "securepassword123", // 生产环境应从环境变量获取
  roles: [
    { role: "readWrite", db: "tacticore" }, // 应用数据库权限
    { role: "clusterMonitor", db: "admin" }, // 监控权限（可选）
  ],
});

// 3. 创建初始集合
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

// 4. 创建索引
db.users.createIndex({ email: 1 }, { unique: true });

// 5. 插入初始数据
db.users.insertOne({
  username: "admin",
  email: "admin@tacticore.com",
  createdAt: new Date(),
});

// 6. 验证结果
print("========== 初始化完成 ==========");
printjson(db.getUsers());
```

### 2. Docker 环境变量配置

```yaml
# docker-compose.yml
services:
  mongo:
    image: mongo:6
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=rootpassword123
    volumes:
      - mongo_data:/data/db
      - ./infra/docker/mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
```

## 🔐 用户权限管理

### 1. Root 用户（管理员）

- **定义位置**：`docker-compose.yml`
- **作用**：MongoDB 实例的初始管理员账户
- **配置示例**：
  ```yaml
  mongo:
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=rootpassword123
  ```

### 2. 应用用户

- **定义位置**：`init-mongo.js`
- **作用**：特定数据库的操作权限
- **配置示例**：
  ```javascript
  db.createUser({
    user: "tacticore_user",
    pwd: "securepassword123",
    roles: [{ role: "readWrite", db: "tacticore" }],
  });
  ```

## 🔍 数据验证与索引

### 1. 集合验证

```javascript
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
```

### 2. 索引创建

```javascript
// 唯一索引
db.users.createIndex({ email: 1 }, { unique: true });

// 复合索引
db.users.createIndex({ username: 1, email: 1 });

// 文本索引
db.users.createIndex({ username: "text", email: "text" });
```

## 🔄 连接字符串配置

### 1. 格式解析

```text
mongodb://<应用用户>:<应用密码>@mongo:27017/<数据库>?authSource=admin
            ↑              ↑             ↑            ↑
        来自 init-mongo.js  │      服务名(容器网络)  认证数据库
```

### 2. 环境变量配置

```env
MONGO_URI=mongodb://tacticore_user:securepassword123@mongo:27017/tacticore?authSource=admin
```

## 🔧 验证与调试

### 1. 连接数据库

```bash
# 使用 root 用户连接
docker exec -it mongo_container mongosh -u root -p rootpassword123

# 使用应用用户连接
docker exec -it mongo_container mongosh -u tacticore_user -p securepassword123 tacticore
```

### 2. 常用命令

```javascript
// 查看数据库
show dbs

// 切换数据库
use tacticore

// 查看集合
show collections

// 查看用户
show users

// 查看索引
db.users.getIndexes()
```

## 🛡️ 安全建议

1. **生产环境必须**：

   - 使用强密码生成器
   - 通过 Docker Secrets 管理密码
   - 定期轮换凭证

2. **开发环境建议**：
   ```javascript
   // init-mongo.js 使用占位符
   pwd: process.env.MONGO_APP_PASSWORD;
   ```
   ```yaml
   # docker-compose.yml
   mongo:
     environment:
       - MONGO_APP_PASSWORD=${MONGO_APP_PASSWORD}
   ```

## 📝 总结

通过本文，我们完成了：

1. MongoDB 数据库初始化配置
2. 用户权限管理
3. 数据验证与索引创建
4. 连接字符串配置
5. 安全最佳实践

在下一篇文章中，我们将详细介绍 Nginx 配置与优化。
