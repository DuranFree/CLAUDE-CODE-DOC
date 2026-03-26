# Port / Migration — Phase 2: 视觉分析 + 视觉升级指南

This phase runs before any porting begins. Its job is to answer one question:

> "What did the original platform force the developer to compromise on, and how does the target platform do it better?"

**This phase is mandatory for every migration. Never skip it.**

---

## 2.1 读取所有视觉文件

Read every file that defines the visual appearance of the original project. The file types depend on the source platform:

- **Web**: CSS files, HTML structure files, canvas/WebGL/shader code, JS visual effects
- **Unity**: Materials, Shaders, URP/HDRP settings, UI prefabs, animation controllers
- **Godot**: Themes, ShaderMaterial files, Environment resources, AnimationPlayer nodes
- **Unreal**: Materials, Post Process Volumes, UMG widget blueprints, Niagara systems
- **Any platform**: All constant definition files — scan for color values, font definitions, sizes, and any other visual-related constants. These are often hidden in constant or config files outside of the main style files (e.g. JS/TS constant files, C# ScriptableObjects, GDScript constants, UE5 DataAssets, ini config files). Every color, font, and size constant must be captured in the Visual Upgrade Guide.

Do not skim. Read thoroughly.

---

## 2.2 生成视觉清单

After reading all files, generate a complete visual checklist and present to the user:

> "以下是从原版整理的完整视觉清单，请确认没有遗漏。确认后开始生成视觉升级方案。"

```
视觉清单：
- [ ] 界面布局（每个界面的元素位置和层级）
- [ ] 颜色（背景、文字、边框、高亮等所有颜色）
- [ ] 字体（字体类型、大小、粗细、间距）
- [ ] 动画（角色、UI、场景动画）
- [ ] 特效（粒子、光效、技能特效等）
- [ ] 动效（缓入缓出、弹跳、震屏等运动感）
- [ ] 过渡效果（场景切换、界面淡入淡出等）
- [ ] UI元素样式（按钮、面板、图标的视觉风格）
- [ ] 音效触发点（哪些交互或事件会触发音效）
- [ ] 其他视觉表现（任何未归类的视觉细节）
```

Do not proceed until the user confirms.

**Once confirmed, both the feature checklist and visual checklist become the execution contract:**
- Work through each item one by one
- Mark each item as complete (✅) when done
- If an item cannot be completed, notify the user with the reason — never skip silently
- Do not move to the next Phase until all items are either completed or explicitly deferred by the user

---

## 2.3 识别原平台限制

For each visual effect found, classify into:

**A — Native strength**: Port directly.

**B — Workaround**: Original platform couldn't do this properly, developer used a hack. Examples vary by source platform:
- Web: `box-shadow` faking glow, CSS `perspective` faking 3D, Canvas JS faking particles
- Unity (older): sprite-based fake lighting, UI Image faking post-processing
- Godot: CanvasItem shader faking effects the engine handles natively in newer versions
- Any platform: anything that looks like a hack or approximation

**C — Missing entirely**: Effect doesn't exist because the platform can't support it.
- Real post-processing (Bloom, Color Grading, Vignette)
- Physics-based motion
- Real-time lighting and shadows
- GPU-accelerated particles

---

## 2.4 映射到目标平台原生方案

For every B and C effect, identify the target platform's best native solution.

#### If source is Web → Unity (URP)

| Original (Web) | Unity Native Solution |
|---|---|
| `box-shadow` glow | URP Bloom (Post-processing Volume) |
| `perspective` + `rotateY` card tilt | World Space Quad + real 3D rotation |
| CSS `transition` animations | DOTween / Animation Rigging |
| JS Canvas particles | Particle System (GPU Instanced) |
| `conic-gradient` border particle | Trail Renderer orbiting via DOTween Path |
| `repeating-linear-gradient` texture | Shader Graph procedural texture node |
| `radial-gradient` background | Shader Graph Radial Gradient node |
| `backdrop-filter: blur` | URP Camera Blur (Full Screen Pass) |
| `filter: brightness` hover | Shader `_EmissionStr` property |
| CSS `@keyframes` rotation | Shader `_Time` node (GPU-driven, zero CPU) |
| `mix-blend-mode: overlay` | URP Custom Blend Mode material |
| SVG animated paths | VectorGraphics package or Shader Graph |
| Screen-space vignette | URP Vignette (Post-processing Volume) |
| Film grain CSS filter | URP Film Grain (Post-processing Volume) |
| Color grading CSS filter | URP Color Adjustments (Post-processing Volume) |

#### If source is Web → Godot

| Original (Web) | Godot Native Solution |
|---|---|
| `box-shadow` glow | WorldEnvironment + GlowEffect |
| 3D card tilt | MeshInstance3D in SubViewport |
| JS particles | GPUParticles2D / GPUParticles3D |
| `radial-gradient` background | GradientTexture2D or custom shader |
| CSS animations | Tween / AnimationPlayer |
| SVG animated paths | Path2D + PathFollow2D |
| Screen vignette | ColorRect with shader overlay |
| Post-processing | WorldEnvironment (Environment resource) |
| Procedural textures | ShaderMaterial (GLSL/VisualShader) |

#### If source is Web → Unreal Engine

| Original (Web) | Unreal Native Solution |
|---|---|
| `box-shadow` glow | Post Process Volume → Bloom |
| 3D card tilt | StaticMesh in World Space + Blueprint rotation |
| JS particles | Niagara Particle System |
| `radial-gradient` background | Material Editor: RadialGradientExponential node |
| CSS animations | Sequencer / UMG Animations |
| Screen vignette | Post Process Volume → Vignette |
| Color grading | Post Process Volume → Color Grading LUT |
| Procedural textures | Material Editor function nodes |

#### If source is Web → Native Mobile (Swift/Kotlin)

| Original (Web) | Native Mobile Solution |
|---|---|
| `box-shadow` glow | CALayer shadowColor + Metal shader |
| CSS animations | Core Animation / ObjectAnimator |
| Particles | SceneKit particles / MotionLayout |
| `radial-gradient` | CAGradientLayer / GradientDrawable |
| Blur effect | UIBlurEffect / RenderEffect.createBlurEffect |

#### 其他任何平台组合

Apply the same analysis pattern:
1. Identify every visual effect in the original
2. Classify each as A, B, or C
3. For B and C: find the target platform's native equivalent
4. Document everything in the Visual Upgrade Guide

---

## 2.5 生成视觉升级指南

Create `./plans/visual-upgrade-<project-name>.md`:

```markdown
# Visual Upgrade Guide: <Project Name>
> Source platform: <e.g., Web / Unity 2019 / Godot 3>
> Target platform: <e.g., Unity URP 2022.3>
> Generated from: <list of source files read>

## Summary
<2-3 sentences: visual style goal and biggest upgrades vs original>

## Color Palette
| Variable/Name | Hex | Usage |
|---|---|---|
| ... | ... | ... |

## Layer Architecture
Describe rendering layer order from bottom to top.

---

## Effect-by-Effect Breakdown

### <Effect Name>
**Original implementation**: <how source achieved this>
**Original limitation**: <why this is a workaround>
**Target platform solution**: <exact tool/feature>
**Implementation notes**: <specific settings, parameters>
**Priority**: High / Medium / Low

---

## Implementation Order
1. <effect> — reason
2. ...

## Assets Required
| Asset | Type | Source | Notes |
|---|---|---|---|

## Known Gaps
Effects with no good equivalent in target platform and recommended fallback.
```

---

## 2.6 向用户确认

Show summary:
- How many effects found
- How many being upgraded vs ported as-is vs gaps
- Top 3 most impactful upgrades

Ask:
> "视觉升级方案已生成。是否有需要调整的地方？确认后开始移植。"

Do not proceed until the user confirms.

---

## Phase 2 完成后

写入开发日志，检查 git 状态，然后读取 `04-plan.md`。
