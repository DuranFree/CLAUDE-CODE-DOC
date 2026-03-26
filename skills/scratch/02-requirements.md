# From-Scratch — Phase 1: 需求挖掘

## Grill Me

Interview the user relentlessly about every aspect of the plan until reaching shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

If a question can be answered by exploring the codebase, explore the codebase instead.

**Do not proceed until all major decision branches are resolved.**

---

## 功能清单

Before closing Phase 1, generate a complete feature checklist — every user-facing interaction and functionality identified from the requirements. Present it to the user and ask:

> "以下是根据需求整理的完整功能清单，请确认没有遗漏，确认后才开始开发。"

```
功能清单：
- [ ] 功能1
- [ ] 功能2
- [ ] 交互1（例：左键点击、右键操作、拖拽等）
- [ ] ...
```

Do not proceed until the user confirms the list is complete.

**Once confirmed, this checklist becomes the execution contract:**
- Work through the checklist item by item
- Mark each item as complete (✅) when done
- If an item cannot be completed, notify the user with the reason — never skip silently
- Do not move to the next Phase until all checklist items are either completed or explicitly deferred by the user

---

## Phase 1 完成后

写入开发日志，检查 git 状态，然后读取 `03-prd-plan.md`。
