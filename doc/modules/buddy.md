# buddy 模块

## 模块定位

`src/buddy` 是 Claude Code 终端 UI 中偏体验化、拟人化的一块能力，用来承接 companion/buddy 角色、提示反馈和视觉状态。

规模统计：

- 文件数：6
- 行数：1300
- 代表文件：
  - `buddy/sprites.ts`，514 行
  - `buddy/CompanionSprite.tsx`，371 行
  - `buddy/types.ts`，148 行
  - `buddy/companion.ts`，133 行
  - `buddy/useBuddyNotification.tsx`，98 行

## 模块职责

### 1. 角色视觉表现

`sprites.ts` 和 `CompanionSprite.tsx` 说明该模块负责终端内 companion 的渲染素材和显示逻辑。

### 2. 交互反馈

`useBuddyNotification.tsx` 表明 buddy 不只是静态装饰，它还能感知事件并向用户回馈状态。

### 3. 统一类型定义

`types.ts` 让视觉状态、动作、反应和外部调用接口保持一致。

## 在系统中的位置

这个模块不是 runtime 主链路的一部分，但它直接挂在 REPL 体验上，和以下模块关系紧密：

- `screens/REPL.tsx`
- `components/*`
- `state/AppStateStore.ts`
- `context/notifications.tsx`

## 为什么它单独成目录

buddy 相关代码如果散落在组件目录里，会造成两个问题：

- 动画/素材/状态逻辑混入主 UI
- 产品实验能力难以 feature gate 或独立迭代

单独成目录说明它被当作一个可以独立演进的交互特性。

## 工程判断

这个模块透露出一个很重要的产品事实：Claude Code 的终端体验并不只追求“高效文本工具”，也追求一定程度的人格化与反馈设计。

## 风险与维护点

- 这类模块天然依赖 REPL 具体布局，容易受到 footer、panel、focus 变更影响。
- 如果 companion 状态来自 AppState 或通知流，边界处理不当会让 UI 小特性侵入主状态模型。
