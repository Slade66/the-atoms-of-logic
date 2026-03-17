# `PingContext`：用上下文执行一次数据库连通性检查

## 函数签名

- `func (db *DB) PingContext(ctx context.Context) error`

- **所属包：** `database/sql`

### 参数

- `ctx context.Context`：本次连通性检测使用的上下文，可用于设置超时或取消信号。

### 返回值

- `err error`：如果数据库可达且检测成功，返回 `nil`；如果超时、被取消或连接失败，则返回对应错误。

## 代码示例

```go
package main

import (
    "context"
    "database/sql"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func initDB(dsn string) (*sql.DB, error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }

    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    if err := db.PingContext(ctx); err != nil {
        db.Close()
        return nil, err
    }

    return db, nil
}
```

## 注意事项

1. **常用于启动时验证连接配置：** `sql.Open` 本身不会立即建立连接，通常会紧跟一次 `PingContext` 来确认 DSN、网络和数据库服务是否正常。

2. **要优先使用带超时的上下文：** 如果直接传入 `context.Background()`，当网络异常或数据库长时间无响应时，检测可能阻塞较久；实际使用中通常搭配 `context.WithTimeout`。
