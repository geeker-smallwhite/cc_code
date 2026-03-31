# coordinator 模块

## 模块定位

`src/coordinator` 是一个非常小但战略意义很强的模块。它代表 Claude Code 在多 agent / 协作模式下的 coordinator 语义。

规模统计：

- 文件数：1
- 行数：369
- 核心文件：`coordinator/coordinatorMode.ts`

## 当前可见职责

从 `QueryEngine.ts` 的调用可见，coordinator 模块至少提供：

- `getCoordinatorUserContext`

也就是说，它会向系统 prompt 或运行上下文注入协调者模式需要的额外信息。

## 为什么这个模块值得单独存在

coordinator 模式的语义和普通单 agent 模式不同，通常会涉及：

- 角色分工
- 可用工具集合差异
- 上下文注入差异
- 任务路由与聚合

把这类逻辑单独放在目录里，可以避免把多 agent 模式污染到普通路径。

## 与其他模块的关系

- `tools/AgentTool`：coordinator 很可能决定 agent 编排行为
- `QueryEngine`：在系统上下文中注入协调信息
- `tasks`：实际任务执行仍由 task 系统承担
- `state` / `components`：UI 层会消费 coordinator 相关的 task panel 状态

## 工程判断

虽然当前目录很小，但它不是边角料，而是未来多 agent 能力继续扩张时的重要锚点。小，不代表不重要，只意味着当前快照中大部分协调逻辑还分散在别处。
