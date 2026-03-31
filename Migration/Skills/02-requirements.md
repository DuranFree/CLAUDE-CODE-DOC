# Port / Migration — Phase 1: 需求挖掘 + 深度扫描

## Grill Me

**Do not ask the questions from the KICKOFF file — those are already answered. Instead:**

1. Read all provided files and the original codebase thoroughly
2. Then interview the user **relentlessly** — walk down every branch of the decision tree, one question at a time, resolving dependencies between decisions one-by-one
3. For each question, provide your own recommended answer so the user can agree or redirect
4. **If a question can be answered by exploring the codebase, explore the codebase instead of asking**
5. Do NOT move to the next question until the current one is fully resolved
6. Do NOT stop until every decision, edge case, and ambiguity is resolved — even if it takes many questions

The goal is to surface what the user hasn't thought of yet. Be thorough, not quick.

**Do not proceed to the deep scans or feature checklist until all decision branches are fully resolved.**

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

**深度扫描 3 — 用户交互流程链：**
Do NOT only scan by feature module. Also trace complete user interaction flows end-to-end — from the user's first action to the final result, crossing file boundaries. A single user-facing feature often spans multiple files and will be missed if each file is only read in isolation.

For each interaction flow found, document the complete chain, for example:
```
悬停手牌 → 识别费用缺口 → 高亮符文 → 拖牌 → 自动补足 → 出牌
```

Any flow that crosses 2+ files must be explicitly listed in the feature checklist as a single item, not split across multiple items.

**深度扫描 4 — 美术资源索引：**
使用 KICKOFF 文档中填写的「美术资源路径」进行扫描（该路径已由用户在项目启动时确认），生成 `plans/assets-index.json`：
```json
{
  "generated": "<date>",
  "platform": "<target platform>",
  "source_asset_path": "<KICKOFF 中填写的美术资源路径>",
  "assets": [
    { "name": "filename", "path": "相对于美术资源路径的相对路径", "type": "Texture|Audio|Font|Other", "ported": false }
  ]
}
```
- `ported: false` 表示尚未迁移到目标工程，迁移后更新为 `true`
- 此文件 Phase 1 生成，后续只在资源迁移时更新 `ported` 字段，不重新扫描
- **后续所有 Phase 需要引用美术资源时，必须先查询此文件，不得重新遍历目录**

**⚠️ 数据字段验证规则：**
数据字段存在 ≠ 游戏机制存在。发现任何数据字段时，必须追踪该字段在代码中实际被使用的地方，确认其真实用途，不得仅凭字段名或字段值推断游戏机制。

例如：发现 `hp: 14` 字段，不能直接推断"该单位有独立HP系统"，必须搜索所有使用该字段的代码，确认它是战斗血量还是内部计算用的数值。

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
