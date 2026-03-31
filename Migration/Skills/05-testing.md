# Port / Migration — Phase 4: 行为验证测试

每移植一个系统时使用此文件。每个系统完成后测试全绿才继续下一个。

---

## 核心原则

This is NOT traditional TDD. The original codebase IS the specification for **logic**. Visual appearance is governed by the Visual Upgrade Guide.

**The test standard is: original input A produces output B → ported version must also produce output B.**

**⚠️ 代码是唯一可信来源：**
- 文档（包括 feature-checklist.md、phase-roadmap.md）可能过时或有错
- 每次规划和实现新 Phase 前，必须先读原版代码对应部分，再对比文档
- 发现文档和代码不一致时，以代码为准，立即修正文档
- 不得用文档验证文档，只能用代码验证文档

---

## 与 TDD 的区别

| TDD | 行为验证（移植）|
|---|---|
| Tests drive interface design | Original code defines the interface |
| Test first, then implement | Read original → port → write verification test |
| Tests define desired behavior | Tests confirm behavior matches original |
| Interface can be redesigned | Interface mirrors original intent |

---

## 每个系统的工作流

**Step 0 — 场景完整性审计 + 玩家操作清单对比（每个 Phase 必做）**

每个 Phase 开始前，无论是逻辑、UI 还是视觉 Phase，都必须执行：
1. 读取 feature-checklist.md 中的布局/区域相关条目，列出所有未完成项
2. 如果本 Phase 的工作对象依赖这些未完成的区域/组件，必须先完成前置项或告知用户缺失
3. 如果本 Phase 涉及 UI 或交互，额外执行 `01-start.md` 中的"玩家操作清单对比规则"

**Step 1 — 读原版**

Before porting anything, read the original code for the current system:
- Understand the public interface (methods, inputs, outputs)
- Understand the observable behavior (what does it produce/change?)
- Note any edge cases or non-obvious logic

**Step 2 — 移植**

Port the original logic to the target framework. Use the same logic, same data flow. Do not redesign.

For visual components: implement per the Visual Upgrade Guide, not from the original source.

**Step 3 — 写行为验证测试**

Write tests that assert the ported version produces the same observable outcomes as the original. Use whatever test syntax matches the current engine/language:

```
// Example (pseudocode — adapt to current engine/language)
// Original produced: score = base * multiplier
// Port must produce the same result

test "score calculation matches original behavior":
  result = calculateScore(baseScore, multiplier)
  assert result == expectedOutput
```

**Step 4 — 立即跑测试，修复**

Run the test suite right now — do not wait until all phases are done.

**This is mandatory. Do not proceed to the next system until all tests for the current system are green.**

**Step 5 — 进入下一个系统**

Only move on after all current system tests pass.

---

## 测试设计规则

1. **测试可观察行为，不测实现**

好测试的特征：
- 测试用户/调用者关心的行为
- 只使用公开接口
- 重构后仍然有效
- 描述 WHAT，不描述 HOW

```
// GOOD — 测试可观察行为（用当前引擎/语言的语法）
test "unit dies when damage exceeds HP":
  unit = createUnit(hp: 10)
  applyDamage(unit, 15)
  assert unit.isDead == true

// BAD — 测试实现细节
test "applyDamage calls unit.reduceHP":
  assert mockUnit.reduceHP was called  // ← 不测行为，测实现

// BAD — 绕过接口验证
test "createUnit saves to database":
  createUnit(data)
  row = db.query("SELECT * FROM units")
  assert row exists  // ← 绕过接口

// GOOD — 通过接口验证
test "createUnit makes unit retrievable":
  unit = createUnit(data)
  retrieved = getUnit(unit.id)
  assert retrieved.name == data.name
```

2. **用原版作为参照，不要靠假设**

If unsure what the expected output should be, read the original code — don't guess.

3. **追踪原版的所有防御性检查**

移植时必须追踪原版代码里所有的防御性检查（null check、边界判断、异常处理），确保每一个都在移植版本里有对应的测试覆盖。原版有的防御性检查，移植版必须保留并测试，不得遗漏。

4. **边界条件测试**

每个功能必须包含边界条件测试：无合法目标、空列表、零值、极限值等异常路径，不只测正常流程。

5. **在系统边界测试**

Test the same interface the original exposed, not internal helpers.

---

## Mock 规则

Mock at **system boundaries only**:
- External services (network, payment, etc.)
- Platform-specific APIs that don't exist in the test environment
- Time / randomness

**Do NOT mock:**
- Ported game logic
- Internal systems you are porting
- Anything whose behavior you are trying to verify

---

## 处理原版 Bug

If the original code has a bug:
- **Do not silently fix it** — flag it to the user
- Ask: "Original code has [behavior] which appears to be a bug. Port as-is or fix?"
- Default to porting as-is unless user confirms a fix

---

## 跑测试

**Testing is NON-NEGOTIABLE. Never skip, even if user is away. If no framework available, write a lightweight TestRunner.**

**Before writing any tests**, check if the standard test framework is installed:
- If not installed → automatically install, inform user
- If installation fails and user is present → ask:
  1. 重试安装
  2. 跳过，使用替代方案（自己写 TestRunner）
  3. 取消
- If installation fails and user is away → automatically use lightweight TestRunner, log in dev-log.md

**⚠️ 如果当前在使用 TestRunner 替代方案：**
- 每个 Phase 开始前**主动尝试安装标准测试框架**
- 安装成功 → 立即将所有现有测试迁移到标准框架，通知用户
- 安装失败 → 继续用 TestRunner，下个 Phase 再试
- 不要等用户提醒，主动找机会切换回标准框架

Always announce status:
- `⚙️ [测试框架] 未找到标准测试框架，正在安装 <框架名称>...`
- `✅ [测试框架] <框架名称> 已安装，启用标准测试框架。`
- `✅ [测试框架] <框架名称> 安装成功，启用标准测试框架。`
- `❌ [测试框架] <框架名称> 安装失败，启用自定义 TestRunner 替代方案。`

**两层测试策略：**

**Layer 1 — 逻辑测试（每个 RED→GREEN 后立即跑）**
- Pure computation, state machines, rules, AI behavior, numerical calculations
- No engine required — fast, seconds
- `🟢 [逻辑测试] 运行轻量级逻辑测试，无需引擎...`

**Layer 2 — 引擎测试（每个系统完成后跑）**
- Rendering, physics, input, platform-specific APIs
- Requires headless/batchmode — slower
- `🔵 [引擎测试] 启动引擎无头模式跑测试，请稍候...`

引擎无头模式示例（根据当前引擎自行判断）：
- Unity: `-batchmode -runTests -nographics`
- Godot: `--headless -s res://tests/run_tests.gd`
- Unreal: `-nullrhi -unattended`
- Web: runs headless by default

跑完报告结果：
- `✅ [逻辑测试] 全部通过 (X/X)`
- `✅ [引擎测试] 全部通过 (X/X)`
- `❌ [逻辑测试] X 个失败，开始修复...`
- `❌ [引擎测试] X 个失败，开始修复...`

跑测试前先 kill 冲突进程：
- Windows: `taskkill /F /IM <process>.exe`
- Mac/Linux: `pkill -f <process>`

---

## 每个系统完成后检查清单

```
[ ] Original code for this system has been read and understood
[ ] Ported logic matches original intent
[ ] Visual components implemented per Visual Upgrade Guide
[ ] Verification tests assert observable outcomes, not implementation
[ ] Tests use the same inputs the original used
[ ] All tests pass before moving to next system
[ ] Any intentional deviations are logged and user-confirmed
```

---

## 所有系统移植完成后

写入开发日志，检查 git 状态，然后读取 `05b-tech-debt.md`。
