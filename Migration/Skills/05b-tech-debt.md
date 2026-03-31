# Port / Migration — Phase 5b: Tech-Debt Cleanup

功能全部完成、测试全绿之后，架构优化之前进入此阶段。

## 何时使用

- 所有功能 Phase 完成
- tech-debt.md 积累了超过 3 条 Medium/Low 条目
- 准备进入架构优化（Phase 6）之前

---

## 流程

**Step 1 — 读取 tech-debt.md**

读取项目的 `plans/tech-debt.md`（或等效文件），列出所有未解决条目。

按优先级分组：
- **High**（如果有遗漏）→ 必须立即修，不进入此 Phase
- **Medium** → 本 Phase 全部修完
- **Low** → 评估后选择性修，不影响流程

**Step 2 — 分类处理**

对每条 Medium/Low：
1. **确认是否仍然有效** — 代码可能已经自然修复了，读文件确认
2. **评估修复成本** — 1 行改动 vs 跨文件重构
3. **按成本排序，从小到大处理**

不要打包成一个大 commit，每修一类问题单独 commit。

**Step 3 — 执行修复**

每条修完后：
- 立即跑测试（用 MCP run_tests 或 batchmode）
- 测试全绿才继续下一条
- 更新 tech-debt.md：已解决的条目标记 ✅ 或删除

**Step 4 — 确认清单**

tech-debt.md 中所有 Medium 条目处理完毕后，向用户展示：
- 本 Phase 修复了哪些条目
- 哪些 Low 选择跳过及原因
- 剩余未解决条目（如果有）

**Step 5 — 进入架构优化**

tech-debt 清理完成，读取 `06-architecture.md` 进入架构优化阶段。

---

## 原则

- 不做超出 tech-debt.md 范围的"顺手改"
- 不在此阶段引入新功能或新抽象
- 测试是唯一通过标准，不依赖代码审美判断
