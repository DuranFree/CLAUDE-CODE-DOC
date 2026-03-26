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

**Step 1 — 规划（写代码前）**
- Confirm with user what interface changes are needed
- Confirm which behaviors to test (prioritize)
- Identify opportunities for deep modules (small interface, deep implementation)
- Design interfaces for testability
- Get user approval on the plan

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
```typescript
// Testable
function processOrder(order, paymentGateway) {}

// Hard to test
function processOrder(order) {
  const gateway = new StripeGateway(); // ← bad
}
```

2. **Return results, don't produce side effects**
```typescript
// Testable
function calculateDiscount(cart): Discount {}

// Hard to test
function applyDiscount(cart): void {
  cart.total -= discount; // ← bad
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

---

## 深模块原则

**Deep module** = small interface + lots of implementation (preferred)
**Shallow module** = large interface + little implementation (avoid)

Ask: Can I reduce methods? Simplify params? Hide more complexity inside?

---

## 好测试 vs 坏测试

```typescript
// GOOD
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});

// BAD — tests implementation
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
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

所有 Phase 完成、项目跑起来后，读取 `05-architecture.md`。
