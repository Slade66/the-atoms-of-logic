---
created: 2026-03-19T23:05:07+08:00
---

# MCP Host 和 Client 的关系

Host = 应用本体（VSCode）
Client = Host 里面负责“说 MCP 协议”的那一小部分

Go 视角类比：
```go
type VSCode struct {
    MCPClient *Client
}
```

VSCode（Host） = 整个应用
MCP Client = 其中一个模块

很多人以为：
Host 和 Client 是两个服务 ❌
或两个进程 ❌
👉 实际上：
Client 是 Host 的一部分（一个模块）
