# 005 · 更灵动的拖拽排序：插入线 + 由近及远的避让
**Livelier drag-to-reorder: a drop line plus a near-to-far ripple of neighbors**

> 分类：交互 Interaction · 原帖：[2026-10-29](https://x.com/Kubai087/status/2115823717024735708) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理；「实现要点」代码为本仓库示例。原帖附约 13 秒演示视频。

## 原理

常见的拖拽排序只是"拖到哪、列表就瞬间重排"，或者所有元素一起平移，缺少物理感。现实里把一样东西塞进一排物件中间时，紧挨着的会先被挤开、挤得最多，远处的晚一点、少一点——像一道涟漪向外扩散。

所以拖拽时要做三件事：**及时的光标反馈**、**一条清楚的落点提示线**、**落点两侧元素按距离递减地依次让开**。

## 规则 / 数值

1. **光标**：可拖拽元素悬停时显示指针手（`pointer` / `grab`），拖动中显示抓手（`grabbing`）。
2. **抬起**：被拖元素跟随指针，并"浮"出列表，不能挡住落点——横向排列时抬到上方；纵向列表时偏到一侧并缩小到约 90%。它原来的位置随即收拢。
3. **落点提示线**：在将要落下的缝隙里画一条细而淡的线（横排用竖线、竖列用横线），颜色约为文字色的 15%。
   - 出现时**从中间向两端展开**；
   - 落点改变时**滑动**到新缝隙，而不是闪现；
   - 落点就是元素原位时**不显示**。
4. **涟漪式避让**：以提示线为中心，两侧元素向外让开。设 n 为与线相隔的层数（紧挨着的 n = 0）：
   - **位移 = A × 0.7ⁿ**（图标类 A ≈ 10px，卡片类 A ≈ 8px）
   - **延迟 = 24ms × n**
   - 用一个快速、几乎不过冲的弹簧，约 0.25s 内停稳。越近动得越早、越多。
5. **放下**：元素落入缝隙，所有元素弹簧过渡到新位置。按 `Esc` 或拖到列表外放手 = 取消，一切弹回原位。
6. **每种运动用各自的曲线**：跟手用"手感"曲线，抬起用一个小小的 pop，归位用弹簧；不要全局一个 `ease-out`。
7. **只用 transform 做位移**，不触发布局重排；同时支持**键盘重排**；`prefers-reduced-motion` 下只保留提示线，去掉避让和弹簧动画。

## 实现要点（示例实现，非原帖代码）

```ts
// 计算每个元素相对插入位置的避让偏移（纵向列表示例）
const AMP = 8;          // 卡片 8px，图标 10px
const DECAY = 0.7;
const STAGGER_MS = 24;

function rippleOffsets(count: number, dropIndex: number) {
  // dropIndex：插入线位于第 dropIndex 个元素之前
  return Array.from({ length: count }, (_, i) => {
    const above = i < dropIndex;
    const n = above ? dropIndex - 1 - i : i - dropIndex; // 紧挨着的为 0
    const dist = AMP * Math.pow(DECAY, n);
    return { y: above ? -dist : dist, delay: n * STAGGER_MS };
  });
}
```

```css
.item {
  transition: transform 250ms cubic-bezier(0.34, 1.2, 0.64, 1); /* 轻微过冲的弹簧近似 */
  will-change: transform;
}
.drop-line {
  height: 2px;
  background: color-mix(in srgb, currentColor 15%, transparent);
  transform-origin: center;
  animation: line-in 160ms ease-out;
}
@keyframes line-in { from { transform: scaleX(0); } to { transform: scaleX(1); } }

@media (prefers-reduced-motion: reduce) {
  .item { transition: none; }
}
```

项目里已有拖拽库（dnd-kit、Framer Motion `Reorder`、SortableJS 等）时，在它的回调里套用上面的偏移和延迟即可，不要另起一套拖拽逻辑。

## 可直接用的 Prompt（精简版）

> Upgrade drag-to-reorder on the lists, rows or docks I name (reuse the project's drag helper). Cursor: grab on hover, grabbing while dragging. The dragged item follows the pointer, lifted clear of the drop spot (above a row; to the side and ~90% scale in a list); its old slot closes. Show a thin, faint drop line (~15% of text color) in the target gap; it grows from its center, glides between gaps, and hides over the item's own slot. Neighbors on both sides lean away: item n from the line (0 = adjacent) moves A × 0.7^n (A ≈ 10px icons, 8px cards), starting 24ms × n later, on a quick low-overshoot spring settling in ~0.25s. On drop everything springs into place; Esc or dropping outside cancels and springs back. Use distinct curves per motion, transforms only, keyboard reordering, and under reduced motion keep only the line.

原版 Prompt 见[原帖](https://x.com/Kubai087/status/2115823717024735708)。

## 常见错误
- 所有元素同时、同距离平移 → 看起来像整块滑动，没有"推挤"感。
- 用 `top` / `margin` 改位置 → 触发布局，拖动时掉帧。
- 被拖元素盖在落点上 → 用户看不到会落在哪里。
- 提示线瞬间出现、瞬间跳位 → 显得生硬；也不要在原位显示提示线。
- 忘了取消路径（Esc、拖出列表）和键盘重排。
- 无视 `prefers-reduced-motion`。

## 出处
- 原帖：https://x.com/Kubai087/status/2115823717024735708
- 作者：[@Kubai087](https://x.com/Kubai087)
