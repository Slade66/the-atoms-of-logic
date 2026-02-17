# Zap

## 简介与安装
- **是什么：** Zap 是 Uber 开源的高性能日志库，旨在解决标准库 `log` 性能不足和缺乏结构化支持的问题。
- **核心优势：**
    - **高性能：** 基于“零内存分配”理念设计，比标准库快数倍。
    - **结构化：** 原生支持键值对日志，便于 ELK/Loki 等日志系统解析。
    - **日志分级：** 提供细粒度的日志级别（Debug/Info/Error 等），弥补标准库 `log` 的不足。
- **安装命令：** `go get -u go.uber.org/zap`

## 日志缓冲与强制落盘
- **底层原理：** 为了实现极致性能，Zap 采用了“异步缓冲写入”策略。日志生成后不会立即进行 I/O 操作（写磁盘/打印），而是先暂存在内存缓冲区中。
- **潜在风险：** 如果程序意外退出或正常结束时缓冲区未清空，残留在缓冲区内的最后几条日志将会丢失。
- **解决方案：** 养成好习惯，在创建 Logger 后立即调用 `defer logger.Sync()`。这能保证程序退出前强制刷新缓冲区，将所有剩余日志安全落盘。

## 三种开箱即用的 Logger
- **`zap.NewExample()` —— 极简模式：**
    - **适用场景：** 仅用于编写文档示例或单元测试，不建议在实际项目中使用。
    - **配置特点：** 只有最基础的日志信息，去掉了时间戳、调用方等所有“噪音”。
    - **输出样例：** `{"level":"debug","msg":"Something happened"}`
- **`zap.NewDevelopment()` —— 开发模式：**
    - **适用场景：** 本地开发与调试。
    - **配置特点：** 像 `fmt.Println` 一样直观。开启 Debug 级别，使用彩色文本格式而非 JSON，并自动记录堆栈信息，方便肉眼快速定位错误。
    - **输出样例：** `2025-11-09T14:03:21.123+0800 DEBUG main.go:10 Something happened`
- **`zap.NewProduction()` —— 生产模式：**
    - **适用场景：** 线上部署。
    - **配置特点：** 机器友好。开启 Info 级别，使用 JSON 格式便于 ELK/Splunk 解析；启用采样器防止高频错误日志打爆磁盘。
    - **输出样例：** `{"level":"info","ts":1731138201.123,"caller":"main.go:10","msg":"Something happened"}`

## Development vs Production
- | 维度 | NewDevelopment (🛠️ 开发模式) | NewProduction (🚀 生产部署) |
  | :--- | :--- | :--- |
  | **设计目标** | **人眼友好**：格式直观，方便本地排错 | **机器友好**：JSON 结构化，方便 ELK/Splunk 解析 |
  | **编码格式** | Console Encoder (纯文本，Level 带颜色) | JSON Encoder (标准 JSON 格式) |
  | **最低级别** | Debug (调试信息全量输出) | Info (忽略 Debug，减少噪音) |
  | **时间字段** | ISO8601 (`2024-11-09T14:03:21.123+0800`) | Epoch 浮点数 (`1731138201.123`) |
  | **调用信息** | `main.go:10` (高亮显示，紧跟 Level) | `"caller":"main.go:10"` (作为独立 JSON 字段) |
  | **堆栈跟踪** | **Warn** 及以上级别自动打印堆栈 | **Error** 及以上级别自动打印堆栈 |
  | **日志采样** | 禁用 (每一条日志都打印) | 启用 (默认: 同一日志每秒前100条保留，其后每100条保留1条) |
  | **完整样例** | `2024-11-09T14:03:21.123+0800	DEBUG	zap-demo/main.go:10	连接数据库失败	{"url": "http://db.local", "attempt": 3}` | `{"level":"info","ts":1731138201.123,"caller":"zap-demo/main.go:10","msg":"用户登录成功","user_id":12345,"ip":"192.168.1.1"}` |

## 设置全局日志器
- **痛点（不使用全局日志器前）：**
    - **层层传递：** 必须把 `logger` 像接力棒一样塞进每个函数的参数列表，代码极其啰嗦。
    - ```go
      // 典型的 "参数污染"：logger 被迫穿透所有层级
      func main() {
          logger := zap.NewExample()
          UserController(logger) // 传！
      }
      
      func UserController(logger *zap.Logger) {
          UserService(logger)   // 传！
      }
      
      func UserService(logger *zap.Logger) {
          logger.Info("终于可以用 logger 了...") 
      }
      ```
- **解决方案（使用全局日志器后）：**
    - **方法：** 在 `main` 函数启动时，使用 `zap.ReplaceGlobals(logger)` 将你配置好的 Logger 注册为全局单例。
    - **效果：** 任意地方直接调用 `zap.L()`（Logger）或 `zap.S()`（SugaredLogger），无需传参，随时随地打印。
    - ```go
      // 1. 在 main 入口处注册一次（通常在 init 或 main 开头）
      func main() {
          logger, _ := zap.NewProduction()
          defer logger.Sync()
          
          // 关键：将这个 logger 设为全局单例
          zap.ReplaceGlobals(logger) 
          
          UserController() // 看！干干净净，不需要传 logger
      }

      // 2. 在任意深层函数中直接使用
      func UserService() {
          // zap.L() 直接获取全局 Logger，无需任何传参
          zap.L().Info("用户服务被调用") 
          zap.S().Infof("亦可获取 SugaredLogger: %s", "方便")
      }
      ```

## SugaredLogger（加糖模式）
- **核心定位：** 适合业务逻辑复杂、对微秒级性能不敏感的场景。
- **语法特点：** 支持 `printf` 格式化，接受松散的键值对（`interface{}`），代码写起来像标准库一样舒服。
- **代码示例：**
- ```go
    sugar := logger.Sugar() // 1. 创建糖衣 logger
    
    // 方式 A：结构化（推荐，键值对）
    sugar.Infow("获取URL失败",
        "url", "http://example.com",
        "attempt", 3,
    )
    
    // 方式 B：Printf 风格（传统）
    sugar.Infof("获取 %s 失败，重试次数: %d", "http://example.com", 3)
    ```

## Logger（原始模式）
- **核心定位：** 适合高频热点代码（如网关、中间件核心循环）。
- **语法特点：** 只接受强类型字段（`zap.Field`），杜绝反射，零内存分配。
- **代码示例：**
- ```go
    // 注意：没有 printf 风格的方法
    logger.Info("获取URL失败",
        zap.String("url", "http://example.com"),
        zap.Int("attempt", 3),
    )
    ```

## SugaredLogger vs Logger
- ```plain
  +-------------+-----------------------------+----------------------------+
  | 特性        | Logger (原始)                | SugaredLogger (加糖)       |
  +-------------+-----------------------------+----------------------------+
  | 核心优势    | 极致性能，零内存分配           | 开发效率，书写便捷          |
  | 语法风格    | 仅支持强类型结构化             | 支持 Printf 及松散键值对    |
  | 参数类型    | zap.Field (如 zap.String)    | 任意类型 (interface{})      |
  | 适用场景    | 网关、基础库、热点循环         | 业务逻辑、Web服务、脚本      |
  | 性能等级    | ★★★★★ (最快)             | ★★★★☆ (略慢，仍极快)    |
  +-------------+-----------------------------+----------------------------+
  ```

## 日志级别
- Zap 提供了从 Debug 到 Fatal 的全套分级打印函数，每个级别对应特定的语义和副作用：
- ```go
  // 1. 常规记录（无副作用）
  logger.Debug("调试", zap.String("var", "abc"))   // 仅在 Development 模式或 Level <= Debug 时打印
  logger.Info("信息", zap.Int("status", 200))      // 正常业务里程碑
  logger.Warn("警告", zap.Duration("cost", 3*time.Second)) // 潜在隐患（如慢请求）
  logger.Error("错误", zap.Error(err))            // 功能受损，但程序继续运行

  // 2. 特殊行为（有副作用）
  logger.DPanic("调试恐慌") // 在 Development 模式下触发 panic，Production 模式下仅记录 Error
  logger.Panic("恐慌")      // 记录日志后，立即触发 panic()
  logger.Fatal("致命")      // 记录日志后，立即调用 os.Exit(1) 终止程序
  ```

## 结构化日志字段的构造
- **核心作用：** 将变量转换为日志系统可识别的强类型字段，避免使用反射，提升性能。
- **常用构造函数：**
    - **基础类型：** `zap.String("k", "v")`, `zap.Int("k", 123)`, `zap.Bool("k", true)`
    - **错误处理：** `zap.Error(err)` —— 自动提取错误消息并记录（推荐）。
    - **时间相关：** `zap.Duration("cost", time.Second)`, `zap.Time("now", time.Now())`
    - **兜底方案：** `zap.Any("k", v)` —— 使用反射处理任意类型（性能略低，仅在类型不确定时使用）。
- **使用示例：**
    - ```go
      logger.Info("用户下单失败",
          // 1. 基础类型（高性能）
          zap.String("order_id", "ORD-123456"),
          zap.Int("amount", 998),
          
          // 2. 特殊类型（优化展示）
              zap.Error(errors.New("库存不足")), // 自动包含堆栈
              zap.Duration("latency", 350*time.Millisecond), // 自动格式化时间
              
              // 3. 任意类型（反射兜底）
              zap.Any("raw_data", map[string]int{"a": 1}), 
          )
          /*
          输出 (JSON):
          {
            "msg": "用户下单失败",
            "order_id": "ORD-123456",
            "amount": 998,
            "error": "库存不足",
            "latency": "0.35s",
            "raw_data": {"a":1}
          }
          */
        )
      ```

## 日志上下文
- **痛点：**
    - 在处理一次 HTTP 请求时，必须在每一行日志里手动重复添加 `request_id`，既繁琐又容易遗漏。
    - ```go
        // 每一行都要重复写 zap.String("req_id", "req-123")，很痛苦
        logger.Info("开始处理", zap.String("req_id", "req-123"))
        logger.Info("参数校验", zap.String("req_id", "req-123"))
        logger.Warn("库存不足", zap.String("req_id", "req-123"))
      ```
- **解决方案：**
    - **原理：** 使用 `logger.With(...)` 创建一个绑定了公共字段的衍生 Logger。
    - **示例：**
    - ```go
        // 1. 创建衍生 Logger：自动携带 "req_id": "req-123"
        reqLogger := logger.With(zap.String("req_id", "req-123"))
        
        // 2. 后续直接打印：自动包含上述字段，无需重复写
        reqLogger.Info("开始处理") // -> {"msg":"开始处理", "req_id":"req-123"}
        reqLogger.Info("参数校验") // -> {"msg":"参数校验", "req_id":"req-123"}
        reqLogger.Warn("库存不足") // -> {"msg":"库存不足", "req_id":"req-123"}
      ```
