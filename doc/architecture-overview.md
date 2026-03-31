# Claude Code 总体架构总览

## 1. 架构定位

Claude Code 可以看成四层叠加：

1. 启动与装配层
2. 会话与 agent runtime 层
3. 交互与终端 UI 层
4. 扩展与外部系统接入层

它不是“前后端分离”的结构，而是一个在同一进程中同时承载：

- 命令行入口
- React/Ink 终端应用
- 模型对话内核
- 工具/任务执行器
- 权限与沙箱决策
- 插件/技能/MCP 扩展面
- bridge/remote/direct-connect 网络会话

## 2. 主调用链

### 2.1 启动链

`src/entrypoints/cli.tsx`
-> fast-path 判断
-> `src/main.tsx`
-> `run()`
-> `Commander`
-> `preAction`
-> `init() / migrations / settings / plugins / remote settings`
-> 进入交互 REPL 或非交互 print/subcommand

### 2.2 交互式链路

`main.tsx`
-> `showSetupScreens()`
-> `launchRepl()`
-> `components/App`
-> `screens/REPL`
-> `AppState`
-> `QueryEngine` / `query()`
-> `services/api/claude`
-> `runTools()`

### 2.3 非交互式链路

`main.tsx`
-> `cli/print.ts`
-> `QueryEngine`
-> `query()`
-> `services/api/claude`
-> tool orchestration
-> 结构化输出 / 文本输出

### 2.4 远程与桥接链路

`entrypoints/cli.tsx`
-> `bridgeMain()` / direct-connect / ssh / remote-control
-> `bridge/*` / `remote/*` / `server/*`
-> 在本地或远程侧创建会话通道
-> 仍然复用主 REPL/runtime

## 3. 分层视角

### 3.1 启动与装配层

主要由以下模块组成：

- `entrypoints`
- `bootstrap`
- `main.tsx`
- `setup.ts`
- `interactiveHelpers.tsx`
- `replLauncher.tsx`

这一层负责解决“系统怎么正确启动”：

- 哪些 flag 需要 fast-path
- 是否是 interactive
- 当前客户端类型是什么
- 设置、信号、迁移、遥测何时初始化
- 最终落到哪种运行模式

### 3.2 runtime 层

主要由以下模块组成：

- `QueryEngine.ts`
- `query.ts`
- `Tool.ts`
- `tools.ts`
- `Task.ts`
- `tasks.ts`
- `services`

这一层负责解决“Claude 一轮对话内部到底如何运行”：

- 如何组 prompt
- 如何发送 API 请求
- 如何处理工具调用
- 如何做 continuation、compact、retry、budget 控制
- 如何追踪 usage、cost、任务输出、权限拒绝

### 3.3 UI 层

主要由以下模块组成：

- `screens`
- `components`
- `hooks`
- `ink`
- `context`
- `keybindings`
- `vim`
- `buddy`

这一层负责解决“用户在终端里怎样看见、输入、操作、切换状态”。

要注意的是，这个项目的 UI 层不薄。它不是一个简单的“把文本打出来”的 CLI，而是一个完整的终端应用。

### 3.4 扩展层

主要由以下模块组成：

- `commands`
- `skills`
- `plugins`
- `services/mcp`
- `tools/*`
- `remote` / `bridge` / `server`

这一层负责解决“Claude Code 如何扩展自身能力边界”。

扩展方式至少有四种：

- slash command 扩展
- tool 扩展
- skill 扩展
- plugin / MCP 扩展

## 4. 核心对象关系

### 4.1 `AppState` 不是全部状态

很多人第一次读仓库会默认“状态都在 React store 里”。这里不是这样。

仓库中至少有四类状态：

- bootstrap 级全局进程状态
- AppState UI/会话状态
- QueryEngine 的对话状态
- task/agent 子状态

这四者并不是重复，而是分工不同：

- bootstrap 负责最早、最轻、全局可读的状态
- AppState 负责 REPL/UI 可观察状态
- QueryEngine 负责 headless/会话内累计状态
- task state 负责后台执行单元

### 4.2 命令与工具是两种边界

- command 面向用户输入，例如 `/config`、`/review`、`/plugin`
- tool 面向模型能力，例如 `BashTool`、`FileEditTool`、`AgentTool`

它们会联动，但不是同一种 abstraction。

常见误区是把 command 看成“tool 的别名”。其实不是：

- command 更像 UI/工作流动作
- tool 更像 agent 采样循环里的能力接口

### 4.3 `services` 是业务执行层

`services` 目录承担大量真正的业务逻辑：

- Claude API 调用
- MCP client/auth
- tool execution
- compact
- analytics
- settings sync
- plugin installation
- memory extraction

如果把 `components` 看成交互壳，那么 `services` 才是大部分执行语义的落点。

### 4.4 `utils` 是实际上的平台底座

理论上 `utils` 应该只放公共小函数；这里并不是。

`utils` 已经吸收了大量中等复杂度甚至高复杂度子系统：

- message 构造
- sessionStorage
- bash parser
- attachments
- config/settings
- permissions/sandbox
- shell/git/github
- plugin/skill/mcp 助手

因此很多真正重要的“运行细节”并不在 `services`，而在 `utils`。

## 5. 交互模式矩阵

### 5.1 交互 REPL

特征：

- 终端 UI 完整
- 权限对话与通知可视化
- task/teammate/footer/overlay 丰富
- AppState 驱动明显

关键文件：

- `screens/REPL.tsx`
- `components/*`
- `hooks/*`

### 5.2 print / automation

特征：

- 无完整 UI
- 重点在结构化输出和稳定自动化
- 依赖 `QueryEngine` + `print.ts`

关键文件：

- `cli/print.ts`
- `QueryEngine.ts`
- `query.ts`

### 5.3 SDK / headless

特征：

- 强调 schema、状态事件、消息序列化
- 通过 `entrypoints/sdk/*` 和 `agentSdkTypes.ts` 暴露契约

### 5.4 remote / bridge / assistant

特征：

- 会话可能跨进程、跨机器或跨控制面
- 需要 transport、permission bridge、event adapter

关键文件：

- `bridge/*`
- `remote/*`
- `server/*`

## 6. 安全与权限横切

权限相关代码横跨多个目录：

- `hooks/useCanUseTool.tsx`
- `utils/permissions/*`
- `components/permissions/*`
- `utils/sandbox/*`
- `tools/BashTool/*`
- `tools/PowerShellTool/*`

其中最重要的设计点有三个：

1. deny 规则会在模型看到工具前就过滤一部分工具。
2. 交互审批与自动模式并存，而不是二选一。
3. sandbox 配置不是手工静态配置，而是从 Claude Code 的权限语义推导出来。

这说明安全不是外围逻辑，而是 runtime 主路径的一部分。

## 7. 扩展机制全景

### 7.1 skills

技能来自本地目录、bundled skills、插件技能、MCP 资源派生技能。

### 7.2 plugins

插件支持：

- 内建插件
- 市场安装
- 本地目录注入
- 缓存与刷新

### 7.3 MCP

MCP 在这个仓库里不是“接几个工具”那么简单，它会带来：

- tools
- commands
- resources
- auth
- approvals
- resource prefetch

### 7.4 subagents / teams / tasks

AgentTool 加上 tasks 系统，使 Claude Code 能从“一个 agent”演化成“agent 协作平台”。

## 8. 模块关系地图

可以用一句话概括各核心模块：

- `entrypoints`：轻量入口和 schema 边界
- `bootstrap`：最早可用的全局状态
- `screens`：顶层终端场景
- `state`：UI 状态容器
- `services`：执行语义中台
- `tools`：模型可调用能力集合
- `tasks`：后台执行单元
- `commands`：用户交互命令集合
- `components`：UI 组件库
- `utils`：超大型基础能力底座

## 9. 阅读仓库时最重要的两个原则

### 9.1 先分清“装配层”和“能力层”

很多大文件是在做装配，不一定在做真正的业务；例如 `main.tsx` 主要是在拼装运行模式，不是所有行为都应该往里面找。

### 9.2 先认主链路，再认扩展面

正确顺序是：

1. 启动
2. REPL / print 入口
3. QueryEngine / query
4. tool / task / API
5. 再看 MCP / plugin / skill / remote

否则会很容易被 feature gate 和目录体量淹没。
