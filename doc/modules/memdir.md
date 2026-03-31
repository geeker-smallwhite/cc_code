# memdir 模块

## 模块定位

`src/memdir` 负责 Claude Code 的 memory 目录语义，包括路径管理、memory 文件类型和相关内容发现逻辑。

规模统计：

- 文件数：8
- 行数：1736
- 代表文件：
  - `memdir/memdir.ts`，507 行
  - `memdir/teamMemPaths.ts`，292 行
  - `memdir/paths.ts`，278 行
  - `memdir/memoryTypes.ts`，271 行
  - `memdir/findRelevantMemories.ts`，141 行

## 模块职责

### 1. memory 目录约定

定义 memory 文件放在哪里、叫什么、怎样被识别。

### 2. memory 类型建模

`memoryTypes.ts` 负责把不同 memory 文件、作用域、语义建模出来。

### 3. 路径与作用域解析

`paths.ts`、`teamMemPaths.ts` 说明 memory 不只是一种单路径目录，还涉及 team 或不同作用域下的 path 变体。

### 4. 相关 memory 检索

`findRelevantMemories.ts` 负责把当前上下文下最相关的 memory 找出来。

## 在运行时中的位置

该模块会影响：

- `QueryEngine` 的 memory prompt 注入
- `attachments` / `query context` 预取
- 多 agent / team 模式下的共享知识

## 与其他模块的关系

- `services/SessionMemory`
- `utils/memory/*`
- `skills` 和 `tools/AgentTool` 的长期记忆逻辑
- `context.ts` / `query.ts` 的 prompt 组装链

## 工程价值

这说明 Claude Code 的“记忆”不是一个抽象标签，而是明确落在文件系统与目录协议上的能力。

## 维护风险

- memory 路径规则如果变化，会同时影响 prompt 注入、附件发现、技能加载和 session 恢复。
- 这是一个典型“看起来像 util，实际上是协议层”的目录，变更时需要格外谨慎。
