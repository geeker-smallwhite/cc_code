# assistant 模块

## 模块定位

`src/assistant` 在当前快照里非常小，但它承接的是一个独立产品能力：`claude assistant` 相关的会话记录与恢复辅助逻辑。

规模统计：

- 文件数：1
- 行数：87
- 当前可见核心文件：`assistant/sessionHistory.ts`

## 当前快照中的角色判断

从目录体量看，这个模块不是完整的 assistant 子系统，更像是被裁剪后留下的会话历史辅助层。

需要特别注意的是：

- 当前快照里 `assistant` 目录只有 1 个文件。
- 但其他代码位置仍引用了 `assistant/sessionDiscovery.js`、`assistant/AssistantSessionChooser.js` 这类路径。

这说明两种可能性至少有一种成立：

1. 当前源码快照并不完整，assistant 相关文件被裁掉了。
2. 部分能力只在内部或特定构建形态中存在。

因此，阅读本模块时应把它当成“assistant 能力的残余可见面”，而不是完整实现。

## 主要职责

基于文件名和调用点，这个模块的职责主要是：

- 管理 assistant 会话历史
- 为 assistant 模式提供可恢复的 session 索引
- 支持 `claude assistant` 的历史发现与切换

## 与其他模块的关系

- 与 `dialogLaunchers.tsx` 协同：用于会话选择/安装 wizard 的交互入口
- 与 `remote`、`bridge` 协同：assistant 模式本质上是远端会话观察/附着能力的一部分
- 与 `history.ts`、`sessionStorage.ts` 有语义相邻关系：都在处理会话连续性，但 assistant 更偏“跨会话入口”

## 工程价值

这个模块虽然小，但它揭示出 Claude Code 并不只有“当前终端这一条会话”，而是显式支持附着、发现、恢复和多入口接入。

## 风险与注意点

- 当前快照缺少 assistant 目录下的其余文件，单靠这一目录无法完整还原 assistant 体系。
- 如果未来要继续做 assistant 功能的文档或重构，必须结合 `remote`、`bridge`、`dialogLaunchers.tsx`、`main.tsx` 一起读。
