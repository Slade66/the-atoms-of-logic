---
description: 解决每行日志重复添加公共字段的痛点，通过衍生 Logger 自动携带上下文信息。
date: 2026-02-27T15:51:38+08:00
---

# Zap With 日志上下文绑定

### 🧠 痛点：公共字段重复书写

在处理一次 HTTP 请求时，每一行日志都必须手动重复添加 `request_id` 等公共字段，既繁琐又容易遗漏。

❌ **灾难做法：**
```go
logger.Info("开始处理", zap.String("req_id", "req-123"))
logger.Info("参数校验", zap.String("req_id", "req-123"))
logger.Warn("库存不足", zap.String("req_id", "req-123"))
```

### 🧠 解决方案：`logger.With()` 创建衍生 Logger

使用 `logger.With(...)` 创建一个绑定了公共字段的衍生 Logger。后续通过该衍生 Logger 打印的所有日志，都会自动携带预设字段，无需重复书写。

✅ **优雅做法：**
```go
// 1. 创建衍生 Logger：自动携带 "req_id": "req-123"
reqLogger := logger.With(zap.String("req_id", "req-123"))

// 2. 后续直接打印：自动包含上述字段
reqLogger.Info("开始处理") // -> {"msg":"开始处理", "req_id":"req-123"}
reqLogger.Info("参数校验") // -> {"msg":"参数校验", "req_id":"req-123"}
reqLogger.Warn("库存不足") // -> {"msg":"库存不足", "req_id":"req-123"}
```