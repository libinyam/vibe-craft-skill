# 004 · 递进式圆角 + 连续曲率圆角（超椭圆）
**Concentric (progressive) radii + continuous-curvature corners (superellipse)**

> 分类：形状与圆角 Shape · 原帖：[2026-10-28](https://x.com/Kubai087/status/2115469767650021796)（作者重发的修订版；初版为 [2115461268723360073](https://x.com/Kubai087/status/2115461268723360073)）· 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理；「实现要点」代码为本仓库示例。原帖附约 13 秒演示视频。

## 原理

**要点一 · 等距内缩（递进式圆角）**：表单、嵌套卡片里，内层元素和外层容器的圆角如果随手写，转角处的间距会忽宽忽窄，看起来"不规整"。让每一层都满足：

> **内圆角 = 外圆角 − 间距**（r = R − p），内外两条圆弧共用一个圆心

这样转角处的间距和直边处一样宽，层次就整齐了。（评论区有网友指出，直接对 AI 说"等距内缩"这个名词往往就够用；作者回复认可，可作为补充，同时仍给出完整版 Prompt。）

**要点二 · 连续圆角 / 超椭圆**：普通 `border-radius` 是一段圆弧直接接直线，曲率在接缝处突变，会有一道隐约的"折痕"。连续曲率圆角（类似 iOS 的圆角）过渡更顺。CSS 写法：

```css
border-radius: 32px;
corner-shape: superellipse(1.25);
```

原帖说明：Chrome / Edge 139+ 支持；Safari、Firefox 暂不支持，会自动退回普通圆角。

## 规则 / 数值

1. **由外向内推**：对话框 / 卡片 → 面板、输入框、按钮 → chip、tag。容器圆角 R、内边距 p 时，内层取 `R − p`。
   - 例：对话框 32、内边距 16 → 面板和按钮 16；16 的面板里内缩 8 的 chip → 8。
2. **同一层级共用一个圆角值。**
3. **最小 4px**，再小就不再递减。
4. **胶囊（pill）和圆形保持全圆**，不参与递减。
5. **所有 padding、gap、radius 落在 4px 栅格上。**
6. 项目有圆角 token 就改 token，不要硬编码。
7. **连续圆角做渐进增强**：保留 `border-radius`，在 `@supports (corner-shape: superellipse(1.25))` 里全局加 `corner-shape: superellipse(1.25)`；胶囊和圆形显式设 `corner-shape: round`。
8. 改完后输出清单：`元素: 旧值 -> 新值`。

## 实现要点（示例实现，非原帖代码）

```css
:root {
  --radius-dialog: 32px;
  --space-dialog: 16px;
  /* 内层 = 外层 − 间距，max() 保证不低于 4px */
  --radius-panel: max(4px, calc(var(--radius-dialog) - var(--space-dialog))); /* 16px */
  --space-panel: 8px;
  --radius-chip:  max(4px, calc(var(--radius-panel) - var(--space-panel)));   /* 8px */
}

.dialog { border-radius: var(--radius-dialog); padding: var(--space-dialog); }
.panel, .dialog .btn, .dialog input { border-radius: var(--radius-panel); }
.panel { padding: var(--space-panel); }
.chip  { border-radius: var(--radius-chip); }
.pill, .avatar { border-radius: 9999px; }

/* 连续曲率：渐进增强 */
@supports (corner-shape: superellipse(1.25)) {
  :where(*) { corner-shape: superellipse(1.25); }
  .pill, .avatar { corner-shape: round; }
}
```

Tailwind：在 `theme.extend.borderRadius` 里定义 `dialog: '32px', panel: '16px', chip: '8px'`，配合 `p-4`（16px）/ `p-2`（8px），保持 `rounded-dialog p-4 > rounded-panel p-2 > rounded-chip` 的配对关系。

> **补充提醒（本仓库整理）**：作者在发帖后转发了 [@MrWillCom](https://x.com/MrWillCom) 的补充——`superellipse()` 的参数会改变视觉上"等效"的圆角大小，因此在不支持的浏览器上回退到同一个 `border-radius` 数值时，圆角观感可能偏差较大；他的做法是用 Sass mixin 按参数换算回退值。若视觉一致性要求高，请在 Chrome 与 Safari 中分别比对。

## 可直接用的 Prompt（精简版）

> Make the corner radii on the screens/components I name consistent, then switch to continuous corners (ask if I haven't named them; edit radius tokens if they exist). Rule: inner radius = outer radius − padding (r = R − p), so nested corners share a center. Work outside-in: dialog/card → panels, inputs, buttons → chips/tags (e.g. dialog 32 + padding 16 → 16; chip inset 8 in a 16 panel → 8). Same level = same radius; never below 4; pills and circles stay fully round; keep padding, gap and radius on a 4px grid. Then add `corner-shape: superellipse(1.25)` inside `@supports` as progressive enhancement (pills/circles get `corner-shape: round`). Finish by listing every change as "element: old -> new".

原版 Prompt 见[原帖](https://x.com/Kubai087/status/2115469767650021796)。

## 常见错误
- 内外圆角写成同一个值（如都是 16）→ 内层转角显得"鼓"出来，间距在角上变窄。
- 内层圆角比外层还大 → 视觉上像是两张不相干的卡片叠在一起。
- 胶囊按钮也套用 R − p 公式 → 胶囊变成圆角矩形。
- 调大或调小 superellipse 参数后，只在 Chrome 里看效果 → Safari / Firefox 回退为普通圆角时观感偏差明显。
- 间距和圆角不在 4px 栅格上 → 递减后出现 6、10、14 这类零碎值。

## 出处
- 原帖（修订版）：https://x.com/Kubai087/status/2115469767650021796
- 初版：https://x.com/Kubai087/status/2115461268723360073
- 作者：[@Kubai087](https://x.com/Kubai087)
