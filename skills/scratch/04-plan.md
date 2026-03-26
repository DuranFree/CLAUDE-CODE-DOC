# Port / Migration — Phase 3: 移植计划

## 3.1 探索原版逻辑代码

Read the original code for game/app logic. Understand:
- Core systems and how they interact
- Data flow between systems
- Any non-obvious coupling or shared state
- Which systems are independent vs tightly coupled

**Note**: Visual systems will be rebuilt per the Visual Upgrade Guide from Phase 2, not ported from source. Logic systems are ported faithfully.

---

## 3.2 垂直切片

Each phase ports ONE complete system end-to-end.

**规则：**
- Each slice produces a working, testable piece of the ported system
- A completed slice can be verified against original behavior on its own
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
