---
title: "Chapter1: 项目初始化"
description: 项目初始化全记录：从零搭建全栈博客基础架构
date: 2025-04-10T23:15:22+08:00
slug: hugo-to-my-blog/chapter1
series:
  - 自主开发全栈博客系统全过程
---

### 📦 项目初始化全记录：从零搭建全栈博客基础架构

#### 🛠️ 初始化过程详解

1. **版本控制初始化**

```bash
git init
# 创建.gitignore文件 node_modules/ .env .DS_Store *.log dist/ 等
```

2. **前端脚手架搭建**

```bash
npm create vite@latest
# 交互式选择：
# ✔ Project name: client
# ✔ Framework: React
# ✔ Variant: TypeScipt
```

3. **核心依赖安装**

```bash
cd client && npm install
# 关键依赖清单：
- @mui/material     # Material Design实现
- react-router-dom   # 路由管理
- axios             # HTTP客户端
```

4. **后端服务初始化**

```bash
mkdir server && cd server
npm init -y
npm install express mongoose cors dotenv
# 创建基础结构：mkdir ...
```

#### 📂 项目结构规范

```tree
TactiCore/
├── .github/                   # GitHub 配置
│   ├── workflows/             # CI/CD 流水线
│   └── ISSUE_TEMPLATE/        # Issue 模板
├── client/                    # 前端 (Vite + React)
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
├── server/                    # 后端 (Express + MongoDB)
│   ├── src/
│   │   ├── config/            # 环境配置
│   │   ├── constants/
│   │   ├── controllers/       # 控制器
│   │   ├── jobs/              # 定时任务
│   │   ├── middleware/        # 中间件
│   │   │   ├── auth.ts        # 认证
│   │   │   └── security/      # 安全相关
│   │   ├── migrations/        # 数据库迁移
│   │   ├── models/            # 数据模型
│   │   ├── routes/            # API 路由
│   │   │   └── v1/            # 版本化路由
│   │   ├── schemas/           # 数据校验
│   │   ├── services/          # 业务逻辑
│   │   ├── types/             # TS 类型
│   │   ├── utils/             # 工具类
│   │   │   ├── logger.ts      #  日志
│   │   │   ├── crypto/        # 加密工具
│   │   │   └── apiError.ts    # 错误处理
│   │   └── app.ts             # 主应用
│   ├── test/
│   │   ├── integration/
│   │   └── unit/
│   └── postman/             # Postman 测试集合
├── docs/                    # 文档中心
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

#### 💡 首次运行验证

**前端启动**：

```bash
cd client
npm run dev
```

访问 http://localhost:5173 验证 Vite 启动正常

**后端健康检查**：后面再完成

```javascript
// server/src/index.js
app.get("/api/health", (req, res) => {
  res.json({
    status: "active",
    timestamp: new Date().toISOString(),
  });
});
```

#### 📌 详细设置指南

为了更详细地了解项目设置过程，请参考以下子页面：

- [项目设置指南](./setup) - 包含环境配置、前端设置和后端设置的详细说明
