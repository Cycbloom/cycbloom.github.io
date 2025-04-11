---
title: "环境变量与配置管理：安全与验证"
description: 🔐 使用 Joi 进行环境变量管理与验证
date: 2025-04-10T23:15:22+08:00
slug: hugo-to-my-blog/chapter1/env-config
series:
  - 自主开发全栈博客系统全过程
---

## 🎯 目标

在本篇文章中，我们将完成：

- 环境变量管理
- 配置验证
- 安全性考虑
- 最佳实践

## 📦 依赖安装

首先，我们需要安装必要的依赖：

```bash
npm install joi @types/joi
```

## ⚙️ 环境变量配置

### 1. 基础配置

```env
# .env
PORT=3000
MONGO_URI=mongodb://tacticore:securepassword123@mongo:27017/tacticore?authSource=admin
JWT_SECRET=your_jwt_secret_here
NODE_ENV=development
```

### 2. 开发环境配置

```env
# .env.development
PORT=3000
MONGO_URI=mongodb://tacticore:securepassword123@mongo:27017/tacticore?authSource=admin
JWT_SECRET=dev_secret_key
NODE_ENV=development
DEBUG=true
```

### 3. 生产环境配置

```env
# .env.production
PORT=3000
MONGO_URI=mongodb://tacticore:securepassword123@mongo:27017/tacticore?authSource=admin
JWT_SECRET=production_secret_key
NODE_ENV=production
DEBUG=false
```

## 🔍 配置验证

### 1. 使用 Joi 进行验证

```typescript
// src/core/config/config.module.ts
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import * as Joi from "joi";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ".env",
      validationSchema: Joi.object({
        PORT: Joi.number().default(3000),
        MONGO_URI: Joi.string().required(),
        JWT_SECRET: Joi.string().min(16).required(),
        NODE_ENV: Joi.string()
          .valid("development", "production", "test")
          .default("development"),
        DEBUG: Joi.boolean().default(false),
      }),
      validationOptions: {
        allowUnknown: true,
        abortEarly: true,
      },
    }),
  ],
  exports: [ConfigModule],
})
export class ConfigurationModule {}
```

### 2. 验证规则说明

| 规则                           | 作用                    |
| ------------------------------ | ----------------------- |
| `Joi.number().default(3000)`   | 必须是数字，默认值 3000 |
| `Joi.string().required()`      | 必填字符串              |
| `Joi.string().min(16)`         | 最小长度 16 的字符串    |
| `Joi.string().valid(...)`      | 限定允许的枚举值        |
| `Joi.boolean().default(false)` | 布尔值，默认 false      |

## 🔐 安全性考虑

### 1. 敏感信息处理

```typescript
// src/core/config/config.service.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";

@Injectable()
export class AppConfigService {
  constructor(private configService: ConfigService) {}

  get isDevelopment(): boolean {
    return this.configService.get("NODE_ENV") === "development";
  }

  get mongoUri(): string {
    return this.configService.get<string>("MONGO_URI");
  }

  get jwtSecret(): string {
    return this.configService.get<string>("JWT_SECRET");
  }
}
```

### 2. 环境变量加载顺序

```typescript
ConfigModule.forRoot({
  envFilePath: [".env.${process.env.NODE_ENV}", ".env"],
});
```

## 🛠️ 最佳实践

### 1. 配置分层

```typescript
// src/core/config/database.config.ts
export const databaseConfig = () => ({
  uri: process.env.MONGO_URI,
  options: {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
});

// src/core/config/jwt.config.ts
export const jwtConfig = () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: "1d",
});
```

### 2. 配置验证错误处理

```typescript
// src/core/config/config.module.ts
ConfigModule.forRoot({
  validationSchema: Joi.object({...}),
  validationOptions: {
    allowUnknown: true,
    abortEarly: true,
  },
  load: [databaseConfig, jwtConfig],
  isGlobal: true,
  envFilePath: '.env',
  expandVariables: true,
})
```

## 🔧 常见问题与解决方案

### 1. 环境变量未加载

**问题**：环境变量未被正确加载。

**解决方案**：

```typescript
// 确保在应用启动时加载环境变量
import * as dotenv from "dotenv";
dotenv.config();
```

### 2. 类型安全问题

**问题**：TypeScript 类型检查不完整。

**解决方案**：

```typescript
// 定义配置接口
interface AppConfig {
  port: number;
  mongoUri: string;
  jwtSecret: string;
  nodeEnv: string;
}

// 使用类型断言
const config = {
  port: parseInt(process.env.PORT, 10),
  mongoUri: process.env.MONGO_URI,
  jwtSecret: process.env.JWT_SECRET,
  nodeEnv: process.env.NODE_ENV,
} as AppConfig;
```

## 📝 总结

通过本文，我们完成了：

1. 环境变量管理
2. 配置验证
3. 安全性考虑
4. 最佳实践
5. 常见问题解决方案

这些配置为我们的应用提供了：

- 安全的配置管理
- 类型安全的配置访问
- 环境特定的配置
- 配置验证机制

至此，我们已经完成了整个项目的基础架构搭建。在接下来的章节中，我们将开始实现具体的业务功能。
