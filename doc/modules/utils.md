# utils 模块

## 模块定位

`src/utils` 是这个仓库最大的目录，也是最容易被低估的目录。名字叫 `utils`，但它实际上已经承担了平台底座层的大量职责。

规模统计：

- 文件数：564
- 行数：180487
- 目录内最大根文件：
  - `utils/messages.ts`，5512 行
  - `utils/sessionStorage.ts`，5105 行
  - `utils/hooks.ts`，5022 行
  - `utils/bash/bashParser.ts`，4436 行
  - `utils/attachments.ts`，3997 行

## 一个核心判断

不要把这个目录理解成“公共小工具函数集合”。对 Claude Code 来说，`utils` 更像“被历史和工程演化沉淀下来的基础设施平台层”。

很多真正重要的逻辑都在这里：

- message 构造
- session 持久化
- prompt/context 组装
- shell 解析与安全
- settings/config 体系
- plugins/skills/mcp 助手
- telemetry/log/debug
- swarm/task/todo/memory

## 子目录地图

### 扩展与插件相关

- `plugins`，44 文件，20522 行
- `skills`，1 文件，311 行
- `mcp`，2 文件，457 行

### Shell 与执行安全相关

- `bash`，23 文件，12306 行
- `permissions`，24 文件，9409 行
- `swarm`，22 文件，7549 行
- `powershell`，3 文件，2305 行
- `shell`，10 文件，3069 行
- `processUserInput`，4 文件，1767 行

### 配置与平台环境

- `settings`，19 文件，4562 行
- `telemetry`，9 文件，4044 行
- `nativeInstaller`，5 文件，3018 行
- `secureStorage`，6 文件，629 行
- `sandbox`，2 文件，997 行

### 集成与产品化能力

- `claudeInChrome`，7 文件，2338 行
- `computerUse`，15 文件，2163 行
- `deepLink`，6 文件，1388 行
- `teleport`，4 文件，955 行

### 任务/记忆/建议

- `task`，5 文件，1223 行
- `suggestions`，5 文件，1213 行
- `background`，2 文件，333 行
- `memory`，2 文件，20 行
- `todo`，1 文件，18 行

### 其他基础能力

- `git`，3 文件，1075 行
- `filePersistence`，2 文件，413 行
- `messages`，2 文件，386 行
- `model`，16 文件，2710 行
- `hooks`，17 文件，3721 行
- `dxt`，2 文件，314 行
- `github`，1 文件，29 行
- `ultraplan`，2 文件，476 行

## 根文件热点

`utils` 的复杂度不只在子目录，也在一批根文件里：

### `utils/messages.ts`

这是消息构造与转换核心之一。`query.ts`、API 层、session storage 都深度依赖它。

### `utils/sessionStorage.ts`

这是会话持久化中枢，负责：

- transcript
- metadata
- file pointer
- attribution snapshot
- mode/agent setting 保存

从体量看，它几乎是一个独立子系统。

### `utils/hooks.ts`

别被文件名骗了，这不是几个轻量 hook helper，而是运行时 hooks 执行层的一大块逻辑。

### `utils/attachments.ts`

Claude Code 的输入上下文不仅有文本，还有附件、memory、文件引用、图片等，这个文件就是重要中枢。

## 重点子域详解

### 1. `utils/plugins`

这是整个 `utils` 最大的子目录，甚至超过很多顶层模块。它承担：

- 插件加载
- 缓存目录管理
- 版本化缓存
- plugin 命令/skills 加载
- 插件标识与解析

这说明插件系统的实际复杂度很大一部分沉在 `utils`，而不在 `plugins` 顶层目录。

### 2. `utils/bash`

bash 相关能力远不只是“执行命令”：

- parser
- command analysis
- 安全判定
- 只读验证
- heredoc / redirect / wildcard 处理

这是 Claude Code shell 能力安全性的基础。

### 3. `utils/permissions`

权限体系的很多核心逻辑都在这里，例如：

- permission result
- deny rule 匹配
- permission setup
- filesystem / dangerous command 识别

它和 `hooks/useCanUseTool.tsx`、`tools/BashTool/*` 共同构成安全主链。

### 4. `utils/swarm`

多 agent / teammate / swarm 的很多实际底层工具沉在这里，而不是都放在 `tasks` 或 `tools/AgentTool` 中。

这说明 swarm 既有产品语义，也有大量底层辅助逻辑。

### 5. `utils/settings`

settings 不是单一 JSON 读写，它包括：

- source 优先级
- 校验
- managed settings
- change detector
- cache
- MDM / policy 相关处理

### 6. `utils/model`

模型选择、provider、canonical name、上下文窗口、能力支持这些逻辑大多在这里提供。

### 7. `utils/telemetry`

说明埋点、追踪、会话 tracing 等能力已经系统化，而不是散点日志。

### 8. `utils/claudeInChrome` 与 `utils/computerUse`

这是两个很产品化的特殊集成面，证明 Claude Code 并非局限于本地 shell/文件编辑。

## 为什么 `utils` 会长成这样

这是大型平台型仓库非常常见的演化结果：

1. 最开始确实只是工具函数
2. 随着业务增长，一些跨层能力难以找到合适归属
3. 它们被逐步沉进 `utils`
4. 最终 `utils` 变成“没有正式归属但非常核心”的大底座

Claude Code 的 `utils` 正处于这个阶段。

## 与其他模块的关系

`utils` 基本被所有核心模块依赖：

- `main.tsx`
- `QueryEngine.ts`
- `query.ts`
- `services/*`
- `tools/*`
- `components/*`
- `hooks/*`
- `state/*`

换句话说，这里是全仓库最强的依赖汇聚点。

## 工程风险

### 1. 目录名掩盖复杂度

新读者会天然低估这里的系统性，导致错判架构重心。

### 2. 职责漂移

一些本应上升为正式子系统的逻辑仍沉在 `utils`，例如：

- session storage
- message pipeline
- plugin loader
- shell security

### 3. 依赖中心化

`utils` 被所有地方依赖，任何改动都可能产生广泛影响。

## 建议的阅读方式

不要试图一次性读完 `utils`。更合理的方式是按主链路分专题阅读：

### 想理解对话消息

读：

- `utils/messages.ts`
- `utils/attachments.ts`
- `utils/processUserInput/*`
- `utils/sessionStorage.ts`

### 想理解权限与 shell 安全

读：

- `utils/permissions/*`
- `utils/bash/*`
- `utils/powershell/*`
- `utils/sandbox/*`

### 想理解配置与运行环境

读：

- `utils/settings/*`
- `utils/config.*`
- `utils/auth.*`
- `utils/model/*`

### 想理解扩展系统

读：

- `utils/plugins/*`
- `utils/skills/*`
- `utils/mcp/*`

## 小结

`utils` 是整个仓库最不像“辅助层”的目录。它实际上保存了 Claude Code 运行细节的大量真相。

如果只读 `services` 而不读 `utils`，你只能理解到“系统做了什么”；只有把 `utils` 一起读，才能理解“系统具体是怎么做到的”。
