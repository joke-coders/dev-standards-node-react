# 项目模板与脚手架

## 1. 项目初始化清单

创建新项目时必须完成以下配置：

### 基础配置

- [ ] `.gitignore` — 排除 `node_modules/`, `.env`, `dist/`, IDE 配置等
- [ ] `.editorconfig` — 统一编辑器设置（缩进、换行符、字符集）
- [ ] `README.md` — 项目说明、启动步骤、环境要求
- [ ] `LICENSE` — 开源许可证（如适用）

### 代码质量

- [ ] ESLint / Pylint / golangci-lint — 代码检查
- [ ] Prettier / Black / gofmt — 代码格式化
- [ ] Husky + lint-staged — Git 提交前自动检查（Node.js 项目）
- [ ] TypeScript `strict: true`（TS 项目必须开启严格模式）

### 环境管理

- [ ] `.env.example` — 环境变量模板（不含敏感值）
- [ ] 区分环境：`development`, `staging`, `production`
- [ ] 敏感配置使用环境变量或密钥管理服务

### CI/CD

- [ ] 单元测试配置
- [ ] CI 流水线（构建 → 测试 → 检查 → 部署）
- [ ] Docker / docker-compose 配置

---

## 2. 后端项目模板

### Node.js / NestJS

```
backend/
├── src/
│   ├── main.ts                    # 入口
│   ├── app.module.ts              # 根模块
│   ├── common/                    # 公共代码
│   │   ├── constants/             # 常量
│   │   │   └── error-codes.ts     # 错误码定义
│   │   ├── decorators/            # 自定义装饰器
│   │   ├── dto/                   # 公共 DTO
│   │   │   └── pagination.dto.ts  # 分页请求 DTO
│   │   ├── exceptions/            # 自定义异常
│   │   │   └── business.exception.ts
│   │   ├── filters/               # 异常过滤器
│   │   │   └── global-exception.filter.ts
│   │   ├── guards/                # 守卫
│   │   │   └── auth.guard.ts
│   │   ├── interceptors/          # 拦截器
│   │   │   ├── response.interceptor.ts  # 统一响应包装
│   │   │   └── logging.interceptor.ts
│   │   ├── interfaces/            # 公共接口/类型
│   │   │   └── response.interface.ts    # 统一响应类型
│   │   ├── middlewares/           # 中间件
│   │   ├── pipes/                 # 管道
│   │   └── utils/                 # 工具函数
│   ├── config/                    # 配置模块
│   │   └── configuration.ts
│   ├── database/                  # 数据库
│   │   ├── entities/              # 实体（可按模块组织）
│   │   ├── migrations/            # 迁移文件
│   │   └── seeds/                 # 种子数据
│   └── modules/                   # 业务模块
│       ├── auth/
│       │   ├── auth.module.ts
│       │   ├── auth.controller.ts
│       │   ├── auth.service.ts
│       │   ├── dto/
│       │   ├── guards/
│       │   └── strategies/
│       └── users/
│           ├── users.module.ts
│           ├── users.controller.ts
│           ├── users.service.ts
│           ├── dto/
│           │   ├── create-user.dto.ts
│           │   └── update-user.dto.ts
│           └── entities/
│               └── user.entity.ts
├── test/
│   ├── unit/
│   └── e2e/
├── .env.example
├── .eslintrc.js
├── .prettierrc
├── nest-cli.json
├── tsconfig.json
└── package.json
```

### Python / FastAPI

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                    # 入口
│   ├── core/                      # 核心配置
│   │   ├── config.py              # 配置类
│   │   ├── database.py            # 数据库连接
│   │   ├── security.py            # 认证/加密
│   │   ├── exceptions.py          # 自定义异常
│   │   ├── error_codes.py         # 错误码
│   │   └── deps.py                # 依赖注入
│   ├── middleware/                 # 中间件
│   │   ├── error_handler.py       # 全局异常处理
│   │   └── logging.py             # 日志中间件
│   ├── models/                    # 数据库模型
│   │   └── user.py
│   ├── schemas/                   # Pydantic 模型
│   │   ├── common.py              # 通用响应/分页
│   │   └── user.py
│   ├── api/                       # 路由
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   └── users.py
│   │   └── router.py
│   └── services/                  # 业务逻辑
│       └── user_service.py
├── migrations/                    # Alembic 迁移
├── tests/
├── .env.example
├── pyproject.toml
├── Dockerfile
└── docker-compose.yml
```

---

## 3. 前端项目模板

### React (Vite + TypeScript)

```
frontend/
├── public/
│   └── favicon.ico
├── src/
│   ├── main.tsx                   # 入口
│   ├── App.tsx                    # 根组件
│   ├── router/                    # 路由配置
│   │   └── index.tsx
│   ├── components/                # 公共组件
│   │   ├── common/
│   │   │   ├── PageHeader/
│   │   │   │   ├── index.tsx
│   │   │   │   └── styles.module.css
│   │   │   ├── DataTable/
│   │   │   ├── Pagination/
│   │   │   ├── StatusTag/
│   │   │   ├── EmptyState/
│   │   │   ├── LoadingState/
│   │   │   └── ErrorBoundary/
│   │   ├── layout/
│   │   │   ├── AppLayout/
│   │   │   ├── Sidebar/
│   │   │   └── Navbar/
│   │   └── feedback/
│   │       └── Toast/
│   ├── pages/                     # 页面
│   │   └── users/
│   │       ├── index.tsx          # 列表页
│   │       ├── detail.tsx
│   │       ├── create.tsx
│   │       └── components/        # 页面私有组件
│   ├── hooks/                     # 自定义 Hooks
│   │   ├── useAuth.ts
│   │   ├── usePagination.ts
│   │   └── useDebounce.ts
│   ├── services/                  # API 请求
│   │   ├── http.ts                # HTTP 客户端封装
│   │   ├── types.ts               # 请求/响应类型
│   │   └── modules/
│   │       ├── auth.ts
│   │       └── user.ts
│   ├── stores/                    # 状态管理
│   ├── utils/                     # 工具函数
│   ├── constants/                 # 常量
│   ├── types/                     # 全局类型
│   └── styles/                    # 全局样式
│       ├── globals.css
│       └── variables.css
├── .env.example
├── .eslintrc.cjs
├── .prettierrc
├── tsconfig.json
├── vite.config.ts
└── package.json
```

---

## 4. Docker 配置模板

### 后端 Dockerfile

```dockerfile
# 多阶段构建
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN npm run build

FROM node:20-alpine
RUN addgroup -g 1001 -S appuser && adduser -S appuser -u 1001
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
USER appuser
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

---

## 5. .env.example 模板

```bash
# ===== 应用配置 =====
NODE_ENV=development
PORT=3000
API_PREFIX=/api/v1

# ===== 前端 Mock =====
# VITE_API_MOCK_DATA=true   # true: 使用 mock 数据 | false: 请求真实 API

# ===== 数据库 =====
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=

# ===== Redis =====
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# ===== 认证 =====
JWT_SECRET=
JWT_EXPIRES_IN=7d

# ===== 第三方服务 =====
# SMTP_HOST=
# SMTP_PORT=
# SMTP_USER=
# SMTP_PASS=

# ===== 文件存储 =====
# S3_BUCKET=
# S3_REGION=
# S3_ACCESS_KEY=
# S3_SECRET_KEY=
```
