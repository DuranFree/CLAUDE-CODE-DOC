# Godot Visual Design Rules

> 使用方式：在做任何视觉相关工作时，将此文件与技能文件一起给 Claude Code。
> Target: Godot 4.x

---

## 设计思维

在写任何代码之前，先确定视觉方向：

- **风格定位**：选择一个明确的美术风格并坚持执行。例如：暗黑奇幻、赛博朋克、水墨国风、极简现代、像素复古、豪华卡牌等。风格要鲜明，不要模糊。
- **记忆点**：这个界面让玩家记住的是什么？确定一个核心视觉亮点。
- **禁止通用 AI 美术**：不要使用毫无个性的默认效果——普通白色 UI、平淡的颜色、没有过渡的直接切换。每一个效果都要有设计感。

---

## 设计稿还原工作流

> ⚠️ [写入memory] 首次读取本规则后，追加到项目 memory 索引，作为"还原设计稿类任务"的触发器

收到"还原设计稿 / 按 XX 稿做 UI"类任务时，禁止用"视觉一致""基本还原"这类模糊标准交付。过去反复踩坑：只做大框，漏掉 chip、标签、副标题、效果层等细节，交付时说"完成了"，用户验收才发现缺件。

**强制四步工作流：**

1. **开始前枚举**：列出设计稿里所有可视子元素（框架 / 背景 / 标签 / chip / 图标 / 副标题 / 效果层…），逐个编号 N1–Nx 写成清单告诉用户
2. **实现时标注**：每完成一段代码，明确标注「实现 N2、N5」
3. **完成后对账**（强制）：截图对比设计稿，逐个编号核对 ✅ / ❌ / ⚠️，缺失项必须显式列出
4. **所有编号 ✅ 才能说「完成」**；做了 7/10 不等于完成

**禁用表述：** "基本还原""视觉一致""大致对齐"——这些是偷懒借口，不得作为交付结论。

---

## 视觉效果实现优先级

> ⚠️ [写入memory] 首次读取本规则后，追加到项目 memory 索引，作为"收到视觉需求时的决策流程"触发器

**收到任何视觉效果需求（发光 / Glow / 模糊 / 描边 / 粒子 / 后处理），按以下顺序决策：**

1. **引擎原生功能** — WorldEnvironment（Glow / DOF / Vignette）、ShaderMaterial、VisualShader、GPUParticles
2. **材质/Shader 模拟** — 用 VisualShader 写自定义效果，仍属于引擎能力范围
3. **手动代码模拟** — 用 TextureRect 叠加、代码控制 modulate 模拟效果

**手动模拟是最后手段，只在引擎能力确实无法覆盖时才使用。**

禁止直接跳到手动方案，必须先确认引擎原生能否实现。

---

## 工具选择规范

### 什么时候用 ShaderMaterial / VisualShader
- 卡牌发光、边框特效、溶解效果
- 程序化纹理（噪点、渐变、图案）
- 悬停高亮、选中效果
- 任何需要动态参数控制的材质效果
- **不要用普通 Sprite2D 或 TextureRect 模拟这些效果**

### 什么时候用 GPUParticles2D / GPUParticles3D
- 施法、攻击、死亡、升级粒子特效
- 环境氛围粒子（漂浮尘埃、魔法光点）
- 轨迹效果
- 粒子数量超过 500 时必须用 GPUParticles，不用 CPUParticles
- **不要用 CPUParticles 做大量粒子**

### 什么时候用 Tween / AnimationPlayer
- **Tween**：卡牌移动、翻转、出牌动画、UI 进场退场、数值跳动（固定终点，有缓动）
- **AnimationPlayer**：复杂的多属性组合动画、角色动画、需要复用的动画序列
- 离散状态切换动画（进退场/出牌/翻转）必须有缓动，不允许线性
- 持续追踪型动画（跟随鼠标/手牌扇形/实时血条）不强制缓动，`_process` Lerp 反而更自然
- **不要用 `_process()` 手写固定终点插值动画；动态追踪目标时 `_process` Lerp 是正确选择**

### 什么时候用 WorldEnvironment
- 场景氛围必须配置以下效果：
  - **Glow** — 发光效果，强度不要过曝
  - **Color Correction** — 色彩风格化，根据美术风格调整
  - **Vignette** — 边缘暗角，增加沉浸感
  - **Depth of Field** — 焦外模糊，用于强调焦点元素（可选）
  - **Fog** — 雾效，用于营造氛围（3D 项目，可选）
- **不要用 CanvasLayer 上的 ColorRect 模拟全屏效果**（除非 WorldEnvironment 无法满足）

> ⚠️ **使用前提——CanvasLayer 内的节点不受 WorldEnvironment 影响：**
> - `CanvasLayer` 创建独立渲染层，其内部所有节点**绕过 WorldEnvironment 后处理**（包括 Glow）
> - 需要 Glow 效果的 UI 元素：不要放在 CanvasLayer 下，改挂到主场景的 Control 节点
> - 如果必须在 CanvasLayer 内实现发光：用 `ShaderMaterial` 在材质层模拟，或用 `BackBufferCopy` 采样后处理结果
>
> 实现任何 UI 发光效果前，先确认节点是否在 CanvasLayer 内。

### 什么时候用 Theme / StyleBox
- 所有 UI 元素必须通过 Theme 统一管理样式
- 按钮、面板、标签的颜色、字体、间距都在 Theme 里定义
- 不要在每个节点上单独设置样式
- **StyleBoxFlat** — 实色背景、圆角、边框
- **StyleBoxTexture** — 使用纹理的复杂边框
- **StyleBoxLine** — 分隔线

---

## 颜色规范

- 建立统一的颜色常量文件，所有颜色从这里引用，不要硬编码
- 主色、辅色、强调色、背景色至少各定义一个
- 发光效果的颜色要比基础色亮（配合 WorldEnvironment Glow 使用）
- 暗色主题：背景不要纯黑，用带色调的深色

```gdscript
# 正确 — 统一管理
class_name GameColors

const GOLD := Color(0.78, 0.67, 0.43)
const GOLD_GLOW := Color(1.5, 1.2, 0.8)
const BACKGROUND := Color(0.04, 0.04, 0.06)
const ACCENT_BLUE := Color(0.2, 0.6, 1.0)

# 错误 — 硬编码
$Card.modulate = Color(0.78, 0.67, 0.43)
```

---

## 动画规范

- 离散状态切换动画（进退场/出牌/翻转）必须有缓动，不允许线性
- 持续追踪型动画（跟随鼠标/手牌扇形/实时血条）不强制缓动，线性 Lerp 反而更自然
- 卡牌出牌动画时长：0.2-0.4 秒
- UI 进场动画时长：0.3-0.5 秒
- 重要事件（胜利、失败）可以用震屏 + 慢动作强调
- 动画必须支持打断和重置
- 不要用 `_process()` 手写固定时长插值循环；持续追踪目标时 `_process` Lerp 是正确选择

```gdscript
# ✅ 固定终点 — Tween，有缓动
var tween := create_tween()
tween.set_ease(Tween.EASE_OUT)
tween.set_trans(Tween.TRANS_BACK)
tween.tween_property(card, "position", target_pos, 0.3)

# ✅ 动态追踪目标 — _process Lerp，线性插值自然流畅
func _process(delta: float) -> void:
    position = position.lerp(_target_pos, delta * speed)

# ❌ 错误 — 用 Tween 追踪动态目标（每帧 kill/restart，性能差）
func _process(delta: float) -> void:
    if _tween: _tween.kill()
    _tween = create_tween()
    _tween.tween_property(self, "position", _target_pos, 0.1)  # ← 错误
```

---

## 卡牌视觉规范（适用于卡牌游戏）

- 卡牌必须有悬停状态（放大 + 发光）
- 拖拽时卡牌要有倾斜效果（rotation 跟随移动方向）
- 可出牌的卡牌要有视觉提示（轻微上移 + 发光边框）
- 费用不足的卡牌要有视觉压制（modulate 变暗）
- 卡牌放置到场地要有落地特效（GPUParticles + 震屏）

```gdscript
# 卡牌悬停效果示例
func _on_mouse_entered() -> void:
    var tween := create_tween()
    tween.set_ease(Tween.EASE_OUT)
    tween.set_trans(Tween.TRANS_BACK)
    tween.tween_property(self, "scale", Vector2(1.1, 1.1), 0.15)
    tween.parallel().tween_property(self, "z_index", 10, 0.0)
```

---

## 信号规范

- 视觉效果的触发必须通过信号解耦，不要在逻辑代码里直接调用视觉函数
- 逻辑层发出信号，视觉层监听信号

```gdscript
# 正确 — 解耦
# 逻辑层
signal card_played(card_data)
emit_signal("card_played", card_data)

# 视觉层监听
func _on_card_played(card_data: CardData) -> void:
    play_card_animation(card_data)

# 错误 — 逻辑和视觉混在一起
func play_card() -> void:
    # 逻辑...
    $AnimationPlayer.play("card_out")  # ← 视觉代码混入逻辑
```

---

## 禁止行为

- ❌ 用普通 `TextureRect` 模拟发光效果
- ❌ 用 `_process()` 手写固定终点插值动画（改用 Tween）；动态追踪目标时 `_process` Lerp 是正确选择
- ❌ 颜色硬编码在节点属性里
- ❌ 粒子效果数量超过 500 还用 `CPUParticles`
- ❌ UI 切换没有任何过渡动画
- ❌ 在逻辑代码里直接调用视觉函数
- ❌ 背景纯黑或纯白没有任何氛围
- ❌ 所有 UI 元素单独设置样式而不用 Theme

---

## 每次做视觉前的检查清单

```
[ ] 确定了明确的视觉风格方向
[ ] 颜色从 GameColors 常量引用
[ ] 发光效果用 ShaderMaterial 或 WorldEnvironment Glow
[ ] 动画用 Tween（固定终点，有缓动）或 _process Lerp（动态追踪）
[ ] 粒子用 GPUParticles（大量）或 CPUParticles（少量）
[ ] WorldEnvironment 已配置氛围效果
[ ] 所有 UI 样式通过 Theme 统一管理
[ ] 视觉触发通过信号解耦，不混入逻辑代码
[ ] 有悬停、选中、禁用三种状态的视觉区分
```
