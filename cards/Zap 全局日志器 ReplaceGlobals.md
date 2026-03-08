---
description: 解决 Logger 层层传参的污染问题，通过全局单例实现任意位置直接调用。
date: 2026-02-27T15:51:38+08:00
---

# Zap 全局日志器 ReplaceGlobals

### 🧠 痛点：Logger 层层传参污染

不使用全局日志器时，必须把 `logger` 像接力棒一样塞进每个函数的参数列表，造成严重的"参数污染"，代码极其啰嗦。

❌ **灾难做法：**
```go
func main() {
    logger := zap.NewExample()
    UserController(logger) // 传！
}

func UserController(logger *zap.Logger) {
    UserService(logger)    // 传！
}

func UserService(logger *zap.Logger) {
    logger.Info("终于可以用 logger 了...")
}
```

### 🧠 解决方案：`zap.ReplaceGlobals()` 注册全局单例

在 `main` 函数启动时，调用 `zap.ReplaceGlobals(logger)` 将配置好的 Logger 注册为全局单例。之后在任意位置直接通过 `zap.L()`（Logger）或 `zap.S()`（SugaredLogger）获取实例，无需传参。

✅ **优雅做法：**
```go
// 1. 在 main 入口注册一次
func main() {
    logger, _ := zap.NewProduction()
    defer logger.Sync()

    zap.ReplaceGlobals(logger) // 关键：设为全局单例

    UserController() // 干干净净，不需要传 logger
}

// 2. 在任意深层函数中直接使用
func UserService() {
    zap.L().Info("用户服务被调用")
    zap.S().Infof("亦可获取 SugaredLogger: %s", "方便")
}
```