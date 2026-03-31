# ink 模块

## 模块定位

`src/ink` 是 Claude Code 的终端渲染和输入底座。它不是简单地“使用了 upstream Ink”，而更像是一个带定制能力的 fork/扩展层。

规模统计：

- 文件数：96
- 行数：19859
- 子目录规模：
  - `components`，18 文件，2243 行
  - `events`，10 文件，797 行
  - `hooks`，12 文件，677 行
  - `layout`，4 文件，563 行
  - `termio`，9 文件，2271 行
- 代表文件：
  - `ink/ink.tsx`，1723 行
  - `ink/screen.ts`，1486 行
  - `ink/render-node-to-output.ts`，1462 行
  - `ink/selection.ts`，917 行
  - `ink/parse-keypress.ts`，801 行

## 模块职责

### 1. 渲染树到终端输出

`render-node-to-output.ts`、`screen.ts` 表明这一层在控制：

- 节点布局
- 差异更新
- 最终输出到 terminal buffer

### 2. 输入事件与按键解析

`parse-keypress.ts`、`events/*`、`hooks/*` 提供低层输入事件体系。

### 3. 选择与交互状态

`selection.ts` 等文件说明终端 UI 里存在选择、高亮、光标/焦点行为，不只是纯文本刷新。

### 4. termio 能力

`termio` 子目录负责更贴近终端协议的事情，例如状态行、osc、终端能力判断等。

## 与 `src/ink.ts` 的关系

- `src/ink`：底层实现
- `src/ink.ts`：项目对外导出的统一 facade，并加上 ThemeProvider

这种分层是合理的，能避免底层实现细节污染全仓库 import 面。

## 为什么这个目录体量大

因为 Claude Code 的 UI 并不是简单 stdout 输出，而是：

- 有复杂布局
- 有实时更新
- 有交互焦点
- 有快捷键和选择
- 有面板/overlay

这套体验必须靠低层终端框架支撑。

## 工程判断

`ink` 目录接近 2 万行，说明作者已经不满足于“默认 Ink 能给什么就用什么”，而是在自己掌控终端框架行为。

## 维护风险

- 这是 UI 的地基，任何输入/渲染 bug 都会辐射到整个产品。
- 自定义 terminal rendering 层越深，升级上游或跨平台适配的成本越高。
