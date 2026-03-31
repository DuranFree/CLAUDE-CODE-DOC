# From-Scratch — Phase 1: 需求挖掘

## Grill Me

**Do not ask the questions from the KICKOFF file — those are already answered. Instead:**

1. Read all provided files and the codebase thoroughly
2. Then interview the user **relentlessly** — walk down every branch of the decision tree, one question at a time, resolving dependencies between decisions one-by-one
3. For each question, provide your own recommended answer so the user can agree or redirect
4. **If a question can be answered by exploring the codebase, explore the codebase instead of asking**
5. Do NOT move to the next question until the current one is fully resolved
6. Do NOT stop until every decision, edge case, and ambiguity is resolved — even if it takes many questions

The goal is to surface what the user hasn't thought of yet. Be thorough, not quick.

**Do not proceed to the feature checklist until all decision branches are fully resolved.**

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
