# 后端 API 开发规范

## 1. 统一响应格式

所有 API 接口必须返回统一的 JSON 响应结构，**无论成功还是失败**。

### 基础响应结构

```json
{
  "code": 0,
  "message": "success",
  "data": null
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `code` | `number` | ✅ | 业务状态码，0 表示成功，非 0 表示失败 |
| `message` | `string` | ✅ | 人类可读的描述信息 |
| `data` | `any \| null` | ✅ | 响应数据，失败时为 `null` |

### 成功响应示例

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 1,
    "name": "张三",
    "email": "zhangsan@example.com"
  }
}
```

### 失败响应示例

```json
{
  "code": 10001,
  "message": "用户不存在",
  "data": null
}
```

---

## 2. 错误码定义

错误码采用**分段设计**，每个模块使用独立的编号范围，便于定位问题来源。

### 错误码分段规则

| 范围 | 模块 | 说明 |
|------|------|------|
| `0` | 全局 | 成功 |
| `10000-10999` | 通用错误 | 参数校验、权限、限流等 |
| `11000-11999` | 用户模块 | 注册、登录、认证 |
| `12000-12999` | 订单模块 | 创建、支付、退款 |
| `13000-13999` | 商品模块 | 商品、库存、分类 |
| `20000-29999` | 第三方服务 | 支付网关、短信、邮件 |
| `90000-99999` | 系统错误 | 数据库、缓存、内部异常 |

### 错误码定义模板

**每个错误码必须同时定义 code 和 message**，且 message 要对用户友好：

```typescript
// TypeScript 示例
export const ErrorCodes = {
  // ===== 通用错误 10000-10999 =====
  PARAM_INVALID:        { code: 10001, message: '请求参数不合法' },
  UNAUTHORIZED:         { code: 10002, message: '未登录或登录已过期' },
  FORBIDDEN:            { code: 10003, message: '无权限访问' },
  RESOURCE_NOT_FOUND:   { code: 10004, message: '资源不存在' },
  RATE_LIMITED:         { code: 10005, message: '请求过于频繁，请稍后再试' },
  DUPLICATE_REQUEST:    { code: 10006, message: '重复请求，请勿重复提交' },

  // ===== 用户模块 11000-11999 =====
  USER_NOT_FOUND:       { code: 11001, message: '用户不存在' },
  USER_ALREADY_EXISTS:  { code: 11002, message: '用户已存在' },
  PASSWORD_INCORRECT:   { code: 11003, message: '密码错误' },
  TOKEN_EXPIRED:        { code: 11004, message: '令牌已过期' },
  ACCOUNT_DISABLED:     { code: 11005, message: '账号已被禁用' },

  // ===== 系统错误 90000-99999 =====
  INTERNAL_ERROR:       { code: 90001, message: '服务器内部错误' },
  DB_ERROR:             { code: 90002, message: '数据库操作失败' },
  CACHE_ERROR:          { code: 90003, message: '缓存服务异常' },
  THIRD_PARTY_ERROR:    { code: 90004, message: '第三方服务异常' },
} as const;
```

```python
# Python 示例
from enum import Enum
from dataclasses import dataclass

@dataclass(frozen=True)
class ErrorCode:
    code: int
    message: str

class Errors:
    # ===== 通用错误 =====
    PARAM_INVALID      = ErrorCode(10001, "请求参数不合法")
    UNAUTHORIZED        = ErrorCode(10002, "未登录或登录已过期")
    FORBIDDEN           = ErrorCode(10003, "无权限访问")
    RESOURCE_NOT_FOUND  = ErrorCode(10004, "资源不存在")

    # ===== 用户模块 =====
    USER_NOT_FOUND      = ErrorCode(11001, "用户不存在")
    USER_ALREADY_EXISTS = ErrorCode(11002, "用户已存在")

    # ===== 系统错误 =====
    INTERNAL_ERROR      = ErrorCode(90001, "服务器内部错误")
    DB_ERROR            = ErrorCode(90002, "数据库操作失败")
```

```java
// Java 示例
public enum ErrorCode {
    SUCCESS(0, "success"),
    PARAM_INVALID(10001, "请求参数不合法"),
    UNAUTHORIZED(10002, "未登录或登录已过期"),
    USER_NOT_FOUND(11001, "用户不存在"),
    INTERNAL_ERROR(90001, "服务器内部错误");

    private final int code;
    private final String message;

    ErrorCode(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() { return code; }
    public String getMessage() { return message; }
}
```

```go
// Go 示例
type AppError struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
}

var (
    ErrParamInvalid    = AppError{Code: 10001, Message: "请求参数不合法"}
    ErrUnauthorized    = AppError{Code: 10002, Message: "未登录或登录已过期"}
    ErrUserNotFound    = AppError{Code: 11001, Message: "用户不存在"}
    ErrInternalError   = AppError{Code: 90001, Message: "服务器内部错误"}
)
```

---

## 3. 分页规范

**所有列表查询接口必须实现分页**，严禁一次返回全部数据。

### 分页请求参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `page` | `number` | `1` | 页码，从 1 开始 |
| `pageSize` | `number` | `20` | 每页条数，最大不超过 100 |
| `sortBy` | `string` | `createdAt` | 排序字段（可选） |
| `sortOrder` | `'asc' \| 'desc'` | `desc` | 排序方向（可选） |

### 分页响应格式（固定结构）

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.list` | `T[]` | 当前页数据列表 |
| `data.pagination.page` | `number` | 当前页码 |
| `data.pagination.pageSize` | `number` | 每页条数 |
| `data.pagination.total` | `number` | 总记录数 |
| `data.pagination.totalPages` | `number` | 总页数 |

### 分页实现模板

```typescript
// TypeScript / NestJS 示例
export class PaginationDto {
  @IsOptional()
  @IsInt()
  @Min(1)
  @Type(() => Number)
  page: number = 1;

  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(100)
  @Type(() => Number)
  pageSize: number = 20;
}

// Service 层分页查询
async findWithPagination(dto: PaginationDto) {
  const { page, pageSize } = dto;
  const skip = (page - 1) * pageSize;

  const [list, total] = await this.repository.findAndCount({
    skip,
    take: pageSize,
    order: { createdAt: 'DESC' },
  });

  return {
    list,
    pagination: {
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
    },
  };
}
```

```python
# Python / FastAPI 示例
from pydantic import BaseModel, Field
from typing import TypeVar, Generic, List

T = TypeVar("T")

class PaginationMeta(BaseModel):
    page: int
    page_size: int
    total: int
    total_pages: int

class PaginatedResponse(BaseModel, Generic[T]):
    list: List[T]
    pagination: PaginationMeta

async def paginate(query, page: int = 1, page_size: int = 20):
    total = await query.count()
    items = await query.offset((page - 1) * page_size).limit(page_size).all()
    return {
        "list": items,
        "pagination": {
            "page": page,
            "page_size": page_size,
            "total": total,
            "total_pages": math.ceil(total / page_size),
        }
    }
```

---

## 4. 统一异常处理

后端必须有**全局异常处理器**，确保：
- 任何未捕获的异常都返回统一格式
- 生产环境不暴露堆栈信息
- 所有异常都记录日志
- 区分业务异常和系统异常

### 业务异常类

```typescript
// TypeScript 示例
export class BusinessException extends Error {
  constructor(
    public readonly code: number,
    public readonly msg: string,
  ) {
    super(msg);
    this.name = 'BusinessException';
  }

  static from(errorCode: { code: number; message: string }) {
    return new BusinessException(errorCode.code, errorCode.message);
  }
}

// 使用方式
throw BusinessException.from(ErrorCodes.USER_NOT_FOUND);
```

```python
# Python 示例
class BusinessException(Exception):
    def __init__(self, code: int, message: str):
        self.code = code
        self.message = message
        super().__init__(message)

    @classmethod
    def from_error(cls, error: ErrorCode):
        return cls(error.code, error.message)
```

### 全局异常过滤器

```typescript
// NestJS 全局异常过滤器
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger('ExceptionFilter');

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();

    if (exception instanceof BusinessException) {
      // 业务异常 — 返回对应错误码
      response.status(200).json({
        code: exception.code,
        message: exception.msg,
        data: null,
      });
      return;
    }

    if (exception instanceof BadRequestException) {
      // 参数校验异常
      const exceptionResponse = exception.getResponse() as any;
      response.status(200).json({
        code: ErrorCodes.PARAM_INVALID.code,
        message: Array.isArray(exceptionResponse.message)
          ? exceptionResponse.message[0]
          : exceptionResponse.message,
        data: null,
      });
      return;
    }

    // 未知异常 — 记录日志，返回通用错误
    this.logger.error('Unhandled exception', exception instanceof Error ? exception.stack : exception);
    response.status(200).json({
      code: ErrorCodes.INTERNAL_ERROR.code,
      message: process.env.NODE_ENV === 'production'
        ? ErrorCodes.INTERNAL_ERROR.message
        : String(exception),
      data: null,
    });
  }
}
```

```python
# FastAPI 全局异常处理
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(BusinessException)
async def business_exception_handler(request: Request, exc: BusinessException):
    return JSONResponse(content={
        "code": exc.code,
        "message": exc.message,
        "data": None,
    })

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unhandled exception: {exc}", exc_info=True)
    return JSONResponse(content={
        "code": 90001,
        "message": "服务器内部错误" if IS_PRODUCTION else str(exc),
        "data": None,
    })
```

---

## 5. 事务管理

**涉及多表写操作时，必须在最小范围内使用事务。**

### 核心原则

1. **最小范围** — 事务只包裹必要的数据库操作，不要包含网络 IO、文件操作等
2. **快速失败** — 任一步骤失败立即回滚，不要吞掉异常
3. **幂等设计** — 重试时不会产生副作用
4. **避免长事务** — 事务持有锁的时间越短越好

### 事务使用模板

```typescript
// TypeScript / TypeORM 示例
async transferFunds(fromId: number, toId: number, amount: number) {
  // 事务只包裹数据库操作
  return this.dataSource.transaction(async (manager) => {
    const from = await manager.findOne(Account, {
      where: { id: fromId },
      lock: { mode: 'pessimistic_write' },
    });
    const to = await manager.findOne(Account, {
      where: { id: toId },
      lock: { mode: 'pessimistic_write' },
    });

    if (!from || !to) throw BusinessException.from(ErrorCodes.RESOURCE_NOT_FOUND);
    if (from.balance < amount) throw BusinessException.from(ErrorCodes.INSUFFICIENT_BALANCE);

    from.balance -= amount;
    to.balance += amount;

    await manager.save(Account, [from, to]);
    await manager.save(TransferLog, {
      fromId, toId, amount, createdAt: new Date(),
    });
  });
  // 事务外：发送通知等非关键操作
}
```

```python
# Python / SQLAlchemy 示例
async def transfer_funds(from_id: int, to_id: int, amount: Decimal):
    async with async_session() as session:
        async with session.begin():  # 自动 commit/rollback
            from_acc = await session.get(Account, from_id, with_for_update=True)
            to_acc = await session.get(Account, to_id, with_for_update=True)

            if not from_acc or not to_acc:
                raise BusinessException.from_error(Errors.RESOURCE_NOT_FOUND)
            if from_acc.balance < amount:
                raise BusinessException.from_error(Errors.INSUFFICIENT_BALANCE)

            from_acc.balance -= amount
            to_acc.balance += amount

            session.add(TransferLog(from_id=from_id, to_id=to_id, amount=amount))
    # 事务外：发送通知
```

---

## 6. 接口设计规范

### RESTful 约定

| 操作 | HTTP 方法 | URL 模式 | 示例 |
|------|-----------|----------|------|
| 查询列表 | `GET` | `/resources` | `GET /api/users` |
| 查询详情 | `GET` | `/resources/:id` | `GET /api/users/123` |
| 创建 | `POST` | `/resources` | `POST /api/users` |
| 全量更新 | `PUT` | `/resources/:id` | `PUT /api/users/123` |
| 部分更新 | `PATCH` | `/resources/:id` | `PATCH /api/users/123` |
| 删除 | `DELETE` | `/resources/:id` | `DELETE /api/users/123` |

### 接口版本控制

```
/api/v1/users
/api/v2/users
```

### 请求参数校验

所有接口入参必须进行校验，不信任任何客户端输入：

```typescript
// 使用 DTO + class-validator
export class CreateUserDto {
  @IsString()
  @Length(2, 50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: '密码必须包含大小写字母和数字',
  })
  password: string;

  @IsOptional()
  @IsPhoneNumber()
  phone?: string;
}
```

---

## 7. 日志规范

### 日志级别使用

| 级别 | 用途 | 示例 |
|------|------|------|
| `ERROR` | 影响业务的异常 | 数据库连接失败、第三方服务超时 |
| `WARN` | 潜在问题但不影响运行 | 重试成功、缓存降级 |
| `INFO` | 关键业务节点 | 用户注册、订单创建、支付完成 |
| `DEBUG` | 开发调试信息 | SQL 语句、请求/响应详情 |

### 日志必须包含

- 请求 ID（traceId）— 用于链路追踪
- 用户 ID（如已登录）
- 操作描述
- 关键参数（脱敏后）
- 耗时（对于关键操作）

### 敏感信息脱敏

日志中**严禁**输出以下明文信息：密码、Token、密钥、完整身份证号、完整银行卡号、完整手机号。

---

## 8. 接口幂等性

以下类型的接口必须保证幂等：

| 场景 | 实现方式 |
|------|----------|
| 支付/下单 | 唯一业务号 + 数据库唯一约束 |
| 表单提交 | 请求头 `X-Idempotency-Key` + Redis 去重 |
| 回调通知 | 根据通知 ID 去重 |

---

## 9. 接口限流

所有对外接口应配置合理的限流策略：

| 类型 | 策略 | 示例 |
|------|------|------|
| 全局限流 | 令牌桶 / 滑动窗口 | 1000 req/s |
| 用户级限流 | 按 userId 或 IP | 60 req/min |
| 接口级限流 | 按接口路径 | 登录接口 5 req/min |
