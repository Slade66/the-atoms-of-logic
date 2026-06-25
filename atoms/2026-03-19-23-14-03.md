---
created: 2026-03-19T23:14:03+08:00
---

# MCP 的标准传输方式

MCP 当前支持三种传输方式：

stdio：本地两个进程之间通信，通常是 client 把 server 当子进程启动，通过标准输入输出传 JSON-RPC。没有网络开销。

Streamable HTTP：通过 HTTP 通信，客户端发 POST，服务端返回 JSON 或 SSE 流，适合远端部署。

SSE（旧 transport）：已废弃，被 Streamable HTTP 取代，只是兼容旧实现时还会看到。

