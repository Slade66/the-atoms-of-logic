---
description: 用 Markdown 文件定义可重复执行的任务序列，并通过斜杠命令一键调用。
date: 2026-03-02T17:39:28+08:00
---

# Google Antigravity Workflows

### 🧠 Workflow 是什么

**本质定义：** Workflow 是以 `.md` 文件形式存储的、预定义好的有序任务步骤序列。其核心价值在于将重复性操作标准化固化，确保 Agent 严格遵循既定轨迹执行，从而有效避免人工操作的步骤遗漏和 AI 随性发挥。

**字符限制：** Workflow 文件单文件字符数上限为 12,000 字符。

```markdown
1. 拉取最新代码：执行 `git pull origin main`
2. 安装依赖：执行 `npm install`
3. 构建产物：执行 `npm run build`
4. 上传到服务器并重启服务
```

### 🧠 Workflow vs Rules

- **Rules（规则）：** 提供持久性的上下文背景，作用在提示词（Prompt）层面，是“通用的行为准则”。

- **Workflows（工作流）：** 提供结构化的步骤序列，作用在执行流程层面，是“具体的行动剧本”。

### 🧠 如何调用 Workflow

**调用语法：** 在 Agent 对话框中输入 `/工作流名称` 即可触发。例如：`/deploy-prod`、`/review-pr`。

**嵌套调用：** Workflow 可以互相调用。例如 `/workflow-1` 内部的步骤可以写"调用 `/workflow-2`"和"调用 `/workflow-3`"，实现模块化的流程组合。

### 🧠 创建 Workflow

1. 点击编辑器 Agent 面板顶部 `...` 下拉菜单，打开 Customizations（自定义）面板。
2. 进入 Workflows 面板。
3. 选择创建范围：
   - `+ Global`：创建全局 Workflow，在所有工作区均可访问。
   - `+ Workspace`：创建仅限当前工作区的 Workflow。

### 🧠 让 Agent 自动生成 Workflow

**自动复盘对话生成 Workflow**：当你已经手动带着 Agent 走完了一套完整流程后，可以直接让它复盘并固化为 Workflow 文件——它会根据对话历史自动提取步骤并生成规范的 `.md` 文件，无需手动编写。