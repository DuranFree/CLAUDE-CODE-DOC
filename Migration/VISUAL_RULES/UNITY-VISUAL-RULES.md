# Unity Visual Design Rules

> 使用方式：在做任何视觉相关工作时，将此文件与技能文件一起给 Claude Code。
> Target: Unity 2022.x+ / URP

---

## 设计思维

在写任何代码之前，先确定视觉方向：

- **风格定位**：选择一个明确的美术风格并坚持执行。例如：暗黑奇幻、赛博朋克、水墨国风、极简现代、像素复古、豪华卡牌等。风格要鲜明，不要模糊。
- **记忆点**：这个界面让玩家记住的是什么？一个特别的光效？一种独特的卡牌质感？确定一个核心视觉亮点。
- **禁止通用 AI 美术**：不要使用毫无个性的默认效果——普通白色 UI、平淡的颜色、没有过渡的直接切换。每一个效果都要有设计感。

---

## 视觉效果实现优先级

> ⚠️ [写入CLAUDE.md] 收到任何视觉效果需求时，必须按此优先级顺序决策

**收到任何视觉效果需求（发光 / Bloom / 模糊 / 描边 / 粒子 / 后处理），按以下顺序决策：**

1. **引擎原生功能** — URP Post Processing（Bloom / Color Grading / Vignette）、Shader Graph、VFX Graph、Particle System
2. **材质/Shader 模拟** — 用 Shader Graph 写自定义效果，仍属于引擎能力范围
3. **手动代码模拟** — 用贴图叠加、UI Image、代码控制颜色模拟效果

**手动模拟是最后手段，只在引擎能力确实无法覆盖时才使用。**

禁止直接跳到手动方案，必须先确认引擎原生能否实现。

---

## 工具选择规范

### 什么时候用 Shader Graph
- 卡牌发光、边框特效、溶解效果
- 程序化纹理（六边形网格、噪点、渐变）
- 悬停高亮、选中效果
- 任何需要动态参数控制的材质效果
- **不要用普通 Sprite 或 UI Image 模拟这些效果**

### 什么时候用 VFX Graph
- 粒子特效（施法、攻击、死亡、升级）
- 环境氛围粒子（漂浮尘埃、魔法光点）
- 轨迹效果
- GPU 粒子数量超过 1000 时必须用 VFX Graph，不用 Particle System
- **不要用 Particle System 做大量粒子**

### 什么时候用 DOTween
- 卡牌出牌、翻转、攻击动画（移动到固定目标位）
- UI 元素进场/退场、弹窗、淡入淡出
- 数值跳动（伤害数字、得分）、进度条填充
- 缓入缓出、弹跳、震屏
- 多段序列动画（Sequence）

**不要用 DOTween 的场景（改用 Update/LateUpdate Lerp）：**
- 目标位置每帧都在变化（手牌扇形、悬停跟随、拖拽预览）
- `Lerp(current, target, dt * speed)` 持续追踪模式
- 用 DOTween 做持续追踪 = 每帧 kill + restart，性能差且不稳定

### 什么时候用 URP Post Processing
- 场景氛围必须开启以下效果：
  - **Bloom** — 发光效果，Intensity 0.3-0.8，不要过曝
  - **Color Adjustments** — 色彩风格化，根据美术风格调整
  - **Vignette** — 边缘暗角，增加沉浸感，强度不超过 0.4
  - **Depth of Field** — 焦外模糊，用于强调焦点元素（可选）
  - **Film Grain** — 胶片颗粒，增加质感（可选，强度要低）
- **不要用 Sprite 或 UI Image 叠加模拟这些后处理效果**

> ⚠️ **使用前提——Canvas 渲染模式必须正确：**
> - `Screen Space - Overlay`：Canvas 在后处理**之后**合成，Bloom 等效果**对 UI 完全无效**
> - `Screen Space - Camera`：Canvas 走相机管线，后处理生效 ✅（推荐）
> - `World Space`：同上，后处理生效 ✅
>
> 实现任何 UI 发光效果前，先确认 Canvas Render Mode。Overlay 模式下再好的 PostProcessVolume 配置也碰不到 UI。

### 什么时候用 UI Toolkit vs uGUI
- **UI Toolkit**：复杂的游戏界面、需要数据绑定的 UI、需要动态生成大量元素
- **uGUI**：简单的世界空间 UI、需要 3D 效果的卡牌界面
- 两者不要混用，选一个坚持到底

---

## 颜色规范

- 建立统一的颜色常量文件，所有颜色从这里引用，不要硬编码
- 主色、辅色、强调色、背景色至少各定义一个
- 发光效果的颜色要比基础色亮 2-3 倍（HDR 颜色）
- 暗色主题：背景不要纯黑，用 `#0a0a0f` 这类带色调的深色

```csharp
// 正确 — 统一管理
public static class GameColors
{
    public static readonly Color Gold = new Color(0.78f, 0.67f, 0.43f);
    public static readonly Color GoldGlow = new Color(2.4f, 2.0f, 1.3f); // HDR
    public static readonly Color Background = new Color(0.04f, 0.04f, 0.06f);
}

// 错误 — 硬编码
GetComponent<Image>().color = new Color(0.78f, 0.67f, 0.43f);
```

---

## 动画规范

**选择原则：有固定终点 → DOTween（必须有缓动）；目标持续变化 → Update/LateUpdate Lerp（线性 Lerp 反而更自然）。**

- 离散状态切换动画（进退场/出牌/翻转）必须有缓动，不允许线性
- 持续追踪型动画（跟随鼠标/手牌扇形/实时血条）不强制缓动，Lerp 反而更自然
- 卡牌出牌动画时长：0.2-0.4 秒
- UI 进场动画时长：0.3-0.5 秒
- 重要事件（胜利、失败）可以用震屏 + 慢动作强调
- 动画必须支持打断和重置，不能锁死输入

```csharp
// ✅ 有固定终点 — DOTween，有缓动
transform.DOMove(targetPos, 0.3f)
         .SetEase(Ease.OutBack)
         .SetTarget(gameObject);

// ✅ 持续追踪动态目标 — Update Lerp，线性插值自然流畅
void LateUpdate()
{
    _currentPos = Vector2.Lerp(_currentPos, _targetPos, Time.deltaTime * speed);
    rt.anchoredPosition = _currentPos;
}

// ❌ 错误 — 用 DOTween 追踪动态目标（每帧 kill/restart，性能差且不稳定）
void Update()
{
    DOTween.Kill(rt);
    rt.DOAnchorPos(_targetPos, 0.1f); // ← 每帧重启，错误
}
```

---

## 卡牌视觉规范（适用于卡牌游戏）

- 卡牌必须有悬停状态（放大 + 发光）
- 拖拽时卡牌要有倾斜效果（3D rotation）
- 可出牌的卡牌要有视觉提示（轻微上移 + 发光边框）
- 费用不足的卡牌要有视觉压制（变暗 + 饱和度降低）
- 卡牌放置到场地要有落地特效（粒子 + 震屏）

---

## 禁止行为

- ❌ 用普通 `UI Image` 模拟发光效果
- ❌ 用 `Update()` 手写固定终点插值动画（改用 DOTween）；持续追踪动态目标时 Update Lerp 是正确选择
- ❌ 颜色硬编码在组件里
- ❌ 粒子效果数量超过 1000 还用 `Particle System`
- ❌ UI 切换没有任何过渡动画
- ❌ 所有元素同一个字体大小
- ❌ 背景纯黑或纯白没有任何氛围

---

## 每次做视觉前的检查清单

```
[ ] 确定了明确的视觉风格方向
[ ] 颜色从统一的常量文件引用
[ ] 发光效果用 Shader Graph 或 URP Bloom，不用 UI Image
[ ] 动画用 DOTween（固定终点，有缓动）或 Update Lerp（动态追踪）
[ ] 粒子用 VFX Graph（大量）或 Particle System（少量）
[ ] Post Processing Volume 已配置
[ ] 有悬停、选中、禁用三种状态的视觉区分
```
