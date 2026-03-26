# From-Scratch — Phase 5: 架构优化

项目完成并运行后才进入此阶段。

## 何时使用

- Code is getting hard to maintain
- Tests are hard to write
- Changing one thing breaks many others
- AI needs to bounce between many files to understand one concept

---

## 流程

**Step 1 — 有机探索代码库**

Note where you experience friction:
- Where does understanding one concept require bouncing between many small files?
- Where are modules so shallow that the interface is nearly as complex as the implementation?
- Where do tightly-coupled modules create integration risk?
- Which parts are untested or hard to test?

**Step 2 — 列出候选模块**

Show a numbered list. For each:
- **Cluster**: which modules/concepts are involved
- **Why they're coupled**: shared types, call patterns, co-ownership
- **Dependency category**: in-process / local-substitutable / remote-owned / true external
- **Test impact**: what existing tests would be replaced

Ask the user which to explore. Do NOT propose interfaces yet.

**Step 3 — 并行设计多个接口方案**

Spawn 3+ parallel sub-agents with different constraints:
- Agent 1: "Minimize interface — aim for 1-3 entry points max"
- Agent 2: "Maximize flexibility — support many use cases"
- Agent 3: "Optimize for the most common caller — make default case trivial"
- Agent 4 (if applicable): "Ports & adapters pattern for cross-boundary dependencies"

Each sub-agent outputs: interface signature, usage example, what complexity it hides, dependency strategy, trade-offs.

Compare in prose. Give a strong recommendation — be opinionated.

**Step 4 — 创建 GitHub Issue RFC（如已关联 GitHub）**

```
## Problem
- Which modules are shallow and tightly coupled
- What integration risk exists in the seams
- Why this makes the codebase harder to maintain

## Proposed Interface
- Interface signature (types, methods, params)
- Usage example
- What complexity it hides internally

## Dependency Strategy
- In-process / Local-substitutable / Ports & adapters / Mock

## Testing Strategy
- New boundary tests to write
- Old tests to delete
- Test environment needs

## Implementation Recommendations
- What the module should own
- What it should hide
- What it should expose
- How callers should migrate
```

---

## 依赖分类

- **In-process**: pure computation, no I/O — always deepenable, merge and test directly
- **Local-substitutable**: has local test stand-ins — deepenable with stand-in
- **Remote but owned**: own services across network — define port interface, inject transport, use in-memory adapter in tests
- **True external**: third-party (Stripe, Twilio) — mock at the boundary only

**Testing principle: replace, don't layer.**
- Delete old unit tests on shallow modules once boundary tests exist
- Write new tests at the deepened module's interface boundary
- Tests assert on observable outcomes, not internal state
