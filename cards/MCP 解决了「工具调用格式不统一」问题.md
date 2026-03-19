---
created: 2026-03-19T21:12:56+08:00
---

# MCP 解决了工具调用格式不统一的问题

假设你在做一个 AI IDE，要让 LLM 可以调用不同工具
查代码（code_search）
查日志（log_search）
查数据库（db_query）

### 没有 MCP 时：工具调用的数据结构不一致

系统里存在多套工具调用格式。

如果没有 MCP，数据格式不是统一的，每个团队自己写自己的。

工具 A（代码搜索服务，Go 团队写的）：
```json
{
  "action": "code_search",
  "payload": {
    "keyword": "MCP"
  }
}
```

工具 B（日志系统，Java 团队写的）：
```json
{
  "tool": "log_search",
  "input": {
    "query": "error",
    "timeRange": "1h"
  }
}
```

工具 C（数据库服务，Python 团队写的）：
```json
{
  "name": "db_query",
  "args": {
    "sql": "SELECT * FROM users"
  }
}
```

问题：LLM 混用格式
LLM 很难稳定调用工具，规则一多，LLM 更容易“用错格式”，同一个意图，模型可能生成不同格式。
因为 LLM 可能混用了不同工具的调用方式。
每接一个新工具，都要学一套新格式

一次它输出：
```json
{
  "tool": "log_search",
  "input": {
    "query": "panic"
  }
}
```

另一次它输出：
```json
{
  "action": "log_search",
  "payload": {
    "keyword": "panic"
  }
}
```

### 有了 MCP：通过定义统一协议，让所有工具调用结构一致

把所有工具都用同一个格式：
```json
{
  "method": "tools/call",
  "params": {
    "name": "log_search",
    "arguments": {
      "query": "order-service error",
      "timeRange": "1h"
    }
  }
}
```

- 所有工具调用都是 "method": "tools/call"
- 都在 params.name 放工具名
- 都在 params.arguments 放参数

LLM 更容易学会调用
因为模型只需要学会一种调用方式，就能调用所有工具：
调工具 → method = "tools/call"
工具名 → params.name
参数 → params.arguments
LLM 只需要：选工具名填参数
这样调用成功率会高很多。

