# 项目启动文档 — 移植开发

> 使用方式：填写完成后给 Claude Code，告诉它读取以下路径的 Skill 文件，然后按 Skill 流程开始。

---

## Skill 路径

- **Skill Index 路径**：E:\claudeCode\DOC\skills\migration\00-INDEX.md（例：/Users/me/skills/port/00-INDEX.md）
- **引擎规范文件路径**：E:\claudeCode\DOC\DEV_RULES\UNITY-RULES.md（例：/Users/me/skills/UNITY-RULES.md）
- **视觉规范文件路径**：E:\claudeCode\DOC\VISUAL_RULES\UNITY-VISUAL-RULES.md（例：/Users/me/skills/UNITY-VISUAL-RULES.md，没有就写"无"）

> 启动指令：`请读取 [Skill Index 路径] 和 [引擎规范文件路径]，然后按 Skill 流程开始。`

---

## 引擎路径

- **引擎可执行文件路径**：C:\Program Files\Unity\Hub\Editor\2022.3.62f3c1\Editor\Unity.exe（例：C:\Godot\Godot_v4.6.exe 或 /Applications/Unity/Unity.app/Contents/MacOS/Unity）

---

## 项目文件路径

- **原始代码路径**：E:\claudeCode\FWTCG_V3d_V9（原版项目的完整路径，例：E:\Projects\OriginalGame）
- **目标工程路径**：E:\claudeCode\unity\FWTCG_UNITY_V2（移植后的新项目路径，例：E:\Projects\NewGame）
- **美术资源路径**：E:\claudeCode\FWTCG_V3d_V9\tempPic（图片、音效、字体等资源的位置）
- **其他资源路径**：E:\claudeCode\FWTCG_V3d_V9（配置文件、数据文件等，没有就写"无"）

>启动后第一件事：把E:\claudeCode\DOC 路径下的 CLAUDE.md 复制到目标工程路径的根目录下，完成后提示用户：✅ CLAUDE.md 已复制到项目根目录。只读取目标工程路径下的 CLAUDE.md，忽略原始代码路径下的任何 CLAUDE.md 文件。（CLAUDE.md 模板路径例：E:\skills\CLAUDE.md）
---

## 基本信息

- **项目名称**：FWTCG_UNITY_V2
- **原始平台/框架**：Web JS（例如：Web JS / Unity 2019 / Flash）
- **目标引擎/框架**：Unity（Unity / Godot / Unreal / Web）
- **目标平台**：PC（PC / Mobile / Web / Console）
- **开发语言**：中文

---

## 移植方式

- **移植类型**：纯1:1移植+视觉升级（纯1:1移植 / 移植+视觉升级 / 移植+部分重设计）
- **视觉目标**：升级到目标引擎原生效果（保持原版视觉 / 升级到目标引擎原生效果）

---

## 已知 Bug

无
（原版有哪些已知问题，移植时是保留还是修复，没有就写"无"）

---

## 故意要改的地方

无
（有哪些地方不是1:1移植，而是要刻意修改或改进，没有就写"无"）

---

## 技术限制

无
（目标设备最低配置、性能要求、特殊限制，没有就写"无"）

---

## 不需要移植的功能

无
（原版有但这次不需要移植的功能，没有就写"无"）

---

## 其他备注

无
