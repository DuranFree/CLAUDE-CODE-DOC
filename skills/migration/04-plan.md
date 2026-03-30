# Port / Migration — Phase 3: 移植计划

## 3.1 探索原版逻辑代码

Read the original code for game/app logic. Understand:
- Core systems and how they interact
- Data flow between systems
- Any non-obvious coupling or shared state
- Which systems are independent vs tightly coupled

**Note**: Visual systems will be rebuilt per the Visual Upgrade Guide from Phase 2, not ported from source. Logic systems are ported faithfully.

## 3.1 规划前必须先读原版代码

**⚠️ 代码是唯一可信来源：**
- 文档（包括 feature-checklist.md、phase-roadmap.md）可能过时或有错
- 规划每个新 Phase 之前，必须先读原版代码对应部分，再对比文档
- 发现文档和代码不一致时，以代码为准，立即修正文档
- 不得用文档验证文档，只能用代码验证文档

---

Each phase ports ONE complete system end-to-end.

**规则：**
- **每个 Phase 必须是一个可玩的最小 demo** — 玩家能实际操作，不能只是后台逻辑
- 从最小可玩状态开始（哪怕只能打一张牌），每个 Phase 逐步扩展功能
- 不要先做所有后台再做 UI，每个切片都必须包含足够的 UI 让玩家能操作
- Each slice produces a working, playable piece of the ported system
- Prefer many thin slices over few thick ones
- **Logic**: port faithfully, behavior must match original
- **Visuals**: implement per Visual Upgrade Guide, do not translate from source

---

## 3.3 与用户确认

Present as a numbered list. For each phase show:
- **Title**: which system is being ported
- **Original files involved**: which source files are being ported
- **Visual handling**: "rebuilt per guide" or "ported directly"
- **Verifiable by**: how behavior can be confirmed against original

Ask: Does the granularity feel right? Are the dependencies in the right order?

Iterate until the user approves.

## 3.3.1 功能 × Phase 对照表

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

## 3.3.2 Phase 依赖门

功能对照表生成后，必须检查 Phase 之间的依赖关系：

**规则：**
- 如果某个 Phase 的工作对象（区域/组件/系统）在功能清单中还是 `[ ]`，该 Phase 不能启动
- 必须先完成前置项，或插入一个前置 Phase
- 典型例子：增强型 Phase（视觉打磨/动画/特效/优化）不能在基础型 Phase（布局/结构/骨架）之前启动
- 检查方式：遍历每个 Phase 涉及的功能清单条目，确认其前置条目全部已分配到更早的 Phase

如果发现依赖缺失，立即告知用户并调整 Phase 顺序。

---

## 3.3.3 Pre-mortem（事前风险分析）

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

---

## 3.4 写计划文件

Create `./plans/port-<project-name>.md`:

```
# Port Plan: <Project Name>

> Original codebase: <path>
> Target framework: <framework>
> Asset path: <path>
> Visual Upgrade Guide: ./plans/visual-upgrade-<project-name>.md

## Architectural decisions
- **Intentional behavior changes**: <list or "none — pure 1:1 port">
- **Known bugs to fix**: <list or "none">
- **Asset strategy**: direct copy / conversion required
- **Visual strategy**: rebuilt using native platform capabilities per Visual Upgrade Guide

---

## Phase 1: <System Name>
**Original files**: ...
**What to port**: ...
**Visual handling**: rebuilt per guide / ported directly
**Acceptance criteria**:
- [ ] Behavior matches original: <specific behavior>
- [ ] Behavior matches original: <specific behavior>
- [ ] Tests written and all passing before moving to next phase

## Phase 2: <System Name>
...
```

---

## Phase 3 完成后

写入开发日志，检查 git 状态，然后读取 `05-testing.md` 开始移植第一个系统。
