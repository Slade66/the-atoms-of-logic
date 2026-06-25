---
created: 2026-03-19T22:57:28+08:00
---

# MCP 协议的工作全流程

**简化的调用流程：**
```
LLM -> 生成“调用哪个工具、传什么参数”
MCP Client -> 把这个调用交给 MCP Server
MCP Server -> 真正请求 GitHub API
```

**完整的MCP工作流程：**
```text
[你]
  |
  v
[VSCode]
  |
  | (1) 用户输入自然语言、工具列表、上下文 -> 发送给 LLM
  v
[OpenAI API (LLM)]
  |
  | (2) 返回 tool call（MCP 格式）
  v
[VSCode (MCP Client)]
  |
  | (3) 调用 MCP Server
  v
[MCP Server (GitHub Adapter)]
  |
  | (4) 调用 GitHub API（把 MCP 格式 → 转成 GitHub API）
  v
[GitHub Server]
  |
  | (5) 返回结果
  v
[MCP Server]
  |
  | (6) 返回 MCP 响应
  v
[VSCode]
  |
  | (7) 发送结果给 LLM / 展示给用户
  v
[你]
```

