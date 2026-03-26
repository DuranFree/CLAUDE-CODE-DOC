# Port / Migration Development — Skill Index

## 使用方式
开始项目时告诉 Claude Code：
> "请读取 port/00-INDEX.md 和 PROJECT-KICKOFF-port.md 然后开始"

---

## 执行顺序

每完成一个阶段才读取下一个文件，不要一次性读取所有文件。

| 阶段 | 文件 | 内容 |
|---|---|---|
| 开始前 | `01-start.md` | 全局规则（版本控制、开发日志）— 立即读取 |
| Phase 1 | `02-requirements.md` | Grill Me + 功能清单 + 深度扫描 |
| Phase 2 | `03-visual.md` | 视觉分析 + 视觉升级指南 |
| Phase 3 | `04-plan.md` | 移植计划拆分 |
| Phase 4 | `05-testing.md` | 行为验证测试（每个系统移植时使用）|
| Phase 5 | `06-architecture.md` | 架构优化（移植全部完成后）|

---

## 核心原则

The original codebase defines the behavioral standard. The visual and technical implementation should be rebuilt using the target platform's native strengths — not translated 1:1 from the source.

---

## 立即执行

1. 读取 `01-start.md`
2. 读取 `02-requirements.md` 开始 Phase 1
