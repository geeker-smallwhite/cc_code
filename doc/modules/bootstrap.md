# bootstrap 模块

## 模块定位

`src/bootstrap` 是最早期、最轻量、最全局的进程级状态层。它不是 UI store，也不是会话引擎，而是“在系统足够早的时候就必须可用”的状态容器。

规模统计：

- 文件数：1
- 行数：1758
- 核心文件：`bootstrap/state.ts`

## 为什么这个模块重要

很多大型 CLI/agent 项目都会出现一个问题：在真正的应用状态初始化之前，已经有大量逻辑需要读取 session id、cwd、client type、telemetry 句柄、cost 计数器或启动期配置。

`bootstrap/state.ts` 就是为了解决这个问题而存在。

它让以下代码可以在非常早期安全读取状态：

- 启动层
- API 层
- cost 统计
- prompt 上下文
- 远程/bridge 辅助逻辑
- 某些 feature gate 下的全局行为

## 状态内容概览

从代码可直接看到，这里维护的状态范围极广，包括：

- 原始 cwd、projectRoot、当前 cwd
- session id、parent session id、session source
- total cost / token / duration / lines changed
- model usage、main loop model、sdk betas
- interactive/client type/question preview format
- telemetry logger、meter、counter
- inline plugins、chrome flag、session bypass permissions
- cron tasks、session-created teams、trust 标记
- invoked skills、teleported session info
- remote mode、direct connect server url
- prompt cache 与 beta header latch
- prompt id、last request id、post-compaction 标记

## 模块职责

### 1. 会话级基础标识

比如：

- `sessionId`
- `projectRoot`
- `originalCwd`
- `clientType`

这些值在系统的很多地方都被读，但不适合放进 React store。

### 2. 统计与遥测累加器

`cost-tracker.ts`、API 计数、OTel 相关逻辑都依赖 bootstrap state 中的计数器和 provider。

### 3. 启动期和跨层共享的小型全局状态

例如：

- trust 是否建立
- 是否 remote mode
- 是否需要特殊 beta headers
- 系统 prompt section cache

这些状态如果进 UI store，反而会增加耦合。

## 与 `state/AppStateStore.ts` 的区别

这是理解仓库的重要分界：

- `bootstrap/state.ts`：全局进程态，强调“早”和“轻”
- `state/AppStateStore.ts`：REPL 可观察应用态，强调“UI 驱动”和“切片订阅”

两者不是重复实现，而是分层不同。

## 设计优点

- 让启动层和基础设施层不必依赖 React
- 便于 headless 路径复用
- 把 session/telemetry/model/cache 这类横切状态集中起来

## 维护风险

文件顶部已经明确提醒“不要再往这里加太多全局状态”。这说明作者也意识到这里已经很接近“全局状态黑洞”。

当前风险主要有两个：

- 状态面过大，未来容易继续膨胀
- 某些字段的真实所有权可能变得模糊，既不像纯 bootstrap，也不像纯 runtime

## 阅读建议

读这个文件时，不要把它当成普通配置文件，而要把它看成“系统运行的最低层控制面状态”。

最值得优先关注的部分是：

- session / cwd / projectRoot
- cost / token / usage
- trust / permissions / remote mode
- prompt cache / beta header latch
