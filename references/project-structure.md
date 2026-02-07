# 项目目录结构规范

> 技术栈：NestJS + React (TypeScript) + PostgreSQL + Redis

```
project-root/
│
├── backend/                            # NestJS 后端
│   ├── src/
│   │   ├── main.ts                     # 入口：启动、全局管道/过滤器/拦截器注册
│   │   ├── app.module.ts               # 根模块
│   │   │
│   │   ├── common/                     # ── 公共层（跨模块复用） ──
│   │   │   ├── constants/
│   │   │   │   ├── error-codes.ts      # 错误码定义（code + message）
│   │   │   │   └── index.ts
│   │   │   ├── decorators/
│   │   │   │   ├── current-user.decorator.ts   # @CurrentUser() 参数装饰器
│   │   │   │   ├── public.decorator.ts         # @Public() 跳过认证
│   │   │   │   └── index.ts
│   │   │   ├── dto/
│   │   │   │   ├── pagination.dto.ts           # 分页请求 DTO
│   │   │   │   └── index.ts
│   │   │   ├── entities/
│   │   │   │   └── base.entity.ts              # 基础实体（id, createdAt, updatedAt, deletedAt）
│   │   │   ├── exceptions/
│   │   │   │   ├── business.exception.ts       # 业务异常类
│   │   │   │   └── index.ts
│   │   │   ├── filters/
│   │   │   │   └── global-exception.filter.ts  # 全局异常过滤器
│   │   │   ├── guards/
│   │   │   │   ├── jwt-auth.guard.ts           # JWT 认证守卫
│   │   │   │   ├── roles.guard.ts              # 角色权限守卫
│   │   │   │   └── index.ts
│   │   │   ├── interceptors/
│   │   │   │   ├── response.interceptor.ts     # 统一响应包装 { code, message, data }
│   │   │   │   ├── logging.interceptor.ts      # 请求日志（耗时、traceId）
│   │   │   │   └── index.ts
│   │   │   ├── interfaces/
│   │   │   │   ├── response.interface.ts       # IApiResponse<T>, IPaginatedData<T>
│   │   │   │   └── index.ts
│   │   │   ├── middlewares/
│   │   │   │   └── trace-id.middleware.ts       # 注入 traceId
│   │   │   ├── pipes/
│   │   │   │   └── parse-int-or-default.pipe.ts
│   │   │   └── utils/
│   │   │       ├── hash.util.ts                # bcrypt 封装
│   │   │       ├── pagination.util.ts          # 分页计算工具
│   │   │       └── index.ts
│   │   │
│   │   ├── config/                     # ── 配置层 ──
│   │   │   ├── app.config.ts           # 应用配置（port, prefix）
│   │   │   ├── database.config.ts      # PostgreSQL 连接配置
│   │   │   ├── redis.config.ts         # Redis 连接配置
│   │   │   ├── jwt.config.ts           # JWT 密钥/过期时间
│   │   │   └── index.ts
│   │   │
│   │   ├── database/                   # ── 数据库层 ──
│   │   │   ├── migrations/             # TypeORM 迁移文件（时间戳命名）
│   │   │   │   └── 20250207_create_users_table.ts
│   │   │   ├── seeds/                  # 种子数据
│   │   │   │   └── admin-user.seed.ts
│   │   │   └── data-source.ts          # TypeORM DataSource（CLI 用）
│   │   │
│   │   └── modules/                    # ── 业务模块（每个模块自包含） ──
│   │       │
│   │       ├── auth/
│   │       │   ├── auth.module.ts
│   │       │   ├── auth.controller.ts
│   │       │   ├── auth.service.ts
│   │       │   ├── dto/
│   │       │   │   ├── login.dto.ts
│   │       │   │   └── register.dto.ts
│   │       │   └── strategies/
│   │       │       └── jwt.strategy.ts
│   │       │
│   │       ├── users/
│   │       │   ├── users.module.ts
│   │       │   ├── users.controller.ts
│   │       │   ├── users.service.ts
│   │       │   ├── dto/
│   │       │   │   ├── create-user.dto.ts
│   │       │   │   ├── update-user.dto.ts
│   │       │   │   └── query-user.dto.ts   # 列表查询（继承 PaginationDto）
│   │       │   └── entities/
│   │       │       └── user.entity.ts
│   │       │
│   │       └── [other-modules]/        # 其他业务模块结构同上
│   │           ├── xx.module.ts
│   │           ├── xx.controller.ts
│   │           ├── xx.service.ts
│   │           ├── dto/
│   │           └── entities/
│   │
│   ├── test/
│   │   ├── unit/                       # 单元测试（镜像 src/modules 结构）
│   │   │   └── modules/
│   │   │       └── users/
│   │   │           └── users.service.spec.ts
│   │   ├── integration/                # 集成测试
│   │   │   └── users.e2e-spec.ts
│   │   └── jest-setup.ts
│   │
│   ├── .env.example                    # 环境变量模板
│   ├── .eslintrc.js
│   ├── .prettierrc
│   ├── nest-cli.json
│   ├── tsconfig.json
│   ├── tsconfig.build.json
│   ├── package.json
│   └── Dockerfile
│
├── frontend/                           # React 前端
│   ├── public/
│   │   └── favicon.ico
│   ├── src/
│   │   ├── main.tsx                    # 入口
│   │   ├── App.tsx                     # 根组件（路由、全局 Provider）
│   │   │
│   │   ├── components/                 # ── 公共组件（跨页面复用） ──
│   │   │   ├── common/
│   │   │   │   ├── PageHeader/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── DataTable/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── Pagination/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── StatusTag/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── EmptyState/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── LoadingState/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── ErrorBoundary/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── ConfirmModal/
│   │   │   │   │   └── index.tsx
│   │   │   │   └── FileUpload/
│   │   │   │       └── index.tsx
│   │   │   ├── layout/
│   │   │   │   ├── AppLayout/
│   │   │   │   │   └── index.tsx       # 整体布局（Sidebar + Header + Content）
│   │   │   │   ├── Sidebar/
│   │   │   │   │   └── index.tsx
│   │   │   │   └── Navbar/
│   │   │   │       └── index.tsx
│   │   │   └── feedback/
│   │   │       └── Toast/
│   │   │           └── index.tsx
│   │   │
│   │   ├── pages/                      # ── 页面（按业务模块组织） ──
│   │   │   ├── auth/
│   │   │   │   ├── Login/
│   │   │   │   │   └── index.tsx
│   │   │   │   └── Register/
│   │   │   │       └── index.tsx
│   │   │   ├── users/
│   │   │   │   ├── UserList/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── UserDetail/
│   │   │   │   │   └── index.tsx
│   │   │   │   ├── UserCreate/
│   │   │   │   │   └── index.tsx
│   │   │   │   └── components/         # 页面私有组件（仅此模块页面使用）
│   │   │   │       └── UserForm.tsx
│   │   │   └── dashboard/
│   │   │       └── index.tsx
│   │   │
│   │   ├── router/                     # ── 路由配置 ──
│   │   │   ├── index.tsx               # 路由表
│   │   │   └── guards.tsx              # 路由守卫（登录检查等）
│   │   │
│   │   ├── services/                   # ── API 请求层 ──
│   │   │   ├── http.ts                 # Axios 实例封装（拦截器、Token 注入、Mock 拦截、统一错误处理）
│   │   │   ├── types.ts                # ApiResponse<T>, PaginatedData<T> 等类型
│   │   │   └── modules/
│   │   │       ├── auth.ts
│   │   │       └── user.ts
│   │   │
│   │   ├── mock_data/                  # ── Mock 数据（VITE_API_MOCK_DATA=true 时启用） ──
│   │   │   ├── index.ts                # Mock 路由注册表（URL → handler 映射）
│   │   │   ├── _helpers.ts             # 工具函数（mockSuccess, mockPaginate, mockDelay）
│   │   │   └── modules/               # 按业务模块组织，与 services/modules/ 一一对应
│   │   │       ├── auth.mock.ts
│   │   │       └── user.mock.ts
│   │   │
│   │   ├── hooks/                      # ── 自定义 Hooks ──
│   │   │   ├── useAuth.ts
│   │   │   ├── usePagination.ts
│   │   │   └── useDebounce.ts
│   │   │
│   │   ├── stores/                     # ── 全局状态（Zustand） ──
│   │   │   └── auth.store.ts
│   │   │
│   │   ├── types/                      # ── 全局类型 ──
│   │   │   ├── api.d.ts
│   │   │   └── env.d.ts
│   │   │
│   │   ├── constants/                  # ── 常量 ──
│   │   │   └── index.ts
│   │   │
│   │   ├── utils/                      # ── 工具函数 ──
│   │   │   ├── format.ts              # 日期/金额格式化
│   │   │   └── storage.ts             # localStorage 封装
│   │   │
│   │   └── styles/                     # ── 全局样式 ──
│   │       ├── globals.css
│   │       └── variables.css
│   │
│   ├── .env.example
│   ├── .eslintrc.cjs
│   ├── .prettierrc
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── package.json
│   └── Dockerfile
│
├── docs/                               # 文档中心
│   ├── requirements/                   # 需求文档
│   │   └── v1.0-user-module.md
│   ├── design/                         # 设计文档
│   │   ├── architecture.md             # 架构设计
│   │   ├── database-erd.md             # ER 图 / 数据库设计
│   │   └── api-spec/                   # API 接口文档（如不用 Swagger 自动生成）
│   │       └── users.md
│   ├── plans/                          # 开发计划
│   │   └── sprint-01.md
│   ├── assets/                         # 设计图、截图等静态资源
│   │   └── wireframes/
│   ├── decisions/                      # ADR (Architecture Decision Records)
│   │   └── 001-use-typeorm.md
│   └── changelog.md                    # 变更日志
│
├── docker-compose.yml                  # 本地开发环境编排（PostgreSQL + Redis + backend + frontend）
├── .gitignore
├── .editorconfig
└── README.md                           # 项目说明、启动步骤
```

## 目录职责说明

### backend

| 目录 | 职责 | 放什么 | 不放什么 |
|------|------|--------|----------|
| `common/` | 跨模块复用的基础设施代码 | 异常类、守卫、拦截器、DTO基类、工具函数 | 业务逻辑 |
| `config/` | 配置集中管理 | 数据库连接、Redis、JWT、应用配置 | 业务常量 |
| `database/` | 数据库生命周期管理 | 迁移文件、种子数据、DataSource | 实体定义（实体跟随业务模块） |
| `modules/xx/` | 自包含的业务模块 | controller、service、dto、entity | 跨模块公共代码 |
| `modules/xx/dto/` | 该模块的请求/响应数据校验 | CreateXxDto、UpdateXxDto、QueryXxDto | 通用分页DTO（放 common） |
| `modules/xx/entities/` | 该模块的数据库实体 | XxEntity | 其他模块的实体 |

### frontend

| 目录 | 职责 | 放什么 | 不放什么 |
|------|------|--------|----------|
| `components/common/` | 跨页面复用的通用组件 | Pagination、DataTable、StatusTag | 仅某个页面用的组件 |
| `components/layout/` | 全局布局相关组件 | AppLayout、Sidebar、Navbar | 业务组件 |
| `pages/xx/` | 按业务模块组织的页面 | 列表页、详情页、创建页 | 公共组件 |
| `pages/xx/components/` | 该模块页面的私有组件 | UserForm（仅 users 模块用） | 跨页面组件（应提升到 components/） |
| `services/modules/` | API 请求函数 | userApi.getList()、userApi.create() | 业务逻辑、UI 逻辑 |
| `mock_data/modules/` | Mock 数据（与 services/modules 一一对应） | user.mock.ts、auth.mock.ts | 真实业务逻辑、组件代码 |
| `hooks/` | 可复用的状态逻辑 | usePagination、useDebounce | 一次性逻辑（留在页面内） |
| `stores/` | 全局客户端状态 | 认证状态、主题切换 | 服务端数据缓存（用 React Query） |

### docs

| 目录 | 职责 |
|------|------|
| `requirements/` | 产品需求文档、用户故事 |
| `design/` | 架构设计、数据库设计、API 规格 |
| `plans/` | Sprint 计划、里程碑、排期 |
| `assets/` | 设计稿、线框图、截图 |
| `decisions/` | 架构决策记录（ADR）— 记录为什么选 TypeORM 而非 Prisma 等 |

## 文件存放决策流程

新建文件时按此流程判断位置：

```
创建新文件
  │
  ├─ 是后端代码？
  │   ├─ 跨模块复用？ → backend/src/common/{对应子目录}/
  │   ├─ 配置相关？   → backend/src/config/
  │   ├─ 数据库迁移？ → backend/src/database/migrations/
  │   └─ 业务代码？   → backend/src/modules/{模块名}/{对应子目录}/
  │
  ├─ 是前端代码？
  │   ├─ 跨页面复用的 UI 组件？ → frontend/src/components/common/
  │   ├─ 布局组件？             → frontend/src/components/layout/
  │   ├─ 仅某模块使用的组件？   → frontend/src/pages/{模块}/components/
  │   ├─ 页面？                 → frontend/src/pages/{模块}/
  │   ├─ API 请求？             → frontend/src/services/modules/
│   ├─ Mock 数据？            → frontend/src/mock_data/modules/
  │   ├─ 可复用的 Hook？        → frontend/src/hooks/
  │   ├─ 全局状态？             → frontend/src/stores/
  │   └─ 工具函数/常量/类型？   → frontend/src/utils/ | constants/ | types/
  │
  └─ 是文档？
      ├─ 需求相关？   → docs/requirements/
      ├─ 设计相关？   → docs/design/
      ├─ 计划排期？   → docs/plans/
      ├─ 图片/设计稿？→ docs/assets/
      └─ 技术决策？   → docs/decisions/
```
