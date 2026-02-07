# 前端开发规范

## 1. 组件抽象原则

### 核心规则

**创建页面时，如果有相同内容的展示，必须抽象为公共组件。** 具体判断标准：

| 条件 | 动作 |
|------|------|
| 相同 UI 出现在 2 个以上页面 | **必须**抽象为公共组件 |
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

### 列表页面标准模板

每个列表页面的结构应保持一致：

```tsx
export function UserListPage() {
  // 1. 状态
  const [filters, setFilters] = useState<UserFilters>({});
  const [pagination, setPagination] = useState({ page: 1, pageSize: 20 });

  // 2. 数据请求
  const { data, isLoading, error } = useQuery({
    queryKey: ['users', filters, pagination],
    queryFn: () => userApi.getList({ ...filters, ...pagination }),
  });

  // 3. 渲染 — 统一结构
  return (
    <div>
      <PageHeader title="用户管理" actions={<CreateButton />} />
      <SearchForm fields={filterFields} onSearch={setFilters} />

      {isLoading && <LoadingState />}
      {error && <ErrorState error={error} />}
      {data && (
        <>
          <DataTable columns={columns} data={data.list} />
          <Pagination
            page={data.pagination.page}
            pageSize={data.pagination.pageSize}
            total={data.pagination.total}
            onChange={(page, pageSize) => setPagination({ page, pageSize })}
          />
        </>
      )}
    </div>
  );
}
```

---

## 2. 目录结构

```
src/
├── components/         # 公共组件（跨页面复用）
│   ├── common/
│   ├── layout/
│   └── feedback/
├── pages/              # 页面组件
│   └── users/
│       ├── index.tsx           # 列表页
│       ├── detail.tsx          # 详情页
│       ├── create.tsx          # 创建页
│       └── components/         # 页面私有组件（仅此页面使用）
│           └── UserForm.tsx
├── hooks/              # 自定义 Hooks
│   ├── useAuth.ts
│   ├── usePagination.ts
│   └── useDebounce.ts
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
