---
created: 2026-03-17T17:33:00+08:00
---

# `QueryContext`：用上下文执行查询并返回多行结果集

## 函数签名

- `func (db *DB) QueryContext(ctx context.Context, query string, args ...any) (*Rows, error)`

- **所属包：** `database/sql`

### 参数

- `ctx context.Context`：本次查询使用的上下文，可用于控制超时、截止时间或取消信号。

- `query string`：要执行的 SQL 查询语句。

- `args ...any`：SQL 语句中的占位参数，会按顺序绑定到查询中。

### 返回值

- `rows *sql.Rows`：表示多行查询结果集的对象，需要遍历读取每一行数据。

- `err error`：执行查询时返回的错误；若成功则为 `nil`。

## 代码示例

```go
package main

import (
    "context"
    "database/sql"
    "time"
)

type User struct {
    ID   int64
    Name string
}

func listUsers(ctx context.Context, db *sql.DB) ([]User, error) {
    queryCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()

    rows, err := db.QueryContext(queryCtx, "SELECT id, name FROM users ORDER BY id")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var users []User
    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Name); err != nil {
            return nil, err
        }
        users = append(users, user)
    }

    if err := rows.Err(); err != nil {
        return nil, err
    }

    return users, nil
}
```

## 注意事项

1. **查询成功后要记得关闭 `rows`：** `QueryContext` 返回的 `*sql.Rows` 底层会占用连接相关资源。

2. **遍历结束后要检查 `rows.Err()`：** 迭代过程中发生的错误，不一定会在 `QueryContext` 调用时直接返回，应该在 `for rows.Next()` 结束后补一次 `rows.Err()` 检查。

3. **它适合返回多条记录的场景：** 如果你明确只取一条记录，通常用 `QueryRowContext` 更直接；如果是增删改语句，则应使用 `ExecContext`。
