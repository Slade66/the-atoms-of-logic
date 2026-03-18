---
created: 2026-03-18T13:41:06+08:00
---

1. 安装 SQLite 驱动

- **执行：** `go get github.com/mattn/go-sqlite3`

- **需要 CGO：** 构建时需要 `gcc`，还要启用 `CGO_ENABLED=1`。

2. 打开/创建数据库

- **SQLite 的特点：** 文件不存在就自动创建。

- **代码示例：**

    ```go
    package main

    import (
        "database/sql"
        "fmt"

        _ "github.com/mattn/go-sqlite3"
    )

    func main() {
        db, err := sql.Open("sqlite3", "./test.db")
        if err != nil {
            panic(err)
        }
        defer db.Close()

        fmt.Println("数据库连接成功")
    }
    ```

3. 执行 SQL 语句

    - **代码示例（创建表）：**
    
        ```go
        createTableSQL := `
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            age INTEGER
        );
        `

        _, err = db.Exec(createTableSQL)
        if err != nil {
            panic(err)
        }

        fmt.Println("表创建成功")
        ```
