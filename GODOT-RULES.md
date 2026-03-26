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

## 常见坑

- `_ready()` 里访问其他节点时，确保那个节点已经在场景树里
- `@export` 变量在编辑器里设置后，运行时不要再用代码覆盖
- `Variant` 类型的警告默认是 warning，但很多项目设置为 error，一律避免使用
- `int / int` 在 GDScript 里结果还是 `int`，需要浮点除法用 `float(a) / float(b)`
- 信号连接推荐用代码连接而不是编辑器连接，方便追踪
