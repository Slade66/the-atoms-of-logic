## VS Code MCP 配置文件样例

在项目根目录下的 `.vscode/mcp.json` 中配置自定义 MCP（Model Context Protocol）服务的典型样例如下：

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

## mcp.json 核心配置字段详解

针对样例配置，各个配置字段的具体含义如下：

- **`servers`**：声明所有 MCP 服务配置的容器对象，支持同时配置多个服务。
- **`test`**：定义该服务的标识 ID，在 VS Code of Copilot 或 MCP 管理界面中显示的即为此名称。
- **`type: "stdio"`**：指定通信传输协议为标准输入输出方式（`stdio`）。VS Code 会通过标准输入（stdin）向 MCP 服务发送指令请求，并通过其标准输出（stdout）接收解析结果。
- **`command: "npm"`**：指定启动 MCP 服务进程时执行的基准命令行程序。
- **`args: ["run", "start"]`**：指定追加给启动命令的参数列表。此例中，执行的完整命令为 `npm run start`，用以拉起项目 `package.json` 中的 `start` 脚本。
- **`dev`**：开发阶段的专用配置块，用于辅助实现修改自动重启和挂载调试器。
- **`dev.watch: "src/**/*.ts"`**：指定需要监听的 TypeScript 源码路径。当这些文件发生变化时，VS Code 会自动热重启该 MCP 服务进程。
- **`dev.debug.type: "node"`**：指定调试模式对应的运行时环境为 Node.js，方便开发者在代码中附加断点进行单步调试。

## MCP 服务的启动与热重载工作机制

根据上述配置，当启动 VS Code 或手动唤醒 MCP 服务时，其运行机制如下：

1. **服务初始化**：VS Code 读取 `.vscode/mcp.json` 配置，定位到 `test` 服务，拉起守护进程并执行 `npm run start`。
2. **长连接通信**：VS Code 与启动的服务进程建立持久化的 `stdio` 数据通道。
3. **源码变更热重启**：开发过程中，一旦监听到 `src` 目录下的 TypeScript 源码发生保存更改，服务进程会立即被销毁并重新初始化，无需手动介入。
4. **附加调试支持**：当需要排查 bug 时，可以通过 VS Code 调试面板启动 Node 调试工作流，直接挂载到该进程上执行代码分析。

## 常见补充配置项（cwd、env 与 envFile）

除上述核心字段外，常见的实用配置项还包括：

- **`cwd`**（Current Working Directory）：指定该 MCP 命令行程序启动时的执行路径（默认情况下为当前 VS Code 工作区的根目录）。
- **`env`**：允许以键值对的形式向 MCP 服务传递所需的环境变量（如 `API_KEY`、`SERVER_PORT` 等）。
- **`envFile`**：允许指定一个 `.env` 配置文件的路径，从中批量加载外部环境变量。

## VS Code 官方 MCP 相关参考文档

- **在 VS Code 里添加和管理 MCP 服务器**（`mcp.json` 放置路径、基本字段配置）：  
  https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- **MCP 开发指南**（`dev.watch`、`dev.debug` 接口示例说明）：  
  https://code.visualstudio.com/api/extension-guides/ai/mcp
- **VS Code API 详解**（`command`、`args`、`env`、`cwd` 等核心属性的官方接口定义）：  
  https://code.visualstudio.com/api/references/vscode-api
