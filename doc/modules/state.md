# state 模块

## 模块定位

`src/state` 是 Claude Code REPL 的主应用状态层。它定义 AppState 结构、store、provider 和状态变更的外围适配。

规模统计：

- 文件数：6
- 行数：1191
- 代表文件：
  - `state/AppStateStore.ts`，569 行
  - `state/AppState.tsx`，200 行
  - `state/onChangeAppState.ts`，171 行
  - `state/teammateViewHelpers.ts`，141 行

## 模块职责

### 1. 定义权威 AppState 结构

`AppStateStore.ts` 定义了非常大的状态树，内容覆盖：

- settings / model / verbose
- toolPermissionContext
- remote/bridge 状态
- tasks / agent registry
- mcp clients/tools/commands/resources
- plugins 状态
- notifications / elicitation
- file history / attribution / todos
- speculation / prompt suggestion
- tungsten / bagel / companion 等 feature 状态

### 2. 提供 store 与 selector 订阅

`AppState.tsx` 用 `useSyncExternalStore` 来提供高粒度订阅，而不是简单把整个状态扔进 context 导致全量 re-render。

### 3. 外部状态同步

`onChangeAppState.ts` 负责将 state 变更映射到外部副作用或外部消费格式。

## 设计亮点

### 1. `useSyncExternalStore`

这是非常关键的设计选择。它说明作者知道 AppState 足够大，必须用 selector 式订阅来避免渲染雪崩。

### 2. Provider 不可嵌套

`AppStateProvider` 会显式阻止嵌套 provider，防止状态源混乱。

### 3. 与 settings 变更联动

`useSettingsChange` 和 `applySettingsChange` 把外部 settings 更新同步进 AppState。

## 与 `bootstrap/state.ts` 的关系

二者是分层状态：

- `bootstrap/state.ts`：早期、轻量、进程级
- `state/AppState*`：UI/REPL 主状态

理解这点非常重要，否则很容易误以为仓库有两套互相打架的 store。

## 工程观察

从字段数量就能看出，Claude Code 的 REPL 不是“消息框 + 输入框”，而是一个状态极其丰富的终端应用。

## 维护风险

- AppState 字段持续增加会让边界变得模糊。
- 任何新特性如果都默认加到 AppState，状态树会继续膨胀。
- 好在当前已经用了 selector 机制，否则性能压力会更明显。
