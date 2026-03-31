# migrations 模块

## 模块定位

`src/migrations` 负责版本升级时的设置和行为迁移，是 Claude Code 配置兼容性的防线。

规模统计：

- 文件数：11
- 行数：603
- 代表文件：
  - `migrations/migrateEnableAllProjectMcpServersToSettings.ts`，118 行
  - `migrations/migrateSonnet45ToSonnet46.ts`，67 行
  - `migrations/migrateAutoUpdatesToSettings.ts`，61 行

## 模块职责

### 1. 旧配置向新配置迁移

随着产品持续演化，设置项名称、默认值和配置位置可能变化，migration 用来保证升级后不直接破坏用户体验。

### 2. 模型默认值切换

例如从旧 model 名称迁移到新 model 名称。

### 3. MCP、自动更新等配置迁移

这些迁移说明仓库的很多能力都在持续变化，配置模型不是静态的。

## 运行时位置

`main.tsx` 的 `preAction` 阶段会调用迁移逻辑，因此它属于“真正启动前的兼容性保障层”。

## 工程意义

有独立 migration 目录说明：

- 作者重视向后兼容
- 配置结构确实在不断演进
- 不愿意把兼容性代码散落进主逻辑

## 维护建议

- migration 应保持幂等
- 旧迁移不要随意删除，因为恢复旧项目状态时仍可能触发
- 迁移文件命名已经采用“做什么迁移”的方式，这很利于追踪历史
