---
title: "Chapter1: 项目初始化与基础架构搭建"
description: 📦 从零开始搭建全栈博客系统的基础架构
date: 2025-04-11T22:00:00+08:00
slug: hugo-to-my-blog/chapter1
tag:
  - 技术
  - React
  - NestJs
  - Docker
series:
  - 自主开发全栈博客系统全过程
image:
math:
license:
hidden: false
comments: true
---

## 🎬 初始化

### 🔧 初始化过程详解

1️⃣ **版本控制初始化**  
📦 使用 `git init` 初始化 Git 仓库  
🚫 创建 `.gitignore` 文件排除以下内容：

```bash
node_modules/  # 第三方依赖
.env           # 敏感配置
.DS_Store      # Mac系统文件
*.log          # 日志文件
dist/          # 构建产物
coverage/      # 测试报告
```

2️⃣ **前端脚手架搭建**  
⚡ 通过 Vite 快速生成项目：

```bash
npm create vite@latest
```

▸ 项目名称：`frontend`  
▸ 框架选择：`React` + `TypeScript`  
💡 优势：组件化开发 + 类型安全

3️⃣ **核心依赖安装**  
🔌 进入项目目录执行安装：

```bash
cd frontend && npm install
```

📚 关键依赖说明：

| 依赖包             | 功能                   |
| ------------------ | ---------------------- |
| `@mui/material`    | Material Design 组件库 |
| `react-router-dom` | SPA 路由管理           |
| `@reduxjs/toolkit` | 状态管理工具           |
| `axios`            | HTTP 客户端            |

4️⃣ **后端服务初始化**  
🛠️ 创建 NestJS 项目：

```bash
mkdir backend && cd backend
npm install -g @nestjs/cli
nest new . --package-manager npm
```

### 📂 项目结构规范

项目结构的合理规划对于项目的长期维护和扩展十分关键。我们的全栈博客系统采用如下结构：

```tree
TactiCore/
├── .github/                   # GitHub 配置
│   ├── workflows/             # CI/CD 流水线
│   └── ISSUE_TEMPLATE/        # Issue 模板
├── frontend/                  # 前端 (Vite + React)
│   ├── public/
│   │   ├── favicons/          # 多尺寸网站图标
│   │   └── robots.txt
│   └── src/
│       ├── api/               # API 请求封装
│       ├── assets/            # 静态资源
│       │   ├── fonts/
│       │   ├── images/
│       │   └── styles/        # 全局样式
│       ├── components/        # 组件库
│       │   ├── ui/            # 原子组件
│       │   ├── shared/        # 业务组件
│       │   └── providers/     # Context 提供者
│       ├── features/          # Redux Toolkit 模块
│       ├── hooks/             # 自定义 Hooks
│       ├── layouts/           # 页面布局
│       ├── locales/           # 国际化
│       │   ├── en/
│       │   └── zh/
│       ├── pages/             # 路由页面
│       ├── stores/            # 状态管理
│       ├── test/              # 测试套件
│       │   ├── e2e/           # 端到端测试
│       │   ├── integration/
│       │   ├── unit/
│       │   └── mocks/         # Mock 数据
│       ├── theme/             # 主题系统
│       │   ├── dark/
│       │   └── light/
│       ├── types/             # TS 类型定义
│       ├── utils/             # 工具函数
│       │   ├── helpers/
│       │   ├── validation/
│       │   └── performance/   # 性能监控
│       └── main.tsx
├── backend/                   # 后端 (NestJS + MongoDB)
│   ├── src/
│   │   ├── app.module.ts      # 根模块
│   │   ├── main.ts            # 入口文件
│   │   ├── core/              # 核心模块
│   │   │   ├── config/        # 环境配置
│   │   │   │   └── config.module.ts
│   │   │   ├── exceptions/    # 全局异常处理
│   │   │   └── decorators/    # 自定义装饰器
│   │   ├── database/          # 数据库模块
│   │   │   ├── database.module.ts
│   │   │   └── schemas/       # MongoDB Schema
│   │   ├── users/             # 用户模块示例
│   │   │   ├── dto/           # 数据传输对象
│   │   │   ├── entities/      # 数据库实体
│   │   │   ├── users.module.ts
│   │   │   ├── users.service.ts
│   │   │   └── users.controller.ts
│   │   ├── common/            # 通用工具
│   │   │   ├── filters/       # 异常过滤器
│   │   │   ├── interceptors/  # 拦截器
│   │   │   └── pipes/         # 数据管道
│   │   └── health/            # 健康检查模块
│   ├── test/                  # 测试套件
│   │   ├── e2e/              # 端到端测试
│   │   └── unit/             # 单元测试
│   ├── docker/               # Docker 配置
│   │   └── entrypoint.sh     # 容器启动脚本
│   ├── .env.example          # 环境变量模板
│   └── nest-cli.json         # Nest CLI 配置
├── docs/                     # 文档中心
│   ├── api.md               # API 文档
│   ├── architecture.md      # 架构设计
│   ├── setup.md             # 环境配置
│   └── CHANGELOG.md         # 更新日志
├── infra/                   # 基础设施
│   ├── docker/              # Docker 配置
│   │   ├── nginx/
│   │   └── mongo/
│   ├── k8s/                 # Kubernetes 配置
│   └── monitoring/          # 监控配置
│       ├── prometheus/
│       └── grafana/
├── scripts/                 # 自动化脚本
├── .vscode/                 # IDE 配置
├── .husky/                  # Git hooks
├── .editorconfig            # 编辑器规范
├── .eslintrc                # 代码检查
├── .prettierrc              # 代码格式化
├── .env.example             # 环境变量模板
├── .gitattributes           # Git 配置
├── .gitmessage              # 提交信息模板
├── docker-compose.yml       # 容器编排
├── package.json             # 全局脚本
├── CONTRIBUTING.md          # 贡献指南
└── README.md                # 项目总览
```

采用前后端分离模式，前端基于 React+vite 生态，后端使用 NestJS 框架。前端的`src`目录中涵盖了从基础的 API 封装、静态资源管理，到复杂的组件库构建、状态管理以及国际化等功能模块。后端的`src`目录则包含了根模块、核心模块（如环境配置、异常处理）、数据库模块、用户模块示例以及通用工具等。合理的项目结构设计为后续的开发、测试、部署以及系统扩展都提供了坚实的基础，能够显著提升开发效率与代码的可维护性。

## 🚀 后端 MVP

在完成第一部分的项目初始化后，我们为全栈博客系统搭建起了初步框架，接下来便要进入后端的 MVP（最小可行产品）实现阶段，着手搭建后端基础项目结构，这是使后端服务能够正常运行并提供基础功能的关键一步。

### ⚙️ 核心配置

1️⃣ **安装必要依赖**：为了构建功能完备的后端服务，我们需要安装一系列必要的依赖包。

- `@nestjs/mongoose`和`mongoose`用于连接和操作 MongoDB 数据库，这对于存储博客数据至关重要。
- `dotenv`能帮助我们加载环境变量，便于管理不同环境下的配置信息。
- `class-validator`和`class-transformer`用于数据验证和转换，保障数据的准确性和一致性。
- `@nestjs/config`用于管理应用的配置，`@nestjs/swagger`用于生成美观且实用的 API 文档，`@nestjs/passport`和`passport-jwt`则用于实现基于 JWT 的身份验证功能。
- 开发环境下，安装`@types/passport-jwt`以获取类型定义，方便 TypeScript 项目开发。

```bash
npm install @nestjs/mongoose mongoose dotenv class-validator class-transformer
npm install @nestjs/config @nestjs/swagger @nestjs/passport passport-jwt
npm install --save-dev @types/passport-jwt
```

2️⃣ **环境变量配置 (.env)**：合理配置环境变量能使我们的后端服务适应不同的运行环境。在`.env`文件中，我们设置了服务运行的端口`PORT`，这里默认设为`3000`。`MONGO_URI`用于指定 MongoDB 数据库的连接地址，需填入正确的数据库地址及相关认证信息。`JWT_SECRET`则是用于 JWT 身份验证的密钥，务必妥善保管，不要泄露。

```env
PORT=3000
MONGO_URI=mongodb://tacticore:pwd_tacticore@mongo:27017/tacticore?authSource=admin
JWT_SECRET=your_jwt_secret_here
```

### 🧩 实现核心模块

1. **创建基础模块结构**：利用 Nest CLI 工具，我们可以快速创建基础模块结构。`nest generate module core/config`用于创建配置模块，管理应用的各种配置信息；`nest generate module database`创建数据库模块，负责与数据库的连接和交互；`nest generate controller health`则创建了健康检查控制器，用于监控后端服务的运行状态。

   ```bash
   nest generate module core/config
   nest generate module database
   nest generate controller health
   ```

2. **配置模块 (src/core/config/config.module.ts)**：在配置模块中，我们引入了`@nestjs/config`模块，并使用`Joi`库对环境变量进行验证。`Joi`是一个流行的 JavaScript 数据验证库，在 NestJS 中常与`@nestjs/config`结合使用。首先需要安装`joi`及其类型定义（TypeScript 项目需要）：

   ```bash
   npm install joi @types/joi
   ```

   安装完成后，在配置模块中引入并定义验证规则：

   ```typescript
   // src/config/config.module.ts
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

   这里的验证规则如下：

   - `Joi.number().default(3000)`：表示`PORT`必须是数字，默认值为`3000`。
   - `Joi.string().required()`：说明`MONGO_URI`是必填的字符串。
   - `Joi.string().min(16)`：要求`JWT_SECRET`是最小长度为`16`的字符串。
   - `Joi.string().valid(...)`：限定`NODE_ENV`只能是`development`、`production`、`test`中的一个，默认值为`development`。
     如果`.env`文件缺少`MONGO_URI`或`JWT_SECRET`等必填项，启动时就会报错，例如：

   ```text
   Error: Config validation error: "MONGO_URI" is required
   ```

3. **数据库模块 (src/database/database.module.ts)**：数据库模块负责连接到 MongoDB 数据库。通过`MongooseModule.forRootAsync`方法，我们可以在运行时动态配置数据库连接字符串，这里从环境变量`process.env.MONGO_URI`中获取连接地址。

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

### 🩺 健康检查端点 (src/health/health.controller.ts)

1. **健康检查端点**：健康检查端点用于检查后端服务是否正常运行。通过`@Controller('health')`装饰器定义了控制器的路由前缀为`health`。`@Get()`装饰器表示该方法处理 HTTP GET 请求，`@ApiOperation`和`@ApiResponse`装饰器用于在 Swagger 文档中描述该接口的功能和响应信息。当访问`/health`端点时，会返回服务的状态和当前时间戳。

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

### 🔧 调整入口文件 (src/main.ts)

在入口文件`src/main.ts`中，我们创建了 Nest 应用实例，并使用`app.useGlobalPipes`方法启用了全局验证管道。该管道配置了`whitelist: true`，表示只接受验证通过的属性，`forbidNonWhitelisted: true`则禁止接受未通过验证的属性，从而确保输入数据的合法性。最后，应用监听指定端口，并在控制台输出应用运行的地址。

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 全局验证管道
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

### ✅ 验证步骤

完成上述配置后，我们可以通过以下方式验证后端服务是否正常运行：

1. **运行测试**：执行`npm run test`命令，运行项目中的测试用例，确保各个模块的功能符合预期。
2. **启动开发模式**：使用`npm run start:dev`命令，以开发模式启动后端服务，便于在开发过程中实时查看代码变更带来的效果。
3. **验证端点**：通过`curl http://localhost:3000/health`命令，向健康检查端点发送请求，验证服务是否正常运行。如果服务正常，将返回类似如下的结果：

```json
{
  "status": "UP",
  "timestamp": "2025-04-11T12:00:00.000Z"
}
```

### 🛠️ 推荐工具链

为了进一步提升后端开发效率和服务质量，我们可以引入一些工具：

| 工具           | 用途             | 安装命令                            |
| -------------- | ---------------- | ----------------------------------- |
| NestJS Swagger | API 文档生成     | `npm install @nestjs/swagger`       |
| NestJS CQRS    | 复杂业务逻辑管理 | `npm install @nestjs/cqrs`          |
| NestJS Cache   | 缓存管理         | `npm install @nestjs/cache-manager` |
| Jest           | 单元测试         | 内置                                |
| Compodoc       | 项目文档生成     | `npm install @compodoc/compodoc`    |

NestJS Swagger 可以自动生成详细的 API 文档，方便团队协作和接口调试；NestJS CQRS 有助于管理复杂的业务逻辑；NestJS Cache 用于缓存频繁访问的数据，提高系统性能；Jest 作为内置的单元测试框架，能够保证代码质量；Compodoc 则可以生成全面的项目文档，方便团队成员了解项目结构和功能。

通过以上步骤，我们初步完成了后端的 MVP 实现，搭建起了基础项目结构，为后续添加更多业务功能奠定了坚实基础。

## 🐳 容器化部署

在完成第二部分后端基础项目结构搭建后，为了实现项目的容器化部署，便于在不同环境中快速、稳定地运行，接下来我们将深入探讨 Docker 配置相关内容。Docker 能够将应用及其依赖打包成一个独立的镜像，确保在任何环境中都能以相同的方式运行，大大提升了项目部署的效率和一致性。

### 📁 文件结构预览

根据项目结构规划，Docker 相关配置主要分布在以下位置：

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
|       ├── Dockerfile.prod                # 前端镜像配置
|       └── Dockerfile.dev
└── backend/
        ├── Dockerfile.prod                # 后端镜像配置
        └──Dockerfile.dev
```

下面我们详细介绍具体的配置步骤：

### 🖥️ 前端镜像配置 (frontend/Dockerfile.prod)

```dockerfile
# frontend/Dockerfile.prod
# ================= 构建阶段 =================
FROM node:22-alpine AS builder
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

# ================= 生产阶段 =================
FROM nginx:alpine AS prod
COPY --from=builder /app/dist /usr/share/nginx/html
COPY infra/docker/nginx/tacticore.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

1. **构建阶段**：基于`node:22-alpine`镜像创建一个名为`builder`的构建阶段。设置工作目录为`/app`，将前端项目的`package*.json`文件复制到容器内，执行`npm ci`安装依赖。这里使用`npm ci`而非`npm install`，`npm ci`是专门为自动化环境设计的依赖安装命令，它严格按照`package-lock.json`的版本安装依赖，不修改任何文件，能确保生产环境与开发环境的依赖完全一致，避免“在我机器上是好的”问题，同时显著加快 Docker 镜像构建速度。接着将整个前端项目复制到容器内，并执行`npm run build`进行项目构建。
2. **生产阶段**：基于`nginx:alpine`镜像创建生产阶段。从`builder`阶段复制编译好的前端文件到`/usr/share/nginx/html`目录，这是 Nginx 默认的静态资源目录。同时，将项目自定义的 Nginx 配置文件`infra/docker/nginx/tacticore.conf`复制到`/etc/nginx/conf.d/default.conf`，覆盖默认的 Nginx 配置。通过`EXPOSE 80`声明容器监听端口为 80，最后使用`CMD ["nginx", "-g", "daemon off;"]`以前台模式启动 Nginx 服务，确保 Docker 容器运行时 Nginx 服务处于运行状态。

### 🗄️ 后端镜像配置 (backend/Dockerfile.prod)

```dockerfile
# 开发阶段
FROM node:22-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:22-alpine AS production
WORKDIR /app
COPY --from=development /app/dist ./dist
COPY --from=development /app/package*.json ./
RUN npm ci --production
CMD ["node", "dist/main.js"]
```

1. **开发阶段**：基于`node:22-alpine`镜像创建名为`development`的开发阶段。设置工作目录为`/app`，复制后端项目的`package*.json`文件，执行`npm ci`安装依赖，再将整个后端项目复制到容器内，并执行`npm run build`进行项目构建。
2. **生产阶段**：同样基于`node:22-alpine`镜像创建`production`阶段。从`development`阶段复制编译后的文件到`/app/dist`目录，同时复制`package*.json`文件。执行`npm ci --production`安装生产环境所需的依赖，这里`--production`参数会跳过安装`devDependencies`。最后通过`CMD ["node", "dist/main.js"]`启动后端服务。

### 🧩 数据库初始化 (infra/docker/mongo/init-mongo.js)

```javascript
// ==================== 安全增强 ====================
// 1. 切换到目标数据库（显式创建数据库）
db = db.getSiblingDB("tacticore");

// 2. 创建专用用户
db.createUser({
  user: "tacticore",
  pwd: "pwd-tacticore", // 生产环境应从环境变量获取
  roles: [
    { role: "readWrite", db: "tacticore" }, // 应用数据库权限
    { role: "clusterMonitor", db: "admin" }, // 监控权限（可选）
  ],
});

// 3. 创建初始集合（可选）
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

// 4. 创建索引（可选）
db.users.createIndex({ email: 1 }, { unique: true });

// 5. 插入初始数据（可选）
db.users.insertOne({
  username: "admin",
  email: "admin@tacticore.com",
  createdAt: new Date(),
});

// 6. 验证结果（调试用）
print("========== 初始化完成 ==========");
printjson(db.getUsers());
```

在这个脚本中，首先切换到目标数据库`tacticore`，若数据库不存在则会自动创建。然后创建一个专用用户`tacticore`，并赋予其在`tacticore`数据库的读写权限以及在`admin`数据库的监控权限（可选）。接着可以根据项目需求创建初始集合`users`，并定义集合的数据校验规则，要求`username`和`email`字段为必填，且类型为字符串。为`email`字段创建唯一索引，确保`email`的唯一性。还可以插入一条初始数据用于测试。最后输出初始化完成的信息，并打印当前用户列表用于调试。在生产环境中，密码应从环境变量获取，以提高安全性。

### 🌐 Nginx 配置 (infra/docker/nginx/tacticore.conf)

```nginx
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

Nginx 在项目中承担着重要角色。通过这个配置文件，Nginx 监听端口 80。在前端路由处理方面，当用户访问根路径时，Nginx 会从`/usr/share/nginx/html`目录查找对应的文件或目录，若未找到则返回`index.html`，这对于单页应用（SPA）的路由处理非常关键。在后端代理方面，当接收到以`/api/`开头的请求时，Nginx 会将请求转发到`http://backend:3000`，同时设置`Host`和`X-Real-IP`头部信息，隐藏后端真实端口和细节，增强安全性。

### 🎛 docker-compose 编排

```yaml
services:
  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile.prod
      target: prod # 默认生产环境
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

`docker-compose.yml`用于编排多个 Docker 容器服务。这里定义了三个服务：

1. **frontend**：基于前端项目根目录构建镜像，使用`frontend/Dockerfile.prod`，目标为生产环境。将容器的 80 端口映射到主机的 8080 端口，并且依赖`backend`服务，确保`backend`服务先启动。
2. **backend**：基于后端项目目录构建镜像，使用`backend/Dockerfile.prod`。通过`env_file`加载`./backend/.env`文件中的环境变量，设置`NODE_ENV`为`production`。将主机的`./backend/.env`文件挂载到容器内的`/app/.env`，实现环境变量的同步。依赖`mongo`服务，确保`mongo`服务先启动。将容器的 3000 端口映射到主机的 3000 端口。
3. **mongo**：使用`mongo:6`官方镜像。将`mongo_data`卷挂载到容器内的`/data/db`目录，用于持久化存储 MongoDB 数据。将`./infra/docker/mongo/init-mongo.js`文件只读挂载到`/docker-entrypoint-initdb.d/init-mongo.js`，在容器启动时会自动执行该脚本进行 MongoDB 的初始化。设置`MONGO_INITDB_ROOT_USERNAME`和`MONGO_INITDB_ROOT_PASSWORD`环境变量，用于初始化 MongoDB 的 root 用户。将容器的 27017 端口映射到主机的 27017 端口。

### 🚀 启动命令

完成上述配置后，可以使用以下命令启动项目：

```bash
# 构建并启动所有服务
docker-compose up --build

# 访问前端
http://localhost:8080

# 验证服务
docker-compose ps
```

`docker-compose up --build`命令会构建所有服务的镜像并启动容器。构建过程中会根据各个`Dockerfile`的配置进行操作。构建完成后，前端服务可通过`http://localhost:8080`访问。`docker-compose ps`命令用于查看当前正在运行的容器服务状态，确保各个服务正常启动。

通过以上详细的 Docker 配置，我们能够将全栈博客系统的前端、后端以及数据库服务以容器化的方式进行部署和管理，为项目的稳定运行和后续扩展提供有力支持。

## ⚡ Docker 热更新

在完成第三部分的 Docker 配置后，项目已经可以以容器化的方式稳定部署和运行。但在开发过程中，频繁修改代码后都重新构建和启动容器会极大地降低开发效率。因此，为了提升开发体验，接下来我们将实现 Docker 热重载，这样在修改代码后，无需重启容器就能实时看到代码变更的效果。

### 🖥️ 前端热更新实现方案

#### 调整前端 Dockerfile

```dockerfile
# frontend/Dockerfile.dev
# ================= 开发阶段 =================
FROM node:22-alpine AS dev
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

这里创建了一个专门用于开发环境的 `Dockerfile.dev`。基于`node:22-alpine`镜像，设置工作目录为`/app`，复制前端项目的`package*.json`文件并执行`npm install`安装依赖。暴露端口 5173，这是 Vite 开发服务器默认的端口。最后启动 Vite 开发服务器。

#### 添加 docker-compose.dev.yml

```yaml
services:
  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile.dev
      target: dev
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "5173:5173"
    environment:
      - NODE_ENV=development
```

在`docker-compose.dev.yml`中，定义了前端服务在开发环境下的配置。通过`volumes`将本地的`./frontend`目录挂载到容器内的`/app`目录，实现代码的实时同步。同时，使用`/app/node_modules`防止本地的`node_modules`覆盖容器内的依赖。将容器的 5173 端口映射到主机的 5173 端口，方便在本地访问开发服务器。设置环境变量`NODE_ENV`为`development`。

#### 配置 Vite 开发服务器

```javascript
// frontend/vite.config.ts
export default defineConfig({
  server: {
    host: true, // 允许外部访问
    port: 5173,
    watch: {
      usePolling: true, // 解决容器内文件监听问题
    },
  },
});
```

在 Vite 配置文件中，设置`host: true`允许外部访问开发服务器，`port: 5173`指定开发服务器的端口。`watch.usePolling: true`解决了在 Docker 容器内文件监听失效的问题，通过轮询的方式检测文件变化。

### 🔄 前端热更新工作原理

```mermaid
sequenceDiagram
    participant 本地编辑器
    participant Docker容器
    participant 浏览器

    本地编辑器->>Docker容器: 修改代码文件
    Docker容器->>Vite服务器: 检测文件变化
    Vite服务器->>Vite服务器: 增量编译
    Vite服务器->>浏览器: 通过 WebSocket 推送更新
    浏览器->>浏览器: 自动刷新页面
```

当在本地编辑器修改代码文件后，由于`volumes`的挂载，容器内的代码也会实时更新。Vite 服务器检测到文件变化后进行增量编译，然后通过 WebSocket 将更新推送给浏览器，浏览器自动刷新页面，无需手动操作。

### ✅ 前端热更新验证

1. **启动开发环境**

   ```bash
   docker-compose up frontend
   ```

2. **修改前端代码**

   ```javascript
   // frontend/src/App.tsx
   // 随便修改一些地方
   ```

3. **观察控制台输出**

   ```text
   [vite] page reload src/App.tsx
   ```

4. **浏览器自动刷新**
   无需手动操作，页面将自动更新

### 🗄️ 后端热重载完整解决方案

#### 拆分后端 Dockerfile

```dockerfile
# backend/Dockerfile.dev
FROM node:22-alpine AS development
WORKDIR /app

# 安装开发依赖
RUN npm install -g nodemon
COPY package*.json ./
RUN npm ci
COPY . .

# 启动开发服务器
CMD ["npm", "run", "start:dev:docker"]
```

```dockerfile
# backend/Dockerfile.prod
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:22-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --production
CMD ["node", "dist/main.js"]
```

将后端的 Dockerfile 拆分为开发环境和生产环境两个文件。在开发环境的 Dockerfile 中，安装`nodemon`用于文件监控和热重载，复制项目文件并安装依赖，最后启动开发服务器。生产环境的 Dockerfile 则先进行代码构建，然后将构建后的文件复制到生产环境镜像中，只安装生产环境所需的依赖。

#### 调整 docker-compose 配置

```yaml
# docker-compose.yml (生产配置)
services:
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
```

```yaml
# docker-compose.dev.yml (开发配置)
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
```

在`docker-compose.yml`中配置生产环境的后端服务，使用`Dockerfile.prod`进行构建。在`docker-compose.dev.yml`中配置开发环境的后端服务，使用`Dockerfile.dev`进行构建，通过`volumes`实现代码实时同步，防止覆盖容器内的依赖。

#### nodemon 配置

```json
// backend/package.json
{
  "scripts": {
    "start:dev:docker": "nodemon --legacy-watch src/main.ts"
  },
  "devDependencies": {
    //版本供参考
    "@types/nodemon": "^1.19.5",
    "nodemon": "^3.1.0", // 新增热重载工具
    "ts-node": "^10.9.2" // 明确 TS 运行时版本
  }
}
```

```json
// backend/nodemon.json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node src/main.ts",
  "legacyWatch": true,
  "delay": "1"
}
```

在`package.json`中添加`start:dev:docker`脚本，使用`nodemon`监控`src`目录下的文件变化。在`nodemon.json`中配置监控的目录、文件类型、忽略的文件等，使用`ts-node`直接运行 TypeScript 文件。

### ✅ 后端热重载验证

1. **修改后端代码**

   ```typescript
   // backend/src/main.ts
   // 随便修改
   ```

2. **观察容器日志**

   ```text
   [nodemon] restarting due to changes...
   Server running on http://localhost:3000
   ```

3. **API 测试**

   ```bash
   curl http://localhost:3000/health
   ```

### 🚀 一键启动方案

#### 生产环境

```bash
docker-compose -f docker-compose.yml up --build
```

#### 开发环境（完整热更新）

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

在主目录下创建批处理文件`dev-up.bat`：

```batch
@echo off
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up %*
```

这个脚本可以方便地启动开发环境，将`docker-compose.yml`和`docker-compose.dev.yml`的配置合并使用。

通过以上前端和后端的热重载配置，我们在开发过程中可以极大地提高效率，实时看到代码修改的效果。同时，通过不同的 Dockerfile 和`docker-compose`配置，清晰地分离了开发环境和生产环境的配置，确保生产环境的稳定性和安全性。

## 🚨 问题与解决方案

在完成前面的 Docker 配置及热重载设置后，在实际使用过程中可能会遇到一些问题。接下来将针对用户和密码管理、文件路径及环境变量加载等方面的常见问题进行分析，并给出相应的解决方案。

### 🔐 用户和密码管理问题

#### 用户和密码定义关系

用户和密码的创建涉及两个关键文件，它们有着不同的作用和配置方式。

- **Root 用户 (管理员)**：在 `docker-compose.yml` 中定义，用于 MongoDB 实例的初始管理员账户，对整个 MongoDB 实例拥有完全控制权。

```yaml
mongo:
  environment:
    - MONGO_INITDB_ROOT_USERNAME=root
    - MONGO_INITDB_ROOT_PASSWORD=rootpassword123
```

- **应用用户 (tacticore)**：在 `init-mongo.js` 中定义，仅限特定数据库的操作权限。

```javascript
db.createUser({
  user: "tacticore",
  pwd: "pwd-tacticore",
  roles: [{ role: "readWrite", db: "tacticore" }],
});
```

- **连接字符串 (MONGO_URI)**：配置在 `.env` 或 `docker-compose.yml` 中，格式为 `mongodb://<应用用户>:<应用密码>@mongo:27017/<数据库>?authSource=admin`。使用 `authSource=admin` 是因为 MongoDB 要求非 root 用户必须在 `admin` 数据库进行身份验证。

#### 修改用户密码

- **修改 Root 用户**：直接修改 `docker-compose.yml` 的环境变量。
- **修改应用用户**：更新 `init-mongo.js` 中的 `pwd` 字段，并重新初始化容器。

```bash
docker-compose down -v  # 删除数据卷
docker-compose up -d
```

#### 安全建议

- **生产环境**：使用强密码生成器生成复杂密码，通过 Docker Secrets 管理密码（而非明文存储），定期轮换凭证。
- **开发环境**：在 `init-mongo.js` 中使用占位符，然后在 `docker-compose.yml` 中注入。

```javascript
// init-mongo.js 使用占位符
pwd: process.env.MONGO_APP_PASSWORD;
```

```yaml
mongo:
  environment:
    - MONGO_APP_PASSWORD=${MONGO_APP_PASSWORD}
```

### 📂 Docker 构建前端镜像时找不到 Nginx 配置文件问题

#### 错误原因

```text
COPY infra/docker/nginx/tacticore.conf → 文件未找到
```

这是由于 Docker 构建上下文路径和文件引用路径不匹配。项目结构中，配置文件在 `infra` 目录，而 Dockerfile 在 `client` 目录。

#### 解决方案

在 `docker-compose.yml` 中指定正确的构建上下文。

```yaml
services:
  frontend:
    build:
      context: . # 将构建上下文设为项目根目录
      dockerfile: frontend/Dockerfile
```

关键原理是 Docker 只能访问 `context` 指定目录下的文件，所有 `COPY` 指令的路径都是相对于 `context` 目录的。

### 📌 环境变量加载问题

#### Docker Compose 的环境变量优先级问题

在 `docker-compose.yml` 中同时使用了两种环境变量加载方式：

```yaml
env_file: ./backend/.env # 方式1：从文件加载
environment: # 方式2：直接设置
  - MONGO_URI=${MONGO_URI} # 引用宿主机环境变量
```

根据 Docker 的变量优先级规则，`environment` 直接定义的变量会覆盖 `env_file` 中的同名变量，`${VAR}` 语法会尝试读取宿主机环境变量（而非文件中的变量）。由于宿主机没有定义 `MONGO_URI` 环境变量，导致最终值为空字符串。

#### 解决方案

统一使用 `env_file`。

```diff
services:
  backend:
    env_file:
      - ./backend/.env
    environment:
      - NODE_ENV=production
-     - MONGO_URI=${MONGO_URI}  # 删除此行
-     - JWT_SECRET=${JWT_SECRET} # 删除此行
```

变量覆盖机制如下：

| 配置方式      | 优先级 | 适用场景           |
| ------------- | ------ | ------------------ |
| `environment` | 高     | 需要覆盖文件配置时 |
| `env_file`    | 低     | 集中管理多个变量   |

通过以上对常见问题的分析和解决方案，可以更好地应对 Docker 配置和使用过程中出现的状况，确保项目的稳定运行。

## ⚙️ 通用配置

在全栈博客系统的开发过程中，完成基础架构搭建、Docker 配置等核心部分后，一系列通用基础设施配置同样不容忽视，它们将为后续的高效开发与协作奠定坚实基础。以下详细介绍通用配置部分。

### 🛠️ 统一编辑器规范（.editorconfig）

通过.editorconfig 文件可以确保团队成员在不同编辑器下保持一致的代码风格。

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 2
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2

[*.json]
indent_size = 2

[Makefile]
indent_style = tab
```

- `root = true`：标记此为根配置文件，编辑器将停止向上级目录搜索其他.editorconfig 文件，防止继承父目录配置导致规则冲突。其技术原理遵循就近原则，优先使用项目根目录配置。

### 📝 Git 提交模板（.gitmessage）

规范的 Git 提交信息有助于团队成员快速了解代码变更内容。

```gitmessage
# .gitmessage
# Commit type:
#   feat     - 新功能    - 用户注册功能
#   fix      - Bug修复   - 登录页面表单验证错误
#   docs     - 文档变更  - 更新API接口文档
#   style    - 代码格式  - 调整代码缩进
#   refactor - 代码重构  - 提出公共函数
#   test     - 测试相关  - 添加单元测试
#   perf     - 性能优化  - 优化性能
#   chore    - 构建/依赖 - 更新依赖
#   revert   - 回滚      - 回滚到上一个提交
#   build    - 构建      - 添加构建脚本
#   ci       - CI/CD     - 更新CI配置
#   release  - 发布      - 发布新版本
# --------------------
# Scope (可选):
#   frontend, backend, infra, docs, config
#   作用域层级不超过2级（如feat(backend/login)）
#   新模块首次提交使用feat(module): initialize
# --------------------
# Subject (必填):
#   不超过50个字符，首字母小写，结尾不加句号
# --------------------
# Body (可选):
#   详细描述修改内容
#   使用祈使句（如"Add"而非"Added"）
#   说明变更动机（Why > What）
#   技术方案摘要（如有必要）
# --------------------
# Footer (可选):
#   BREAKING CHANGE: 重大变更说明
# --------------------
# Issues (可选):
#   Close #123, Fix #456

{type}({scope}): {subject}

[body]

[footer]
```

- **配置生效步骤**：执行`git config commit.template .gitmessage`使配置生效。
- **强制校验（推荐）**：

  - 首先在项目根目录执行`npm install husky @commitlint/cli @commitlint/config-conventional --save-dev`。其中，`husky`是 Git hooks 管理工具，`@commitlint/cli`是提交信息校验命令行工具，`@commitlint/config-conventional`是社区标准校验规则。
  - 接着执行`npx husky-init && npm install`初始化 husky 配置，此时会生成.husky 目录结构：

    ```text
    .husky/
    ├── _
    │   └── .gitignore
    └── pre-commit
    ```

  - 在根目录创建配置文件.commitlintrc.json，内容如下：

    ```json
    {
      "extends": ["@commitlint/config-conventional"],
      "rules": {
        "header-max-length": [2, "always", 100]
      }
    }
    ```

  - 最后在.husky/commit-msg 中添加`npx commitlint --edit $1`结合 Husky 使用。

- **完整提交示例**：

  ```text
  feat(payment): 集成支付宝扫码支付

  - 添加支付宝SDK初始化配置
  - 实现扫码支付核心逻辑
  - 增加支付结果回调处理

  BREAKING CHANGE: 支付接口返回结构变更
  Closes #112, #113
  Refs: #98
  ```

- **常见问题排查**：

  | 问题现象                   | 解决方案                                             |
  | -------------------------- | ---------------------------------------------------- |
  | `command not found: husky` | 重新执行`npm install husky --save-dev`               |
  | 钩子未生效                 | 检查.husky 目录是否在.git 同级目录                   |
  | Windows 权限问题           | 在 PowerShell 执行`Set-ExecutionPolicy RemoteSigned` |
  | 需要跳过校验               | `git commit --no-verify` (慎用)                      |

### 🔍 Git 属性配置（.gitattributes）

.gitattributes 文件用于定义 Git 如何处理不同类型的文件。

```text
# .gitattributes
* text=auto eol=lf

*.sh text eol=lf

*.md linguist-language=Markdown
*.ts linguist-language=TypeScript
*.tsx linguist-language=TSX

package-lock.json binary
yarn.lock binary
```

例如，`* text=auto eol=lf`表示自动检测文本文件，并将换行符设置为 LF；`package-lock.json binary`则将`package - lock.json`文件标记为二进制文件，避免在合并时出现不必要的冲突。

### 🚨 基础 ESLint 配置（.eslintrc）

ESLint 用于检查和规范代码质量。

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "react", "import"],
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "rules": {
    "react/react-in-jsx-scope": "off",
    "import/order": [
      "error",
      {
        "groups": [
          "builtin",
          "external",
          "internal",
          "parent",
          "sibling",
          "index"
        ],
        "newlines-between": "always"
      }
    ]
  }
}
```

以`import/order`规则为例，它的作用是统一模块导入顺序。分组优先级如下：

- `builtin`：Node.js 内置模块（如`path`）
- `external`：`node_modules`依赖
- `internal`：项目内部别名路径
- `parent`：父目录引用
- `sibling`：同级目录
- `index`：目录索引文件
  例如：

```javascript
// ✅ 正确顺序
import path from "path"; // builtin
import React from "react"; // external
import Button from "@/components/Button"; // internal
import utils from "../utils"; // parent
import styles from "./styles.module.css"; // sibling
```

### 🎨 Prettier 格式化配置（.prettierrc）

Prettier 用于代码格式化，保持代码风格一致。

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

例如，`printWidth`设置为 100，表示一行代码的最大宽度为 100 个字符；`singleQuote`设置为`true`表示使用单引号。

### 🐶 Husky 提交规范（.husky/）

Husky 可以在 Git 操作的特定阶段执行脚本，确保代码规范。在配置 Husky 时，lint:staged 发挥着重要作用。

#### **安装依赖**

```bash
npm install lint-staged --save-dev
# 安装ESLint依赖
npm install eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
# 在项目根目录执行
npm install prettier --save-dev
```

#### **修改`package.json`**

```json
{
  "scripts": {
    "test": "echo \"Tests placeholder\"",
    "lint:staged": "lint-staged"
  },
  "lint-staged": {
    "*.{js,ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

在此配置中，`lint-staged`定义了对暂存的特定文件（如`.js`、`.ts`、`.tsx`文件）执行`eslint --fix`进行代码检查和修复，以及`prettier --write`进行代码格式化。 3. **修改`.husky/pre-commit`**：

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint:staged
```

修改后，在每次执行`git commit`进行提交时，Husky 的`pre-commit`钩子会自动运行`npm run lint:staged`，对暂存文件进行预先检查和格式化，确保提交的代码符合团队设定的代码规范。

整体来看，Husky 结合 lint:staged，能够在代码提交的关键节点，通过自动化的脚本执行，有效提升代码质量，减少因代码风格不一致等问题引发的潜在错误和沟通成本。

### 🌱 基础环境模板（.env.example）

.env.example 文件用于定义项目所需的环境变量模板，方便团队成员了解和配置环境。

```env
# 前端
VITE_API_BASE_URL=http://localhost:4000/api/v1
VITE_SENTRY_DSN=

# 后端
PORT=4000
MONGO_URI=mongodb://mongo:27017/blog
JWT_SECRET=change_this_in_production
```

例如，前端通过`VITE_API_BASE_URL`配置后端 API 的基础地址；后端通过`MONGO_URI`配置 MongoDB 的连接字符串。

通过完成以上通用基础设施配置，能够极大提升团队协作效率，减少代码风格不一致、提交信息不规范等问题，为全栈博客系统的持续开发提供有力保障。
