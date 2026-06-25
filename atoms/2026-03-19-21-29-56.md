---
created: 2026-03-19T21:29:56+08:00
---

# MCP 解决了「能力发现」问题

LLM 怎么知道“有哪些工具可以用”？

### before

只能靠手写 prompt：
你可以使用以下工具：xxx、xxx、xxx

工具列表是“写死的”，导致：
新加一个工具 → 要改 prompt
删一个工具 → 也要改 prompt
很容易不同步

### after

系统可以提供：
```json
{
  "tools": [
    {
      "name": "get_weather",
      "description": "...",
      "inputSchema": { ... }
    },
    {
      "name": "search",
      "description": "...",
      "inputSchema": { ... }
    }
  ]
}
```

从：
“你得提前告诉 LLM 有哪些工具”

变成：
“系统可以动态告诉 LLM 有哪些工具”

MCP 让工具从“写死配置”变成“可被系统发现的能力集合”。工具不再写死在 prompt
