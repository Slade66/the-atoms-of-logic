---
created: 2026-03-19T22:44:14+08:00
---

# MCP 协议中的 LLM 起什么作用？

LLM 负责“决定”，程序负责“执行”

LLM 不直接调用服务，只是生成一个“标准化的调用意图”，“决定调用什么”，具体的调用由 MCP Client 执行。

LLM 擅长的是：
- 理解你的自然语言意图
- 决定该做什么，选工具
- 组织参数
比如把「issue，标题是 xxx，内容是 yyy 变成一种结构化表示。」变成一种结构化表示。

LLM 根据用户输入的自然语言，和可调用的工具列表，产出类似这种结构化结果返回：
```json
{
  "method": "tools/call",
  "params": {
    "name": "create_issue",
    "arguments": {
      "owner": "octo-org",
      "repo": "demo-repo",
      "title": "Bug: login fails",
    }
  }
}
```

这个东西不是 GitHub API 请求本身。而是 MCP 协议的消息，MCP Client 会把这个消息发给 MCP Server，后者要把这个消息转成真正的 API 调用。

