---
created: 2026-03-17T17:34:00+08:00
---

# `ExecContext`：用上下文执行不返回结果集（增、删、改）的 SQL 语句

## 函数签名

- `func (db *DB) ExecContext(ctx context.Context, query string, args ...any) (Result, error)`

- **所属包：** `database/sql`

### 参数

- `ctx context.Context`：本次执行使用的上下文，可用于控制超时、截止时间或取消信号。

- `query string`：要执行的 SQL 语句，通常是 `INSERT`、`UPDATE`、`DELETE` 或 DDL 语句。

- `args ...any`：SQL 语句中的占位参数，会按顺序绑定到语句中。

### 返回值

- `result sql.Result`：执行结果对象，可继续调用 `LastInsertId` 或 `RowsAffected` 获取附加信息。

- `err error`：执行 SQL 时返回的错误；若成功则为 `nil`。

## 代码示例

```go
package main

import (
    "context"
    "database/sql"
    "time"
)

func updateUserName(ctx context.Context, db *sql.DB, id int64, name string) error {
    execCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()

    result, err := db.ExecContext(execCtx, "UPDATE users SET name = ? WHERE id = ?", name, id)
    if err != nil {
        return err
    }

    affected, err := result.RowsAffected()
    if err != nil {
        return err
    }
    if affected == 0 {
        return sql.ErrNoRows
    }

    return nil
}
```

## 注意事项

1. **它不适合查询数据：** `ExecContext` 用于执行不返回结果集的语句；如果要读取记录，应使用 `QueryContext` 或 `QueryRowContext`。

2. **返回的是 `sql.Result`，不是具体数据：** 常见用法是继续调用 `RowsAffected()` 查看影响行数，或在插入场景下调用 `LastInsertId()` 获取新记录 ID；但具体支持情况取决于驱动。

3. **更新或删除时要关注影响行数：** SQL 成功执行不代表一定改到了数据，很多业务场景下还需要检查 `RowsAffected()` 是否为 0。
