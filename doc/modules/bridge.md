# bridge 模块

## 模块定位

`src/bridge` 负责 Claude Code 的 remote-control / bridge 环境能力。它让本地机器可以作为“可被远程控制的执行环境”存在，而不是只作为一个本地 CLI。

规模统计：

- 文件数：31
- 行数：12613
- 代表文件：
  - `bridge/bridgeMain.ts`，2999 行
  - `bridge/replBridge.ts`，2406 行
  - `bridge/remoteBridgeCore.ts`，1008 行
  - `bridge/initReplBridge.ts`，569 行
  - `bridge/sessionRunner.ts`，550 行

## 模块职责

### 1. bridge 环境生命周期管理

`bridgeMain.ts` 是此目录的核心文件。它负责：

- bridge 环境注册
- 轮询工作项
- session spawn / reconnect / heartbeat
- backoff 与错误处理
- worktree/临时目录生命周期
- logger 与状态显示

从代码可见，这不是简单的“开个 websocket”，而是一套长生命周期 supervisor。

### 2. REPL bridge

`replBridge.ts`、`initReplBridge.ts` 负责把“当前 REPL 会话暴露给远端控制面”的能力接进交互模式。

这意味着 bridge 不只有命令模式，也有“会话内保持连接”的模式。

### 3. remote bridge core

`remoteBridgeCore.ts` 承接更底层的桥接控制逻辑，避免所有细节都堆在 `bridgeMain.ts`。

### 4. session runner / worker 生成

`sessionRunner.ts` 说明 bridge 会真的去拉起下游 Claude 会话，而不是纯转发。

## 在产品里的角色

bridge 是 Claude Code 从“本地开发助手”走向“远程执行环境”的关键一步。它让用户可以通过 Claude.ai 或其他控制面驱动本机/环境里的 Claude Code。

## 与其他模块的关系

- `entrypoints/cli.tsx`：bridge fast-path 入口
- `main.tsx`：交互 REPL bridge 的后续接入
- `remote`：远端会话消息/权限桥
- `cli/transports/*`：网络传输层
- `tasks` / `tools/AgentTool`：bridge 会话里依然运行 agent/task 系统
- `utils/worktree.ts`：隔离工作目录

## 一个重要区分

不要把 `bridge` 和 `remote` 视为同一件事：

- `bridge` 更偏“把本地环境开放出去”
- `remote` 更偏“管理远端会话消息与权限流”

两者协作，但职责不同。

## 工程复杂度来源

bridge 目录之所以大，主要是因为它同时处理：

- 控制面 API
- 长连接/轮询
- 会话 spawn
- token refresh
- backoff 与超时
- worktree 清理
- 用户可见状态

这是一套典型的“产品级远程环境控制”复杂度。

## 维护风险

- 网络错误、认证过期、session 生命周期交织，容易形成难测边界情况。
- `bridgeMain.ts` 和 `replBridge.ts` 都已是大文件，未来仍有继续拆分空间。
