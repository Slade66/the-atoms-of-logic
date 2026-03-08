---
description: 控制日志是否附加调用者文件行号，以及从哪个级别开始自动打印调用栈。
date: 2026-02-27T15:51:38+08:00
---

# Zap WithCaller 与 AddStacktrace 配置

### 🧠 `zap.WithCaller(true)` —— 调用者信息

**作用：** 控制每条日志是否自动附加调用者所处的文件名和行号（例如 `devicepuller/logger.go:35`）。

**注意：** 在 `zap.NewDevelopment()` 中，此配置默认开启。显式加不加这行配置，效果是一样的。

### 🧠 `zap.AddStacktrace()` —— 堆栈跟踪阈值

**作用：** 控制 Zap 从哪一个日志级别开始，自动在输出中附加完整的调用栈（Stacktrace）信息。

**默认机制：** 对于 `zap.NewDevelopment()` 产生的 Logger，默认触发阈值是 `WarnLevel`。即调用 `Warn`、`Error` 及以上级别时，都会自动附带一串方法调用堆栈。