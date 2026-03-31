# 对话与执行运行时

## 1. 文档范围

这份文档关注“Claude Code 如何完成一次 agentic turn”。重点覆盖这些根文件：

- `src/QueryEngine.ts`
- `src/query.ts`
- `src/Tool.ts`
- `src/tools.ts`
- `src/commands.ts`
- `src/Task.ts`
- `src/tasks.ts`
- `src/context.ts`
- `src/history.ts`
- `src/cost-tracker.ts`
- `src/costHook.ts`
- `src/ink.ts`

这些文件共同定义了 Claude Code 的核心运行模型。

## 2. 运行时总链路

一次完整 turn 可以概括为：

1. 用户输入或 SDK 输入形成消息
2. `QueryEngine` 整理状态和系统上下文
3. `query()` 进入循环
4. `services/api/claude.ts` 发起采样
5. 模型返回普通消息或 tool_use
6. `runTools()` 执行工具
7. tool_result 回流消息序列
8. 必要时 compact / retry / continuation
9. 最终写回状态、usage、history、session storage

## 3. `src/QueryEngine.ts`

### 3.1 模块定位

`QueryEngine` 是 headless conversation runtime 的核心对象。它把“一次会话中跨多个 turn 持续存在的状态”封装起来。

文件规模：

- 文件数：1
- 行数：1295

### 3.2 主要状态

从代码可见，它至少持有：

- `mutableMessages`
- `abortController`
- `permissionDenials`
- `totalUsage`
- `readFileState`
- discovered skills
- loaded memory path tracking

这说明 `QueryEngine` 不是简单的 `ask()` 包装，而是会话级状态容器。

### 3.3 主要职责

- 保存会话消息历史
- 构建/扩展系统提示
- 包装 `canUseTool` 以跟踪权限拒绝
- 管理 read file cache
- 跟踪 usage、session persistence、structured output
- 调用 `query()` 驱动一轮轮执行

### 3.4 为什么它重要

它是 REPL 逻辑抽离出来的“可复用 agent runtime”。没有这个类，print/SDK/headless 模式会被迫复用 UI 驱动的实现。

## 4. `src/query.ts`

### 4.1 模块定位

`query.ts` 是 agent 采样循环本身。`QueryEngine` 更像 orchestrator，而 `query.ts` 是 turn machine。

文件规模：

- 文件数：1
- 行数：1729

### 4.2 它处理的核心问题

- 请求开始与流式事件产出
- thinking 规则与 message 合法性
- token budget / task budget
- auto compact / reactive compact / microcompact
- tool call 执行与结果注入
- stop hooks / failure hooks
- continuation 与恢复
- prompt 过长、max_output_tokens 等异常恢复路径

### 4.3 模块特征

这是标准意义上的“状态机文件”。它包含大量跨迭代状态：

- messages
- toolUseContext
- compact tracking
- recovery count
- pending tool summary
- stop hook 状态
- turn count

它不是简单串行流程，而是带恢复、压缩、预算和 hook 的 agent loop。

## 5. `src/Tool.ts`

### 5.1 模块定位

`Tool.ts` 定义工具系统的核心抽象，是模型能力层的基础契约。

文件规模：

- 文件数：1
- 行数：792

### 5.2 关键职责

- 定义 `Tool` / `Tools`
- 定义 `ToolUseContext`
- 提供 tool 查找与匹配函数
- 定义工具输入输出的共享语义

### 5.3 为什么要重视 `ToolUseContext`

这个类型很大，意味着工具可以访问很多系统能力：

- AppState
- 会话消息
- 通知
- 文件与缓存
- 任务系统
- MCP/插件上下文
- UI 回调

这既说明工具系统很强，也说明工具边界必须被权限体系严格保护。

## 6. `src/tools.ts`

### 6.1 模块定位

`tools.ts` 是模型可见工具池的装配器。

文件规模：

- 文件数：1
- 行数：389

### 6.2 主要职责

- 汇总所有内置工具
- 根据 feature gate 和环境变量动态决定工具是否存在
- 支持 tool preset
- 过滤被 deny rule 全局拒绝的工具
- 结合 MCP 工具形成最终工具池

### 6.3 一个关键设计点

工具过滤不是等模型调用时才做，而是会在“模型看到工具列表之前”先做一层预过滤。这个设计直接影响安全与模型行为质量。

## 7. `src/commands.ts`

### 7.1 模块定位

这是 slash command 系统的总装配文件，不是 Commander 子命令文件。

文件规模：

- 文件数：1
- 行数：754

### 7.2 主要职责

- 汇总内置 slash commands
- 加入 feature-gated commands
- 加入 skill dir commands
- 加入 bundled skills
- 加入 builtin plugin skill commands
- 加入 plugin commands
- 加入动态 skills
- 对命令做 enable/filter/remote mode 过滤

### 7.3 和 `tools.ts` 的区别

- `commands.ts` 面向用户
- `tools.ts` 面向模型

两者都能扩展，但承担的是两种完全不同的协议边界。

## 8. `src/Task.ts` 与 `src/tasks.ts`

### 8.1 `Task.ts`

这个文件定义任务系统的基础类型。

文件规模：

- 文件数：1
- 行数：125

核心内容：

- `TaskType`
- `TaskStatus`
- terminal 状态判断
- `TaskContext`
- `TaskStateBase`
- task id 生成规则

它把“后台执行单元”从工具调用中抽象了出来。

### 8.2 `tasks.ts`

这个文件则负责把具体 task 实现注册成统一列表。

文件规模：

- 文件数：1
- 行数：39

它采用与 `tools.ts` 类似的模式：

- 汇总内置任务类型
- 用 feature gate 按需加载 workflow / monitor task
- 提供 `getAllTasks()` 和 `getTaskByType()`

## 9. `src/context.ts`

### 9.1 模块定位

虽然名字很短，但这是系统 prompt 上下文的重要来源文件。

文件规模：

- 文件数：1
- 行数：189

### 9.2 主要职责

- 生成 git 状态快照
- 生成 system context
- 生成 user context
- 注入 CLAUDE.md、日期、cache breaker 等内容

### 9.3 价值

这个文件说明 Claude Code 的 prompt 上下文不是一坨字符串拼接，而是结构化来源拼装。

## 10. `src/history.ts`

### 10.1 模块定位

负责 prompt/history/pasted content 的持久化和读取，不只是简单 up-arrow 历史。

文件规模：

- 文件数：1
- 行数：464

### 10.2 关键职责

- 写入 JSONL 历史
- 维护当前 session 与当前 project 的历史视图
- 处理 pasted text / image 引用和外置存储
- 支持逆序读取和去重

### 10.3 为什么重要

在 Claude Code 里，历史不是“交互细节”，而是工作流连续性的一部分。

## 11. `src/cost-tracker.ts` 与 `src/costHook.ts`

### 11.1 `cost-tracker.ts`

文件规模：

- 文件数：1
- 行数：323

职责：

- 累计会话 cost、duration、token usage、代码修改行数
- 恢复/保存 session 级 cost 状态
- 格式化 model usage 与 total cost 输出

### 11.2 `costHook.ts`

文件很小，但它把 cost 相关逻辑以 hook 或触发点的方式接到运行时里。

这对 print/REPL 统计展示和 session resume 都很关键。

## 12. `src/ink.ts`

### 12.1 模块定位

这是项目对 Ink 的统一出口，而不是底层实现本身。

文件规模：

- 文件数：1
- 行数：85

### 12.2 职责

- 用 `ThemeProvider` 包一层 render/createRoot
- 统一导出项目级 Theme 组件和底层 Ink 能力

### 12.3 价值

它让全仓库的终端 UI 都自动享有统一主题能力，而不用在每个 render 点重复包裹 provider。

## 13. 这套运行时的三个核心设计

### 13.1 会话状态与单轮状态分离

- `QueryEngine` 管跨 turn 状态
- `query.ts` 管单轮迭代状态

这是一个健康的边界。

### 13.2 命令、工具、任务三分

- command：用户入口动作
- tool：模型调用能力
- task：后台执行单元

把这三层分开，是 Claude Code 能支持多 agent、后台任务、REPL 和 automation 共存的根本原因。

### 13.3 prompt/context 不是纯字符串，而是结构化生成

`context.ts`、`attachments.ts`、`messages.ts`、`services/api/claude.ts` 一起形成了一条结构化 prompt 生产链，这和很多简单 agent 项目差别很大。

## 14. 阅读建议

如果你想真正搞懂 agent turn，建议按这个顺序读代码：

1. `Tool.ts`
2. `tools.ts`
3. `QueryEngine.ts`
4. `query.ts`
5. `services/api/claude.ts`
6. `services/tools/*`
7. `Task.ts`
8. `tasks/*`

## 15. 小结

Claude Code 的核心运行时不是 UI，而是“QueryEngine + query loop + tool/task orchestration”。

只要这套心智模型建立起来，后面的 MCP、plugin、skill、bridge、remote 都会更容易理解，因为它们本质上都是在给这套 runtime 注入新的上下文、能力或 transport。
