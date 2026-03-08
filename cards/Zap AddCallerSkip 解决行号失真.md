---
description: 当 Logger 被二次封装后 Caller 行号指向包装器而非业务代码，用 AddCallerSkip(1) 跳过中间栈帧修正。
date: 2026-02-27T15:51:38+08:00
---

# Zap AddCallerSkip 解决行号失真

### 🧠 痛点：包装器导致 Caller 行号失真

如果你在 `logger.go` 中用接口把 Zap 包装了一层（例如自定义一个全局方法 `log.Infof`，内部再调用 Zap），那么 Zap 默认探测到的 Caller 永远只会指向你的包装器文件的行号，而不是真正触发日志的业务代码行号。

❌ **失真表现：**
```text
logger.go:35  用户登录失败    ← 永远指向包装器，毫无定位价值
```

### 🧠 解决方案：`zap.AddCallerSkip(1)`

在初始化 Zap 时传入 `zap.AddCallerSkip(1)`。

```go
logger, _ := zap.NewDevelopment(zap.AddCallerSkip(1))
```

### 🧠 底层原理：跳过中间栈帧

`AddCallerSkip(1)` 告诉 Zap 在探测调用栈寻找真正调用者时，"再多往上跳 1 层栈帧"，忽略掉中间的包装函数，直接抓出躲在幕后真正调用 `log.Infof` 的那个业务代码的文件名和行号。

✅ **修正后表现：**
```text
controller.go:72  用户登录失败    ← 精准指向业务代码
```