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
