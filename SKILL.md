---
name: vibe-craft
description: Craft-level UI details that make a vibe-coded product feel designed rather than generated. Use when building or polishing UI components (buttons, menus, dropdowns, cascading/nested menus, drag-to-reorder / sortable lists, cards, dialogs, forms, inputs, login/landing pages, hero images), when adding visual texture effects (dither, ASCII, grain), when fixing corner radii or nested-card alignment, or whenever the user asks to make an interface feel more refined, tactile, polished, "有质感", or less "AI-looking". Based on @Kubai087's series 《999 个小技巧，让你 Vibe 的产品更有质感》.
---

# Vibe Craft · 让 Vibe 出来的产品更有质感

AI 生成的界面往往"能用但没质感"：按钮是一块平色、菜单一碰就乱跳、圆角层层打架、大图平铺直叙。这个技能收集的是一条条**具体、可量化**的小技巧——每条都给出原理、数值和实现方式，让你在写组件时把细节一次做对。

技巧来源：[@Kubai087（枯白）](https://x.com/Kubai087) 的 X 栏目《999 个小技巧，让你 Vibe 的产品更有质感》。本仓库是改写后的规则摘要，原帖附演示视频，强烈建议去看原帖。

## 使用方式（给 Agent）

1. **先匹配，再动手。** 当任务涉及下表任一类组件或效果时，先读对应的 `reference/` 文件，再写代码。不要凭印象实现。
2. **尊重现有体系。** 项目里已有组件库 / 设计系统 / design tokens 时，在原有组件和 token 上改造（新增 variant、改 token 值），保留其 API、props、键盘交互与无障碍；只有在没有现成组件时才新建。
3. **数值写成 token / 常量。** 文件里给出的数值（时长、阴影、圆角、阈值）集中定义成命名常量或 CSS 变量，不要散落成魔法数字。
4. **跟随主题。** 亮色 / 暗色都从 token 派生，不要为暗色另写一套组件。
5. **完成后自查。** 对照每个 reference 文件末尾的「常见错误」逐条检查。

## 核心原则

- **质感来自分层，而不是特效。** 一个"有手感"的表面 = 渐变面 + 描边 + 顶部高光 + 底部暗线 + 两层投影；每层都很克制。拒绝大面积发光、强模糊和 Web 2.0 式高光。
- **交互要猜意图。** 用户的手不是直线移动的；给移动路径留容错（安全区）、给打开 / 关闭留延时，界面才会显得"懂你"。
- **几何要自洽。** 嵌套元素的圆角要同心（内 = 外 − 间距），间距与圆角落在统一栅格上；能用连续曲率就别留折痕。
- **一切随尺寸缩放。** 边框、阴影偏移、按压位移都应按字号比例缩放，而不是固定像素。
- **动效有物理感但短。** 按压 / 悬停在 120–200ms 量级；展开类动画可以带一点回弹（overshoot），回弹本身就是手感的一部分。
- **效果服务内容。** 纹理类效果（Dither / ASCII）要加遮罩让画面退后，读起来像纹理而不是噪点；并且参数可调、可复位。
- **渐进增强。** 新 CSS 特性（如 `corner-shape`）用 `@supports` 包裹，旧浏览器自然回退。

## 技巧索引

### 组件 Components
| # | 技巧 | 什么时候读 |
|---|---|---|
| 001 | [拟物按钮：有点击手感的五层材质](reference/components/001-tactile-skeuomorphic-button.md) | 写 / 改任何 Button、按钮组、工具栏按钮 |

### 交互 Interaction
| # | 技巧 | 什么时候读 |
|---|---|---|
| 002 | [级联菜单：安全三角区 + 三个延时](reference/interaction/002-cascading-menu-safe-triangle.md) | 写 / 改下拉菜单、多级菜单、右键菜单、导航 mega menu |
| 005 | [拖拽排序：插入线 + 涟漪式避让](reference/interaction/005-drag-reorder-ripple.md) | 写 / 改拖拽排序、可排序列表、Dock、标签页重排 |

### 质感效果 Texture & Effects
| # | 技巧 | 什么时候读 |
|---|---|---|
| 003 | [Dither 与 ASCII 参数化效果](reference/effects/003-dither-ascii-effect.md) | 登录页 / 卡片 / Hero 图需要纹理感、复古像素感或科技感 |

### 形状与圆角 Shape
| # | 技巧 | 什么时候读 |
|---|---|---|
| 004 | [递进式圆角 + 连续曲率（超椭圆）](reference/shape/004-concentric-radius-superellipse.md) | 嵌套卡片、对话框、表单、任何"圆角套圆角"的布局 |

> 机器可读索引见 [`tips.json`](tips.json)。新技巧持续追加中（原作者每日更新）。

## 出处与致谢

所有技巧的创意与原始方案归 [@Kubai087](https://x.com/Kubai087) 所有。每个 reference 文件末尾都链接了原帖；如需完整原文、演示视频与原版 Prompt，请访问原帖并关注作者。
