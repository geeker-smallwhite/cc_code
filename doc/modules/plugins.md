# plugins 模块

## 模块定位

`src/plugins` 是顶层插件注册层，负责告诉系统有哪些内建插件以及 bundled plugin 入口在哪里。

规模统计：

- 文件数：2
- 行数：182
- 子目录：`bundled`
- 代表文件：
  - `plugins/builtinPlugins.ts`，159 行
  - `plugins/bundled/index.ts`，23 行

## 注意事项

不要被这个目录的体量误导。Claude Code 的插件系统并不小，真正的大量逻辑分布在：

- `services/plugins/*`
- `utils/plugins/*`
- `commands/plugin/*`

`src/plugins` 更像“插件注册表入口”，不是完整插件框架。

## 模块职责

### 1. 内建插件目录

`builtinPlugins.ts` 负责定义系统自带插件集合。

### 2. 提供 builtin plugin skill commands

从根级 `commands.ts` 的调用可见，这里还会把 builtin plugin 里暴露出的 skill commands 汇总出来。

### 3. bundled plugin 装配入口

`bundled/index.ts` 说明某些插件能力随产品一起打包。

## 与其他模块的关系

- `commands/plugin/*`：插件管理 UI
- `services/plugins/PluginInstallationManager.ts`：后台安装
- `utils/plugins/pluginLoader.ts`：缓存、加载、清理
- `commands.ts`：把插件命令并入 slash command 系统

## 工程判断

插件体系被拆成“顶层注册层 + utils/services 执行层 + commands 管理层”，这是比较健康的结构。

## 维护要点

- `plugins` 目录适合放注册与定义，不适合继续堆积执行逻辑。
- 真正的插件系统行为变更，通常不应只改这里，还要看 `utils/plugins` 和 `services/plugins`。
