# From-Scratch — Phase 4: TDD 实现

## 核心原则

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests should not.

**Good tests** exercise real code paths through public APIs. They describe *what* the system does, not *how*.

**Bad tests** are coupled to implementation — they mock internal collaborators, test private methods, or break when you refactor without changing behavior.

---

## 反模式：横向切片

**DO NOT write all tests first, then all implementation.**

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
```

---

## 工作流

**Step 0 — 场景完整性审计 + 玩家操作清单对比（每个 Phase 必做）**

每个 Phase 开始前，无论是逻辑、UI 还是视觉 Phase，都必须执行：
1. 读取 feature-checklist.md 中的布局/区域相关条目，列出所有未完成项
2. 如果本 Phase 的工作对象依赖这些未完成的区域/组件，必须先完成前置项或告知用户缺失
3. 如果本 Phase 涉及 UI 或交互，额外执行 `01-start.md` 中的"玩家操作清单对比规则"

**Step 1 — 规划（写任何代码之前）**

- [ ] 确认需要什么接口改动
- [ ] 确认哪些行为要测试（优先级排序）
- [ ] 识别深模块机会（小接口，大实现）
- [ ] 为可测试性设计接口
- [ ] 列出要测试的行为（不是实现步骤）
- [ ] 获取用户批准

问用户："公开接口应该长什么样？哪些行为最重要？"

**你不可能测试所有东西。** 和用户确认哪些行为最关键，把测试精力放在核心路径和复杂逻辑上，不是每个边界情况。

**Step 2 — Tracer Bullet**
```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

**Step 3 — 增量循环**
```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
RUN:   Execute test suite immediately — do not batch
```

规则：
- One test at a time
- Only enough code to pass the current test
- **Run tests immediately after each GREEN — never accumulate**
- Don't anticipate future tests

**Step 4 — 重构**

After all tests pass:
- Duplication → Extract function/class
- Long methods → Break into private helpers
- Shallow modules → Combine or deepen
- Feature envy → Move logic to where data lives

**Never refactor while RED.**

---

## 接口设计规则

1. **Accept dependencies, don't create them**

Pass external dependencies in rather than creating them internally:

```
// Testable — dependency injected
function resolveCombat(attacker, defender, rules) {}

// Hard to test — dependency created internally
function resolveCombat(attacker, defender) {
  const rules = new GameRules(); // ← bad, can't be replaced in tests
}
```

2. **Return results, don't produce side effects**

```
// Testable — returns result
function calculateDamage(attack, defense): int {}

// Hard to test — modifies state directly
function applyDamage(unit, amount): void {
  unit.hp -= amount; // ← hard to verify without checking internal state
}
```

3. **Small surface area** — fewer methods = fewer tests needed

---

## Mock 规则

Mock at **system boundaries only**:
- External APIs (payment, email, etc.)
- Databases (prefer test DB when possible)
- Time / randomness

**Do NOT mock:**
- Your own classes/modules
- Internal collaborators
- Anything you control

**设计易于 Mock 的接口：**

用依赖注入，不要在内部创建依赖：

```
// Easy to mock — 依赖从外部传入
function resolveCombat(attacker, defender, rules) {}

// Hard to mock — 依赖在内部创建
function resolveCombat(attacker, defender) {
  const rules = new GameRules();  // ← 无法在测试中替换
}
```

**优先用 SDK 风格接口，不要用通用 fetcher：**

```
// GOOD — 每个函数独立可 mock
const gameAPI = {
  getUnit: (id) => ...,
  dealDamage: (unitId, amount) => ...,
  endTurn: () => ...,
};

// BAD — mock 时需要条件逻辑
const gameAPI = {
  call: (action, params) => ...  // ← 难以 mock
};
```

SDK 风格的好处：
- 每个 mock 只返回一种固定结构
- 测试里不需要条件逻辑
- 一眼看出测试用了哪些操作

---

## 深模块原则

**Deep module** = small interface + lots of implementation (preferred)
**Shallow module** = large interface + little implementation (avoid)

Ask: Can I reduce methods? Simplify params? Hide more complexity inside?

---

## 好测试 vs 坏测试

好测试是**集成风格**的——通过真实接口测试，而不是 mock 内部实现。

好测试的特征：
- 测试用户/调用者关心的行为
- 只使用公开 API
- 重构后仍然有效
- 描述 WHAT，不描述 HOW
- 每个测试只有一个逻辑断言
- **必须包含边界条件测试：无合法目标、空列表、零值、极限值等异常路径，不只测正常流程**

坏测试的红旗：
- Mock 内部协作者
- 测试私有方法
- 断言调用次数/顺序
- 重构但行为没变时测试却失败
- 测试名称描述 HOW 不是 WHAT

```
// GOOD — 测试可观察行为（用当前引擎/语言的语法）
test "player can complete turn with valid action":
  state = createGameState()
  result = processTurn(state, validAction)
  assert result.status == "completed"

// BAD — 测试实现细节
test "processTurn calls actionService.execute":
  assert mockActionService.execute was called  // ← 不测行为，测实现

// BAD — 绕过接口验证
test "createUnit saves to database":
  createUnit(data)
  row = db.query("SELECT * FROM units")
  assert row exists  // ← 绕过接口直接查数据库

// GOOD — 通过接口验证
test "createUnit makes unit retrievable":
  unit = createUnit(data)
  retrieved = getUnit(unit.id)
  assert retrieved.name == data.name  // ← 通过公开接口验证
```

---

## 每个循环的检查清单

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```

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

**⚠️ 如果当前在使用 TestRunner 替代方案：** 在每个 Phase 开始前检查标准测试框架是否已可用。一旦可用，立即将所有现有测试迁移到标准框架，不要继续使用 TestRunner。

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

**Layer 2 — 引擎测试（每个 Phase 完成后跑）**
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

## 每个 Phase 完成后

写入开发日志，检查 git 状态，继续下一个 Phase。

所有 Phase 完成、项目跑起来后，读取 `04b-tech-debt.md`。
