---
title: "项目初始化与结构设计"
description: 项目初始化与结构设计
date: 2025-04-11T19:06:24+08:00
slug: hugo-to-my-blog/chapter1/project-init
series:
  - 自主开发全栈博客系统全过程
---

## 🛠️ 初始化过程详解

### 1. 版本控制初始化

```bash
git init
# 创建 .gitignore 文件，增加以下内容：
node_modules/
.env
.DS_Store
*.log
dist/
coverage/
```

### 2. 前端脚手架搭建

```bash
npm create vite@latest
# 交互式选择：
# ✔ Project name: frontend
# ✔ Framework: React
# ✔ Variant: TypeScipt
```

### 3. 核心依赖安装

```bash
cd client && npm install
# 关键依赖清单：
- @mui/material     # Material Design实现
- react-router-dom   # 路由管理
- @reduxjs/toolkit
- axios             # HTTP客户端
```

### 4. 后端服务初始化(NestJS)

```bash
mkdir backend && cd backend

# 使用 Nest CLI 创建项目
npm install -g @nestjs/cli
nest new . --package-manager npm
```

## 📂 项目结构规范

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

在构建现代化全栈应用时，项目结构的设计往往决定着后期维护效率和系统扩展能力。我的博客系统架构采用前后端分离模式，以前端 React+vite 生态与后端 NestJS 框架为核心，在保证开发效率的同时，为系统演进预留充足空间。
