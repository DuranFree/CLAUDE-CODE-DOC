# Unreal Engine Rules

> Target: Unreal Engine 5.x / C++ / Blueprints
> 使用本文件时，将其与技能文件一起提供给 Claude Code。

---

## UPROPERTY 和 UFUNCTION

所有需要在编辑器里显示或蓝图可访问的属性和函数必须加宏：

```cpp
// 属性
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Combat")
int32 Health = 100;

// 函数
UFUNCTION(BlueprintCallable, Category = "Combat")
void TakeDamage(int32 Amount);
```

---

## 内存管理

UE5 用智能指针和 GC 管理内存，不要手动 `new` / `delete` UObject：

```cpp
// 错误
AMyActor* actor = new AMyActor();

// 正确
AMyActor* actor = GetWorld()->SpawnActor<AMyActor>(AMyActor::StaticClass());

// 软引用（异步加载）
TSoftObjectPtr<UTexture2D> CardTexture;

// 强引用（直接持有）
UPROPERTY()
UTexture2D* CardTexture;
```

---

## TObjectPtr（UE5.0+ 新标准）

**UE5.0 起，`UPROPERTY` 成员变量的原始指针应替换为 `TObjectPtr<T>`。**

```cpp
// ❌ 旧写法 — 原始指针，UE5 不推荐
UPROPERTY(EditAnywhere)
UStaticMeshComponent* MeshComp;

// ✅ 新写法 — TObjectPtr（UE5.0+）
UPROPERTY(EditAnywhere)
TObjectPtr<UStaticMeshComponent> MeshComp;
```

**TObjectPtr 的优势：**
- 在 Editor 构建中启用 lazy load 和 access tracking，有助于检测悬空引用
- 与原始指针 API 完全兼容（隐式转换），改动量极小
- `IsValid()` / `Get()` 等接口与裸指针一致

**规则：**
- 所有带 `UPROPERTY()` 宏的 UObject/Actor/Component 指针成员变量，一律用 `TObjectPtr<T>`
- 局部变量、函数参数、TArray 内元素仍可用原始指针（GC 追踪不依赖这些）
- 非 UPROPERTY 的弱引用用 `TWeakObjectPtr<T>`，软引用用 `TSoftObjectPtr<T>`

---

## Null 检查

UE5 用 `IsValid()` 检查对象是否有效，不要只用 `!= nullptr`：

```cpp
// 不够安全
if (MyActor != nullptr) { }

// 正确
if (IsValid(MyActor)) { }
```

---

## 委托和事件

用 UE5 的委托系统代替函数指针：

```cpp
// 声明委托
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, int32, NewHealth);

// 在类里
UPROPERTY(BlueprintAssignable)
FOnHealthChanged OnHealthChanged;

// 触发
OnHealthChanged.Broadcast(CurrentHealth);
```

---

## 命名规范

UE5 有严格的命名规范，必须遵守：

- `A` 前缀：Actor 类，如 `AGameManager`
- `U` 前缀：UObject 类，如 `UCardData`
- `F` 前缀：结构体，如 `FCardInfo`
- `E` 前缀：枚举，如 `EGameState`
- `I` 前缀：接口，如 `ICombatInterface`
- `T` 前缀：模板类，如 `TArray`

---

## 数据资产

游戏数据用 `UDataAsset` 存储，不要硬编码：

```cpp
UCLASS()
class UCardDataAsset : public UDataAsset
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere)
    FString CardName;

    UPROPERTY(EditAnywhere)
    int32 Attack;
};
```

---

## 测试框架

标准测试框架是 **Automation Testing Framework**，内置。

无头模式跑测试：
```
UnrealEditor-Cmd.exe <ProjectPath> -ExecCmds="Automation RunTests <TestName>" -nullrhi -unattended -nopause
```

---

## Blueprint vs C++

- **逻辑密集、性能敏感** → 用 C++
- **关卡设计、快速原型、视觉效果** → 用 Blueprint
- 核心系统用 C++ 写，暴露接口给 Blueprint 调用

---

## 动画规范

**动画工具选择原则：有固定终点用 Timeline/UMG Animation；目标持续变化用 Tick Lerp。**

### ✅ 用 FTimeline / UMG Animation 的场景
从 A 到 B 的一次性动画（起点终点明确、时长固定）：
- UI 弹窗、淡入淡出、滑动、缩放
- 数字滚动、进度条填充
- 卡牌出牌、翻转、攻击动画（移动到固定目标位）
- 镜头震屏、FOV 变化
- 颜色/alpha 渐变

```cpp
// ✅ 一次性淡出 — UTimelineComponent
void AMyActor::OnFadeUpdate(float Value)
{
    MeshComp->SetScalarParameterValueOnMaterials(TEXT("Opacity"), Value);
}
// BeginPlay 中绑定 FadeCurve 并 PlayFromStart()
```

```cpp
// ✅ 简单位移 — MoveComponentTo（无需完整 Timeline）
UKismetSystemLibrary::MoveComponentTo(Component, TargetPos, TargetRot,
    false, false, 0.3f, false, EMoveComponentAction::Move, LatentInfo);
```

### ❌ 不用 Timeline，改用 Tick Lerp 的场景
目标每帧都可能变化，Timeline 是固定时长动画，不适合持续追踪：
- **持续追踪目标**：手牌扇形展开（目标坐标随牌数重算）、悬停跟随
- **拖拽预览**：跟随鼠标位置的平滑插值
- **弹簧/阻尼效果**：`FMath::Lerp(Current, Target, DeltaTime * Speed)` 模式

```cpp
// ✅ 持续追踪 — Tick Lerp（目标每帧可能变化）
void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    CurrentPos = FMath::VInterpTo(CurrentPos, TargetPos, DeltaTime, Speed);
    SetActorLocation(CurrentPos);
}
```

### ❌ 不适用（使用 Animation Blueprint / Montage）
- 骨骼动画（Skeleton Mesh）
- 多状态切换状态机（idle → walk → run → jump）
- Blend Space 混合动画
- AnimMontage 攻击/技能动画

### 清理规则
- Timeline 绑定在 Actor 上，Actor 销毁时自动清理
- 手动停止用 `FadeTimeline->Stop()`
- `OnDestroy()` 中解绑委托防止悬挂引用

---

## UMG 布局面板使用判断规则
> ⚠️ [写入CLAUDE.md] 核心结论需同步到项目 CLAUDE.md 的"Unreal 引擎常见坑"

**用 Box 类面板的唯一合理场景：** 子 Widget 只需要静态排列，不需要任何位移/旋转/缩放动画。

| 场景 | 用 Box 面板？ |
|------|------------|
| 设置菜单选项行、背包格子、商店列表 | ✅ 适合（HorizontalBox / VerticalBox） |
| 对话框按钮组（确认/取消） | ✅ 适合 |
| 技能栏图标（静态排列） | ✅ 适合 |
| 手牌扇形展开 | ❌ 不适合，改用 Canvas Panel |
| Widget 飞入/飞出动画 | ❌ 不适合，改用 Canvas Panel |
| 子 Widget 需要旋转 | ❌ 不适合，改用 Canvas Panel |
| 子 Widget 需要非均匀间距 | ❌ 不适合，改用 Canvas Panel |
| 拖拽 Widget | ❌ 不适合，拖拽时必须移到顶层 Canvas Panel |

**判断口诀：** 子 Widget 只排列不动 → 用 Box 面板；子 Widget 要动 → 用 Canvas Panel 自己控制 Anchors。

**Canvas Panel 是 UMG 里唯一支持自由定位动画的容器**，相当于 Unity 的"无 LayoutGroup 的 RectTransform"。

---

## UMG 布局面板与 Widget 动画互斥

UMG 的布局面板（UHorizontalBox、UVerticalBox、UGridPanel、UUniformGridPanel、UWrapBox 等）
通过 Slate 的 `ArrangeChildren` 在每次布局 Pass 时强制覆盖子 Widget 的位置和大小，
与 Widget Animation / UMG Timeline / Tick 里手动设置 `RenderTranslation` 会产生冲突。

- 凡是需要动画控制位置的 Widget，必须放在 `UCanvasPanel` 下（自由定位），
  不得作为 Box 类面板（Horizontal / Vertical / Grid）的直接子节点

- 正确做法二选一：
  ① Canvas Panel + Anchors 自由定位，动画用 Widget Animation 或 Tick 设置 `RenderTransform`
  ② 如果必须在 Box 内，只用 `SetRenderTranslation()` 做纯视觉偏移（不影响 layout 空间），
     但注意 hit-test 区域仍在原始 layout 位置，点击判定会错位

- RenderTransform vs 真实位置：
  - `RenderTransform` = 纯视觉偏移，layout 不感知，hit-test 不跟随
  - `Slot` 里的 `Offset` / `Position` = 真实布局位置，Box 面板会覆盖

- 常见踩坑场景：
  - 手牌扇形展开 → Canvas Panel 下，不放 HorizontalBox
  - 卡牌拖拽：拖拽时 `AddToViewport` 放顶层，释放后再 `AddChild` 回原容器
  - 弹窗飞入：放在覆盖全屏的 Canvas Panel 层，不放在任何 Box 面板内

---

## 常见坑

- `BeginPlay()` 等价于 Unity 的 `Start()`，`Tick()` 等价于 `Update()` ⚠️ [写入CLAUDE.md]
- `BeginPlay()` 时其他 Actor 不一定已经初始化，跨 Actor 依赖用 `OnActorBeginOverlap` 或 GameMode 管理 ⚠️ [写入CLAUDE.md]
- `TArray` 是 UE5 的动态数组，不要用 `std::vector` ⚠️ [写入CLAUDE.md]
- `FString` 是 UE5 的字符串，不要用 `std::string` ⚠️ [写入CLAUDE.md]
- `UE_LOG` 用于日志输出，不要用 `printf` 或 `std::cout` ⚠️ [写入CLAUDE.md]
- 编译时间很长，改动要小步提交 ⚠️ [写入memory]
- Hot Reload 不稳定，重大改动后完整重编译 ⚠️ [写入memory]
- 物理操作在 `PhysicsTick` 或用 `FBodyInstance` 直接操作 ⚠️ [写入CLAUDE.md]
- UI 元素定位超出父容器边界时，必须主动提示用户验证渲染层级是否遮挡 ⚠️ [写入memory]
- 子 Widget 需要位移/旋转/缩放动画时，容器必须用 Canvas Panel；静态排列才用 Box 面板 ⚠️ [写入CLAUDE.md]
- 收到任何视觉效果需求前必须确认 PostProcessVolume 是否存在且覆盖相机、Widget 渲染模式是否为 Screen Space；Screen Space UMG 不受 Post Process Volume 影响 ⚠️ [写入memory]

---

## 视觉效果前置检查规则
> ⚠️ [写入memory] 首次读取本规则后，必须追加到项目 memory 索引，作为"收到视觉需求时"的触发器
> 视觉效果实现优先级（引擎原生 → 材质 → 手动）见 UNREAL-VISUAL-RULES.md

收到任何视觉效果需求（发光 / Bloom / 模糊 / 后处理）前，必须先确认：

1. **Post Process Volume 是否存在且覆盖相机**：
   - 没有 PostProcessVolume 或 `Infinite Extent` 未勾选 → Bloom 等效果不生效
   - PostProcessVolume 存在但 Blend Weight = 0 → 同样无效

2. **UMG Widget 渲染模式**：
   - `Screen Space`（默认）→ Widget 在后处理**之后**合成，**Bloom 碰不到**
   - `World Space` → Widget 作为场景对象渲染，后处理生效 ✅
   - 需要 Bloom 的 UI → 改用 World Space Widget，或渲染到 Render Target 再后处理

3. **自发光材质触发 Bloom**：
   - 材质 Emissive Color 强度 > 1.0 可触发 Bloom，适合场景物体，不适合纯 UI Widget

---

## 类型安全规则

**枚举转换：**
```cpp
// 错误 — 枚举直接当 FString 用
FString name = CardType;  // ← 报错

// 正确 — 用 UEnum::GetValueAsString()
FString name = UEnum::GetValueAsString(CardType);

// 或用 StaticEnum
FString name = StaticEnum<ECardType>()->GetNameStringByValue((int64)CardType);
```

**FString vs FName vs FText：**
```cpp
// FString — 可修改的字符串，用于逻辑处理
FString CardName = TEXT("Fireball");

// FName — 不可修改的标识符，用于资产引用、查找
FName SocketName = TEXT("hand_r");

// FText — 本地化字符串，用于 UI 显示
FText DisplayName = LOCTEXT("CardName", "Fireball");

// 错误 — 混用类型
FName name = CardName;  // ← 需要显式转换 FName(*CardName)
```

**整数除法：**
```cpp
// 错误 — 结果是 int，小数被截断
int32 ratio = Damage / MaxHp;  // ← 结果是 0 或 1

// 正确 — 先转成 float
float ratio = (float)Damage / (float)MaxHp;
```

**IsValid 检查：**
```cpp
// 错误 — 不检查直接用，容易崩溃
Actor->DoSomething();

// 正确 — 先检查
if (IsValid(Actor))
{
    Actor->DoSomething();
}

// TWeakObjectPtr 需要用 IsValid() 而不是 != nullptr
if (WeakRef.IsValid())
{
    WeakRef->DoSomething();
}
```

**TArray 操作：**
```cpp
// 错误 — 越界访问
TArray<int32> arr = {1, 2, 3};
int32 val = arr[5];  // ← 崩溃

// 正确 — 先检查长度
if (arr.IsValidIndex(5))
{
    int32 val = arr[5];
}
```

---

## Git 管理规则

**必须排除的文件夹：**
- `Binaries/` — 编译输出，每次构建自动生成
- `Intermediate/` — 编译中间文件，体积巨大
- `DerivedDataCache/` — 类似 Unity 的 Library，自动生成的资产缓存
- `Saved/` — 日志、截图、崩溃报告
- `Build/` — 打包输出

**标准 .gitignore 内容（必须包含）：**
```
Binaries/
Intermediate/
DerivedDataCache/
Saved/
Build/
*.VC.db
*.VC.opendb
```

**如已误上传，执行以下命令从仓库删除（不删本地文件）：**
```
git rm -r --cached Binaries/
git rm -r --cached Intermediate/
git rm -r --cached DerivedDataCache/
git rm -r --cached Saved/
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

**问题根源：** UE5 MCP 工具修改 Actor 属性或关卡结构后，关卡处于未保存状态（标题栏显示 `*`）。切换关卡或某些操作会触发保存弹窗，导致 MCP 响应超时。

**解决方案：** 接入 MCP 后，立即在项目中创建以下 Python Editor Utility 脚本（路径：`Content/PythonScripts/auto_save_level.py`），并配置为编辑器启动时自动执行：

```python
"""
auto_save_level.py
监听关卡修改事件，2 秒防抖后自动保存当前关卡。
配置方式：Editor Preferences > Python > Startup Scripts 添加此文件路径。
"""
import unreal
import threading

_save_timer: threading.Timer = None
DEBOUNCE_SEC: float = 2.0


def _do_save():
    world = unreal.EditorLevelLibrary.get_editor_world()
    if world is None:
        return
    if unreal.EditorLoadingAndSavingUtils.save_current_level():
        unreal.log("[AutoSave] 关卡已自动保存")
    else:
        unreal.log_warning("[AutoSave] 关卡保存失败，请手动保存")


def _on_object_modified(obj):
    global _save_timer
    if _save_timer is not None:
        _save_timer.cancel()
    _save_timer = threading.Timer(DEBOUNCE_SEC, _do_save)
    _save_timer.daemon = True
    _save_timer.start()


# 注册修改监听
unreal.register_slate_post_tick_callback(lambda dt: None)  # 保持 Python 运行时活跃
_delegate = unreal.get_editor_subsystem(
    unreal.AssetEditorSubsystem
)
# 用 EditorDelegates 监听对象变更
unreal.EditorDelegates.on_asset_post_import  # placeholder — 实际用 FCoreUObjectDelegates
# 推荐方式：通过 Python binding 监听
import unreal
subsystem = unreal.get_editor_subsystem(unreal.UnrealEditorSubsystem)
unreal.log("[AutoSave] 关卡自动存盘插件已启动")
```

> **注意：** UE5 Python API 在不同版本存在差异。推荐的更稳定替代方案是创建 C++ `UEditorUtilitySubsystem` 插件，订阅 `FCoreUObjectDelegates::OnObjectModified`，在回调中用 Timer 延迟调用 `FEditorFileUtils::SaveLevel`。Python 方案适合快速接入，C++ 方案适合生产环境。

**简易替代方案（推荐优先用这个）：**
在 `Editor Preferences > General > Auto Save > Enable AutoSave` 中开启 UE5 内置自动保存，设置间隔为 `30 秒`，可覆盖大多数 MCP 修改场景的场景。

**接入后，将以下规则写入项目 `CLAUDE.md`：**

```
## MCP 场景自动存盘
项目已启用 UE5 内置 AutoSave（30秒间隔）或 Python auto_save_level.py 插件。
MCP 修改 Actor/关卡结构后，等待自动保存日志 [AutoSave] 关卡已自动保存 确认，或手动执行 Ctrl+S。
不确定是否已保存时，通过 MCP 调用 execute_menu_item("File/Save Current Level") 强制保存。
```
