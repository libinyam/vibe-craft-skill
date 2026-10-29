# 002 · 级联菜单：安全三角区 + 三个延时
**Cascading menus that don't lose their submenu: safe triangle + delays**

> 分类：交互 Interaction · 原帖：[2026-10-26](https://x.com/Kubai087/status/2114729985143677258) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，数值取自原帖；「实现要点」代码为本仓库示例。原帖附约 23 秒演示视频。评论区作者与网友提到，这一交互源自多年前 Amazon 商品分类菜单的经典设计（业界常称 "menu-aim"）。

## 原理

鼠标从父菜单斜着走向右侧子菜单时，必然会擦过下面几行。普通菜单"碰到就切换"，于是子菜单被切走、用户扑空。

解法：以鼠标**上一刻**的位置为顶点，连到子菜单靠近父菜单那条边的上、下两个角，形成一个三角形。指针在三角形内 = 正在奔向子菜单，此时路过的行一律不响应；离开三角形、或在某行停留够久（1.2s），才切换。再配合"延时打开"和"延时关闭"，菜单就会显得稳。

## 规则 / 数值

| 机制 | 数值 | 说明 |
|---|---|---|
| 打开延时 | **220ms** | 悬停某行 220ms 后才生效：文件夹行打开子菜单，普通行收起更深的子菜单；提前离开则取消。触摸输入立即生效。 |
| 安全三角 | 顶点 = 倒数第二次 `pointermove` 位置 | 记录最近两次指针位置，用较旧的一次做顶点，避免三角形退化成一个点。另一个条件：指针位于父菜单朝子菜单一侧的最后 `min(90px, max(56px, 父菜单宽 × 43%))` 范围内。 |
| 安全窗口 | 每 **80ms** 复查，最长 **1200ms** | 在安全区内悬停到别的行，不立刻切换；指针离开安全区就切，或从开始悬停起满 1200ms 也切（指针停住不动时也能拿到新子菜单）。 |
| 关闭延时 | **360ms** | 指针离开触发按钮、所有已打开层级和安全区后，360ms 再整体关闭；及时回来则取消。点击外部立即关闭。 |
| 拖拽悬停 | **420ms** | 拖动条目经过文件夹行时，用 420ms 而不是 220ms 打开。 |
| 结束保护 | — | 指针一旦进入已打开的子菜单，所有保护解除。 |

**高亮必须由组件状态驱动**（hovered / active / skipped），不能依赖 CSS `:hover`，否则安全区内被跨过的行仍会亮起。

### 结构与键盘
- Props：一个锚点（点击打开的触发按钮）+ 一棵条目树 `{ label, icon?, href?, children? }`；有 children 的是文件夹行，否则点击跳转。
- 每一层顶部用小标题显示所属文件夹名。
- 键盘：方向键移动；→ / Enter 打开文件夹；← 返回上一层；Esc 全部关闭并把焦点还给锚点。
- 角色：`menu` / `menuitem`，文件夹行带 `aria-expanded`。
- 已有菜单组件时，把这套逻辑并入其现有子菜单实现，保留 API、键盘与无障碍。

### 几何与动效
- 子菜单默认向右展开，与父菜单重叠 8px，首行与父行对齐；右侧空间不够时翻到左侧，安全区随之改用子菜单的右边缘。
- 每层宽 210–260px（随内容），最大高 `min(420px, 100vh - 24px)` 并独立滚动；内边距 6px，圆角 12px，1px 边框，`backdrop-filter: blur(18px) saturate(135%)`。
- 行高 32px，圆角 8px，行距 4px，padding `0 8px`，间隙 8px；图标 18px，文字 14px/500，文件夹行带 16px 右箭头。层标题 11px/600、弱化色、padding `4px 8px 2px`。
- 子菜单入场：从 `translateX(-8px) scale(0.88, 0.98)`、透明度 0 开始；透明度 170ms，transform 360ms，曲线 `cubic-bezier(0.2, 1.45, 0.35, 1)`（带轻微回弹，是手感的一部分，别去掉）。
- 根菜单从锚点以同样方式展开：`translateY(-6px) scale(0.96, 0.86)`，左上角为原点。
- 文件夹图标在子菜单打开时由"合上"变"打开"，460ms，`cubic-bezier(0.22, 1, 0.36, 1)`；两个图标结构相同就做路径变形，否则交叉淡入并让前片轻微抬起。

### 主题 token
| token | 亮色 | 暗色（中性灰，无蓝调） |
|---|---|---|
| surface | `rgba(255,255,255,.96)` | `rgba(34,34,34,.96)` |
| border | `rgba(0,0,0,.14)` | `rgba(255,255,255,.12)` |
| row ink | `#263140` | `#E4E4E4` |
| row hover | `rgba(15,23,42,.07)` | `rgba(255,255,255,.08)` |
| quiet ink | `#7A8491` | `#9A9A9A` |
| chevron | `#94A3B8` | `#8A8A8A` |
| shadow | `0 18px 34px rgba(15,23,42,.12), 0 2px 4px rgba(15,23,42,.05), inset 0 1px 0 rgba(255,255,255,.74)` | `0 18px 34px rgba(0,0,0,.42), 0 2px 4px rgba(0,0,0,.2), inset 0 1px 0 rgba(255,255,255,.06)` |

单色品牌图标在暗色下反色；切换主题时颜色、边框、阴影约 500ms 交叉过渡。

### 工程要求
- 所有时长集中成一组命名常量。
- 安全区判定写成纯函数并加单元测试（点在三角形内：三条边叉积同号）。
- 可选的调试叠层：画出安全区（蓝色 16% 填充 + 蓝色虚线描边）和正在计时的计时器，默认关闭。

## 实现要点（示例实现，非原帖代码）

```ts
// menu-timing.ts
export const MENU_TIMING = {
  openDelay: 220,
  dragOpenDelay: 420,
  closeDelay: 360,
  safeRecheck: 80,
  safeMaxWait: 1200,
} as const;

// safe-zone.ts —— 纯函数，便于单测
type P = { x: number; y: number };
const cross = (a: P, b: P, c: P) => (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);

export function pointInTriangle(p: P, a: P, b: P, c: P): boolean {
  const d1 = cross(a, b, p), d2 = cross(b, c, p), d3 = cross(c, a, p);
  const hasNeg = d1 < 0 || d2 < 0 || d3 < 0;
  const hasPos = d1 > 0 || d2 > 0 || d3 > 0;
  return !(hasNeg && hasPos);
}

/** submenuRect: 子菜单的 DOMRect；side: 子菜单在父菜单哪一侧 */
export function inSafeZone(
  pointer: P, apex: P, parentRect: DOMRect, submenuRect: DOMRect, side: 'right' | 'left',
): boolean {
  const edgeX = side === 'right' ? submenuRect.left : submenuRect.right;
  const top = { x: edgeX, y: submenuRect.top };
  const bottom = { x: edgeX, y: submenuRect.bottom };
  const band = Math.min(90, Math.max(56, parentRect.width * 0.43));
  const inBand = side === 'right'
    ? pointer.x >= parentRect.right - band
    : pointer.x <= parentRect.left + band;
  return inBand && pointInTriangle(pointer, apex, top, bottom);
}
```

用法要点：在 `pointermove` 里维护 `[prev, curr]` 两个点，判定时用 `prev` 当 `apex`；悬停新行时若 `inSafeZone` 为真，则启动 80ms 轮询，直到离开安全区或累计 1200ms 再切换。

## 可直接用的 Prompt（精简版）

> Build (or retrofit into the existing menu) a cascading menu with a safe triangle and three delays. Open delay 220ms (touch acts immediately; drag-over 420ms). While a submenu is open, form a triangle from the older of the last two pointer positions to the submenu's near-edge corners; if the pointer is inside it (and within the last min(90px, max(56px, 43% of parent width)) of the parent), ignore rows it crosses. A row hovered inside the zone switches only after the pointer leaves the zone or 1200ms pass (recheck every 80ms). Close the whole menu 360ms after the pointer leaves everything; outside click closes at once. Drive highlight from state, not :hover. Full keyboard support with menu/menuitem roles and aria-expanded. Submenus overlap parent by 8px, flip left when needed, spring-in on cubic-bezier(0.2, 1.45, 0.35, 1). Timings as named constants; safe-zone math as a pure unit-tested helper; optional debug overlay.

完整 Prompt（含全部几何、动效与主题数值）见[原帖](https://x.com/Kubai087/status/2114729985143677258)。

## 常见错误
- 用当前指针位置做三角形顶点 → 三角形退化，安全区失效。
- 只做安全区不做超时 → 指针停在三角区里不动时永远切不过去。
- 用 CSS `:hover` 画高光 → 被跨过的行依然闪烁。
- 子菜单与父菜单之间留缝 → 指针在缝里触发关闭（应重叠 8px）。
- 翻转到左侧后安全区仍用左边缘计算。
- 去掉回弹曲线 → 展开动作变得生硬。

## 出处
- 原帖：https://x.com/Kubai087/status/2114729985143677258
- 作者：[@Kubai087](https://x.com/Kubai087)
