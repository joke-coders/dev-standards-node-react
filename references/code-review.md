# 代码审查与优化指南

## 1. 审查流程

Claude 审查代码时应按以下顺序进行：

```
1. 安全性 → 2. 正确性 → 3. 可维护性 → 4. 性能 → 5. 风格
```

**安全问题永远优先于风格问题。**

---

## 2. 审查清单

### 正确性

- [ ] 函数是否处理了所有边界条件（空值、零值、最大值、负值）
- [ ] 异步操作是否正确 await，是否处理了 reject
- [ ] 条件分支是否覆盖了所有情况（else / default case）
- [ ] 循环是否有正确的终止条件（无死循环）
- [ ] 类型转换是否安全（string → number 是否处理 NaN）
- [ ] 并发场景是否有竞态条件
- [ ] 资源（文件句柄、数据库连接、定时器）是否正确释放

### 可维护性

- [ ] 函数长度是否 ≤ 50 行（超过考虑拆分）
- [ ] 函数参数是否 ≤ 4 个（超过用对象参数）
- [ ] 嵌套层级是否 ≤ 3 层（超过用 early return / extract method）
- [ ] 命名是否清晰表达意图（不用 `data`, `info`, `temp`, `flag`）
- [ ] 是否有必要的注释（Why，非 What）
- [ ] 魔法数字/字符串是否提取为常量
- [ ] 重复代码是否已抽象

### 性能

- [ ] 是否有 N+1 查询问题
- [ ] 大数据集操作是否分批处理
- [ ] 是否有不必要的深拷贝
- [ ] 正则表达式是否有回溯爆炸风险
- [ ] 定时器/监听器是否在组件销毁时清理
- [ ] 数据库查询是否利用了索引

---

## 3. 代码坏味道识别

### 高优先级重构

| 坏味道 | 特征 | 重构手法 |
|--------|------|----------|
| **超长函数** | > 50 行 | Extract Method |
| **超大类** | > 300 行 | Extract Class / 模块拆分 |
| **重复代码** | 3+ 处相同/相似逻辑 | Extract → 公共函数/组件 |
| **深层嵌套** | if > 3 层 | Early Return / Guard Clause |
| **过长参数列表** | > 4 个参数 | Parameter Object |
| **Feature Envy** | 频繁访问另一个类的数据 | Move Method |
| **God Object** | 一个类做了太多事 | 按职责拆分 |

### 示例：深层嵌套重构

```typescript
// ❌ 重构前：深层嵌套
async function processOrder(order: Order) {
  if (order) {
    if (order.status === 'pending') {
      if (order.items.length > 0) {
        const user = await getUser(order.userId);
        if (user) {
          if (user.balance >= order.total) {
            // 处理逻辑...
          }
        }
      }
    }
  }
}

// ✅ 重构后：Guard Clause + Early Return
async function processOrder(order: Order) {
  if (!order) throw new Error('Order is required');
  if (order.status !== 'pending') return;
  if (order.items.length === 0) return;

  const user = await getUser(order.userId);
  if (!user) throw BusinessException.from(ErrorCodes.USER_NOT_FOUND);
  if (user.balance < order.total) throw BusinessException.from(ErrorCodes.INSUFFICIENT_BALANCE);

  // 处理逻辑...
}
```

---

## 4. 优化方向

### 可读性优化

```typescript
// ❌ 不清晰
if (status === 1 || status === 3 || status === 7) { ... }

// ✅ 清晰
const ACTIVE_STATUSES = [Status.PENDING, Status.PROCESSING, Status.REVIEW];
if (ACTIVE_STATUSES.includes(status)) { ... }
```

```typescript
// ❌ 含义不明
setTimeout(retry, 86400000);

// ✅ 含义清晰
const ONE_DAY_MS = 24 * 60 * 60 * 1000;
setTimeout(retry, ONE_DAY_MS);
```

### 健壮性优化

```typescript
// ❌ 脆弱：假设数据总是存在
const name = response.data.user.profile.name;

// ✅ 健壮：可选链 + 默认值
const name = response?.data?.user?.profile?.name ?? 'Unknown';
```

```typescript
// ❌ 脆弱：不处理错误
const data = JSON.parse(input);

// ✅ 健壮：处理异常
let data: unknown;
try {
  data = JSON.parse(input);
} catch {
  throw BusinessException.from(ErrorCodes.PARAM_INVALID);
}
```

### 性能优化

```typescript
// ❌ N+1 查询
for (const order of orders) {
  const user = await userRepo.findOne(order.userId); // 每次循环一条 SQL
}

// ✅ 批量查询
const userIds = orders.map(o => o.userId);
const users = await userRepo.find({ where: { id: In(userIds) } });
const userMap = new Map(users.map(u => [u.id, u]));
for (const order of orders) {
  const user = userMap.get(order.userId);
}
```

---

## 5. 审查输出格式

审查结果应结构化输出：

```markdown
## 代码审查结果

### 🔴 严重问题

**[S-01] SQL 注入风险**
- 📍 位置：`user.service.ts:L35`
- 💡 问题：直接拼接用户输入到 SQL 查询
- ✅ 修复：
  ```typescript
  // 修复前
  const sql = `SELECT * FROM users WHERE name = '${name}'`;
  // 修复后
  const users = await repo.find({ where: { name } });
  ```

### 🟠 警告

**[W-01] 缺少错误处理**
- 📍 位置：`payment.service.ts:L78`
- 💡 问题：第三方支付接口调用未捕获异常
- ✅ 修复：添加 try-catch 并做降级处理

### 🟡 建议

**[I-01] 函数过长**
- 📍 位置：`order.service.ts:L120-L195` (75行)
- 💡 建议：拆分为 `validateOrder()`, `processPayment()`, `updateInventory()`

### 🔵 优化

**[O-01] 可利用缓存**
- 📍 位置：`config.service.ts:L20`
- 💡 建议：系统配置查询频繁但极少变更，可加 5 分钟缓存
```

---

## 6. 技术债务评估

对于存量代码，按以下维度评估技术债务并排优先级：

| 维度 | 权重 | 评估标准 |
|------|------|----------|
| 安全风险 | 最高 | 是否存在可被利用的漏洞 |
| 稳定性影响 | 高 | 是否可能导致线上故障 |
| 开发效率影响 | 中 | 是否显著拖慢开发速度 |
| 用户体验影响 | 中 | 是否影响用户使用 |
| 维护成本 | 低 | 是否增加理解和修改的难度 |
