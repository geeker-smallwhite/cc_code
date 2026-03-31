# cli 模块

## 模块定位

`src/cli` 承担 Claude Code 的命令行 I/O 基础设施，尤其是非交互路径、结构化输出和 transport 层。

规模统计：

- 文件数：19
- 行数：12355
- 子目录：`handlers`、`transports`
- 代表文件：
  - `cli/print.ts`，5594 行
  - `cli/transports/ccrClient.ts`，998 行
  - `cli/handlers/plugins.ts`，878 行
  - `cli/structuredIO.ts`，859 行
  - `cli/transports/WebSocketTransport.ts`，800 行

## 不要把 `cli` 和 `commands` 混淆

- `cli`：外层命令行运行基础设施
- `commands`：REPL 内 slash commands

`cli` 解决的是“这个进程如何和外界以命令行协议交流”，不是“用户在会话里打什么 slash command”。

## 模块职责

### 1. print/headless 主路径

`print.ts` 是整个目录最关键的文件，也是全仓库最大的文件之一。它负责：

- `-p/--print` 模式
- 结构化输出
- stream-json 输出
- SDK control request/response
- 会话恢复与 hydration
- settings sync / remote settings 下载
- permission prompt 适配
- hook event 转发
- 远端 session 状态变更

这说明非交互模式不是“少一个 UI 而已”，而是一套独立复杂路径。

### 2. Structured I/O

`structuredIO.ts` 表明 Claude Code 对机器消费场景有明确支持，而不是仅面向人类终端输出。

### 3. 传输层

`transports/*` 负责 WebSocket、CCR、HybridTransport 等传输实现。

其中 `HybridTransport` 的存在，说明有些场景需要组合多种传输语义，而非单一 socket。

### 4. handlers

`handlers/plugins.ts` 等文件承接部分命令行子场景的专门处理逻辑。

## 子目录理解

### `handlers`

更偏子功能入口处理器，例如插件相关 CLI 流程。

### `transports`

更偏网络/连接抽象，使 CLI 能与 remote/CCR/direct-connect 场景打通。

## 与其他模块的关系

- `main.tsx`：是否进入 print 路径
- `QueryEngine.ts`：headless 对话引擎
- `services/mcp/*`：结构化接入外部能力
- `remote` / `bridge`：远端控制面与会话同步
- `utils/sessionStorage.ts`：恢复、元数据、持久化

## 工程观察

`cli/print.ts` 5594 行这一点非常关键。它说明自动化和编程式使用场景在产品中已经成长为一级复杂度，而不是附加模式。

## 维护风险

- `print.ts` 的功能面太广，几乎触碰所有系统横切能力。
- 非交互模式和交互模式共享大量 runtime，但也有自己特有的状态与 I/O 约束，回归风险高。
