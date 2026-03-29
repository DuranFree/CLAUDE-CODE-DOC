# 项目启动文档 — 移植开发

> 使用方式：填写完成后给 Claude Code，告诉它读取以下路径的 Skill 文件，然后按 Skill 流程开始。

---

## Skill 路径

- **Skill Index 路径**：XXX（例：/Users/me/skills/port/00-INDEX.md）
- **引擎规范文件路径**：XXX（例：/Users/me/skills/UNITY-RULES.md）
- **视觉规范文件路径**：XXX（例：/Users/me/skills/UNITY-VISUAL-RULES.md，没有就写"无"）

> 启动指令：`请读取 [Skill Index 路径] 和 [引擎规范文件路径]，然后按 Skill 流程开始。`

---

## 引擎路径

- **引擎可执行文件路径**：XXX（例：C:\Godot\Godot_v4.6.exe 或 /Applications/Unity/Unity.app/Contents/MacOS/Unity）

---

## 项目文件路径

- **原始代码路径（模板）：XXX（模板的路径，架构和 UI 从这里来）
- **原始代码路径（游戏规则）：XXX（你原版游戏的路径，规则和逻辑从这里来）
- **美术资源路径：XXX（你自己的贴图和资源）
- **移植类型：混合移植——架构来自模板，游戏规则和贴图来自原版游戏
- **交接摘要路径**：XXX（例：E:\Projects\NewGame\handover.md，每次开新窗口读取这个文件继续）

---

## 基本信息

- **项目名称**：XXX
- **原始平台/框架**：XXX（例如：Web JS / Unity 2019 / Flash）
- **目标引擎/框架**：XXX（Unity / Godot / Unreal / Web）
- **目标平台**：XXX（PC / Mobile / Web / Console）
- **开发语言**：XXX

---

## 移植方式

- **移植类型**：XXX（纯1:1移植 / 移植+视觉升级 / 移植+部分重设计）
- **视觉目标**：XXX（保持原版视觉 / 升级到目标引擎原生效果）

---

## 已知 Bug

XXX
（原版有哪些已知问题，移植时是保留还是修复，没有就写"无"）

---

## 故意要改的地方

XXX
（有哪些地方不是1:1移植，而是要刻意修改或改进，没有就写"无"）

---

## 技术限制

XXX
（目标设备最低配置、性能要求、特殊限制，没有就写"无"）

---

## 不需要移植的功能

XXX
（原版有但这次不需要移植的功能，没有就写"无"）

---

## 其他备注

XXX
