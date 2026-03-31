# tools 模块

## 模块定位

`src/tools` 是 Claude Code 的模型能力实现目录。这里定义的不是用户 slash command，而是可以出现在模型工具列表里的正式工具。

规模统计：

- 文件数：184
- 行数：50863
- 主要子目录规模：
  - `BashTool`，18 文件，12414 行
  - `PowerShellTool`，14 文件，8961 行
  - `AgentTool`，20 文件，6784 行
  - `LSPTool`，6 文件，2006 行
  - `FileEditTool`，6 文件，1813 行
  - `SkillTool`，4 文件，1478 行
  - `shared`，2 文件，1370 行
  - `WebFetchTool`，5 文件，1132 行
  - `MCPTool`，4 文件，1087 行
  - `SendMessageTool`，4 文件，998 行

## 与根文件 `tools.ts` 的关系

- `src/tools.ts`：工具池装配与过滤
- `src/tools/*`：每种具体工具的实现

这意味着顶层 `tools.ts` 解决“有哪些工具”，而目录本身解决“工具怎么工作”。

## 模块职责

### 1. Shell 与执行工具

最重的两个工具族是：

- `BashTool`
- `PowerShellTool`

它们之所以大，不只是因为要运行命令，更因为要处理：

- 权限与 deny rule
- 命令安全分类
- 只读/写入判定
- 路径合法性
- 展示与进度反馈

尤其是 `bashPermissions.ts`、`bashSecurity.ts`、`readOnlyValidation.ts` 这些大文件，说明 shell 工具本身就是安全边界。

### 2. Agent 与团队工具

`AgentTool` 是整个目录的另一大核心。它负责：

- 启动 subagent
- worktree / remote 隔离
- background task
- 进度汇报
- 与 `LocalAgentTask` / `RemoteAgentTask` 对接

相关工具还包括：

- `SendMessageTool`
- `TeamCreateTool`
- `TeamDeleteTool`
- `TaskCreate/Get/List/Update/Stop/Output`

这说明工具系统已经不仅是“文件读写 + shell”，而是完整的 agent orchestration 面。

### 3. 文件与代码工具

这部分包括：

- `FileReadTool`
- `FileEditTool`
- `FileWriteTool`
- `NotebookEditTool`
- `GlobTool`
- `GrepTool`
- `LSPTool`

它们是 Claude Code 在代码仓库中行动的基础能力集。

### 4. MCP 与资源工具

包括：

- `MCPTool`
- `McpAuthTool`
- `ListMcpResourcesTool`
- `ReadMcpResourceTool`

说明 MCP 在工具层里有正式位置，而不是 service 层内部的隐藏机制。

### 5. 模式/工作流工具

包括：

- `EnterPlanModeTool`
- `ExitPlanModeTool`
- `EnterWorktreeTool`
- `ExitWorktreeTool`
- `BriefTool`
- `ConfigTool`
- `SkillTool`
- `ToolSearchTool`
- `TodoWriteTool`

这些工具直接改变代理的工作模式或上下文。

### 6. 网络与外部工具

包括：

- `WebFetchTool`
- `WebSearchTool`
- `RemoteTriggerTool`
- `ScheduleCronTool`

进一步扩大了 agent 的能力边界。

## 一个核心设计：工具是安全边界

在 Claude Code 里，工具不是简单函数调用。它们同时牵扯：

- 输入 schema
- user-facing name / description
- permission evaluation
- concurrency safety
- tool result storage
- context modification

这也是为什么 shell 工具和 agent 工具体量会非常大。

## AgentTool 特别说明

`AgentTool` 几乎值得当成一个单独子系统看待。

从代码可见，它支持：

- `subagent_type`
- model override
- background launch
- named agent
- team name
- permission mode
- `isolation: worktree | remote`
- `cwd` override

这已经不是简单“spawn child task”，而是一套 agent delegation 平台。

## 与其他模块的关系

- `Tool.ts`：工具契约定义
- `services/tools/*`：工具执行编排
- `hooks/useCanUseTool.tsx`：工具权限入口
- `tasks/*`：后台工具会落成 task
- `services/mcp/*`：MCP 工具接入
- `utils/permissions/*`：deny rule 与审批语义

## 工程观察

工具目录 5 万多行，说明 Claude Code 的核心价值很大程度上来自“模型到底能安全、稳定地调动多少能力”。

## 维护风险

- Shell 工具与 AgentTool 是最敏感的两个区域，既影响安全又影响能力。
- `tools/*` 和 `utils/permissions/*`、`services/tools/*` 的耦合天然很深，改动需跨层验证。
- 继续增加工具种类时，必须保持工具提示、schema、权限和 UI 表现的一致性。
