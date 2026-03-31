# Unity Engine Rules

> Target: Unity 2022.x+ / C# / URP
> 使用本文件时，将其与技能文件一起提供给 Claude Code。

---

## 命名空间

所有脚本必须声明命名空间，避免命名冲突：

```csharp
// 错误
public class GameManager : MonoBehaviour { }

// 正确
namespace MyGame.Core
{
    public class GameManager : MonoBehaviour { }
}
```

---

## Null 检查

Unity 的 `== null` 对 UnityEngine.Object 有特殊行为，用以下方式：

```csharp
// 对普通 C# 对象
if (obj != null) { }

// 对 Unity 组件/GameObject
if (gameObject != null) { }

// 更安全的方式
if (this != null && gameObject != null) { }
```

---

## GetComponent 缓存

不要在 `Update()` 里反复调用 `GetComponent`，在 `Awake()` 或 `Start()` 里缓存：

```csharp
// 错误 - 每帧都调用，性能差
void Update()
{
    GetComponent<Rigidbody>().AddForce(Vector3.up);
}

// 正确 - 缓存引用
private Rigidbody _rb;
void Awake() => _rb = GetComponent<Rigidbody>();
void Update() => _rb.AddForce(Vector3.up);
```

---

## SerializeField

用 `[SerializeField]` 代替 `public` 暴露字段：

```csharp
// 错误
public int health = 100;

// 正确
[SerializeField] private int health = 100;
```

---

## 协程 vs async/await

Unity 2022+ 支持 async/await，优先使用，但注意生命周期：

```csharp
// 旧方式
IEnumerator WaitAndDo()
{
    yield return new WaitForSeconds(1f);
    DoSomething();
}

// 新方式 (Unity 2022+)
async void WaitAndDo()
{
    await Task.Delay(1000);
    DoSomething();
}
```

---

## ScriptableObject

游戏数据（卡牌属性、技能配置）用 ScriptableObject 存储，不要硬编码：

```csharp
[CreateAssetMenu(fileName = "CardData", menuName = "Game/CardData")]
public class CardData : ScriptableObject
{
    public string cardName;
    public int attack;
    public int defense;
}
```

---

## 测试框架

标准测试框架是 **Unity Test Framework (UTF)**，内置，无需安装。

无头模式跑测试：
```
Unity -batchmode -runTests -testPlatform EditMode -nographics -projectPath <path>
```

EditMode 测试放在 `Assets/Tests/EditMode/` 目录。
PlayMode 测试放在 `Assets/Tests/PlayMode/` 目录。

**⚠️ `-quit` 和 `-runTests` 不能同时使用：**
- `-quit` 会让 Unity 在测试跑完之前退出，导致测试结果丢失
- 使用 `-runTests` 时不加 `-quit`，测试完成后 Unity 会自动退出
- 正确写法：`-batchmode -runTests -testPlatform EditMode -nographics -projectPath <path>`
- 错误写法：`-batchmode -runTests -quit -nographics -projectPath <path>`

---

## 纯逻辑抽离

尽早将不依赖 MonoBehaviour 的纯逻辑抽离到独立的 `.netstandard2.1` 类库，使用 `dotnet test` 跑，速度远快于 Unity 无头模式。

---

## 常见坑

- `Awake()` 早于 `Start()`，跨组件初始化顺序依赖时要注意
- `OnDestroy()` 里必须取消事件订阅，否则内存泄漏
- `Time.deltaTime` 在 `FixedUpdate()` 里用 `Time.fixedDeltaTime`
- `Resources.Load()` 性能差，用 Addressables 或直接引用
- 不要在运行时用 `GameObject.Find()`，提前缓存引用
- `string` 拼接在热路径里用 `StringBuilder`
- Physics 操作放在 `FixedUpdate()`，输入检测放在 `Update()`

---

## 类型安全规则

**枚举转换：**
```csharp
// 错误 — 枚举直接当 string 用
string name = cardType;  // ← 报错

// 正确 — 用 ToString() 或 Enum.GetName()
string name = cardType.ToString();
string name2 = Enum.GetName(typeof(CardType), cardType);
```

**整数除法：**
```csharp
// 错误 — 结果是 int，小数被截断
int ratio = damage / maxHp;  // ← 结果是 0 或 1

// 正确 — 先转成 float
float ratio = (float)damage / (float)maxHp;
```

**值类型 vs 引用类型：**
```csharp
// struct 是值类型，赋值是复制
Vector3 pos = transform.position;
pos.x = 5f;  // ← 不会影响 transform.position，必须重新赋值
transform.position = pos;  // ← 正确

// 错误写法
transform.position.x = 5f;  // ← 编译报错
```

**Null 合并：**
```csharp
// 对普通 C# 对象可以用 ?? 运算符
var target = possibleTarget ?? defaultTarget;

// 对 Unity Object 不要用 ??，因为 == null 被重载过
// 改用三元运算符
var target = possibleTarget != null ? possibleTarget : defaultTarget;
```

---

## Git 管理规则

**必须排除的文件夹：**
- `Library/` — Unity 自动生成的完整缓存目录，绝对不上传
- `Library/PackageCache/` — 包缓存，自动生成，体积巨大
- `Temp/` — 临时编译文件
- `Logs/` — 运行日志
- `obj/` — 编译中间文件

**标准 .gitignore 内容（必须包含）：**
```
[Ll]ibrary/
[Tt]emp/
[Oo]bj/
[Ll]ogs/
```

**如已误上传，执行以下命令从仓库删除（不删本地文件）：**
```
git rm -r --cached Library/
git rm -r --cached Temp/
git rm -r --cached Obj/
git commit -m "chore: remove auto-generated folders from tracking"
git push
```

**确认 .gitignore 已包含以上路径后再 push，否则下次还会上传。**
