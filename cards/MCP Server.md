---
created: 2026-03-19T21:56:03+08:00
---

# MCP Server

MCP Server 是一个运行着的服务，它是对现有 API 的适配。

**MCP Server 的作用：**
1. 把 GitHub API → 变成 MCP 工具
2. 提供工具发现功能。
3. 把 MCP 工具调用，翻译成 API 调用。
    ```
    name = "create_issue"
    arguments = { owner, repo, title, body }

            ⬇️
            
    POST https://api.github.com/repos/{owner}/{repo}/issues
    Body = { title, body }
    ```

**为什么要把 MCP 消息翻译成 API？**

业务系统接受的是私有格式的调用，根本不认识 MCP 协议的消息。

协议翻译：MCP Server 负责把标准化的 MCP 工具调用（MCP 协议消息），翻译成具体服务的 API 调用。返回结果再转回 MCP 格式。

类比适配器模式：
```
统一接口（MCP）
        ↓
适配层（MCP Server）
        ↓
具体实现（GitHub / MySQL / Redis）
```

### MCP Server 运行在哪？

MCP Server 可以运行在你本地，也可以运行在远端服务商那边。这取决于它用的 transport（传输方式）。

**本地运行的情况：**
如果一个 MCP Server 使用 stdio transport，通常就是：
Client 在你的机器上把它当子进程启动
双方通过 stdin/stdout 交换 JSON-RPC 消息

**远端运行的情况：**
如果一个 MCP Server 使用 Streamable HTTP，那它就可以部署在远端平台上，通过 HTTP 与 Client 通信。

