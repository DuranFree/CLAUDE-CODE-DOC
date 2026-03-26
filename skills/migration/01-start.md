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

读取完毕后，立即读取 `02-requirements.md` 开始 Phase 1。
