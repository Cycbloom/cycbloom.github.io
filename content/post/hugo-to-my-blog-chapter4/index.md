---
title: "Chapter4: 后端架构与实现"
description: "深入解析 TactiCore 后端架构设计与核心模块实现"
date: 2025-04-15T14:00:00+08:00
slug: hugo-to-my-blog/chapter4
tag:
  - 技术
  - NestJS
  - TypeScript
  - 后端开发
  - Prisma
  - Docker
series:
  - 自主开发全栈博客系统全过程
image:
math:
license:
hidden: false
comments: true
---

---

## Chapter4: 后端架构与实现

——从认证到任务管理的全栈解析

---

### 一、项目架构概览

TactiCore 后端采用 NestJS 框架构建，遵循模块化设计原则，各功能模块高度解耦。整体架构如下：

```
backend/
├── src/
│   ├── core/           # 核心模块
│   │   ├── auth/       # 认证模块
│   │   └── config/     # 配置模块
│   ├── database/       # 数据库模块
│   │   ├── schemas/    # 数据模型定义
│   │   └── prisma/     # Prisma 服务
│   ├── task/           # 任务管理模块
│   └── main.ts         # 应用入口
├── prisma/             # Prisma 配置
│   ├── schema.prisma   # 数据库模型定义
│   └── migrations/     # 数据库迁移文件
└── scripts/            # 辅助脚本
    └── init-db.sh      # 数据库初始化脚本
```

**架构特点**：

- **模块化设计**：每个功能模块独立封装，通过依赖注入实现松耦合
- **分层架构**：控制器层处理请求，服务层实现业务逻辑，数据访问层与数据库交互
- **类型安全**：全面使用 TypeScript 类型系统，确保代码健壮性
- **容器化部署**：基于 Docker 和 Docker Compose 实现环境一致性

---

### 二、认证模块实现

认证模块是系统的安全基石，我们采用 JWT (JSON Web Token) 实现无状态认证。

#### 1. JWT 策略设计

```typescript
// JwtStrategy 实现
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.getOrThrow<string>("JWT_SECRET"),
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

**关键技术点**：

- **策略模式**：通过继承 `PassportStrategy` 实现自定义认证策略
- **配置注入**：使用 `ConfigService` 获取环境变量中的密钥
- **令牌提取**：从请求头中提取 Bearer 令牌
- **负载验证**：验证 JWT 令牌的有效性并返回用户信息

#### 2. 认证模块配置

```typescript
@Module({
  imports: [
    PassportModule,
    ConfigModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.getOrThrow<string>("JWT_SECRET"),
        signOptions: { expiresIn: "1d" },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [JwtStrategy],
  exports: [JwtModule],
})
export class AuthModule {}
```

**模块设计**：

- **异步配置**：使用 `registerAsync` 实现 JWT 模块的异步配置
- **依赖注入**：通过 `inject` 注入 `ConfigService` 依赖
- **模块导出**：导出 `JwtModule` 供其他模块使用

#### 3. 认证守卫

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```

**使用方式**：

```typescript
@Controller("tasks")
@UseGuards(JwtAuthGuard)
export class TaskController {
  // 控制器方法...
}
```

**设计优势**：

- **全局保护**：通过装饰器轻松为整个控制器添加认证保护
- **灵活配置**：支持基于角色的访问控制扩展
- **异常处理**：自动处理认证失败情况

---

### 三、配置模块设计

配置模块负责管理系统配置，支持环境变量和配置验证。

#### 1. 配置模式定义

```typescript
// 配置模式定义
export const configValidationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid("development", "production", "test")
    .default("development"),
  PORT: Joi.number().default(3000),
  JWT_SECRET: Joi.string().required(),
  POSTGRES_URL: Joi.string().required(),
  MONGODB_URL: Joi.string().required(),
});
```

**验证机制**：

- **类型安全**：使用 Joi 进行运行时类型验证
- **默认值**：为可选配置提供合理的默认值
- **必填项**：标记关键配置为必填项

#### 2. 配置服务实现

```typescript
@Injectable()
export class ConfigService {
  private readonly config: Record<string, any>;

  constructor() {
    this.config = this.validateConfig(process.env);
  }

  get<T>(key: string): T {
    return this.config[key] as T;
  }

  getOrThrow<T>(key: string): T {
    const value = this.get<T>(key);
    if (value === undefined) {
      throw new Error(`配置项 ${key} 未定义`);
    }
    return value;
  }

  private validateConfig(config: Record<string, any>): Record<string, any> {
    const { error, value } = configValidationSchema.validate(config, {
      abortEarly: false,
      allowUnknown: true,
    });

    if (error) {
      throw new Error(`配置验证失败: ${error.message}`);
    }

    return value;
  }
}
```

**服务特点**：

- **类型安全**：支持泛型获取配置值
- **错误处理**：提供 `getOrThrow` 方法确保关键配置存在
- **验证逻辑**：在初始化时验证所有配置项

#### 3. 配置模块注册

```typescript
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

**模块设计**：

- **单例模式**：配置服务作为单例在整个应用中共享
- **依赖导出**：导出配置服务供其他模块使用

---

### 四、数据库模块实现

数据库模块采用 Prisma ORM 实现，支持 PostgreSQL 和 MongoDB 双数据库。

#### 1. Prisma 服务设计

```typescript
@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  constructor() {
    super({
      log: ["query", "info", "warn", "error"],
    });
  }

  async onModuleInit() {
    await this.$connect();
    try {
      await this.$executeRaw`SELECT 1`;
      console.log("数据库连接成功");
    } catch (error) {
      console.error("数据库连接失败:", error);
      throw error;
    }
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

**服务特点**：

- **生命周期钩子**：实现 `OnModuleInit` 和 `OnModuleDestroy` 接口
- **连接管理**：在模块初始化时连接数据库，销毁时断开连接
- **健康检查**：通过执行简单查询验证数据库连接状态

#### 2. 数据库模型定义

```prisma
// Prisma schema 定义
model Task {
  id          String   @id @default(uuid())
  title       String
  description String?
  status      String   @default("todo")
  priority    String   @default("medium")
  dueDate     DateTime?
  tags        String[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

**模型设计**：

- **UUID 主键**：使用 UUID 作为主键，提高安全性和分布式兼容性
- **默认值**：为常用字段提供合理的默认值
- **时间戳**：自动记录创建和更新时间

#### 3. 数据库迁移管理

数据库迁移是确保数据库结构与代码同步的关键环节。在 TactiCore 中，我们通过以下步骤实现迁移：

1. **创建迁移文件**：在 docker 容器控制台

   ```bash
   npx prisma migrate dev --name init
   ```

   这个命令会：

   - 检测模型变化
   - 生成迁移 SQL 文件
   - 应用迁移到数据库
   - 重新生成 Prisma 客户端

2. **迁移文件结构**：

   ```
   prisma/migrations/
   ├── 20250415053129_init/
   │   └── migration.sql
   └── migration_lock.toml
   ```

3. **迁移文件内容**：

   ```sql
   -- CreateTable
   CREATE TABLE "Task" (
       "id" TEXT NOT NULL,
       "title" TEXT NOT NULL,
       "description" TEXT,
       "status" TEXT NOT NULL DEFAULT 'todo',
       "priority" TEXT NOT NULL DEFAULT 'medium',
       "dueDate" TIMESTAMP(3),
       "tags" TEXT[],
       "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
       "updatedAt" TIMESTAMP(3) NOT NULL,
       CONSTRAINT "Task_pkey" PRIMARY KEY ("id")
   );
   ```

4. **自动化迁移**：
   在 Docker 环境中，我们通过初始化脚本自动应用迁移：

   ```bash
   #!/bin/sh
   # 等待数据库准备就绪
   echo "等待数据库准备就绪..."
   sleep 5

   # 运行 Prisma 迁移
   echo "运行数据库迁移..."
   npx prisma migrate deploy

   # 生成 Prisma 客户端
   echo "生成 Prisma 客户端..."
   npx prisma generate

   echo "数据库初始化完成"
   ```

**迁移策略**：

- **开发环境**：使用 `migrate dev` 命令，自动应用迁移并更新客户端
- **生产环境**：使用 `migrate deploy` 命令，仅应用已有迁移，不修改数据库结构
- **版本控制**：迁移文件纳入版本控制，确保团队间数据库结构一致

---

### 五、任务管理模块实现

任务管理模块是系统的核心业务模块，实现了任务的 CRUD 操作。

#### 1. 数据模型定义

```typescript
// 任务状态枚举
export enum TaskStatus {
  TODO = "todo",
  IN_PROGRESS = "inProgress",
  COMPLETED = "completed",
}

// 任务优先级枚举
export enum TaskPriority {
  LOW = "low",
  MEDIUM = "medium",
  HIGH = "high",
}

// 创建任务 DTO
export class CreateTaskDto {
  @ApiProperty({ description: "任务标题" })
  @IsString()
  title: string;

  @ApiProperty({ description: "任务描述", required: false, nullable: true })
  @IsString()
  @IsOptional()
  description: string | null;

  @ApiProperty({
    description: "任务状态",
    enum: TaskStatus,
    default: TaskStatus.TODO,
  })
  @IsEnum(TaskStatus)
  @IsOptional()
  status?: TaskStatus;

  // 其他字段...
}

// 更新任务 DTO
export class UpdateTaskDto extends CreateTaskDto {
  // 继承自 CreateTaskDto，不需要额外字段
}

// 任务响应 DTO
export class TaskResponseDto extends CreateTaskDto {
  @ApiProperty({ description: "任务ID" })
  @IsUUID()
  id: string;

  @ApiProperty({ description: "创建时间" })
  @IsDate()
  createdAt: Date;

  @ApiProperty({ description: "更新时间" })
  @IsDate()
  updatedAt: Date;
}
```

**DTO 设计**：

- **继承关系**：`UpdateTaskDto` 继承自 `CreateTaskDto`，避免代码重复
- **验证装饰器**：使用 `class-validator` 装饰器进行输入验证
- **API 文档**：使用 `@ApiProperty` 装饰器生成 Swagger 文档
- **类型安全**：使用 TypeScript 类型系统确保类型安全

#### 2. 任务服务实现

```typescript
@Injectable()
export class TaskService {
  constructor(private readonly prisma: PrismaService) {}

  // 类型转换函数
  private convertToTaskResponseDto(task: any): TaskResponseDto {
    return {
      ...task,
      status: task.status as TaskStatus,
      priority: task.priority as TaskPriority,
    };
  }

  // 创建任务
  async createTask(createTaskDto: CreateTaskDto): Promise<TaskResponseDto> {
    const task = await this.prisma.task.create({
      data: {
        ...createTaskDto,
        status: createTaskDto.status || TaskStatus.TODO,
        priority: createTaskDto.priority || TaskPriority.MEDIUM,
      },
    });
    return this.convertToTaskResponseDto(task);
  }

  // 获取任务列表
  async getTasks(filter: TaskFilterDto): Promise<TaskResponseDto[]> {
    const where = {
      ...(filter.status && { status: filter.status }),
      ...(filter.priority && { priority: filter.priority }),
      ...(filter.tags && { tags: { hasSome: filter.tags } }),
      ...(filter.startDate && { dueDate: { gte: filter.startDate } }),
      ...(filter.endDate && { dueDate: { lte: filter.endDate } }),
    };

    const tasks = await this.prisma.task.findMany({
      where,
      orderBy: {
        createdAt: "desc",
      },
    });
    return tasks.map((task) => this.convertToTaskResponseDto(task));
  }

  // 其他方法...
}
```

**服务设计**：

- **依赖注入**：通过构造函数注入 `PrismaService`
- **类型转换**：使用 `convertToTaskResponseDto` 方法确保返回类型正确
- **过滤条件**：支持多条件组合过滤
- **默认值**：为可选字段提供合理的默认值

#### 3. 任务控制器实现

```typescript
@ApiTags("tasks")
@Controller("tasks")
@UseGuards(JwtAuthGuard)
export class TaskController {
  constructor(private readonly taskService: TaskService) {}

  @Post()
  @ApiOperation({ summary: "创建新任务" })
  @ApiResponse({
    status: 201,
    description: "任务创建成功",
    type: TaskResponseDto,
  })
  async createTask(
    @Body() createTaskDto: CreateTaskDto
  ): Promise<TaskResponseDto> {
    return this.taskService.createTask(createTaskDto);
  }

  // 其他端点...
}
```

**控制器设计**：

- **路由定义**：使用装饰器定义 HTTP 方法和路径
- **参数绑定**：使用装饰器绑定请求参数
- **API 文档**：使用 Swagger 装饰器生成 API 文档
- **认证保护**：使用 `JwtAuthGuard` 保护所有端点

---

### 六、Docker 部署配置

TactiCore 采用 Docker 和 Docker Compose 实现容器化部署，确保环境一致性。

#### 1. 开发环境 Dockerfile

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
RUN npx prisma generate
CMD ["npm", "run", "start:dev:docker"]
```

**开发环境特点**：

- **热重载**：使用 nodemon 实现代码变更自动重启
- **依赖安装**：使用 `npm ci` 确保依赖版本一致性
- **Prisma 生成**：在构建时生成 Prisma 客户端

#### 2. 生产环境 Dockerfile

```dockerfile
# 开发阶段
FROM node:22-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

# 生产阶段
FROM node:22-alpine AS production
WORKDIR /app
COPY --from=development /app/dist ./dist
COPY --from=development /app/package*.json ./
COPY --from=development /app/node_modules/.prisma ./node_modules/.prisma
COPY --from=development /app/prisma ./prisma
COPY --from=development /app/scripts ./scripts
RUN npm ci --production
# 设置脚本权限
RUN chmod +x ./scripts/init-db.sh
# 使用初始化脚本
CMD ["sh", "-c", "./scripts/init-db.sh && node dist/main.js"]
```

**生产环境特点**：

- **多阶段构建**：使用多阶段构建减小镜像体积
- **依赖优化**：仅安装生产环境依赖
- **数据库初始化**：启动时自动运行数据库迁移
- **脚本执行**：使用 shell 命令组合多个操作

#### 3. Docker Compose 配置

```yaml
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
      - postgres
    ports:
      - "3000:3000"

  mongo:
    image: mongo:6
    volumes:
      - mongo_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=rootpassword123
    ports:
      - "27017:27017"

  postgres:
    image: postgres:16.8-alpine3.20
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres123
      - POSTGRES_DB=tacticore
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  mongo_data:
  postgres_data:
```

**Compose 配置特点**：

- **服务依赖**：使用 `depends_on` 定义服务启动顺序
- **环境变量**：通过 `env_file` 和 `environment` 配置环境变量
- **数据持久化**：使用命名卷保存数据库数据
- **健康检查**：为数据库服务配置健康检查
- **端口映射**：将容器端口映射到主机端口

---

### 七、API 测试与文档

我们使用 Postman 进行 API 测试，并生成 Swagger 文档。

#### 1. Postman 测试集设计

```json
{
  "info": {
    "name": "TactiCore Task API",
    "description": "测试 TactiCore 任务管理 API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "创建任务",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"title\": \"测试任务\",\n  \"description\": \"这是一个测试任务\",\n  \"status\": \"todo\",\n  \"priority\": \"medium\",\n  \"dueDate\": \"2024-12-31T23:59:59Z\",\n  \"tags\": [\"测试\", \"示例\"]\n}"
        },
        "url": {
          "raw": "{{base_url}}/tasks",
          "host": ["{{base_url}}"],
          "path": ["tasks"]
        },
        "description": "创建一个新的任务"
      }
    }
    // 其他请求...
  ],
  "auth": {
    "type": "jwt",
    "jwt": [
      {
        "key": "secret",
        "value": "tacticore-jwt-secret",
        "type": "string"
      },
      {
        "key": "algorithm",
        "value": "HS256",
        "type": "string"
      },
      {
        "key": "headerPrefix",
        "value": "Bearer",
        "type": "string"
      }
    ]
  },
  "variable": [
    {
      "key": "base_url",
      "value": "http://localhost:3000",
      "type": "string"
    },
    {
      "key": "jwt_token",
      "value": "tacticore-jwt-secret",
      "type": "string"
    },
    {
      "key": "task_id",
      "value": "1",
      "type": "string"
    }
  ]
}
```

**测试集特点**：

- **环境变量**：使用变量定义基础 URL 和认证令牌
- **JWT 认证**：配置 JWT 认证方式
- **请求组织**：按功能模块组织请求
- **请求描述**：为每个请求添加详细描述

#### 2. JWT 认证配置

在 Postman 中，JWT 认证通过以下方式配置：

```json
"auth": {
  "type": "jwt",
  "jwt": [
    {
      "key": "secret",
      "value": "tacticore-jwt-secret",
      "type": "string"
    },
    {
      "key": "algorithm",
      "value": "HS256",
      "type": "string"
    },
    {
      "key": "headerPrefix",
      "value": "Bearer",
      "type": "string"
    }
  ]
}
```

**认证配置特点**：

- **密钥配置**：设置 JWT 签名密钥
- **算法指定**：使用 HS256 算法
- **前缀设置**：设置 Bearer 前缀

#### 3. 测试流程

1. **创建任务**：

   - 发送 POST 请求到 `/tasks` 端点
   - 请求体包含任务信息
   - 从响应中获取任务 ID

2. **更新任务**：

   - 将任务 ID 设置到环境变量 `task_id`
   - 发送 PUT 请求到 `/tasks/{{task_id}}` 端点
   - 请求体包含更新的任务信息

3. **获取任务**：

   - 发送 GET 请求到 `/tasks/{{task_id}}` 端点
   - 验证返回的任务信息

4. **删除任务**：
   - 发送 DELETE 请求到 `/tasks/{{task_id}}` 端点
   - 验证任务已被删除

---

### 八、总结与展望

TactiCore 后端采用模块化设计，实现了认证、配置、数据库和任务管理等功能。通过 Docker 容器化部署，确保了环境一致性和部署便捷性。

**技术亮点**：

- **类型安全**：全面使用 TypeScript 类型系统
- **模块化设计**：各功能模块高度解耦
- **依赖注入**：通过 NestJS 依赖注入实现松耦合
- **数据库迁移**：使用 Prisma 管理数据库迁移
- **容器化部署**：基于 Docker 实现环境一致性

**未来展望**：

- **缓存层**：引入 Redis 缓存提高性能
- **消息队列**：集成消息队列处理异步任务
- **监控系统**：添加 Prometheus 和 Grafana 监控
- **CI/CD**：实现自动化测试和部署流程

通过这一系列模块的精心设计和实现，TactiCore 后端为全栈博客系统提供了坚实的技术基础，为后续功能扩展和性能优化奠定了良好的架构基础。

---
