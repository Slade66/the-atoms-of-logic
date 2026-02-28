---
title: 架构分层：main 组装层与 App 业务层
description: main.go 只负责组装依赖并注入 App，业务逻辑全部下沉到 internal/ 的 App 结构体中。
date: 2026-02-27T16:05:35+08:00
tags: [Go, 架构]
---

### 🧠 痛点：`main.go` 沦为上帝函数

当所有逻辑都塞在 `main.go` 里时，它会迅速膨胀成一个既初始化资源、又处理业务、还做错误收尾的"上帝函数"。这样的代码无法被单元测试覆盖（`main` 函数不能被外部调用），也无法复用任何业务逻辑。

```go
// main.go 变成上帝函数：既建连接，又跑业务，无法测试
func main() {
    logger, _ := zap.NewProduction()
    db, _ := sql.Open("postgres", os.Getenv("DB_URL"))

    rows, _ := db.Query("SELECT * FROM users")
    for rows.Next() {
        var name string
        rows.Scan(&name)
        logger.Info("processing", zap.String("user", name))
        // 更多业务逻辑...
    }
}
```

### 🧠 原则：`main.go` 是接线板，`App` 结构体是业务逻辑

**`main.go` 只做四件事：**

1. 初始化基础设施（Logger、数据库连接）
2. 解析 CLI 参数 / 加载配置文件
3. 把上述依赖注入到 `App` 结构体中
4. 调用 `app.Run()` 启动程序

绝不应出现 `for` 循环、业务 `if` 判断或数据处理逻辑。

**`App` 结构体** 位于 `internal/app.go`，是业务逻辑的真正归宿：

- **字段**：持有所有外部依赖（Logger、DB 等），由构造函数注入，自身不负责创建
- **`NewApp`**：唯一的依赖注入入口，参数就是这个结构体所需的一切
- **`Run`**：业务流程的顶层入口，所有 `for` 循环、数据处理、错误处理都在这里展开

`App` 对依赖的来源一无所知——它只管拿来就用。测试时把 mock 依赖传进去，业务逻辑照常可跑。

```go
// internal/app.go —— 业务逻辑的归宿
type App struct {
    logger *zap.Logger
    db     *sql.DB
}

func NewApp(logger *zap.Logger, db *sql.DB) *App {
    return &App{logger: logger, db: db}
}

func (a *App) Run() error {
    rows, err := a.db.Query("SELECT name FROM users")
    if err != nil {
        return fmt.Errorf("query users: %w", err)
    }
    defer rows.Close()

    for rows.Next() {
        var name string
        if err := rows.Scan(&name); err != nil {
            return fmt.Errorf("scan: %w", err)
        }
        a.logger.Info("processing user", zap.String("name", name))
    }
    return rows.Err()
}
```

```go
// main.go —— 纯粹的接线板
func main() {
    if err := run(); err != nil {
        fmt.Fprintf(os.Stderr, "error: %v\n", err)
        os.Exit(1)
    }
}

func run() error {
    logger, _ := zap.NewProduction()
    defer logger.Sync()

    db, err := sql.Open("postgres", os.Getenv("DB_URL"))
    if err != nil {
        return fmt.Errorf("open db: %w", err)
    }
    defer db.Close()

    app := internal.NewApp(logger, db) // 把依赖"插"进去
    return app.Run()                   // 业务全在 App 里
}
```
