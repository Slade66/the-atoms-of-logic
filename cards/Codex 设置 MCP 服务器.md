---
created: 2026-03-19T11:01:15+08:00
---

# Codex 设置 MCP 服务器

1. 面板右上角 Settings
2. Codex settings
3. Open config.toml
4. 在文件中加上：

    ```toml
    [mcp_servers.todo-mcp]
    command = 'A:/Users/liyuz/Desktop/mcp-todo/bin/todo-mcp.exe'
    cwd = 'A:/Users/liyuz/Desktop/mcp-todo'
    enabled = true
    ```
