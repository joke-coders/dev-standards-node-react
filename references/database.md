# 数据库设计与操作规范

## 1. 表设计规范

### 命名规则

| 对象 | 规则 | 示例 |
|------|------|------|
| 表名 | snake_case 复数 | `users`, `order_items` |
| 字段名 | snake_case | `created_at`, `wallet_address` |
| 索引名 | `idx_{表名}_{字段名}` | `idx_users_email` |
| 唯一索引 | `uk_{表名}_{字段名}` | `uk_users_email` |
| 外键 | `fk_{表名}_{关联表}` | `fk_orders_user_id` |

### 必备字段

每张业务表必须包含以下字段：

```sql
CREATE TABLE users (
  id          BIGINT       PRIMARY KEY AUTO_INCREMENT,
  -- ... 业务字段 ...
  created_at  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at  TIMESTAMP    NULL     DEFAULT NULL     -- 软删除（可选但推荐）
);
```

| 字段 | 说明 |
|------|------|
| `id` | 主键，推荐自增或雪花算法 |
| `created_at` | 创建时间，由数据库自动填充 |
| `updated_at` | 更新时间，由数据库自动更新 |
| `deleted_at` | 软删除标记，NULL 表示未删除 |

### 字段类型选择

| 场景 | 推荐类型 | 避免使用 |
|------|----------|----------|
| 主键 | `BIGINT` | `INT`（容易溢出） |
| 金额/精确小数 | `DECIMAL(18,2)` | `FLOAT/DOUBLE`（精度丢失） |
| 短文本 (<256) | `VARCHAR(n)` | `TEXT` |
| 长文本 | `TEXT` | `VARCHAR(65535)` |
| 布尔值 | `TINYINT(1)` / `BOOLEAN` | `ENUM('Y','N')` |
| 时间 | `TIMESTAMP` / `DATETIME` | `VARCHAR` 存时间字符串 |
| JSON 数据 | `JSON` (MySQL 5.7+) | `TEXT` 存 JSON 字符串 |
| 枚举/状态 | `TINYINT` + 代码层映射 | `ENUM`（不利于扩展） |

---

## 2. 索引规范

### 必须建索引的场景

1. `WHERE` 条件中的高频查询字段
2. `JOIN` 关联字段
3. `ORDER BY` 排序字段
4. 唯一性约束字段（如 email、手机号）

### 索引原则

1. **单表索引数量 ≤ 5** — 过多索引降低写入性能
2. **复合索引遵循最左前缀原则** — 把筛选度最高的字段放最左
3. **避免在低区分度字段上建索引** — 如 `status`、`gender`（除非与其他字段组合）
4. **覆盖索引** — 查询字段尽量被索引覆盖，避免回表
5. **避免索引失效** — 不在索引列上使用函数、类型转换、`LIKE '%xxx'`

### 示例

```sql
-- 复合索引：先 status（筛选），再 created_at（排序）
CREATE INDEX idx_orders_status_created ON orders(status, created_at);

-- 唯一索引
CREATE UNIQUE INDEX uk_users_email ON users(email);

-- 覆盖索引：查询只需要 id 和 name
CREATE INDEX idx_users_status_name ON users(status, name);
```

---

## 3. 查询优化

### 禁止事项

| 禁止 | 原因 | 替代方案 |
|------|------|----------|
| `SELECT *` | 浪费带宽，无法利用覆盖索引 | 明确列出需要的字段 |
| 不带 `LIMIT` 的查询 | 可能返回百万行数据 | 始终加 `LIMIT` |
| 在循环中逐条查询 | N+1 问题 | 批量查询 `WHERE id IN (...)` |
| `OFFSET` 大分页 | 性能随页数线性退化 | 游标分页 `WHERE id > lastId` |
| 子查询关联 | 多数情况性能差 | 改用 `JOIN` |

### 大分页优化

```sql
-- ❌ 慢：OFFSET 越大越慢
SELECT * FROM orders ORDER BY id DESC LIMIT 20 OFFSET 100000;

-- ✅ 快：游标分页（需前端传 lastId）
SELECT * FROM orders WHERE id < :lastId ORDER BY id DESC LIMIT 20;

-- ✅ 快：延迟关联（先查主键再回表）
SELECT o.* FROM orders o
INNER JOIN (SELECT id FROM orders ORDER BY id DESC LIMIT 20 OFFSET 100000) t
ON o.id = t.id;
```

---

## 4. 事务与锁

### 事务隔离级别

| 级别 | 脏读 | 不可重复读 | 幻读 | 推荐场景 |
|------|------|-----------|------|----------|
| READ COMMITTED | ✗ | ✓ | ✓ | 大多数 Web 应用（推荐） |
| REPEATABLE READ | ✗ | ✗ | ✓ | MySQL 默认，财务系统 |
| SERIALIZABLE | ✗ | ✗ | ✗ | 极端一致性要求 |

### 锁策略

```sql
-- 悲观锁：确保并发安全，但降低吞吐
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- 乐观锁：高并发读多写少场景
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = :expectedVersion;
```

### 死锁预防

1. **固定加锁顺序** — 多表操作按表名字母序加锁
2. **缩小锁范围** — 锁单行而非锁表
3. **缩短事务时间** — 不在事务中做 IO 操作
4. **设置锁超时** — `innodb_lock_wait_timeout = 5`

---

## 5. 数据库迁移

### 必须遵守的规则

1. **每次变更一个迁移文件** — 不要把多个不相关的变更放在同一个迁移中
2. **迁移必须可回滚** — 每个 `up` 都要有对应的 `down`
3. **不要修改已执行的迁移** — 只能创建新的迁移来修正
4. **大表变更要注意** — 百万级以上表的 ALTER TABLE 需要用 `pt-online-schema-change` 或类似工具
5. **迁移命名** — 使用时间戳前缀 + 描述：`20240115_add_email_to_users`

---

## 6. ORM 使用规范

### 实体与数据库映射

代码中属性使用 camelCase，数据库字段使用 snake_case，通过 ORM 配置自动映射：

```typescript
// TypeORM 示例
@Entity('users')
export class UserEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ name: 'first_name', length: 50 })
  firstName: string;

  @Column({ name: 'wallet_address', length: 42, nullable: true })
  walletAddress?: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;

  @DeleteDateColumn({ name: 'deleted_at' })
  deletedAt?: Date;
}
```

### 关联查询

```typescript
// ✅ 好：用 QueryBuilder 控制加载
const users = await this.userRepo
  .createQueryBuilder('u')
  .leftJoinAndSelect('u.orders', 'o', 'o.status = :status', { status: 'active' })
  .where('u.id = :id', { id })
  .getOne();

// ❌ 坏：eager loading 加载所有关联
@OneToMany(() => Order, order => order.user, { eager: true }) // 不要这么做
orders: Order[];
```
