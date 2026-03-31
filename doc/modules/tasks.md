# tasks 模块

## 模块定位

`src/tasks` 是 Claude Code 的后台执行单元实现目录。它把“一个长时间运行的工作项”抽象成 task，而不是把所有执行都塞进 tool 调用栈里。

规模统计：

- 文件数：12
- 行数：3290
- 子目录：
  - `RemoteAgentTask`，1 文件，856 行
  - `LocalAgentTask`，1 文件，683 行
  - `LocalShellTask`，3 文件，640 行
  - `InProcessTeammateTask`，2 文件，247 行
  - `DreamTask`，1 文件，157 行
- 代表文件：
  - `tasks/RemoteAgentTask/RemoteAgentTask.tsx`，856 行
  - `tasks/LocalAgentTask/LocalAgentTask.tsx`，683 行
  - `tasks/LocalShellTask/LocalShellTask.tsx`，523 行
  - `tasks/LocalMainSessionTask.ts`，479 行

## 模块职责

### 1. 本地 shell task

承接 bash/monitor 等本地命令型后台执行单元。

### 2. 本地 agent task

当 AgentTool 启动本地子 agent，实际的生命周期、进度、输出文件、通知等需要 task 实现来承接。

### 3. 远程 agent task

remote agent 不只是“把本地 agent 搬远端”，还要处理 session url、远端状态、输出路径和 reconnect。

### 4. teammate / dream 等特殊任务

说明任务系统不仅服务传统 shell/agent，也服务更产品化的特殊工作项。

## 与根文件 `Task.ts` / `tasks.ts` 的关系

- `Task.ts`：定义 task 契约与通用类型
- `tasks.ts`：汇总 task 实现
- `tasks/*`：具体执行逻辑

这是一套清晰的三层关系。

## 为什么任务层很关键

没有 tasks，Claude Code 很难优雅支持：

- 后台 agent
- task panel
- foreground/background 切换
- 输出文件跟踪
- 多 agent 并行

所以 task 系统实际上是 agent 平台化的重要基础。

## 与其他模块的关系

- `tools/AgentTool`
- `tools/Task*`
- `state` 中的 `tasks` 字段
- `components/tasks/*`
- `sessionStorage` 和 `task/diskOutput`

## 维护风险

- 任务状态和 UI 可见状态高度耦合，容易出现“运行结束了但 UI 没收敛”这类问题。
- 远程任务和本地任务行为差异大，抽象边界需要持续保持清晰。
