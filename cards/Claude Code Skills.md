---
description: "Skills 是一种语义触发的能力包，解决通用模型的认知稀释问题，把经验封装为可复用的工程资产"
date: 2026-02-28T15:13:36+08:00
---

# Claude Code Skills

### 🧠 Skills 要解决的根本问题

Skills 要解决三个具体问题：**Prompt 不可复用**（每次对话都要重新说明）、**经验无法沉淀**（个人技巧无法传承）、**团队能力难以统一**（规范散落在人脑和文档里）——把个人技巧变成可组合的工程资产。

**为什么 AI 代码助手必然需要 Skills？**
在真实的工程团队里，极少有人能把十几页代码风格指南、各种组件约定全背下来，人类工程师往往会随着习惯偏离规范，但**机器会严格遵守**。通过向 AI 注入 Skills，就能确保生成的代码从一开始就规范且高质量。

**更深远的意义在于，它促成了人类工作模式的剧变（向“CEO”转变）**：
原本我们需要亲自去执行的开发规范与流程，现在我们只需将其提炼成 SOP 即可。人类成为“CEO”，AI 充当执行特定 Skills 的虚拟员工，人类的职责从“亲自写代码”变成了“下发标准化任务并执行检查验收”。

一种朴素的做法是：把所有规范写进 `CLAUDE.md`，让模型每次对话都读取。短期可行，但当知识规模扩大到几十页、上百页时，问题就出现了——每次对话都在为"可能用不到的知识"支付上下文成本。这不仅消耗 tokens，更重要的是，它会稀释模型的注意力。真正需要用到的规则，反而淹没在冗余信息里。

Skills 的答案是**按需加载的认知架构**：与其把所有知识常驻在上下文中，不如把它们封装成可独立触发的能力单元。当模型判断当前任务涉及某个特定领域时，再加载对应的知识与操作流程。

### 🧠 Skills 在 Claude Code 体系中的定位

| 组件 | 回答的问题 |
|---|---|
| **Tools** | 能做什么（行动原语）|
| **SubAgents** | 谁来做（执行分工）|
| **Hooks** | 什么时候检查（流程规则）|
| **Skills** | **怎么做，以及何时做**（认知结构）|

Skills 解决的本质是**认知问题**：在有限的上下文窗口中，让 Agent 在正确的时刻拥有正确的领域知识。

### 🧠 核心定义：Skills 是什么

**Skills 是一种可被语义触发的能力包（Prompt）**，它包含领域知识、执行步骤、输出规范与约束条件，并在需要时渐进式加载到上下文中。

换成企业视角来理解：Tools 是员工的操作工具，SubAgents 是岗位分工，Hooks 是质检流程，`CLAUDE.md` 是企业文化与通用规章。而 **Skills，正是企业的 SOP 体系**——在具体任务发生时按需查阅，并按照步骤执行，输出标准化结果。当 AI 执行代码审查时，它不会即兴发挥，而是参考《代码审查 SOP》，按步骤检查并输出符合模板的报告。

这意味着，当组织的"做事方式"被封装为可语义调用的能力单元，经验就不再依附于老员工的记忆，也不再散落在文档系统中——它变成了模型可以理解、选择和继承的结构。这构成了**企业 AI Agent 化转型**的基石：把企业业务拆解成一个个技能智能体，未来的企业可能就是“少数极客 + 一堆执行标准化技能的虚拟员工”。

### 🧠 语义触发：Skills 如何被激活

传统工具需要用户手动触发——你输入 `/review`，它就审查代码。但 Skills 不同，你只需要说"帮我看看这段代码有没有安全问题"，AI 就能自动判断任务类型并激活对应的 Skill。**这种"语义触发"的设计，让 AI 从执行命令的工具，升级为理解意图的工作伙伴。**

触发机制基于语义推理而非精确匹配：Claude 读取所有 Skills 的 `description` 字段，通过语义理解判断当前对话是否匹配某个 Skill。因此 `description` 是最核心的字段——它包含两部分信息：这个 Skill 做什么，以及什么情况下使用它。

Skills 支持双向触发：用户知道自己要什么时，可以用 `/skill-name` 直接调用；用户只是描述需求时，Claude 自动判断并激活对应 Skill。两种场景都能被无缝覆盖。

### 🧠 如何创建一个 Skill

**1. 创建目录**

```bash
mkdir -p ~/.claude/skills/<skill-name>      # 个人级
mkdir -p .claude/skills/<skill-name>        # 项目级
```

每个 Skill 本质上是一个目录，`SKILL.md` 是必须存在的入口文件，其余文件按需添加：

```text
my-skill/
├── SKILL.md           # 主指令文件（必需）
├── template.md        # 供 Claude 填写的模板
├── examples/
│   └── sample.md      # 示例输出，展示期望格式
└── scripts/
    └── validate.sh    # Claude 可执行的脚本
```

其他文件都是可选的，用来构建更强大的能力（模板文件约束输出，示例文件展示期望结果，脚本文件用于实际执行）。

这里有一个**保持 `SKILL.md` 极度精简的最佳实践**：冗长的大型参考文档或 API 规范不需要每次都加载。你只需要在 `SKILL.md` 中通过 Markdown 链接直接引用（例如：`完整的 API 细节请参阅 [reference.md](reference.md)`），Claude 就会在真正需要查阅时才读取它。这是"渐进式披露架构"的最佳体现：只在用到时才加载对应的知识。

**2. 编写 SKILL.md**

每个 Skill 必须有一个 `SKILL.md`，包含两部分：YAML frontmatter（用 `---` 包裹）+ Markdown 指令内容。

```markdown
---
name: explain-code
description: 用图示和类比解释代码。当解释代码如何工作、讲解代码库或用户问"这怎么工作？"时使用。
---

当解释代码时，始终包括：

1. 先用一个类比：把代码比作日常生活中的事物
2. 画一个图：用 ASCII 图展示流程或结构
3. 逐步讲解：按步骤说明代码发生了什么
4. 提醒一个坑：常见错误或误解是什么？

保持对话式风格。复杂概念可以使用多个类比。
```

**Frontmatter 字段说明：**

- **`name`**：Skill 名称，默认使用目录名，也是斜杠命令名（`/name`）。
- **`description`**：**最重要的字段**，决定 Skill 何时被触发。应包含两部分：这个 Skill 做什么 + 什么情况下使用它。Claude 通过读取此字段进行语义推理来判断是否激活。
- **`argument-hint`**：调用时显示给用户的参数提示，如 `[commit message]`。
- **`allowed-tools`**：白名单，限制 Skill 执行期间可用的工具，如 `Read, Grep, Glob`。
- **`disable-model-invocation: true`**：禁止 Claude 自动触发此 Skill，只允许用户手动通过 `/skill-name` 调用。自动执行有风险时使用。
- **`user-invocable`**：是否在斜杠命令菜单中显示此 Skill。
- **`model`**：为此 Skill 指定特定的模型。
- **`context: fork`**：在隔离的子代理中执行此 Skill，防止中间输出污染主对话上下文。
- **`agent`**：指定子代理类型（`Explore`、`Plan`、`general-purpose` 或自定义），配合 `context: fork` 使用。
- **`hooks`**：生命周期钩子，在 Skill 完成后自动触发质量验证或合规检查。

**3. 测试**

```bash
# 方式一：自然语言触发（Claude 自动判断）
How does this code work?

# 方式二：斜杠命令手动调用
/explain-code src/auth/login.ts
```

**4. 选择存放位置**（决定生效范围）

| 位置 | 路径 | 生效范围 |
|---|---|---|
| 个人 | `~/.claude/skills/<skill-name>/SKILL.md` | 你的所有项目 |
| 项目 | `.claude/skills/<skill-name>/SKILL.md` | 仅当前项目 |

### 🧠 参数传递

Skill 内容中可用的参数变量：

| 变量 | 含义 |
|---|---|
| `$ARGUMENTS` | 全部参数（完整字符串）|
| `$ARGUMENTS[0]` | 第一个参数 |
| `$0` | 同上，简写形式 |
| `$1`, `$2`... | 按位置接收多个参数 |

通过 `/skill-name args` 调用 Skill 时，参数会注入到 Skill 内容中：

```yaml
# 单参数
---
description: Quick git commit
---
Create a git commit with message: $ARGUMENTS
```

```yaml
# 多参数
---
description: Create a pull request
---
Title: $1
Description: $2
```

如果 Skill 内容中没有定义 `$ARGUMENTS`，但调用时传入了参数，Claude Code 会自动在内容末尾追加 `ARGUMENTS: <用户输入>`，确保参数不丢失。

### 🧠 与子代理配合：隔离执行

在 frontmatter 中添加 `context: fork`，Skills 可以在隔离的子代理中执行：

```yaml
---
name: deep-research
description: 深入研究一个主题
context: fork
agent: Explore
---
```

这样做的价值在于：大量中间输出（几百行搜索结果、测试日志）不会回流污染主对话上下文。技术上有两种协作方式：

- **Skill 驱动子代理**（在 Skill 里写任务 + `context: fork`）
- **子代理使用 Skills**（在子代理配置中声明 Skills 作为预加载知识）

两种方式的技术构成不同：

|  | Skill 驱动子代理 | 子代理使用 Skills |
|---|---|---|
| **系统提示词** | 来自 `agent` 类型（如 Explore、Plan）| 来自子代理 markdown 主体 |
| **任务来源** | `SKILL.md` 的内容 | Claude 的委派消息 |
| **同时加载** | `CLAUDE.md` | 预加载的 Skills + `CLAUDE.md` |

注意：`context: fork` 只适用于包含明确任务指令的 Skill。如果 Skill 只是指导性内容而非可执行任务，子代理会因为没有实质任务而空跑一圈。

`agent` 字段指定子代理类型：内置类型有 `Explore`、`Plan`、`general-purpose`，或 `.claude/agents/` 下的自定义子代理；省略时默认使用 `general-purpose`。

> 💡 **一句话生动理解二者的区别**：
> 普通 Skill 像是递给 Claude 一张纸质的**“提示卡片”**；而加上 `context: fork` 后，它就变成了**“开一个新房间，让专门的助手在里面独立完成这件事，最后只把干干净净的结果拿回来交给你”**。

### 🧠 发展趋势：从内置命令到通用生态

在早期版本中，斜杠命令 `/Commands` 和 Skills 是两个独立组件。但在新版 Claude Code 中，**Commands 已合并到 Skills，成为 Skills 的子集**。Skills 的目录结构支持辅助文件（模板、示例、脚本等），因此具有远超传统命令的表达能力。

更宏观来看：**Skills 概念目前已经开始脱离 Claude Code 本身，形成了一种 Agent 的通用技能生态标准。** 从特定工具专属走向全平台的通用设计模式，这是让大模型具备专业化能力的重要途径。

> 💡 **一句话区分 `CLAUDE.md` 和 Skills：**
> **`CLAUDE.md`**：放"Claude 每次对话都**必须**知道的核心规则与文化"。
> **`Skills`**：放"仅特定场景下才会用到的**详细 SOP 指令**和知识盲区"。