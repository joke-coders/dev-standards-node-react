# 前端开发规范

## 1. 组件抽象原则

### 核心规则

**创建页面时，如果有相同内容的展示，必须抽象为公共组件。** 具体判断标准：

| 条件 | 动作 |
|------|------|
| 相同 UI 出现在 2 个以上页面 | **必须**抽象为公共组件（UI 组件阈值：2+） |
| 相同 UI 在同一页面重复 2 次以上 | **必须**抽象为组件 |
| UI 相似但有差异（70%+ 相同） | **应该**抽象为组件 + props 控制差异 |
| 未来可能复用的独立功能块 | **建议**抽象为组件 |

### 典型需要抽象的公共组件

```
components/
├── common/
│   ├── PageHeader/           # 页面标题栏（标题、面包屑、操作按钮）
│   ├── SearchForm/           # 搜索/筛选表单
│   ├── DataTable/            # 通用表格（分页、排序、选择）
│   ├── Pagination/           # 分页组件
│   ├── StatusTag/            # 状态标签（成功/失败/进行中等）
│   ├── ConfirmModal/         # 确认对话框
│   ├── EmptyState/           # 空数据状态
│   ├── LoadingState/         # 加载状态（骨架屏/spinner）
│   ├── ErrorBoundary/        # 错误边界
│   ├── Avatar/               # 用户头像
│   ├── FileUpload/           # 文件上传
│   ├── RichTextEditor/       # 富文本编辑器封装
│   └── ImagePreview/         # 图片预览
├── layout/
│   ├── AppLayout/            # 全局布局
│   ├── Sidebar/              # 侧边栏
│   ├── Navbar/               # 导航栏
│   └── Footer/               # 页脚
└── feedback/
    ├── Toast/                # 轻提示
    ├── Alert/                # 警告提示
    └── ProgressBar/          # 进度条
```

### 组件设计规范

```tsx
// ✅ 好的组件设计 — 通过 props 控制差异
interface StatusTagProps {
  status: 'success' | 'error' | 'pending' | 'warning';
  label?: string;       // 可自定义文案
  size?: 'sm' | 'md';   // 可控大小
}

export function StatusTag({ status, label, size = 'md' }: StatusTagProps) {
  const config = {
    success: { color: 'green', defaultLabel: '成功' },
    error:   { color: 'red',   defaultLabel: '失败' },
    pending: { color: 'blue',  defaultLabel: '进行中' },
    warning: { color: 'yellow',defaultLabel: '警告' },
  };
  const { color, defaultLabel } = config[status];

  return (
    <span className={`tag tag-${color} tag-${size}`}>
      {label ?? defaultLabel}
    </span>
  );
}

// ❌ 坏的做法 — 每个页面各自实现状态展示
// page-a.tsx: <span className="green">成功</span>
// page-b.tsx: <span style={{color: 'green'}}>已完成</span>
// page-c.tsx: <div className="status-ok">Success</div>
```

### 分页组件封装

由于分页在几乎所有列表页面都会出现，**必须封装统一的分页组件**：

```tsx
interface PaginationProps {
  page: number;
  pageSize: number;
  total: number;
  onChange: (page: number, pageSize: number) => void;
  pageSizeOptions?: number[];
}

export function Pagination({
  page,
  pageSize,
  total,
  onChange,
  pageSizeOptions = [10, 20, 50, 100],
}: PaginationProps) {
  const totalPages = Math.ceil(total / pageSize);
  // ...实现分页 UI
}
```

### 自定义 Hook：usePagination

分页逻辑可复用，必须封装为 Hook：

```tsx
// hooks/usePagination.ts
import { useState } from 'react';

interface PaginationState {
  page: number;
  pageSize: number;
}

export function usePagination(initialPageSize: number = 20) {
  const [pagination, setPagination] = useState<PaginationState>({
    page: 1,
    pageSize: initialPageSize,
  });

  const handlePaginationChange = (page: number, pageSize: number) => {
    setPagination({ page, pageSize });
  };

  const resetPage = () => {
    setPagination({ ...pagination, page: 1 });
  };

  return {
    page: pagination.page,
    pageSize: pagination.pageSize,
    pagination,
    onChange: handlePaginationChange,
    resetPage,
  };
}
```

### 列表页面标准模板

每个列表页面的结构应保持一致，使用 `usePagination` Hook 管理分页：

```tsx
export function UserListPage() {
  // 1. 状态
  const [filters, setFilters] = useState<UserFilters>({});
  const { page, pageSize, pagination, onChange: onPaginationChange, resetPage } = usePagination();

  // 2. 数据请求
  const { data, isLoading, error } = useQuery({
    queryKey: ['users', filters, pagination],
    queryFn: () => userApi.getList({ ...filters, ...pagination }),
  });

  // 3. 过滤器变化时重置分页
  const handleFilterChange = (newFilters: UserFilters) => {
    setFilters(newFilters);
    resetPage();
  };

  // 4. 渲染 — 统一结构
  return (
    <div>
      <PageHeader title="用户管理" actions={<CreateButton />} />
      <SearchForm fields={filterFields} onSearch={handleFilterChange} />

      {isLoading && <LoadingState />}
      {error && <ErrorState error={error} />}
      {data && (
        <>
          <DataTable columns={columns} data={data.list} />
          <Pagination
            page={data.pagination.page}
            pageSize={data.pagination.pageSize}
            total={data.pagination.total}
            onChange={onPaginationChange}
          />
        </>
      )}
    </div>
  );
}
```

### 创建/编辑页面标准模板

#### UserForm 私有组件

创建和编辑页面共用一个表单组件，通过 `mode` prop 区分：

```tsx
// pages/users/components/UserForm.tsx
interface UserFormProps {
  mode: 'create' | 'edit';
  defaultValues?: User;
  onSuccess?: () => void;
}

export function UserForm({ mode, defaultValues, onSuccess }: UserFormProps) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<User>({
    defaultValues: defaultValues || {},
  });

  const createMutation = useMutation({
    mutationFn: (data: User) => userApi.create(data),
    onSuccess: () => {
      toast.success('用户创建成功');
      onSuccess?.();
    },
  });

  const updateMutation = useMutation({
    mutationFn: (data: User) => userApi.update(defaultValues?.id!, data),
    onSuccess: () => {
      toast.success('用户更新成功');
      onSuccess?.();
    },
  });

  const onSubmit = (data: User) => {
    if (mode === 'create') {
      createMutation.mutate(data);
    } else {
      updateMutation.mutate(data);
    }
  };

  const isLoading = createMutation.isPending || updateMutation.isPending;

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>姓名</label>
        <input {...register('name', { required: '姓名必填' })} />
        {errors.name && <span>{errors.name.message}</span>}
      </div>

      <div>
        <label>邮箱</label>
        <input {...register('email', { required: '邮箱必填', pattern: /^\S+@\S+$/ })} />
        {errors.email && <span>{errors.email.message}</span>}
      </div>

      <button type="submit" disabled={isLoading}>
        {isLoading ? '提交中...' : mode === 'create' ? '创建' : '更新'}
      </button>
    </form>
  );
}
```

#### UserCreate 页面

```tsx
// pages/users/UserCreate/index.tsx
import { useNavigate } from 'react-router-dom';
import { UserForm } from '../components/UserForm';
import { PageHeader } from '@/components/common/PageHeader';

export default function UserCreatePage() {
  const navigate = useNavigate();

  const handleSuccess = () => {
    // 创建成功后跳转到列表页
    navigate('/users');
  };

  return (
    <div>
      <PageHeader title="新建用户" backTo="/users" />
      <UserForm mode="create" onSuccess={handleSuccess} />
    </div>
  );
}
```

#### UserEdit 页面

```tsx
// pages/users/UserEdit/index.tsx
import { useParams, useNavigate } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';
import { UserForm } from '../components/UserForm';
import { PageHeader } from '@/components/common/PageHeader';
import { LoadingState } from '@/components/common/LoadingState';
import { ErrorState } from '@/components/common/ErrorState';
import { userApi } from '@/services/modules/user';

export default function UserEditPage() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();

  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', id],
    queryFn: () => userApi.getById(Number(id)),
    enabled: !!id,
  });

  const handleSuccess = () => {
    // 编辑成功后跳转到详情页
    navigate(`/users/${id}`);
  };

  if (isLoading) return <LoadingState />;
  if (error) return <ErrorState error={error} />;
  if (!user) return <ErrorState error={new Error('用户不存在')} />;

  return (
    <div>
      <PageHeader title="编辑用户" backTo={`/users/${id}`} />
      <UserForm mode="edit" defaultValues={user} onSuccess={handleSuccess} />
    </div>
  );
}
```

### 页面私有组件提升标准

页面私有组件何时应提升为公共组件：

| 条件 | 处置 |
|------|------|
| 组件**仅在本模块**（1 个以上页面）使用 | 保留在 `pages/{模块}/components/` |
| 组件**跨 2 个以上模块**使用 | 提升至 `components/common/`（如 PageHeader、DataTable） |
| 组件**仅 1 个页面内使用** | 保留在页面文件内，无需单独文件（如复杂的表单字段组件） |

**转移规则**：如果发现私有组件在第二个模块被引用，立即提升为公共组件，避免代码重复。

---

## 2. 目录结构

```
src/
├── components/         # 公共组件（跨页面复用）
│   ├── common/
│   ├── layout/
│   └── feedback/
├── pages/              # 页面组件（按业务模块组织）
│   └── users/
│       ├── UserList/           # 列表页
│       │   └── index.tsx
│       ├── UserDetail/         # 详情页
│       │   └── index.tsx
│       ├── UserCreate/         # 创建页
│       │   └── index.tsx
│       ├── UserEdit/           # 编辑页
│       │   └── index.tsx
│       └── components/         # 页面私有组件（仅此模块页面使用）
│           └── UserForm.tsx
├── hooks/              # 自定义 Hooks
│   ├── useAuth.ts
│   ├── usePagination.ts
│   └── useDebounce.ts
├── router/             # 路由配置
│   ├── index.tsx
│   └── guards.tsx
├── services/           # API 请求层
│   ├── http.ts                 # Axios/Fetch 封装
│   ├── types.ts                # 请求/响应类型
│   └── modules/
│       ├── user.ts
│       └── order.ts
├── stores/             # 全局状态管理
├── utils/              # 工具函数
├── constants/          # 常量定义
├── types/              # 全局类型
└── styles/             # 全局样式
```

---

## 2.1 路由配置

### router/index.tsx — 路由表

使用 `createBrowserRouter` + `React.lazy` 进行懒加载：

```tsx
import { createBrowserRouter, RouteObject } from 'react-router-dom';
import { Suspense } from 'react';
import { ErrorBoundary } from '@/components/common/ErrorBoundary';
import { LoadingState } from '@/components/common/LoadingState';
import { authGuard } from './guards';

// 懒加载页面
const UserListPage = lazy(() => import('@/pages/users/UserList'));
const UserDetailPage = lazy(() => import('@/pages/users/UserDetail'));
const UserCreatePage = lazy(() => import('@/pages/users/UserCreate'));
const UserEditPage = lazy(() => import('@/pages/users/UserEdit'));
const LoginPage = lazy(() => import('@/pages/auth/Login'));

const routes: RouteObject[] = [
  {
    path: '/login',
    element: <LoginPage />,
  },
  {
    path: '/',
    element: <AppLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: 'users',
        children: [
          {
            path: '',
            element: (
              <ErrorBoundary>
                <Suspense fallback={<LoadingState />}>
                  <UserListPage />
                </Suspense>
              </ErrorBoundary>
            ),
          },
          {
            path: ':id',
            element: (
              <ErrorBoundary>
                <Suspense fallback={<LoadingState />}>
                  <UserDetailPage />
                </Suspense>
              </ErrorBoundary>
            ),
          },
          {
            path: 'create',
            element: (
              <ErrorBoundary>
                <Suspense fallback={<LoadingState />}>
                  <UserCreatePage />
                </Suspense>
              </ErrorBoundary>
            ),
          },
          {
            path: ':id/edit',
            element: (
              <ErrorBoundary>
                <Suspense fallback={<LoadingState />}>
                  <UserEditPage />
                </Suspense>
              </ErrorBoundary>
            ),
          },
        ],
      },
    ],
  },
];

export const router = createBrowserRouter(routes);
```

### router/guards.tsx — 认证守卫

路由守卫检查认证状态，未登录则重定向到登录页：

```tsx
import { Navigate } from 'react-router-dom';
import { useAuthStore } from '@/stores/auth.store';

export function authGuard() {
  const { token, isAuthenticated } = useAuthStore();

  // 如果未登录，重定向到登录页
  if (!isAuthenticated || !token) {
    return <Navigate to="/login" replace />;
  }

  return null;
}

// 在路由配置中使用
// 方案 1：包裹页面组件
// <ProtectedRoute element={<UserListPage />} />

// 方案 2：使用 loader （推荐 React Router v6.4+）
// {
//   path: 'users',
//   loader: async () => {
//     if (!useAuthStore.getState().isAuthenticated) {
//       return redirect('/login');
//     }
//     return null;
//   },
//   element: <UserListPage />,
// }
```

---

## 3. API 请求层封装

### HTTP 客户端封装

所有 API 请求必须通过统一的 HTTP 客户端，不允许在组件中直接使用 `fetch` 或 `axios`：

```typescript
// services/http.ts
import axios from 'axios';
import type { ApiResponse, PaginatedData } from './types';

const http = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15000,
});

// 请求拦截 — 注入 Token
http.interceptors.request.use((config) => {
  const token = getToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截 — 统一处理错误
http.interceptors.response.use(
  (response) => {
    const { code, message, data } = response.data as ApiResponse;
    if (code !== 0) {
      // 统一处理业务错误
      if (code === 10002) {
        // Token 过期 → 跳转登录
        redirectToLogin();
        return Promise.reject(new Error(message));
      }
      // 其他业务错误 → Toast 提示
      toast.error(message);
      return Promise.reject(new Error(message));
    }
    return data; // 直接返回 data，上层不用再解构
  },
  (error) => {
    // 网络错误
    if (!error.response) {
      toast.error('网络连接失败，请检查网络');
    } else {
      toast.error(`服务器错误 (${error.response.status})`);
    }
    return Promise.reject(error);
  },
);

export default http;
```

### API 模块

```typescript
// services/modules/user.ts
import http from '../http';
import type { PaginatedData } from '../types';

export interface User {
  id: number;
  name: string;
  email: string;
  status: string;
}

export interface UserListParams {
  page?: number;
  pageSize?: number;
  keyword?: string;
  status?: string;
}

export const userApi = {
  getList: (params: UserListParams) =>
    http.get<PaginatedData<User>>('/api/v1/users', { params }),

  getById: (id: number) =>
    http.get<User>(`/api/v1/users/${id}`),

  create: (data: CreateUserDto) =>
    http.post<User>('/api/v1/users', data),

  update: (id: number, data: UpdateUserDto) =>
    http.patch<User>(`/api/v1/users/${id}`, data),

  delete: (id: number) =>
    http.delete<void>(`/api/v1/users/${id}`),
};
```

---

## 4. 状态管理

### 选择原则

| 场景 | 推荐方案 |
|------|----------|
| 服务端数据（API 响应） | React Query / SWR / TanStack Query |
| 全局 UI 状态（主题、侧边栏） | Zustand / Jotai / Context |
| 表单状态 | React Hook Form / Formik |
| 组件局部状态 | useState / useReducer |

### 关键规则

1. **服务端状态和客户端状态分离** — API 数据用 React Query，不要塞进 Redux/Zustand
2. **状态就近原则** — 状态放在最近的共同父组件，不要什么都往全局放
3. **避免 prop drilling 超过 3 层** — 超过 3 层用 Context 或状态管理库

---

## 5. 错误处理

### 错误边界

每个页面级路由必须包裹错误边界：

```tsx
// components/common/ErrorBoundary.tsx
import { Component, type ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // 上报错误监控
    reportError(error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="error-page">
          <h2>页面出错了</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            重试
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### 加载与空状态

每个数据驱动的页面必须处理以下状态：

```tsx
// 必须处理的 4 种状态
if (isLoading) return <LoadingState />;
if (error)     return <ErrorState error={error} onRetry={refetch} />;
if (!data || data.list.length === 0) return <EmptyState message="暂无数据" />;
return <DataTable data={data.list} />;
```

---

## 6. 性能优化

| 技术 | 应用场景 | 实现 |
|------|----------|------|
| 路由懒加载 | 所有页面级路由 | `React.lazy()` + `Suspense` |
| 组件 memo | 列表项、纯展示组件 | `React.memo()` |
| 虚拟滚动 | 长列表（>100 条可见） | `react-virtuoso` / `tanstack-virtual` |
| 防抖搜索 | 搜索框输入 | `useDebounce(value, 300)` |
| 图片懒加载 | 图片列表、长页面 | `loading="lazy"` / Intersection Observer |
| 请求缓存 | API 重复请求 | React Query `staleTime` 配置 |

---

## 7. 表单处理规范

### 统一规则

1. **表单校验** — 前端校验用于即时反馈，后端校验用于安全保障，**两者都不可省略**
2. **提交防重** — 提交按钮在请求期间必须禁用
3. **加载态** — 提交时显示 loading 状态
4. **错误展示** — 字段级错误显示在字段下方，全局错误显示在表单顶部

```tsx
// 表单提交标准模式
const { mutate, isPending } = useMutation({
  mutationFn: userApi.create,
  onSuccess: () => {
    toast.success('创建成功');
    navigate('/users');
  },
  onError: (err) => {
    // 错误已在 HTTP 拦截器中处理
  },
});

<button type="submit" disabled={isPending}>
  {isPending ? '提交中...' : '提交'}
</button>
```

---

## 8. 国际化准备

即使当前只有一种语言，也应遵循以下规则以便未来扩展：

1. **文案不硬编码** — 所有用户可见文案集中管理在 `constants/` 或 i18n 文件中
2. **日期/数字格式化** — 使用 `Intl` API 或 `dayjs`，不手动拼接
3. **不要拼接句子** — `"共" + count + "条"` → 使用模板 `共 {count} 条`
