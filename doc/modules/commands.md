# commands 模块

## 模块定位

`src/commands` 是 Claude Code 的 slash command 实现集合。它代表的是“用户在 REPL 中能触发哪些工作流动作”。

规模统计：

- 文件数：207
- 行数：26528
- 代表文件：
  - `commands/insights.ts`，3200 行
  - `commands/plugin/ManagePlugins.tsx`，2215 行
  - `commands/plugin/PluginSettings.tsx`，1072 行
  - `commands/plugin/ManageMarketplaces.tsx`，838 行
  - `commands/plugin/BrowseMarketplace.tsx`，802 行

## 模块边界

命令层不是 Commander 子命令层，而是 REPL 内的 slash commands。

例如：

- `/config`
- `/model`
- `/permissions`
- `/plugin`
- `/review`
- `/thinkback`

这些能力的实现都在 `src/commands`，而它们的汇总装配则在根文件 `src/commands.ts`。

## 目录面貌

这个目录很能说明仓库的产品复杂度。它覆盖了大量主题：

- 项目与目录：`add-dir`、`files`、`branch`
- 会话与恢复：`resume`、`session`、`rewind`
- 配置与权限：`config`、`permissions`、`model`、`effort`、`output-style`
- 插件与扩展：`plugin`、`mcp`、`skills`、`reload-plugins`
- 体验功能：`theme`、`keybindings`、`voice`、`chrome`
- 生产力功能：`review`、`diff`、`compact`、`thinkback`
- 集成能力：`ide`、`install-github-app`、`install-slack-app`

## 一个非常有意思的信号

目录统计里有不少子目录只有 1 行或很小的 stub。这通常意味着：

- 某些命令是 feature-gated 的壳
- 当前快照做过裁剪
- 外部发行版和内部发行版的命令面不完全相同

所以不能用“当前文件小”来判断某命令在产品里不重要。

## 模块职责

### 1. 把用户 intent 映射成工作流

命令通常不直接做最底层执行，而是：

- 读取 AppState / settings
- 调用 services
- 触发 UI 对话框
- 注入 prompt
- 改变运行模式

### 2. 承接小型产品界面

插件管理相关命令已经明显不只是“执行动作”，而是终端内 mini-app：

- 管理插件
- 浏览市场
- 修改设置

### 3. 扩展 command 面

根级 `commands.ts` 还会把 skills、plugin commands、bundled skills 和动态 skills 汇总进来，说明 command 面是开放扩展的。

## 与其他模块的关系

- `components`：命令往往会挂载组件或对话框
- `services`：命令的真实业务落地层
- `skills` / `plugins`：扩展命令来源
- `state`：命令读写会话状态
- `tools`：某些命令会影响模型可见能力或运行模式

## 工程观察

这个目录之所以值得重视，是因为它是“Claude Code 产品面”的直观投影。很多用户能感知到的功能，不是直接体现在 tool 上，而是体现在 slash command 上。

## 维护风险

- 命令种类多、表面积广，极易出现行为不一致。
- 插件/MCP/skills 会进一步扩大命令来源，过滤和优先级管理必须清楚。
- 插件管理子目录已经很大，未来最好继续从命令逻辑中拆出更明确的 service/view-model。
