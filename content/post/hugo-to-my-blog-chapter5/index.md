---
title: "Chapter5: 认证系统设计与实现"
description: "深入解析 TactiCore 认证系统的设计与实现过程"
date: 2025-04-19T16:00:00+08:00
slug: hugo-to-my-blog/chapter5
tag:
  - 技术
  - NestJS
  - TypeScript
  - 认证系统
  - React
  - JWT
series:
  - 自主开发全栈博客系统全过程
image:
math:
license:
hidden: false
comments: true
---

---

## Chapter5: 认证系统设计与实现

——从后端到前端的完整认证流程

---

### 一、后端认证系统设计

TactiCore 的认证系统采用 JWT (JSON Web Token) 实现无状态认证，结合 Prisma ORM 进行用户数据管理。

#### 1. 用户数据模型设计

首先，我们在 Prisma schema 中定义用户模型：

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  username  String   @unique
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**模型特点**：

- **UUID 主键**：使用 UUID 作为主键，提高安全性和分布式兼容性
- **唯一约束**：邮箱和用户名设置唯一约束，确保数据唯一性
- **时间戳**：自动记录创建和更新时间

#### 2. 数据库初始化

通过 `seed.ts` 实现数据库初始化，创建默认用户：

```typescript
async function main() {
  // 创建默认管理员用户
  const adminPassword = await bcryptjs.hash("admin123", 10);
  await prisma.user.upsert({
    where: { email: "admin@example.com" },
    update: {},
    create: {
      email: "admin@example.com",
      username: "admin",
      password: adminPassword,
    },
  });

  // 创建测试用户
  const testPassword = await bcryptjs.hash("test123", 10);
  await prisma.user.upsert({
    where: { email: "test@example.com" },
    update: {},
    create: {
      email: "test@example.com",
      username: "test",
      password: testPassword,
    },
  });
}
```

**初始化特点**：

- **密码加密**：使用 bcryptjs 进行密码加密
- **upsert 操作**：使用 upsert 确保数据幂等性
- **默认用户**：创建管理员和测试用户

#### 3. 用户服务实现

用户服务负责处理用户相关的业务逻辑：

```typescript
@Injectable()
export class UsersService {
  constructor(private prisma: PrismaService) {}

  async create(createUserDto: CreateUserDto) {
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);
    return this.prisma.user.create({
      data: {
        ...createUserDto,
        password: hashedPassword,
      },
    });
  }

  async findOne(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
    });
  }

  async findByEmail(email: string) {
    return this.prisma.user.findUnique({
      where: { email },
    });
  }
}
```

**服务特点**：

- **依赖注入**：通过构造函数注入 PrismaService
- **密码加密**：使用 bcrypt 进行密码加密
- **查询方法**：提供多种查询方式

#### 4. 认证服务设计

认证服务处理登录和令牌生成：

```typescript
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService
  ) {}

  async validateUser(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);
    if (user && (await bcrypt.compare(password, user.password))) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user: any) {
    const payload = { email: user.email, sub: user.id };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

**认证特点**：

- **密码验证**：使用 bcrypt 进行密码比对
- **JWT 生成**：使用 JwtService 生成访问令牌
- **数据脱敏**：返回用户信息时移除密码字段

#### 5. CurrentUser 装饰器

自定义装饰器用于获取当前用户信息：

```typescript
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  }
);
```

**装饰器特点**：

- **参数装饰器**：使用 createParamDecorator 创建
- **上下文访问**：通过 ExecutionContext 访问请求上下文
- **用户信息提取**：从请求对象中提取用户信息

#### 6. CORS 配置

在 `main.ts` 中配置跨域资源共享：

```typescript
app.enableCors({
  origin: ["http://localhost:5173"],
  methods: "GET,HEAD,PUT,PATCH,POST,DELETE,OPTIONS",
  credentials: true,
});
```

**CORS 特点**：

- **源限制**：只允许特定前端域名访问
- **方法支持**：支持常用 HTTP 方法
- **凭证支持**：允许携带认证信息

### 二、前端认证系统实现

前端认证系统采用 React Context 和自定义 Hook 实现状态管理。

#### 1. API 设计

在 `api` 目录下定义认证相关的 API 请求：

```typescript
// auth.ts
export const authApi = {
  login: (data: LoginData) => request.post("/auth/login", data),
  getCurrentUser: () => request.get("/auth/me"),
};

// request.ts
const request = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// 请求拦截器
request.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器
request.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("token");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

**API 特点**：

- **请求封装**：统一处理请求和响应
- **令牌管理**：自动添加认证令牌
- **错误处理**：统一处理认证错误

#### 2. 认证上下文

使用 Context 管理认证状态：

```typescript
export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  const [user, setUser] = useState<UserData | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const storedToken = localStorage.getItem("token");
    if (storedToken) {
      setToken(storedToken);
      authApi
        .getCurrentUser()
        .then((response) => setUser(response))
        .catch(() => {
          localStorage.removeItem("token");
          setToken(null);
        })
        .finally(() => setIsLoading(false));
    } else {
      setIsLoading(false);
    }
  }, []);

  const login = async (data: LoginData) => {
    const response = await authApi.login(data);
    const { access_token } = response;
    localStorage.setItem("token", access_token);
    setToken(access_token);
    const userData = await authApi.getCurrentUser();
    setUser(userData);
  };

  const logout = () => {
    localStorage.removeItem("token");
    setToken(null);
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, token, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

**上下文特点**：

- **状态管理**：管理用户信息和令牌状态
- **自动登录**：启动时自动恢复登录状态
- **方法封装**：提供登录和登出方法

#### 3. 登录组件实现

使用 Material-UI 实现登录表单：

```typescript
export default function LoginForm({ onSuccess }: LoginFormProps) {
  const navigate = useNavigate();
  const { login } = useAuth();
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (data: LoginFormData) => {
    setError("");
    setLoading(true);

    try {
      await login(data);
      onSuccess?.();
      navigate("/dashboard");
    } catch (err: any) {
      setError(err.response?.data?.message || "登录失败，请重试");
    } finally {
      setLoading(false);
    }
  };

  return (
    <Container component="main" maxWidth="xs">
      <Paper elevation={3} sx={{ marginTop: 8, padding: 4 }}>
        <BaseForm<LoginFormData>
          onSubmit={handleSubmit}
          defaultValues={{ username: "", password: "" }}
          formTitle="登录到 TactiCore"
          schema={loginSchema}
          submitButtonText={loading ? "登录中..." : "登录"}
        >
          {/* 表单字段 */}
        </BaseForm>
      </Paper>
    </Container>
  );
}
```

**组件特点**：

- **表单验证**：使用 Zod 进行表单验证
- **状态管理**：处理加载和错误状态
- **导航控制**：登录成功后自动导航
- **UI 组件**：使用 Material-UI 实现美观界面

### 三、总结与展望

TactiCore 的认证系统实现了完整的用户认证流程，从后端的用户管理到前端的状态管理，都采用了现代化的技术方案。

**技术亮点**：

- **JWT 认证**：实现无状态认证
- **密码加密**：使用 bcrypt 保护用户密码
- **上下文管理**：使用 React Context 管理认证状态
- **表单验证**：使用 Zod 实现类型安全的表单验证
- **UI 组件**：使用 Material-UI 实现美观的界面

通过这一系列模块的精心设计和实现，TactiCore 的认证系统为用户提供了安全、便捷的认证体验，为系统的其他功能模块奠定了坚实的基础。

---
