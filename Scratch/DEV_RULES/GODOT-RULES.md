# Godot Engine Rules

> Target: Godot 4.x / GDScript
> 使用本文件时，将其与技能文件一起提供给 Claude Code。

---

## 强制类型声明

所有变量必须明确声明类型，不允许让 Godot 自动推断：

```gdscript
# 错误
var owner = get_parent()
var is_dead = false

# 正确
var owner: Node = get_parent()
var is_dead: bool = false
```

函数参数和返回值也必须声明类型：

```gdscript
# 错误
func take_damage(amount):
    pass

# 正确
func take_damage(amount: int) -> void:
    pass
```

---

## 节点引用

不要用字符串路径获取节点，用 `@onready` + 类型声明：

```gdscript
# 错误
var player = get_node("Player")

# 正确
@onready var player: CharacterBody2D = $Player
```

---

## 信号声明

信号必须明确声明参数类型：

```gdscript
# 错误
signal health_changed

# 正确
signal health_changed(new_health: int, max_health: int)
```

---

## 枚举

用枚举代替魔法数字：

```gdscript
# 错误
var state = 0

# 正确
enum State { IDLE, MOVING, ATTACKING }
var state: State = State.IDLE
```

**枚举类型转换规则：**
- 枚举值不能直接当 String 使用，必须先转换
- 用自定义转换函数或内置方式处理

```gdscript
# 错误 — 枚举直接传给期望 String 的函数
some_function(unit.sch_type)  # ← 报错

# 正确 — 先转换
some_function(_sch_type_to_string(unit.sch_type))

# 或用内置转换
some_function(SchType.keys()[unit.sch_type])
```

---

## 类型安全规则

**整数除法：**
```gdscript
# 错误 — 结果是 int，小数被截断
var ratio = damage / max_hp  # ← 结果是 0 或 1

# 正确 — 先转成 float
var ratio: float = float(damage) / float(max_hp)
```

**Null 检查：**
```gdscript
# 正确 — 用 is_instance_valid 检查节点
if is_instance_valid(node):
    node.do_something()

# 正确 — 用 != null 检查普通对象
if data != null:
    process(data)

# 错误 — 不检查直接用，容易崩溃
node.do_something()  # ← 如果 node 已销毁会报错
```

**Variant 避免：**
- 不要使用 Variant 类型，会隐藏类型错误
- 所有变量必须有明确类型声明
- 如果看到 "variable type inferred as Variant" 警告，必须修复

```gdscript
# 错误
var data = get_some_data()  # 类型不明

# 正确
var data: CardData = get_some_data()
```

---

## 空值检查

Godot 4 用 `== null` 或 `is_instance_valid()`，不要假设节点一定存在：

```gdscript
# 正确
if owner != null:
    owner.do_something()

# 更安全
if is_instance_valid(owner):
    owner.do_something()
```

---

## Autoload 访问规则

GDScript autoload 是挂在场景树 `/root/` 下的节点，不是 C++ 单例，访问方式完全不同：

```gdscript
# 错误 — C++ 单例的访问方式，对 GDScript autoload 无效
var gs = Engine.get_singleton("GameState")  # ← 返回 null

# 正确方式 1 — 直接用 autoload 名称（推荐，最简洁）
var gs = GameState

# 正确方式 2 — 用节点路径
var gs = get_node("/root/GameState")
```

**注意**：如果 autoload 名称在 Project Settings 里是 `GameState`，直接写 `GameState` 就能访问，不需要任何 get 调用。

---

## 场景实例化

用 `preload` 而不是 `load`，性能更好：

```gdscript
# 一般
var scene = load("res://scenes/card.tscn")

# 推荐
const CardScene: PackedScene = preload("res://scenes/card.tscn")
```

---

## 测试框架

标准测试框架是 **GUT (Godot Unit Testing)**。
- 开始前检查 GUT 是否已安装
- 未安装则自动安装
- 安装失败则询问用户

无头模式跑测试：
```
godot --headless -s res://addons/gut/gut_cmdln.gd
```

---

## 动画规范

**补间动画一律使用内置 Tween，禁止手写 _process 插值循环。**

适用范围：所有非骨骼动画，包括但不限于：
- UI 弹窗、淡入淡出、滑动、缩放
- 数字滚动、进度条填充
- 卡牌/棋子/道具的移动、旋转、缩放
- 镜头震屏、跟随、FOV 变化
- 颜色/alpha 渐变
- 循环脉冲、呼吸效果

**不适用（继续使用 AnimationPlayer / AnimationTree）：**
- 2D/3D 骨骼动画（Spine、骨骼 rig）
- 多状态切换状态机（idle → walk → run → jump）
- Blend Tree 混合动画

```gdscript
# ❌ 禁止 — 手写 _process 插值
var _alpha: float = 1.0
var _fading: bool = false

func _process(delta: float) -> void:
    if _fading:
        _alpha -= delta / 0.5
        modulate.a = _alpha
        if _alpha <= 0.0:
            _fading = false

# ✅ 正确 — Tween 一行搞定
func fade_out() -> void:
    var tween: Tween = create_tween()
    tween.tween_property(self, "modulate:a", 0.0, 0.5).set_ease(Tween.EASE_OUT)
```

```gdscript
# ❌ 禁止 — 手写 move_toward 循环
func _process(delta: float) -> void:
    position = position.move_toward(target_pos, 300.0 * delta)

# ✅ 正确 — Tween
func move_to(target: Vector2) -> void:
    var tween: Tween = create_tween()
    tween.tween_property(self, "position", target, 0.3).set_ease(Tween.EASE_OUT)
```

**多段序列动画：**
```gdscript
# 战斗冲锋：后撤 → 冲向目标 → 回弹
func charge_attack(back_pos: Vector2, target_pos: Vector2, origin_pos: Vector2) -> void:
    var tween: Tween = create_tween()
    tween.tween_property(self, "position", back_pos, 0.3).set_ease(Tween.EASE_OUT)
    tween.tween_property(self, "position", target_pos, 0.1).set_ease(Tween.EASE_IN)
    tween.tween_property(self, "position", origin_pos, 0.3).set_trans(Tween.TRANS_BACK)
    tween.tween_callback(on_combat_end)
```

**清理规则：**
- 创建新 Tween 前，如果之前有正在运行的 Tween 需要先 `kill()`
- 节点被 `queue_free()` 时，所有通过 `create_tween()` 创建的 Tween 会自动停止，无需手动清理

---

## 常见坑

- `_ready()` 里访问其他节点时，确保那个节点已经在场景树里 ⚠️ [写入CLAUDE.md]
- `@export` 变量在编辑器里设置后，运行时不要再用代码覆盖 ⚠️ [写入CLAUDE.md]
- `Variant` 类型的警告默认是 warning，但很多项目设置为 error，一律避免使用 ⚠️ [写入CLAUDE.md]
- `int / int` 在 GDScript 里结果还是 `int`，需要浮点除法用 `float(a) / float(b)` ⚠️ [写入CLAUDE.md]
- 信号连接推荐用代码连接而不是编辑器连接，方便追踪 ⚠️ [写入CLAUDE.md]
- UI 元素定位超出父容器边界时，必须主动提示用户验证渲染层级是否遮挡 ⚠️ [写入CLAUDE.md]

---

## Git 管理规则

**必须排除的文件夹/文件：**
- `.godot/` — Godot 4 自动生成的项目缓存，绝对不上传
- `export_presets.cfg` — 如含签名密钥或证书信息，不得上传
- `.import/` — Godot 3 的导入缓存（如兼容旧项目）

**标准 .gitignore 内容（必须包含）：**
```
.godot/
export_presets.cfg
```

**如已误上传，执行以下命令从仓库删除（不删本地文件）：**
```
git rm -r --cached .godot/
git rm --cached export_presets.cfg
git commit -m "chore: remove auto-generated folders from tracking"
git push
```

**确认 .gitignore 已包含以上路径后再 push，否则下次还会上传。**

---

## MCP 僵尸进程清理

如果项目使用了引擎 MCP Server，Windows 上 Claude Code 关闭 session 时可能不清理子进程，导致僵尸进程堆积、MCP 连接不稳定。

**首次接入 MCP 时，检查 `.mcp.json` 的 `command` 字段，确认 MCP Server 使用的运行时进程名（如 `node`、`python`、`dotnet` 等），然后在项目 `CLAUDE.md` 的"每次新会话开始时执行"部分加入对应的清理规则：**

```
每次新会话开始时执行：
- 检查 <进程名> 进程数量
- 超过 10 个 → 全部杀掉（Claude Code 会自动重启需要的那个）
- 10 个以下 → 静默继续
```

---

## MCP 场景自动存盘

**问题根源：** Godot MCP 工具（如 `godot-mcp`）修改场景节点后，编辑器场景处于未保存状态。切换场景或关闭编辑器时会弹出保存对话框，导致 MCP 响应超时。

**解决方案：** 接入 MCP 后，立即在项目中创建以下 EditorPlugin（路径：`addons/auto_save/auto_save_plugin.gd`），并在 `Project > Project Settings > Plugins` 中启用：

```gdscript
@tool
extends EditorPlugin

const DEBOUNCE_SEC: float = 2.0
var _timer: float = -1.0
var _dirty: bool = false

func _enter_tree() -> void:
    get_editor_interface().get_edited_scene_root() # warm up
    # 监听场景树变动
    get_tree().node_added.connect(_on_scene_changed)
    get_tree().node_removed.connect(_on_scene_changed)

func _exit_tree() -> void:
    if get_tree().node_added.is_connected(_on_scene_changed):
        get_tree().node_added.disconnect(_on_scene_changed)
    if get_tree().node_removed.is_connected(_on_scene_changed):
        get_tree().node_removed.disconnect(_on_scene_changed)

func _on_scene_changed(_node: Node = null) -> void:
    _dirty = true
    _timer = DEBOUNCE_SEC

func _process(delta: float) -> void:
    if not _dirty: return
    _timer -= delta
    if _timer > 0.0: return
    _dirty = false
    var scene_root: Node = get_editor_interface().get_edited_scene_root()
    if scene_root == null: return
    var path: String = scene_root.scene_file_path
    if path.is_empty(): return
    var packed: PackedScene = PackedScene.new()
    packed.pack(scene_root)
    ResourceSaver.save(packed, path)
    print("[AutoSave] 场景已自动保存: ", path)
```

**注意事项：**
- 该插件只监听节点增删，不监听属性变化。如果 MCP 只修改了属性（如位置、颜色），仍需手动保存或在 MCP 操作后调用 `save_scene`。
- Godot 4.x 内置自动保存（`Editor > Editor Settings > Auto Save On Focus Lost`），可作为补充，但响应更慢。

**插件创建后，将以下规则写入项目 `CLAUDE.md`：**

```
## MCP 场景自动存盘
项目已安装 auto_save EditorPlugin，节点增删后 2 秒自动保存场景。
MCP 修改节点结构后无需手动保存，控制台打印 [AutoSave] 场景已自动保存 确认。
注意：MCP 只修改属性（位置/颜色等）时自动存盘不触发，需手动调用 save_scene 或确认已保存。
```
