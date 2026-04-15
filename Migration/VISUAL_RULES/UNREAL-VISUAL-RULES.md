# Unreal Engine 5 Visual Design Rules

> 使用方式：在做任何视觉相关工作时，将此文件与技能文件一起给 Claude Code。
> Target: Unreal Engine 5.x / Lumen / Nanite

---

## 设计思维

在写任何代码之前，先确定视觉方向：

- **风格定位**：选择一个明确的美术风格并坚持执行。例如：暗黑奇幻、赛博朋克、水墨国风、极简现代、像素复古、豪华卡牌等。风格要鲜明，不要模糊。
- **记忆点**：这个界面让玩家记住的是什么？确定一个核心视觉亮点。
- **禁止通用 AI 美术**：不要使用毫无个性的默认效果——普通白色 UI、平淡的颜色、没有过渡的直接切换。每一个效果都要有设计感。

---

## 视觉效果实现优先级

> ⚠️ [写入CLAUDE.md] 收到任何视觉效果需求时，必须按此优先级顺序决策

**收到任何视觉效果需求（发光 / Bloom / 模糊 / 描边 / 粒子 / 后处理），按以下顺序决策：**

1. **引擎原生功能** — Post Process Volume（Bloom / Color Grading / Vignette）、Material Editor、Niagara、Lumen
2. **材质模拟** — 用 Material Editor + Emissive 通道实现发光，仍属于引擎能力范围
3. **手动代码模拟** — 用 Widget 叠加、Tick 控制颜色模拟效果

**手动模拟是最后手段，只在引擎能力确实无法覆盖时才使用。**

禁止直接跳到手动方案，必须先确认引擎原生能否实现。

---

## 工具选择规范

### 什么时候用 Material Editor
- 卡牌发光、边框特效、溶解效果
- 程序化纹理（噪点、渐变、图案）
- 悬停高亮、选中效果
- 任何需要动态参数控制的材质效果
- **不要用普通 Texture 叠加模拟这些效果**

常用 Material 节点：
- `RadialGradientExponential` — 径向渐变
- `SimpleNoise` / `GradientNoise` — 噪点纹理
- `Fresnel` — 边缘发光
- `DepthFade` — 软粒子边缘

### 什么时候用 Niagara
- 粒子特效（施法、攻击、死亡、升级）
- 环境氛围粒子（漂浮尘埃、魔法光点）
- 轨迹效果、流体模拟
- **不要用旧版 Cascade 粒子系统做新效果**
- GPU Simulation 用于大量粒子，CPU Simulation 用于少量需要精确控制的粒子

### 什么时候用 Sequencer / UMG Animations
- **Sequencer**：过场动画、复杂的多对象协同动画、摄像机运动
- **UMG Animations**：UI 进场退场、按钮交互动效、数值跳动
- **Blueprint Timeline**：简单的单属性动画、循环动画
- **不要用 Tick 事件手写插值动画**

```cpp
// 正确 — 用 Timeline 或 UMG Animation
UFUNCTION()
void PlayCardAnimation();  // 触发预设的 UMG Animation

// 错误 — Tick 手写插值
void Tick(float DeltaTime) {
    CardWidget->SetRenderTranslation(
        FMath::Lerp(CurrentPos, TargetPos, DeltaTime)  // ← bad
    );
}
```

### 什么时候用 Post Process Volume
- 场景氛围必须配置以下效果：
  - **Bloom** — 发光效果，不要过曝
  - **Color Grading** — 色彩风格化，用 LUT 或手动调整
  - **Vignette** — 边缘暗角，增加沉浸感，强度不超过 0.4
  - **Depth of Field** — 焦外模糊，用于强调焦点元素（可选）
  - **Film Grain** — 胶片颗粒，增加质感（可选，强度要低）
  - **Chromatic Aberration** — 色差，增加镜头感（可选，强度极低）
- **不要用 Widget 叠加模拟这些后处理效果**

> ⚠️ **使用前提——Screen Space UMG 不受 Post Process Volume 影响：**
> - `Screen Space` UMG（默认模式）在后处理**之后**合成到屏幕，Bloom 等效果**对 UI 完全无效**
> - `World Space` Widget：作为场景对象渲染，后处理生效 ✅
> - 需要 Bloom 的 UI 发光效果，两种方案：
>   1. 改用 `World Space` Widget Component 放入场景
>   2. 在材质的 `Emissive Color` 通道输出高于 1.0 的值，触发 Bloom（适合场景内物体）
>
> 实现任何 UI 发光效果前，先确认 Widget 的渲染模式。Screen Space 模式下 PostProcessVolume 配置再完整也对 UI 无效。

### 什么时候用 UMG vs 3D Widget
- **UMG**：所有 HUD、菜单、卡牌信息显示
- **3D Widget (Widget Component)**：世界空间里的 UI 元素、悬浮在角色头顶的信息
- 两者不要混用于同一功能

### 什么时候用 Lumen
- 需要实时全局光照的场景
- 动态光源变化效果
- 反射效果
- **移动端或低配 PC 项目关闭 Lumen，改用 Screen Space Reflections**

---

## 颜色规范

- 建立统一的颜色常量，所有颜色从这里引用，不要硬编码
- 主色、辅色、强调色、背景色至少各定义一个
- 发光颜色使用 HDR 值（亮度超过 1.0）配合 Bloom
- 暗色主题：背景不要纯黑，用带色调的深色

```cpp
// 正确 — 统一管理
namespace GameColors
{
    const FLinearColor Gold = FLinearColor(0.78f, 0.67f, 0.43f);
    const FLinearColor GoldGlow = FLinearColor(2.4f, 2.0f, 1.3f); // HDR
    const FLinearColor Background = FLinearColor(0.04f, 0.04f, 0.06f);
}

// 错误 — 硬编码
CardWidget->SetColorAndOpacity(FLinearColor(0.78f, 0.67f, 0.43f));
```

---

## 动画规范

- 离散状态切换动画（进退场/出牌/翻转）必须有缓动，不允许线性
- 持续追踪型动画（跟随鼠标/手牌扇形/实时血条）不强制缓动，Tick Lerp 反而更自然
- 卡牌出牌动画时长：0.2-0.4 秒
- UI 进场动画时长：0.3-0.5 秒
- 重要事件（胜利、失败）可以用震屏 + 慢动作强调（`SetGlobalTimeDilation`）
- 动画必须支持打断和重置，不能锁死输入
- UMG Animation 用 `PlayAnimation` / `StopAnimation` 控制，不要用 `SetVisibility` 直接切换
- 不要用 Tick 手写固定时长插值循环；持续追踪目标时 Tick Lerp（`FMath::VInterpTo`）是正确选择

---

## 卡牌视觉规范（适用于卡牌游戏）

- 卡牌必须有悬停状态（放大 + 发光）
- 拖拽时卡牌要有倾斜效果（RenderTransform Rotation）
- 可出牌的卡牌要有视觉提示（轻微上移 + 发光边框）
- 费用不足的卡牌要有视觉压制（ColorAndOpacity 变暗）
- 卡牌放置到场地要有落地特效（Niagara + 震屏）

---

## Blueprint vs C++ 规范

- **逻辑密集、性能敏感** → 用 C++
- **视觉效果、UI 交互、关卡设计** → 用 Blueprint
- 核心游戏系统用 C++ 写，暴露 `UFUNCTION(BlueprintCallable)` 给 Blueprint 调用
- **不要在 Blueprint 里写复杂逻辑**，Blueprint 只负责调用和视觉

---

## 禁止行为

- ❌ 用普通 `Image Widget` 模拟发光效果
- ❌ 用 `Tick` 手写插值动画
- ❌ 颜色硬编码在 Widget 属性里
- ❌ 用旧版 Cascade 粒子做新特效
- ❌ UI 切换没有任何过渡动画
- ❌ 所有元素同一个字体大小
- ❌ 背景纯黑或纯白没有任何氛围
- ❌ Post Process Volume 未配置就上线

---

## 每次做视觉前的检查清单

```
[ ] 确定了明确的视觉风格方向
[ ] 颜色从统一的常量引用
[ ] 发光效果用 Material Fresnel 或 Post Process Bloom
[ ] 动画用 UMG Animation 或 Blueprint Timeline，有缓动
[ ] 粒子用 Niagara，不用旧版 Cascade
[ ] Post Process Volume 已配置氛围效果
[ ] 核心逻辑在 C++，视觉交互在 Blueprint
[ ] 有悬停、选中、禁用三种状态的视觉区分
```
