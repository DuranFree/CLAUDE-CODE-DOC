# From-Scratch — 全局规则

这些规则适用于整个工作流的每一个阶段，始终有效，不可忽略。

---

## 引擎 MCP 检查

项目启动时，读取 KICKOFF 文档中的目标平台字段，执行一次性 MCP 检查流程：

| 平台 | 推荐 MCP | 安装方式 |
|---|---|---|
| Unity | CoderGamester/mcp-unity | UPM git URL + node build |
| Unreal Engine | chongdashu/unreal-mcp | UE 插件 + Python server |
| Godot | Coding-Solo/godot-mcp | npm install + node |
| Phaser.js / Web | phaserjs/editor-mcp-server | `npx @phaserjs/editor-mcp-server` |

**完整检查流程（只在项目启动时执行一次）：**

**第一步 — 安装检查：**
1. 查看项目根目录是否存在 `.mcp.json` 且包含对应平台的 MCP 配置
2. 未配置 → 告知用户：`⚠️ 检测到目标平台为 [X]，推荐安装 [MCP名] 以支持引擎内部状态查询，是否现在安装？`
   - 用户确认 → 执行安装流程 → 进入第二步
   - 用户跳过 → 记录到 `tech-debt.md`，后续查询场景时改用读取文件，跳过后续步骤

**第二步 — 版本检查：**
3. 已配置 / 刚安装完 → 检查是否有新版本（查对应 GitHub repo 最新 release / npm latest tag）
   - 有更新 → 提示用户：`🔄 [MCP更新] [MCP名] 有新版本，建议更新后再继续，是否现在更新？`
     - 用户确认 → 执行更新 → 进入第三步
     - 用户跳过 → 直接进入第三步（版本滞后属于低优先级，不记录）
   - 无更新 → 直接进入第三步

**第三步 — 可用性验证：**
4. 调用一个最简单的 MCP 工具指令（如获取场景信息、列出项目结构等），验证 MCP 是否正常响应
   - 验证通过 → 输出 `✅ [MCP] [MCP名] 已就绪，后续将优先使用 MCP 查询引擎状态`，继续
   - 验证失败（连接失败 / 权限错误 / 工具无响应 / 账号问题）→ 输出 `⚠️ [MCP] 验证失败：[错误原因]，已降级为读取文件模式`，记录到 `tech-debt.md`，后续改用读取文件

---

## Codex 代码审查工具检查

**完整检查流程（只在项目启动时执行一次）：**

**第一步 — 安装检查：**
1. 检查 `/codex` 命令是否可用（插件是否已安装）
2. 未安装 → 提示用户：`⚠️ Codex 插件未安装，建议安装以支持深度代码审查，是否现在安装？`
   - 用户确认 → 执行安装（`npm install -g @openai/codex` 或对应包管理器）→ 进入第二步
   - 用户跳过 → 记录到 `tech-debt.md`，后续所有 Codex 审查步骤改用 Claude 自身审查，跳过后续步骤
3. 已安装 → 直接进入第二步

**第二步 — 账号与可用性验证：**
4. 尝试简单调用（如 `/codex --version` 或最小任务）验证 Codex 是否正常响应
   - 验证通过 → 输出 `✅ [Codex] 已就绪，核心逻辑 Phase 将在 commit 前自动触发 adversarial-review`，继续
   - 验证失败 → 输出 `⚠️ [Codex] 验证失败，请排查以下可能原因：`
     - 账户欠费 / 额度耗尽 → 检查 OpenAI 账单页面，充值或升级套餐
     - API Key 未配置 / 已失效 → 重新在环境变量中设置 `OPENAI_API_KEY`
     - 网络问题 → 检查代理或防火墙设置
     - 插件版本过旧 → 尝试 `npm update -g @openai/codex`
   - 提示用户：`是否已解决？解决后重新验证，否则后续审查将降级为 Claude 自身审查。`
     - 用户已解决 → 重新执行第二步验证
     - 用户跳过 → 记录到 `tech-debt.md`，后续改用 Claude 自身审查

---

## GitHub 确认

Ask the user:
> "是否需要关联 GitHub 仓库？Architecture Improvement 阶段会用到 GitHub Issues 来记录重构任务。如果没有，该阶段会跳过 Issue 创建步骤。"

If yes, confirm `gh` CLI is installed and logged in before proceeding. If no, skip all `gh issue create` steps silently.

---

## 版本控制规则

After every Phase where all tests pass, check if a git repository is connected:
- **If yes** → automatically commit: `git commit -m "Phase X: <描述> - all tests passing"`
- **If no** → notify the user every time: `⚠️ [版本控制] 未检测到 git 仓库，跳过本次 commit。如需版本控制保护，请关联 git 仓库。`

Never skip the check itself.

---

## 开发日志规则

After every Phase, automatically append to `./logs/dev-log.md` in the current project folder. Create if it doesn't exist:

```
## Phase <number>: <title> — <date>
**Status**: ✅ Completed / ⚠️ Partially completed
**What was done**: <brief summary>
**Decisions made**: <any architectural or design decisions>
**Technical debt**: <anything skipped or deferred, with reason>
**Problems encountered**: <any issues found and how they were resolved>
```

---

## 持久化文件规则

以下文件必须在项目文件夹里维护，贯穿整个开发周期。每个文件的更新方式不同，严格遵守：

### 文件结构
```
<project>/
  plans/
    feature-checklist.md   ← 功能清单，完成打勾，持续更新
    visual-checklist.md    ← 视觉清单，完成打勾，持续更新
    tech-debt.md           ← 技术债，解决后删除，新增时追加
    known-bugs.md          ← 已知 Bug，修复后标记 ✅，新增时追加
    deep-scan-results.md   ← Phase 1 生成，之后不再修改
  logs/
    dev-log.md             ← 每个 Phase 追加，不覆盖
 ```

### 各文件更新规则

**feature-checklist.md**
- Phase 1 生成完整功能清单后创建
- 每完成一个功能项立即更新，标记 ✅
- 不删除任何项目，只更新状态

**visual-checklist.md**
- Phase 2 生成视觉清单后创建
- 每完成一个视觉项立即更新，标记 ✅
- 不删除任何项目，只更新状态

**tech-debt.md**
- 发现技术债时追加
- 解决后删除对应条目
- 格式：`- [ ] <描述> — 原因：<why deferred> — Phase <number>`

**known-bugs.md**
- 发现 bug 时追加
- 修复后标记 ✅，不删除
- 格式：`- [ ] <描述> — 发现于 Phase <number>`

**deep-scan-results.md**
- Phase 1 深度扫描完成后创建
- 包含：硬编码数值、配置文件、交互流程链
- **之后不再修改**

---

## 关键文件 git status 监测规则

项目启动时（只执行一次），识别该项目存在哪些关键索引/参考文件，并将对应的 git status 监测规则写入项目 `CLAUDE.md`：

**必须监测的文件（所有项目）：**
- 项目计划 roadmap 文件 — 已内置于通用 CLAUDE.md，无需额外写入

**按项目类型识别，存在则写入监测规则：**
- 资源索引文件（如 `assets-index.json` 或类似文件）→ 存在时写入：`如果资源索引文件出现在 git status 修改列表中，立即读取它`；同时将路径存入 memory（类型：reference，内容：美术资源索引路径）
- `deep-scan-results.md` — 之后不再修改，无需监测

**写入格式（追加到项目 CLAUDE.md 的会话开始部分）：**
```
检查 git status，如果以下文件出现在修改列表中，立即读取：
- `plans/assets-index.json`（如该文件存在）
```

---

## 玩家操作清单对比规则

每个涉及 UI 或交互的 Phase 开始前，必须先执行以下两步：

**第一步 — 区域存在性检查：**
- 列出项目中所有独立的界面区域/面板/堆/层/页面（按实际项目调整）
- 对比当前场景/页面，标记哪些区域已存在、哪些缺失
- 缺失的区域必须补到功能清单并告知用户，不得跳过

**第二步 — 按操作类型枚举：**
- 每个已存在和待创建的区域，按操作类型列出：左键、右键、悬停、长按、拖拽、快捷键
- 功能和视觉分开建条目（如"悬停放大"是视觉，"悬停查看详情"是功能）
- 对比现有功能清单，发现缺失项立即补充并告知用户

**此规则贯穿整个开发周期，不可忽略。**


