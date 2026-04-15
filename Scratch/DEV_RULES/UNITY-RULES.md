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

Unity 2022+ 支持 async/await，但 `async void` 在 MonoBehaviour 上有严重生命周期风险：

**⚠️ 禁止在 MonoBehaviour 上用 `async void`**

```csharp
// ❌ 危险 — async void 无法被外部 await，对象销毁后回调照跑，导致访问已销毁组件崩溃
async void LoadCard()
{
    await Task.Delay(1000);
    // 如果这 1 秒内 GameObject 被销毁，这里仍然执行，this == null 但代码不知道
    _image.sprite = loadedSprite;  // ← NullReferenceException
}

// ✅ 正确 — 用 async Task + CancellationToken，对象销毁时取消
private CancellationTokenSource _cts;

void OnEnable()  => _cts = new CancellationTokenSource();
void OnDisable() => _cts?.Cancel();
void OnDestroy() => _cts?.Dispose();

async UniTask LoadCardAsync(CancellationToken ct)  // 推荐 UniTask，没有则用 Task
{
    await UniTask.Delay(1000, cancellationToken: ct);
    if (ct.IsCancellationRequested) return;
    _image.sprite = loadedSprite;
}

// 流程控制（WaitForSeconds/逻辑等待）仍可用协程
IEnumerator WaitAndDo()
{
    yield return new WaitForSeconds(1f);
    DoSomething();
}
```

**规则：**
- `async void` 仅允许在事件处理器（如 `UnityEvent`）中使用，内部必须 try-catch
- MonoBehaviour 内异步逻辑一律 `async Task` / `async UniTask`，配合 `CancellationToken`
- 对象销毁前必须取消所有进行中的 Task（`OnDisable` 或 `OnDestroy` 中 `_cts.Cancel()`）

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

## 对象池（Object Pooling）

**在热路径里禁止用 `Instantiate` / `Destroy`。** 每次创建/销毁都触发 GC 分配、内存碎片化，在战斗中密集生成子弹、伤害数字、特效时，帧率会出现明显抖动。

```csharp
// ❌ 错误 — 热路径 Instantiate/Destroy，GC 压力大
void SpawnBullet()
{
    var bullet = Instantiate(bulletPrefab, spawnPos, Quaternion.identity);
    Destroy(bullet, 3f);
}

// ✅ 正确 — 用 Unity 内置对象池（Unity 2021+）
using UnityEngine.Pool;

private ObjectPool<GameObject> _pool;

void Awake()
{
    _pool = new ObjectPool<GameObject>(
        createFunc:    () => Instantiate(bulletPrefab),
        actionOnGet:   obj => obj.SetActive(true),
        actionOnRelease: obj => obj.SetActive(false),
        actionOnDestroy: obj => Destroy(obj),
        defaultCapacity: 20,
        maxSize: 100
    );
}

void SpawnBullet()
{
    var bullet = _pool.Get();
    bullet.transform.SetPositionAndRotation(spawnPos, Quaternion.identity);
    // 用完后归还
    StartCoroutine(ReturnAfterDelay(bullet, 3f));
}

IEnumerator ReturnAfterDelay(GameObject obj, float delay)
{
    yield return new WaitForSeconds(delay);
    _pool.Release(obj);
}
```

**需要对象池的典型场景：** 子弹、特效、伤害数字、音效 AudioSource、粒子系统实例。

---

## 纯逻辑抽离

尽早将不依赖 MonoBehaviour 的纯逻辑抽离到独立的 `.netstandard2.1` 类库，使用 `dotnet test` 跑，速度远快于 Unity 无头模式。

---

## LayoutGroup 使用判断规则
> ⚠️ [写入CLAUDE.md] 核心结论需同步到项目 CLAUDE.md 的"Unity 引擎常见坑"

**用 LayoutGroup 的唯一合理场景：** 子节点只需要静态排列，不需要任何位移/旋转/缩放动画。

| 场景 | 用 LayoutGroup？ |
|------|----------------|
| 设置菜单选项行、背包格子、商店列表 | ✅ 适合 |
| 对话框按钮组（确认/取消） | ✅ 适合 |
| 技能栏图标（静态排列） | ✅ 适合 |
| 手牌扇形展开 | ❌ 不适合 |
| 卡牌进场/离场动画 | ❌ 不适合 |
| 子节点需要旋转 | ❌ 不适合 |
| 子节点需要非均匀间距 | ❌ 不适合 |
| 子节点需要自定义尺寸（如圆形不被拉伸） | ❌ 不适合（`childControlHeight/Width=true` 会强制拉伸） |

**判断口诀：** 子节点只排列不动 → 用 LayoutGroup；子节点要动 → 自己算坐标。

---

## LayoutGroup 与位置动画互斥

Unity 的 LayoutGroup（HorizontalLayoutGroup / VerticalLayoutGroup / GridLayoutGroup）
在 `Canvas.willRenderCanvases` 阶段强制重算并覆盖所有子节点的 `anchoredPosition`，
该阶段发生在 `LateUpdate()` **之后**、渲染之前。

- 凡是需要在 `Update` / `LateUpdate` / DOTween 里手动控制 `anchoredPosition` 的节点，
  不得作为 LayoutGroup 的直接子节点

- 正确做法二选一：
  ① 移除容器上的 LayoutGroup，自己计算 `anchoredPosition`（TCG Engine 方案）
  ② 给子节点加 `LayoutElement.ignoreLayout = true` 脱离布局控制，
     但同时失去自动排列，需自行维护所有位置

- 常见踩坑场景：
  - 手牌扇形展开、卡牌拖拽、进场动画 → 手牌容器不挂 HorizontalLayoutGroup
  - 弹窗/提示从屏幕外飞入 → 直接放在 Canvas 根节点下，不放在任何 LayoutGroup 内
  - 拖拽时需要临时脱离：`SetParent(rootCanvas.transform)` + `LayoutElement.ignoreLayout`

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
- 子节点需要位移/旋转/缩放动画时，容器禁止挂 LayoutGroup；静态排列才用 LayoutGroup ⚠️ [写入CLAUDE.md]
- 收到任何视觉效果需求前必须先确认渲染管线（URP/HDRP/Built-in）和 Canvas 渲染模式；Canvas Overlay 下 URP 后处理完全无效，需改为 Screen Space - Camera ⚠️ [写入CLAUDE.md]

---

## 视觉效果前置检查规则
> ⚠️ [写入CLAUDE.md] 核心结论需同步到项目 CLAUDE.md 的"Unity 引擎常见坑"
> 视觉效果实现优先级（引擎原生 → Shader → 手动）见 UNITY-VISUAL-RULES.md

收到任何视觉效果需求（发光 / Bloom / 模糊 / 描边 / 后处理）前，必须先确认：

1. **渲染管线**：URP / HDRP / Built-in
   - Built-in → 没有 Post Process Stack v2 就没有 Bloom
   - URP / HDRP → 检查 PostProcessVolume 是否存在且 Profile 已配置

2. **Canvas 渲染模式**：
   - `Screen Space - Overlay` → Canvas 在后处理之后渲染，**URP Bloom 完全碰不到**
   - `Screen Space - Camera` → Canvas 走相机渲染管线，后处理生效 ✅
   - `World Space` → 同上，后处理生效 ✅

**Canvas Overlay + URP Bloom 的修法：** Canvas Render Mode 改为 `Screen Space - Camera`，指定主相机，后处理立即生效。

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

**动画工具选择原则：有固定终点用 DOTween，目标持续变化用 Update Lerp。**

### ✅ 用 DOTween 的场景
从 A 到 B 的一次性动画（起点终点明确、时长固定）：
- UI 弹窗、淡入淡出、滑动、缩放
- 数字滚动、进度条填充
- 卡牌出牌、翻转、攻击动画（移动到固定目标位）
- 镜头震屏、FOV 变化
- 颜色/alpha 渐变
- 多段序列（Sequence）

```csharp
// ✅ 一次性淡出 — DOTween
cg.DOFade(0f, 0.5f).SetEase(Ease.OutQuad);

// ✅ 多段序列 — DOTween Sequence
var seq = DOTween.Sequence();
seq.Append(card.DOMove(backPos, 0.3f).SetEase(Ease.OutQuad));
seq.Append(card.DOMove(targetPos, 0.1f).SetEase(Ease.InQuad));
seq.Append(card.DOMove(originPos, 0.3f).SetEase(Ease.OutBack));
seq.OnComplete(() => OnCombatEnd());
```

### ❌ 不用 DOTween，改用 Update/LateUpdate Lerp 的场景
目标每帧都可能变化，用 DOTween 需要每帧 kill + restart，反而更差：
- **持续追踪目标**：手牌扇形展开（每次加减牌目标坐标重算）、悬停跟随
- **拖拽预览**：跟随鼠标位置的平滑插值
- **弹簧/阻尼效果**：`Lerp(current, target, dt * speed)` 模式

```csharp
// ✅ 持续追踪 — Update Lerp（目标每帧可能变化）
void LateUpdate()
{
    _currentPos = Vector2.Lerp(_currentPos, _targetPos, Time.deltaTime * speed);
    rt.anchoredPosition = _currentPos;
}

// ❌ 错误 — 用 DOTween 追踪动态目标（每帧 kill/restart，性能差且不稳定）
void Update()
{
    DOTween.Kill(rt);
    rt.DOAnchorPos(_targetPos, 0.1f); // ← 每帧重启 tween，错误
}
```

### ❌ 不适用（使用 Animator Controller）
- 2D/3D 骨骼动画（Spine、骨骼 rig）
- 多状态切换状态机（idle → walk → run → jump）
- Blend Tree 混合动画

### DOTween 清理规则
- 所有 tween 必须加 `.SetTarget(gameObject)` 便于统一清理
- `OnDestroy` 中调用 `DOTween.Kill(gameObject)`

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

**脚本创建後，将以下规则写入项目 `CLAUDE.md`：**

```
## MCP 场景存盘
MCP 修改组件/GameObject 后必须立即调用 save_scene，防止 dirty 状态触发保存弹窗卡住 MCP。
注意：Build Game Scene（SceneBuilder）内部自带保存，调用后无需额外 save_scene。
```

> ⚠️ 注意：自动存盘脚本（hierarchyChanged + Undo.postprocessModifications）在时序上无法保证在 Unity 弹出保存弹窗之前完成保存，不可靠。**Unity MCP 项目必须手动 save_scene，不依赖自动存盘机制。**
