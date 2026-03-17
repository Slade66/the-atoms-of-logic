# `QueryRowContext`：用上下文执行查询并返回单行结果

## 函数签名

- `func (db *DB) QueryRowContext(ctx context.Context, query string, args ...any) *Row`

- **所属包：** `database/sql`

### 参数

- `ctx context.Context`：本次查询使用的上下文，可用于控制超时、截止时间或取消信号。

- `query string`：要执行的 SQL 查询语句。

- `args ...any`：SQL 语句中的占位参数，会按顺序绑定到查询中。

### 返回值

- `row *sql.Row`：表示单行查询结果的对象，通常对其继续调用 `Scan` 读取字段值。

## 代码示例

```go
package main

import (
    "context"
    "database/sql"
    "errors"
    "time"
)

type User struct {
    ID   int64
    Name string
}

func getUserByID(ctx context.Context, db *sql.DB, id int64) (*User, error) {
    queryCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()

    row := db.QueryRowContext(queryCtx, "SELECT id, name FROM users WHERE id = ?", id)

    var user User
    if err := row.Scan(&user.ID, &user.Name); err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, nil
        }
        return nil, err
    }

    return &user, nil
}
```

## 注意事项

1. **它返回的是 `*sql.Row`，不是错误：** `QueryRowContext` 本身不会直接返回查询错误，常见错误会延迟到 `row.Scan(...)` 时暴露。

2. **查不到数据时要判断 `sql.ErrNoRows`：** 当查询结果为空时，`Scan` 会返回 `sql.ErrNoRows`，这通常表示“没查到记录”，不是程序异常。

3. **适合明确只取一条记录的场景：** 例如按主键或唯一索引查询；如果预期返回多条记录，应使用 `QueryContext`。
