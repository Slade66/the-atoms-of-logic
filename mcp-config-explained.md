# `.vscode/mcp.json` 配置说明（大白话）

你现在的配置：

```json
{
  "servers": {
    "test": {
      "type": "stdio",
      "command": "npm",
      "args": ["run", "start"],
      "dev": {
        "watch": "src/**/*.ts",
        "debug": {
          "type": "node"
        }
      }
    }
  }
}
```

## 每个字段是干什么的

- `servers`
作用：放所有 MCP 服务配置的地方。可以配多个服务。

- `test`
作用：这是这个服务的名字（ID）。你在 VS Code 里看到的就是这个名字。

- `type: "stdio"`
作用：告诉 VS Code 这个 MCP 服务通过“标准输入/标准输出”通信。  
大白话：VS Code 像和命令行程序聊天一样，往它 stdin 发请求、从 stdout 收响应。

- `command: "npm"`
作用：启动服务时先执行的命令。

- `args: ["run", "start"]`
作用：给 `command` 追加参数。  
大白话：组合起来就是执行 `npm run start`，也就是用你 `package.json` 里的 `start` 脚本启动 MCP 服务。

- `dev`
作用：开发模式专用配置。主要帮你“改代码自动重启”和“直接调试”。

- `dev.watch: "src/**/*.ts"`
作用：监听 `src` 下所有 `.ts` 文件变化。  
大白话：你改了这些文件后，VS Code 会重启这个 MCP 服务，省得你手动重启。

- `dev.debug.type: "node"`
作用：告诉 VS Code 用 Node.js 调试器去调这个服务。  
大白话：你可以像平时调 Node 项目一样打断点看变量。

## 你这份配置实际会发生什么

1. VS Code 启动名为 `test` 的 MCP 服务。
2. 它运行 `npm run start`。
3. 通过 stdio 和服务通信。
4. 你改了 `src/**/*.ts` 里文件，服务自动重启。
5. 需要调试时，使用 Node 调试器附加/启动调试流程。

## 常见补充项（你现在没写，但经常会用）

- `cwd`：指定命令在哪个目录执行（不写通常就是工作区目录）。
- `env`：给服务传环境变量（比如 API Key）。
- `envFile`：从 `.env` 文件加载变量。

## 官方文档来源

- 在 VS Code 里添加和管理 MCP 服务器（`mcp.json` 位置、基本配置）  
https://code.visualstudio.com/docs/copilot/customization/mcp-servers

- MCP 开发指南（`dev.watch`、`dev.debug` 示例）  
https://code.visualstudio.com/api/extension-guides/ai/mcp

- VS Code API（`command`、`args`、`env`、`cwd` 等字段语义）  
https://code.visualstudio.com/api/references/vscode-api
