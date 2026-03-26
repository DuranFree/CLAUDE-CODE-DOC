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

## 常见坑

- `BeginPlay()` 等价于 Unity 的 `Start()`，`Tick()` 等价于 `Update()`
- `BeginPlay()` 时其他 Actor 不一定已经初始化，跨 Actor 依赖用 `OnActorBeginOverlap` 或 GameMode 管理
- `TArray` 是 UE5 的动态数组，不要用 `std::vector`
- `FString` 是 UE5 的字符串，不要用 `std::string`
- `UE_LOG` 用于日志输出，不要用 `printf` 或 `std::cout`
- 编译时间很长，改动要小步提交
- Hot Reload 不稳定，重大改动后完整重编译
- 物理操作在 `PhysicsTick` 或用 `FBodyInstance` 直接操作
