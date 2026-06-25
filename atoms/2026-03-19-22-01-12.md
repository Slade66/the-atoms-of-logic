---
created: 2026-03-19T22:01:12+08:00
---

# MCP Client

MCP Host 应用里会有一个通用的 MCP client 实现，但它会为每个 MCP server 实例化一个独立 client。

[[MCP 协议中的 LLM 起什么作用？]]

### MCP Client 和 MCP Server 之间传的是什么？

传递的是 MCP 协议消息（基于 JSON-RPC 结构）

```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "method": "tools/call",
  "params": {
    "name": "create_issue",
    "arguments": {
      "title": "bug",
      "body": "xxx"
    }
  }
}
```


