# types 模块

## 模块定位

`src/types` 提供跨模块共享的数据结构和协议类型，是仓库的公共契约层。

规模统计：

- 文件数：11
- 行数：3446
- 子目录：`generated`
- 代表文件：
  - `types/generated/events_mono/claude_code/v1/claude_code_internal_event.ts`，865 行
  - `types/permissions.ts`，441 行
  - `types/textInputTypes.ts`，387 行
  - `types/plugin.ts`，363 行
  - `types/logs.ts`，330 行

## 模块职责

### 1. 共享领域类型

例如：

- permissions
- plugin
- logs
- text input
- ids
- utils 辅助类型

### 2. 生成类型

`generated` 目录说明部分类型来自自动生成，而不是纯手写。

### 3. 跨层契约稳定点

这些类型被 `components`、`services`、`tools`、`state` 等多个层同时依赖。

## 为什么这个模块重要

像 Claude Code 这样的多层系统，如果没有一个相对稳定的 shared types 层：

- 模块边界会更脆弱
- 变更传播成本更高
- headless / UI / remote / plugin 之间更难协作

## 与其他模块的关系

几乎所有大模块都会依赖 `types`，特别是：

- `state`
- `tools`
- `commands`
- `services`
- `entrypoints`

## 工程观察

`types` 目录本身没有太多业务，但它能反映仓库真正关心的数据模型是什么。比如单独存在 `plugin.ts`、`permissions.ts`、`textInputTypes.ts`，说明这些领域是系统的一等概念。

## 维护建议

- 类型层应尽量保持“契约定义”身份，不要混入业务函数。
- 生成类型和手写类型的边界应清晰，以免手工修改 generated 内容。
