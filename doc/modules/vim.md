# vim 模块

## 模块定位

`src/vim` 负责在 Claude Code 的输入体验中提供 Vim 风格的模态编辑能力。

规模统计：

- 文件数：5
- 行数：1513
- 代表文件：
  - `vim/operators.ts`，556 行
  - `vim/transitions.ts`，490 行
  - `vim/types.ts`，199 行
  - `vim/textObjects.ts`，186 行
  - `vim/motions.ts`，82 行

## 模块职责

### 1. motions

定义光标移动语义。

### 2. operators

定义删除、修改等操作符行为。

### 3. text objects

定义词、块、文本对象的选择与作用范围。

### 4. transitions

定义不同 mode 之间的状态转移。

## 在产品中的意义

这说明 Claude Code 的输入框不是简化版 readline，而是目标明确地支持高级编辑体验。

## 与其他模块的关系

- `components/PromptInput/*`
- `keybindings`
- `ink` 输入解析层

## 工程观察

Vim 模式被单独做成目录是正确的，因为它本质上是一套小状态机。如果把这些逻辑混进 PromptInput，会让输入组件更难维护。

## 维护风险

- 模态编辑天然容易和全局快捷键、选择器快捷键冲突。
- 任何 prompt 输入逻辑变更都要考虑 Vim 模式的行为一致性。
