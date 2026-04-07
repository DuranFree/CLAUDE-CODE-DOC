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

- `Awake()` 早于 `Start()`，跨组件初始化顺序依赖时要注意 ⚠️ [写入CLAUDE.md]
- `OnDestroy()` 里必须取消事件订阅，否则内存泄漏 ⚠️ [写入CLAUDE.md]
- `Time.deltaTime` 在 `FixedUpdate()` 里用 `Time.fixedDeltaTime` ⚠️ [写入CLAUDE.md]
- `Resources.Load()` 性能差，用 Addressables 或直接引用 ⚠️ [写入CLAUDE.md]
- 不要在运行时用 `GameObject.Find()`，提前缓存引用 ⚠️ [写入CLAUDE.md]
- `string` 拼接在热路径里用 `StringBuilder` ⚠️ [写入CLAUDE.md]
- Physics 操作放在 `FixedUpdate()`，输入检测放在 `Update()` ⚠️ [写入CLAUDE.md]
- UI 元素定位超出父容器边界时（pivot + offset 导致跑出 rect 范围），必须主动提示用户验证渲染层级是否遮挡 ⚠️ [写入CLAUDE.md]

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

## 动画规范

**补间动画一律使用 DOTween，禁止手写协程插值。**

适用范围：所有非骨骼动画，包括但不限于：
- UI 弹窗、淡入淡出、滑动、缩放
- 数字滚动、进度条填充
- 卡牌/棋子/道具的移动、旋转、缩放
- 镜头震屏、跟随、FOV 变化
- 颜色/alpha 渐变
- 循环脉冲、呼吸效果

**不适用（继续使用 Animator Controller）：**
- 2D/3D 骨骼动画（Spine、骨骼 rig）
- 多状态切换状态机（idle → walk → run → jump）
- Blend Tree 混合动画

```csharp
// ❌ 禁止 — 手写协程插值
IEnumerator FadeOut(CanvasGroup cg)
{
    float t = 0f;
    while (t < 1f)
    {
        t += Time.deltaTime / 0.5f;
        cg.alpha = Mathf.Lerp(1f, 0f, t);
        yield return null;
    }
}

// ✅ 正确 — DOTween 一行搞定
cg.DOFade(0f, 0.5f).SetEase(Ease.OutQuad);
```

```csharp
// ❌ 禁止 — 手写 MoveTowards 循环
IEnumerator MoveCard(Transform card, Vector3 target)
{
    while (Vector3.Distance(card.position, target) > 0.01f)
    {
        card.position = Vector3.MoveTowards(card.position, target, 5f * Time.deltaTime);
        yield return null;
    }
}

// ✅ 正确 — DOTween
card.DOMove(target, 0.3f).SetEase(Ease.OutQuad);
```

**多段序列动画用 Sequence：**
```csharp
// 战斗冲锋：后撤 → 冲向目标 → 回弹
var seq = DOTween.Sequence();
seq.Append(card.DOMove(backPos, 0.3f).SetEase(Ease.OutQuad));
seq.Append(card.DOMove(targetPos, 0.1f).SetEase(Ease.InQuad));
seq.Append(card.DOMove(originPos, 0.3f).SetEase(Ease.OutBack));
seq.OnComplete(() => OnCombatEnd());
```

**安装方式：**
- Asset Store 免费版：`DOTween (HOTween v2)`
- 或 OpenUPM：`openupm add com.demigiant.dotween`
- 导入后运行 `Tools > Demigiant > DOTween Utility Panel > Setup DOTween`

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

---

## MCP 僵尸进程清理

Unity MCP（mcp-unity）使用 Node.js 作为 Bridge Server。Windows 上 Claude Code 关闭 session 时经常不清理子进程，导致 `node.exe` 僵尸堆积，大量进程同时抢 WebSocket 连接，造成 MCP 超时和连接不稳定。

**首次读取本规则后，必须将以下清理规则写入项目 `CLAUDE.md` 的"每次新会话开始时执行"部分：**

```
**MCP 僵尸进程清理（每次新会话开始时执行）：**
- 执行 `tasklist | grep -c "node.exe"` 检查 node 进程数量
- 如果超过 10 个 → 说明有僵尸 MCP 进程堆积，执行 `cmd.exe //C "taskkill /F /IM node.exe"` 全部杀掉（Claude Code 会自动重启需要的那个）
- 杀完后告知用户：`⚠️ [MCP 清理] 检测到 N 个僵尸 node 进程，已全部清理`
- 10 个以下 → 静默继续
```

---

## MCP 场景自动存盘

**问题根源：** `mcp-unity` 的 `update_component`、`update_gameobject` 等工具直接修改 Unity 场景内存，但不自动保存。场景进入 dirty 状态后，Unity 在进入 Play Mode / 切换场景 / 跑测试时会弹出"是否保存"对话框，导致 MCP 工具等待响应超时。

**解决方案：** 接入 `mcp-unity` 后，立即在项目中创建以下 Editor 脚本（路径：`Assets/Scripts/Editor/AutoSaveScene.cs`）：

```csharp
using UnityEditor;
using UnityEditor.SceneManagement;
using UnityEngine.SceneManagement;

/// <summary>
/// 监听 Hierarchy 变动，2 秒防抖后自动保存场景，防止 MCP 修改后忘记 save_scene 导致弹窗卡住。
/// </summary>
[InitializeOnLoad]
public static class AutoSaveScene
{
    private const double DEBOUNCE_SECONDS = 2.0;
    private static double _lastDirtyTime = -1;
    private static bool _scheduled = false;

    static AutoSaveScene()
    {
        EditorApplication.hierarchyChanged += OnHierarchyChanged;
        EditorApplication.update += OnUpdate;
    }

    private static void OnHierarchyChanged()
    {
        if (Application.isPlaying) return;
        var scene = SceneManager.GetActiveScene();
        if (!scene.isDirty) return;
        _lastDirtyTime = EditorApplication.timeSinceStartup;
        _scheduled = true;
    }

    private static void OnUpdate()
    {
        if (!_scheduled) return;
        if (Application.isPlaying) { _scheduled = false; return; }
        if (EditorApplication.timeSinceStartup - _lastDirtyTime < DEBOUNCE_SECONDS) return;
        _scheduled = false;
        var scene = SceneManager.GetActiveScene();
        if (scene.isDirty)
        {
            EditorSceneManager.SaveScene(scene);
            UnityEngine.Debug.Log("[AutoSave] 场景已自动保存");
        }
    }
}
```

**脚本创建后，将以下规则写入项目 `CLAUDE.md`：**

```
## MCP 场景自动存盘
项目已安装 AutoSaveScene.cs Editor 脚本，场景有变动后 2 秒自动保存。
MCP 修改组件/GameObject 后无需手动调用 save_scene，控制台会打印 [AutoSave] 场景已自动保存 确认。
注意：Build Game Scene（SceneBuilder）内部自带保存，调用后同样无需额外 save_scene。
```
