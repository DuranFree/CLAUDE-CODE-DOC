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

**程序化补间动画一律使用 FTimeline 或 UTimelineComponent，禁止手写 Tick 插值循环。**

适用范围：所有非骨骼动画，包括但不限于：
- UI 弹窗、淡入淡出、滑动、缩放
- 数字滚动、进度条填充
- 卡牌/棋子/道具的移动、旋转、缩放
- 镜头震屏、跟随、FOV 变化
- 颜色/alpha 渐变
- 循环脉冲、呼吸效果

**不适用（继续使用 Animation Blueprint / Montage）：**
- 骨骼动画（Skeleton Mesh）
- 多状态切换状态机（idle → walk → run → jump）
- Blend Space 混合动画
- AnimMontage 攻击/技能动画

**C++ Timeline 用法：**
```cpp
// ❌ 禁止 — 手写 Tick 插值
void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    Alpha -= DeltaTime / 0.5f;
    MeshComp->SetScalarParameterValueOnMaterials(TEXT("Opacity"), Alpha);
}

// ✅ 正确 — UTimelineComponent
// .h
UPROPERTY()
UTimelineComponent* FadeTimeline;

UPROPERTY()
UCurveFloat* FadeCurve;

UFUNCTION()
void OnFadeUpdate(float Value);

// .cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();

    FOnTimelineFloat UpdateDelegate;
    UpdateDelegate.BindUFunction(this, FName("OnFadeUpdate"));
    FadeTimeline->AddInterpFloat(FadeCurve, UpdateDelegate);
    FadeTimeline->PlayFromStart();
}

void AMyActor::OnFadeUpdate(float Value)
{
    MeshComp->SetScalarParameterValueOnMaterials(TEXT("Opacity"), Value);
}
```

**Blueprint 中直接用 Timeline 节点：**
- 在蓝图 EventGraph 中添加 Timeline 节点
- 双击编辑曲线，可视化调整 Ease
- 连接 Update 引脚到 Set 属性节点

**轻量替代方案（简单一次性动画）：**
```cpp
// FLatentActionInfo + UKismetSystemLibrary::MoveComponentTo
// 适合简单的位移/旋转，不需要完整 Timeline

// 或使用第三方插件 BUITween / iTween for UE
// API 更接近 DOTween 风格，适合大量程序化动画
```

**清理规则：**
- Timeline 绑定在 Actor 上，Actor 销毁时自动清理
- 手动停止用 `FadeTimeline->Stop()`
- `OnDestroy()` 中解绑委托防止悬挂引用

---

## 常见坑

- `BeginPlay()` 等价于 Unity 的 `Start()`，`Tick()` 等价于 `Update()` ⚠️ [写入CLAUDE.md]
- `BeginPlay()` 时其他 Actor 不一定已经初始化，跨 Actor 依赖用 `OnActorBeginOverlap` 或 GameMode 管理 ⚠️ [写入CLAUDE.md]
- `TArray` 是 UE5 的动态数组，不要用 `std::vector` ⚠️ [写入CLAUDE.md]
- `FString` 是 UE5 的字符串，不要用 `std::string` ⚠️ [写入CLAUDE.md]
- `UE_LOG` 用于日志输出，不要用 `printf` 或 `std::cout` ⚠️ [写入CLAUDE.md]
- 编译时间很长，改动要小步提交 ⚠️ [写入CLAUDE.md]
- Hot Reload 不稳定，重大改动后完整重编译 ⚠️ [写入CLAUDE.md]
- 物理操作在 `PhysicsTick` 或用 `FBodyInstance` 直接操作 ⚠️ [写入CLAUDE.md]
- UI 元素定位超出父容器边界时，必须主动提示用户验证渲染层级是否遮挡 ⚠️ [写入CLAUDE.md]

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
