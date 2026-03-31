# From-Scratch — Phase 2 & 3: PRD + 计划

## Phase 2: 写 PRD

Summarize the Grill Me session into a structured PRD document. Include:

- Goal and background
- User stories (numbered)
- Functional requirements
- Out of scope
- Open questions (if any remain)

If the requirements include any visual descriptions, generate a visual checklist before writing the PRD and present it to the user:

> "以下是根据需求整理的视觉清单，请确认没有遗漏，确认后才开始开发。"

```
视觉清单：
- [ ] 界面布局（每个界面的元素位置和层级）
- [ ] 颜色（背景、文字、边框、高亮等所有颜色）
- [ ] 字体（字体类型、大小、粗细、间距）
- [ ] 动画（角色、UI、场景动画）
- [ ] 特效（粒子、光效、技能特效等）
- [ ] 动效（缓入缓出、弹跳、震屏等运动感）
- [ ] 过渡效果（场景切换、界面淡入淡出等）
- [ ] UI元素样式（按钮、面板、图标的视觉风格）
- [ ] 音效触发点（哪些交互或事件会触发音效）
- [ ] 其他视觉表现（任何未归类的视觉细节）
```

If no visual descriptions exist in the requirements, skip this checklist.

**Once confirmed, the visual checklist becomes part of the execution contract:**
- Work through each visual item during development
- Mark each item as complete (✅) when done
- If an item cannot be completed, notify the user with the reason — never skip silently

Save PRD as `./plans/prd-<feature-name>.md`.

---

## Phase 3: 计划拆分

Break the PRD into a phased implementation plan using vertical slices (tracer bullets).

### 3.1 探索代码库

If a codebase already exists, explore it to understand current architecture, existing patterns, and integration layers.

### 3.2 识别耐久架构决策

Before slicing, identify high-level decisions unlikely to change:
- Core system boundaries (which systems exist and how they communicate)
- Key data models and their relationships
- Platform-specific constraints (target device, performance limits)
- Third-party service or plugin boundaries
- Save/load data format and storage approach

### 3.3 垂直切片

Each phase is a thin vertical slice cutting through ALL integration layers end-to-end.

**规则：**
- **每个 Phase 必须是一个可玩的最小 demo** — 玩家能实际操作，不能只是后台逻辑
- 从最小可玩状态开始（哪怕只能打一张牌），每个 Phase 逐步扩展功能
- 不要先做所有后台再做 UI，每个切片都必须包含足够的 UI 让玩家能操作
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice must be playable, not just testable
- Prefer many thin slices over few thick ones
- Do NOT include specific file names or function names likely to change
- DO include durable decisions: route paths, schema shapes, data model names

### 3.4 与用户确认

Present the breakdown as a numbered list. For each phase show:
- **Title**: short descriptive name
- **User stories covered**: which user stories from the PRD this addresses

Ask: Does the granularity feel right? Should any phases be merged or split?

Iterate until the user approves.

### 3.4.1 功能 × Phase 对照表

规划完成后，必须生成完整的功能对照表，确保没有功能黑洞：

```
| 功能 | Phase | 状态 | 备注 |
|------|-------|------|------|
| 功能A | Phase 1 | 待做 | |
| 功能B | Phase 2 | 待做 | |
| 功能C | OUT OF SCOPE | - | 原因 |
```

**规则：**
- 功能清单里的每一条必须分配到具体 Phase 或标注为 OUT OF SCOPE
- 不允许"后续再做"的模糊表述
- 每个 Phase 必须明确声明覆盖哪些功能，剩余哪些留后续
- 生成对照表后询问用户：是否有遗漏的功能？

### 3.4.2 Phase 依赖门

功能对照表生成后，必须检查 Phase 之间的依赖关系：

**规则：**
- 如果某个 Phase 的工作对象（区域/组件/系统）在功能清单中还是 `[ ]`，该 Phase 不能启动
- 必须先完成前置项，或插入一个前置 Phase
- 典型例子：增强型 Phase（视觉打磨/动画/特效/优化）不能在基础型 Phase（布局/结构/骨架）之前启动
- 检查方式：遍历每个 Phase 涉及的功能清单条目，确认其前置条目全部已分配到更早的 Phase

如果发现依赖缺失，立即告知用户并调整 Phase 顺序。

### 3.4.3 Pre-mortem（事前风险分析）

每个 Phase 规划完成后，在开始写代码之前，必须执行以下分析：

> "假设这个 Phase 失败了，最可能是哪一步出的问题？有没有更简单的替代方案？"

输出格式：
```
Phase X Pre-mortem：
- 最高风险点：[具体描述]
- 次要风险点：[具体描述]
- 建议替代方案：[如果有的话]
- 结论：按原计划 / 建议调整为 [方案]
```

用户确认后才开始写代码。

### 3.5 写计划文件

Create `./plans/<feature-name>.md`:

```
# Plan: <Feature Name>

> Source PRD: <link or filename>

## Architectural decisions
- **Routes**: ...
- **Schema**: ...
- **Key models**: ...

---

## Phase 1: <Title>
**User stories**: ...
### What to build
...
### Acceptance criteria
- [ ] ...

## Phase 2: <Title>
...
```

---

## Roadmap 修改校验规则

**⚠️ 此规则不只在初次生成时生效，在任何追加或修改 roadmap 时同样强制执行：**

任何时候新增、修改、追加 roadmap 内容，必须重新读取本文件（`03-prd-plan.md`），逐条对照检查：
- 所有规划规则是否还在被遵守？
- Phase 顺序和依赖门是否正确？
- 有没有功能黑洞？
- 最后两个 Phase 是否符合模板？

不得依赖记忆，必须读文件后才能改 roadmap。

---

## 计划模板强制规则

生成计划文件时，**最后两个 Phase 必须固定为**：

```
## Phase N-1: Tech-Debt Cleanup
**内容**: 清理所有功能开发期间积累的 Medium/Low 技术债（来自 tech-debt.md）
**前置条件**: 所有功能 Phase 完成，测试全绿

## Phase N: 架构优化
**内容**: 模块解耦、接口重构、测试覆盖率提升
**前置条件**: Tech-Debt Cleanup 完成
```

这两个 Phase 不需要用户要求，规划时自动加入。

---

## Phase 2 & 3 完成后

写入开发日志，检查 git 状态，然后读取 `04-tdd.md`。
