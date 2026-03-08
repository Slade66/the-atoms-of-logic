---
description: 常用强类型字段构造函数一览，含基础类型、错误处理、时间相关与反射兜底方案。
date: 2026-02-27T15:51:38+08:00
---

# Zap 结构化字段 Field 构造

### 🧠 核心作用

`zap.Field` 系列构造函数将变量转换为日志系统可识别的强类型字段，避免使用反射，在编译期即确定类型，实现零内存分配的极致性能。

## 常用构造函数速查

### 🧠 基础类型

```go
zap.String("key", "value")
zap.Int("key", 123)
zap.Bool("key", true)
zap.Float64("key", 3.14)
```

### 🧠 错误处理

**推荐用法：** `zap.Error(err)` 会自动提取错误消息并以 `"error"` 为键记录，无需手动指定键名。

```go
zap.Error(err) // 输出: "error": "库存不足"
```

### 🧠 时间相关

```go
zap.Duration("cost", 350*time.Millisecond) // 输出: "cost": "0.35s"
zap.Time("now", time.Now())
```

### 🧠 兜底方案：`zap.Any()`

**适用场景：** 当类型不确定时使用。底层通过反射处理任意类型，性能略低于强类型构造函数，仅在无法预知类型时作为兜底。

```go
zap.Any("raw_data", map[string]int{"a": 1})
```

## 完整使用示例

```go
logger.Info("用户下单失败",
    zap.String("order_id", "ORD-123456"),
    zap.Int("amount", 998),
    zap.Error(errors.New("库存不足")),
    zap.Duration("latency", 350*time.Millisecond),
    zap.Any("raw_data", map[string]int{"a": 1}),
)
// 输出 (JSON):
// {
//   "msg": "用户下单失败",
//   "order_id": "ORD-123456",
//   "amount": 998,
//   "error": "库存不足",
//   "latency": "0.35s",
//   "raw_data": {"a":1}
// }
```