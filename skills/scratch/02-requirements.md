# Port / Migration — Phase 1: 需求挖掘 + 深度扫描

## Grill Me

Interview the user relentlessly about every aspect of the migration until reaching shared understanding. Walk down each branch of the decision tree, resolving dependencies one-by-one. For each question, provide your recommended answer.

Key questions to resolve:
- Where is the original codebase? (get the exact path)
- Where are the original art/asset files?
- What is the target framework/platform?
- Are there any intentional behavior changes, or is this a pure 1:1 port?
- Are there any parts of the original that are known to be buggy — should those bugs be ported or fixed?
- What is the priority order for porting (core systems first, UI last, etc.)?

If a question can be answered by exploring the codebase, explore the codebase instead.

**Do not proceed until scope is clear and original paths are confirmed.**

---

## 深度扫描

Before generating the feature checklist, perform two deep scans of the original codebase:

**深度扫描 1 — 硬编码数值和隐藏游戏规则：**
Scan all code for hardcoded numerical values and boundary conditions hidden inside conditional logic (e.g. `if hp > 30`, `damage * 1.5`, `score >= 100`). These are game balance parameters and rules that are not documented anywhere — only in the code. List them all so they are not missed during porting.

**深度扫描 2 — 配置和数据文件：**
Scan for any configuration or data files outside the main codebase (JSON, CSV, XML, INI, YAML, or platform-specific formats) that contain game data such as unit stats, level configs, card definitions, or balance values. These must all be ported or recreated in the target platform.

---

## 功能清单

Generate a complete feature checklist — every user-facing interaction and functionality found in the original. Present to the user:

> "以下是从原版代码中整理的完整功能清单，请确认没有遗漏，确认后才开始移植。"

```
功能清单：
- [ ] 功能1
- [ ] 功能2
- [ ] 交互1（例：左键点击、右键操作、拖拽等）
- [ ] ...

深度扫描发现：
- [ ] 硬编码数值：<列出所有发现>
- [ ] 配置文件：<列出所有发现>
```

Do not proceed until the user confirms the list is complete.

**Once confirmed, this checklist becomes the execution contract:**
- Work through the checklist item by item
- Mark each item as complete (✅) when done
- If an item cannot be completed, notify the user with the reason — never skip silently
- Do not move to the next Phase until all checklist items are either completed or explicitly deferred by the user

---

## Phase 1 完成后

写入开发日志，检查 git 状态，然后读取 `03-visual.md`。
