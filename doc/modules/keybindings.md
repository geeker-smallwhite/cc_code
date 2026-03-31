# keybindings 模块

## 模块定位

`src/keybindings` 负责 Claude Code 终端交互的快捷键体系，包括默认绑定、用户覆盖、校验和 provider 安装。

规模统计：

- 文件数：14
- 行数：3161
- 代表文件：
  - `keybindings/validate.ts`，498 行
  - `keybindings/loadUserBindings.ts`，472 行
  - `keybindings/defaultBindings.ts`，340 行
  - `keybindings/KeybindingProviderSetup.tsx`，308 行
  - `keybindings/resolver.ts`，244 行

## 模块职责

### 1. 默认绑定定义

`defaultBindings.ts` 负责给系统提供默认键位语义。

### 2. 用户配置加载

`loadUserBindings.ts` 说明用户可以覆盖默认键位。

### 3. 规则校验

`validate.ts` 负责避免重复键位、无效配置和不合法绑定进入运行态。

### 4. 运行时解析

`resolver.ts` 把按键事件解析成实际动作。

### 5. Provider 装配

`KeybindingProviderSetup.tsx` 把键位系统接入 React/Ink 组件树。

## 在系统中的作用

对 Claude Code 这种重交互终端应用来说，键位系统不是辅助功能，而是主要输入协议之一。它直接影响：

- PromptInput
- overlay/dialog
- footer 与 panel 切换
- vim mode
- 选择器与列表组件

## 与其他模块的关系

- `ink`：承接原始输入事件
- `hooks`：部分 hook 会消费按键状态
- `components/PromptInput`：重点使用者
- `vim`：会与 modal 编辑模式叠加

## 工程判断

键位系统被单独做成目录而不是几个 util 函数，说明交互复杂度已经足以支撑一套小型输入框架。

## 维护点

- 任何新的全局快捷键都应先检查 resolver 和 validate 的冲突逻辑。
- 需要谨慎处理普通模式、vim 模式、dialog 模式之间的优先级。
