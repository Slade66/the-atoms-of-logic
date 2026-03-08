---
title: Zap 三种开箱即用的 Logger 预设
description: 对比 Example / Development / Production 三种预设的适用场景、编码格式与配置差异。
date: 2026-02-27T15:51:38+08:00
tags: [Go, Zap]
---

### 🧠 `zap.NewExample()` —— 极简模式

**适用场景：** 仅用于编写文档示例或单元测试，不建议在实际项目中使用。输出只保留最基础的日志信息，去掉了时间戳、调用方等所有"噪音"。

```json
{"level":"debug","msg":"Something happened"}
```

### 🧠 `zap.NewDevelopment()` —— 开发模式

**适用场景：** 本地开发与调试。输出像 `fmt.Println` 一样直观，开启 `Debug` 级别，使用彩色纯文本格式并自动记录堆栈信息，方便肉眼快速定位错误。

```text
2025-11-09T14:03:21.123+0800  DEBUG  main.go:10  Something happened
```

### 🧠 `zap.NewProduction()` —— 生产模式

**适用场景：** 线上部署。输出为机器友好的 JSON 格式，开启 `Info` 级别（过滤 Debug 噪音），并启用采样器防止高频错误日志打爆磁盘，便于 ELK / Splunk 解析。

```json
{"level":"info","ts":1731138201.123,"caller":"main.go:10","msg":"Something happened"}
```

### 🧠 Development vs Production 对比

| 维度 | 🛠️ Development | 🚀 Production |
| :--- | :--- | :--- |
| **设计目标** | 人眼友好，方便本地排错 | 机器友好，JSON 结构化 |
| **编码格式** | Console Encoder（纯文本，Level 带颜色） | JSON Encoder |
| **最低级别** | `Debug`（调试信息全量输出） | `Info`（忽略 Debug） |
| **时间字段** | ISO8601（`2024-11-09T14:03:21.123+0800`） | Epoch 浮点数（`1731138201.123`） |
| **堆栈跟踪** | `Warn` 及以上自动打印 | `Error` 及以上自动打印 |
| **日志采样** | 禁用（每条都打印） | 启用（同一日志每秒前 100 条保留，其后每 100 条保留 1 条） |
