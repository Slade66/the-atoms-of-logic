---
created: 2026-03-22T13:53:00+08:00
---

# Tracing 链路追踪

### Tracing 有什么用？

Tracing（链路追踪）就是把“一次请求在整个系统所有服务里的每一步耗时都记录下来，并串成一条时间线”。

你可以看到每一段的耗时，哪一段慢。

没有 Tracing，你就只能猜是哪里慢。

### Trace：一次完整的请求

TraceID 贯穿全链路，串起所有服务日志。

### Span

一个步骤

一个 span（一个计时区间/步骤）

### OpenTelemetry（OTel）

埋点标准：用来在代码里埋点

sample_ratio: 1.0
采样率 100%，每个请求都上报。排障阶段建议 1.0，线上常改成更低。

### Jaeger

后端系统

存储 trace
可视化分析工具。
查询分析

#### 安装

```bash
docker run -d --name jaeger -p 16686:16686 -p 4317:4317 jaegertracing/all-in-one:latest
```

打开 UI 👉 http://localhost:16686

**为什么只需要这两个端口？**
16686 👉 看页面
4317 👉 接收 OTLP gRPC（你 tracing 用的）

## Go 项目接入 tracing

### otlptracegrpc

`go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc`

作用：把 trace 数据发送出去（通过 gRPC），负责把你程序里的 trace 数据发到后端系统。

OTLP = OpenTelemetry Protocol（标准协议）

gRPC = 传输方式

