# From-Scratch — 全局规则

这些规则适用于整个工作流的每一个阶段，始终有效，不可忽略。

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

