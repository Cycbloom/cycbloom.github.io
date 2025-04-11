---
title: "最小化后端实现：NestJS 框架搭建"
description: 🚀 从零开始搭建 NestJS 后端服务，实现核心功能模块
date: 2025-04-10T23:15:22+08:00
slug: hugo-to-my-blog/chapter1/minimal-backend
series:
  - 自主开发全栈博客系统全过程
---

## 🎯 目标

在本篇文章中，我们将实现一个最小化的后端服务，包含以下核心功能：

- 基础项目结构搭建
- 数据库连接配置
- 环境变量管理
- 健康检查接口

## 📦 依赖安装

首先，我们需要安装必要的依赖包：

```bash
# 核心依赖
npm install @nestjs/mongoose mongoose dotenv class-validator class-transformer
npm install @nestjs/config @nestjs/swagger @nestjs/passport passport-jwt

# 开发依赖
npm install --save-dev @types/passport-jwt
```

## ⚙️ 环境变量配置

创建 `.env` 文件，配置必要的环境变量：

```env
PORT=3000
MONGO_URI=mongodb://tacticore:securepassword123@mongo:27017/tacticore?authSource=admin
JWT_SECRET=your_jwt_secret_here
```

## 🏗️ 核心模块实现

### 1. 配置模块 (src/core/config/config.module.ts)

```typescript
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

### 2. 数据库模块 (src/database/database.module.ts)

```typescript
import { Module } from "@nestjs/common";
import { MongooseModule } from "@nestjs/mongoose";

@Module({
  imports: [
    MongooseModule.forRootAsync({
      useFactory: () => ({
        uri: process.env.MONGO_URI,
      }),
    }),
  ],
})
export class DatabaseModule {}
```

### 3. 健康检查控制器 (src/health/health.controller.ts)

```typescript
import { Controller, Get } from "@nestjs/common";
import { ApiOperation, ApiResponse } from "@nestjs/swagger";

@Controller("health")
export class HealthController {
  @Get()
  @ApiOperation({ summary: "服务健康检查" })
  @ApiResponse({ status: 200, description: "服务运行正常" })
  checkHealth() {
    return {
      status: "UP",
      timestamp: new Date().toISOString(),
    };
  }
}
```

## 🚀 应用入口配置

### 主模块 (src/app.module.ts)

```typescript
import { Module } from "@nestjs/common";
import { ConfigurationModule } from "./core/config/config.module";
import { DatabaseModule } from "./database/database.module";
import { HealthController } from "./health/health.controller";

@Module({
  imports: [ConfigurationModule, DatabaseModule],
  controllers: [HealthController],
})
export class AppModule {}
```

### 入口文件 (src/main.ts)

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
    })
  );

  await app.listen(process.env.PORT || 3000);
  console.log(`Application running on: ${await app.getUrl()}`);
}
bootstrap();
```

## 🔍 验证与测试

1. 启动服务：

```bash
npm run start:dev
```

2. 测试健康检查接口：

```bash
curl http://localhost:3000/health
```

预期响应：

```json
{
  "status": "UP",
  "timestamp": "2024-04-11T01:15:22.000Z"
}
```

## 📝 总结

通过本文，我们完成了：

1. 项目基础架构搭建
2. 核心依赖配置
3. 数据库连接设置
4. 环境变量管理
5. 基础健康检查接口

这个最小化的后端实现为我们后续的功能开发提供了坚实的基础。在下一篇文章中，我们将详细介绍 Docker 环境的配置。
