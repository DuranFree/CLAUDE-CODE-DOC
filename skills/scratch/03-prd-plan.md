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
- Route structures / URL patterns
- Database schema shape
- Key data models
- Authentication / authorization approach
- Third-party service boundaries

### 3.3 垂直切片

Each phase is a thin vertical slice cutting through ALL integration layers end-to-end.

**规则：**
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names or function names likely to change
- DO include durable decisions: route paths, schema shapes, data model names

### 3.4 与用户确认

Present the breakdown as a numbered list. For each phase show:
- **Title**: short descriptive name
- **User stories covered**: which user stories from the PRD this addresses

Ask: Does the granularity feel right? Should any phases be merged or split?

Iterate until the user approves.

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

## Phase 2 & 3 完成后

写入开发日志，检查 git 状态，然后读取 `04-tdd.md`。
