---
title: Zap 日志级别速查与使用规范
description: 从 Debug 到 Fatal 的全套分级语义、副作用说明，以及业务代码中的级别选择规范。
date: 2026-02-27T15:51:38+08:00
tags: [Go, Zap]
---

## 常规记录（无副作用）

### 🧠 `Debug` —— 调试细节

**触发条件：** 仅在 Development 模式或日志级别设为 `Debug` 及以下时打印。用于输出开发阶段的变量状态、流程追踪等细粒度信息。

```go
logger.Debug("调试", zap.String("var", "abc"))
```

### 🧠 `Info` —— 业务里程碑

**使用场景：** 针对正常的业务流程节点和里程碑事件打印，如"用户登录成功"、"订单创建完成"。

```go
logger.Info("信息", zap.Int("status", 200))
```

### 🧠 `Warn` —— 潜在隐患

**使用场景：** 针对潜在问题、隐患或需要运维特别关注的情况打印，如慢请求、重试操作、资源即将耗尽。

```go
logger.Warn("警告", zap.Duration("cost", 3*time.Second))
```

### 🧠 `Error` —— 功能受损

**使用场景：** 程序运行出现明确错误、调用失败，但程序仍可继续运行。

```go
logger.Error("错误", zap.Error(err))
```

## 特殊行为（有副作用）

### 🧠 `DPanic` —— 开发模式专属 panic

**行为差异：** 在 Development 模式下触发 `panic`，在 Production 模式下仅记录为 Error 级别日志。设计意图是让开发阶段的严重错误立即暴露，而不影响线上稳定性。

```go
logger.DPanic("调试恐慌")
```

### 🧠 `Panic` —— 记录后 panic

**副作用：** 记录日志后，立即触发 `panic()`。

```go
logger.Panic("恐慌")
```

### 🧠 `Fatal` —— 记录后强杀进程

**副作用：** 记录日志后，立即调用 `os.Exit(1)` 终止程序。所有 `defer` 语句都不会被执行，因此在业务代码中应避免使用。

```go
logger.Fatal("致命")
```
