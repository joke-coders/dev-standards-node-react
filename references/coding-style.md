# 通用编码规范

## 1. 命名规范

### 通用规则

| 类型 | 风格 | 示例 |
|------|------|------|
| 变量/函数 | camelCase | `getUserById`, `isActive` |
| 类/接口/类型 | PascalCase | `UserService`, `CreateUserDto` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `API_BASE_URL` |
| 文件名 (TS/JS) | kebab-case | `user-service.ts`, `create-user.dto.ts` |
| 文件名 (Python) | snake_case | `user_service.py` |
| 文件名 (React 组件) | PascalCase | `UserProfile.tsx`, `DataTable.tsx` |
| 数据库表/字段 | snake_case | `user_profiles`, `created_at` |
| URL 路径 | kebab-case | `/api/user-profiles` |
| 环境变量 | UPPER_SNAKE_CASE | `DATABASE_URL` |

### 命名语义化

```typescript
// ❌ 模糊命名
const d = new Date();
const list = getAll();
const flag = check();
function process(data: any) {}

// ✅ 语义化命名
const createdAt = new Date();
const activeUsers = getActiveUsers();
const isEligible = checkEligibility(user);
function calculateOrderTotal(order: Order): number {}
```

### 布尔值命名

布尔变量和函数以 `is`, `has`, `can`, `should` 开头：

```typescript
const isActive = true;
const hasPermission = checkPermission(user);
const canEdit = isOwner || isAdmin;
function shouldRetry(error: Error): boolean {}
```

### 集合命名

集合类型使用复数名词：

```typescript
const users: User[] = [];           // ✅ 复数
const userMap = new Map<number, User>();  // ✅ 加类型后缀
const userIds = new Set<number>();   // ✅ 复数 + 类型提示
```

---

## 2. 注释规范

### 何时写注释

| 场景 | 是否注释 | 说明 |
|------|----------|------|
| 复杂业务逻辑 | ✅ 必须 | 解释 **为什么** 这样做 |
| 非显而易见的算法 | ✅ 必须 | 解释算法思路 |
| Hack / Workaround | ✅ 必须 | 说明原因和理想方案 |
| 公共 API / 函数签名 | ✅ 推荐 | JSDoc / docstring |
| 显而易见的代码 | ❌ 不写 | `i++ // increment i` 这种无意义注释 |

### 注释格式

```typescript
/**
 * 计算订单折扣金额
 *
 * 折扣规则：
 * - 会员等级 >= Gold 且订单金额 >= 500 → 9折
 * - 使用优惠券 → 按券面值扣减（与会员折扣可叠加）
 * - 最终价格不低于 0
 *
 * @param order - 订单信息
 * @param coupon - 优惠券（可选）
 * @returns 折扣金额（正数）
 */
function calculateDiscount(order: Order, coupon?: Coupon): number {
  // ...
}

// HACK: 第三方库 v2.3 在 Safari 下会崩溃，临时用 polyfill 绕过
// 跟踪 issue: https://github.com/xxx/issues/123
// TODO: 升级到 v2.4 后移除
const result = safariPolyfill(input);

// FIXME: 并发场景下可能出现竞态条件，需要加分布式锁
await updateBalance(userId, amount);
```

### TODO / FIXME 规范

| 标记 | 含义 | 示例 |
|------|------|------|
| `TODO` | 待实现/待优化 | `// TODO: 添加缓存提升性能` |
| `FIXME` | 已知缺陷待修复 | `// FIXME: 并发下有竞态问题` |
| `HACK` | 临时方案需后续改进 | `// HACK: 绕过 SDK bug` |
| `NOTE` | 需要注意的设计决策 | `// NOTE: 故意不加索引，因为写远多于读` |

---

## 3. 文件组织

### 文件大小

| 指标 | 建议值 | 说明 |
|------|--------|------|
| 单文件行数 | ≤ 300 行 | 超过考虑拆分 |
| 单函数行数 | ≤ 50 行 | 超过考虑拆分 |
| 单类方法数 | ≤ 10 个 | 超过考虑职责拆分 |
| 函数参数数 | ≤ 4 个 | 超过用对象参数 |
| 文件导入数 | ≤ 15 个 | 超过说明职责过多 |

### 导入排序

```typescript
// 1. 标准库/运行时
import { readFile } from 'fs/promises';

// 2. 第三方库
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';

// 3. 项目内部模块（绝对路径）
import { BusinessException } from '@/common/exceptions';
import { ErrorCodes } from '@/common/constants';

// 4. 相对路径导入
import { CreateUserDto } from './dto/create-user.dto';
import { UserEntity } from './entities/user.entity';
```

---

## 4. Git 规范

### Commit Message 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 类型

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(user): add login with email` |
| `fix` | 修复 Bug | `fix(order): fix total calculation with discount` |
| `refactor` | 重构（非新功能、非修复） | `refactor(auth): extract token validation` |
| `perf` | 性能优化 | `perf(query): add index for user search` |
| `docs` | 文档变更 | `docs: update API documentation` |
| `style` | 代码格式（不影响运行） | `style: fix indentation` |
| `test` | 测试 | `test(user): add unit tests for service` |
| `chore` | 构建/工具变更 | `chore: upgrade dependencies` |
| `ci` | CI/CD 配置 | `ci: add deploy stage` |

### 分支策略

```
main          ← 生产代码，只接受合并
├── develop   ← 开发主分支
│   ├── feature/user-auth    ← 功能分支
│   ├── feature/order-flow
│   └── fix/login-redirect   ← 修复分支
└── hotfix/critical-bug      ← 紧急修复（从 main 拉出）
```

### 分支命名

```
feature/{module}-{description}   → feature/user-email-login
fix/{module}-{description}       → fix/order-total-calc
hotfix/{description}             → hotfix/payment-timeout
refactor/{description}           → refactor/auth-middleware
```

---

## 5. 错误处理风格

### 通用原则

1. **不吞异常** — 捕获异常后必须处理（记日志、抛出、降级），不能空 catch
2. **精确捕获** — 捕获具体的异常类型，不要 `catch(Exception e)`
3. **fail fast** — 参数校验失败立即返回，不要继续执行
4. **异常 vs 返回值** — 不可恢复的错误用异常，可预期的失败用返回值

```typescript
// ❌ 吞异常
try {
  await riskyOperation();
} catch (e) {
  // 什么都不做
}

// ❌ 用 console.log 代替错误处理
try {
  await riskyOperation();
} catch (e) {
  console.log(e);
}

// ✅ 正确处理
try {
  await riskyOperation();
} catch (error) {
  logger.error('riskyOperation failed', { error, context });
  throw BusinessException.from(ErrorCodes.INTERNAL_ERROR);
}
```

---

## 6. 测试规范

### 测试文件组织

```
tests/
├── unit/           # 单元测试
│   └── services/
│       └── user.service.spec.ts
├── integration/    # 集成测试
│   └── api/
│       └── users.spec.ts
└── e2e/            # 端到端测试
    └── user-flow.spec.ts
```

### 测试命名

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid data', async () => {});
    it('should throw if email already exists', async () => {});
    it('should hash password before saving', async () => {});
  });
});
```

### 测试原则

1. **单一职责** — 每个测试只验证一个行为
2. **独立性** — 测试之间不依赖执行顺序
3. **可读性** — 测试名称清晰描述预期行为
4. **AAA 模式** — Arrange（准备）→ Act（执行）→ Assert（断言）
