# skills 模块

## 模块定位

`src/skills` 是 Claude Code 的技能系统入口。它负责发现技能目录、解析 frontmatter、生成可调用 command/skill，并承接 bundled skills。

规模统计：

- 文件数：20
- 行数：4066
- 子目录：`bundled`
- 代表文件：
  - `skills/loadSkillsDir.ts`，1086 行
  - `skills/bundled/updateConfig.ts`，475 行
  - `skills/bundled/scheduleRemoteAgents.ts`，447 行
  - `skills/bundled/keybindings.ts`，339 行
  - `skills/bundled/loremIpsum.ts`，282 行

## 模块职责

### 1. 技能目录发现

`loadSkillsDir.ts` 负责从多个来源寻找技能：

- user settings
- project settings
- managed/policy settings
- plugin
- bundled
- mcp

### 2. frontmatter 解析

从代码可见，技能 frontmatter 支持的字段很多：

- description
- allowed-tools
- arguments
- when_to_use
- version
- model
- effort
- hooks
- context
- agent
- shell
- user-invocable

这说明 skill 不是简单文本片段，而是一种结构化扩展对象。

### 3. 动态 skills 生成

根级 `commands.ts` 会调用 `getDynamicSkills()`，说明技能系统会在运行时生成新的 command/能力面。

### 4. bundled skills

`bundled/*` 目录说明部分技能随产品分发，而不是完全依赖用户自定义目录。

## 技能系统的真正意义

skills 是 Claude Code 的“软扩展面”。和 plugins、MCP 不同，skills 更像：

- 可声明的工作流资产
- 带 frontmatter 的 prompt/command 组合单元
- 可以被模型和用户共同消费的能力包

## 与其他模块的关系

- `commands.ts`：把技能变成命令
- `tools/SkillTool`：把技能暴露为可调用能力
- `plugins`：插件也能贡献技能
- `services/mcp`：MCP 资源可派生技能
- `utils/markdownConfigLoader.ts`、`utils/frontmatterParser.ts`：底层解析支撑

## 工程观察

这个模块是 Claude Code 可扩展性最有特色的部分之一。它把 markdown/frontmatter 这种低门槛资产变成了正式扩展机制。

## 维护风险

- 技能来源多，去重、优先级和覆盖规则必须清晰。
- skill frontmatter 已经很丰富，继续扩展时要小心保持可理解性。
