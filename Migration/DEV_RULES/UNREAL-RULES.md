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
