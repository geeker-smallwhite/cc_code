# components 模块

## 模块定位

`src/components` 是 Claude Code 的终端 UI 组件库，也是整个仓库第二大目录。它不只是“展示层”，而是相当一部分产品体验和交互逻辑的承载体。

规模统计：

- 文件数：389
- 行数：81892
- 关键子目录规模：
  - `permissions`，51 文件，12199 行
  - `messages`，41 文件，6055 行
  - `PromptInput`，21 文件，5175 行
  - `agents`，26 文件，4545 行
  - `tasks`，12 文件，3950 行
  - `mcp`，13 文件，3932 行
  - `CustomSelect`，10 文件，3023 行
  - `Settings`，4 文件，2577 行
  - `LogoV2`，15 文件，2497 行
  - `design-system`，16 文件，2253 行

## 模块职责

### 1. 终端 UI 组件库

这里包含基础控件、面板、对话框、消息组件、选择器、设置页、任务视图等大量可复用组件。

### 2. 业务化交互载体

很多组件已经不只是无状态 view，而是：

- 直接接权限逻辑
- 直接处理任务状态
- 直接渲染 MCP/agent/plugin 特有信息

### 3. 终端设计系统落点

`design-system` 子目录说明这个项目有自己的终端组件风格体系，而不是单纯依赖原生 Ink 组件。

## 重点子域

### `PromptInput`

这是整个交互系统的核心之一。输入框不仅要处理文字，还要处理：

- suggestion
- file references
- 历史
- 键位
- vim mode
- overlay/footer 交互

### `permissions`

权限 UI 之所以占 12k+ 行，说明权限在产品里不是一个简单确认框，而是完整的人机交互系统。

### `messages`

消息渲染是 Claude Code 体验的中心，包括 assistant message、tool progress、特殊标签、streaming 片段等。

### `agents` 和 `tasks`

这两个目录体现了多 agent / background task 已经深入 UI 层。

### `mcp`

MCP 的 UI 体量不小，说明它是终端产品中的一级概念，而不是不可见后端集成。

## 与 `screens` 的关系

- `screens` 是场景级入口，如 `REPL`、`Doctor`
- `components` 是被场景复用和组合的界面单元

换句话说，`screens` 管“页面/场景”，`components` 管“部件/交互块”。

## 工程观察

很多大型 CLI 项目的 UI 目录只占很小比例，这个仓库不是。组件层 8 万多行，说明终端 UI 本身就是产品核心竞争力的一部分。

## 维护风险

- 组件层已经吸收了相当多业务逻辑，继续膨胀会让 view/business 边界更模糊。
- `PromptInput`、`permissions`、`messages` 这几个区域是最需要关注的大块热点。
