# 009 · 提示框、跟随气泡、就地确认、抽屉与弹窗，到底怎么选？
**Which overlay to open: tooltip, cursor tip, popconfirm, drawer, or dialog**

> 分类：交互 Interaction · 原帖：[2026-11-02](https://x.com/Kubai087/status/2117314098165862612) · Prompt：[回复](https://x.com/Kubai087/status/2117314198430757283) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。原帖附约 40 秒演示视频，观感以视频为准。下面的数值只有提示框延迟（约 500ms）来自原帖；「实现要点」里的选型函数和延迟控制是本仓库示例。颜色、圆角、阴影、字体、间距、动效和既有文案保持不动，只改浮层种类和补上缺失的行为。

## 原理

浮层的质感不在阴影，而在**打断得恰如其分**。说明不该挡住操作，能收回的操作不该先问一句「确定吗」，收不回来的操作也不该只在角落里一闪而过。先问清这件事是补充文字、执行动作，还是填写表单，再选容器：提示框、跟随气泡、点击气泡、就地确认、弹窗、抽屉，或单独一页。

选错的代价是固定的：该就地问的做成整页弹窗，路径变长；该停下来的做成小气泡，影响范围写不下；该用抽屉对照页面的塞进弹窗，用户看不见自己正在改的东西。

## 规则 / 数值

先盘点页面上每一种浮层（提示框、hover card、气泡、确认气泡、抽屉 / sheet、弹窗），以及设计系统里对应的组件。能复用就复用。两种组件做同一件事时标成重复，不要再加第三套。设计系统里还没有撤销 toast 或点击气泡时，先问再补，不要直接新做一套。原帖把 hover card 列进盘点，没有单独给它一种结论：只含文字的按提示框核对，里面有操作的按点击气泡核对。

### 决策流

```
说明？
  ├─ 里面有可点击的操作 → 点击气泡 Popover
  ├─ 说明盖住一整块区域（图表、一块相对独立的图形） → 跟随气泡
  └─ 只解释指针 / 焦点下的那一个元素 → 提示框 Tooltip

操作？
  ├─ 做完还能撤销 → 直接执行，留下撤销（toast 或回收站）
  ├─ 不可逆，且是单条、用户自己的、触发器旁有锚点 → 就地确认 Popconfirm
  └─ 不可逆，且影响大、涉及他人、批量，或必须先做另一个决定 → 弹窗 Dialog

表单？
  ├─ 这件事就是主任务，或需要刷新后仍在、可以分享的地址 → 单独一页
  ├─ 表单长、填写时要对照页面，或需要更宽的横向空间 → 抽屉 Drawer
  └─ 表单短，不用看页面上的其他信息 → 弹窗 Dialog
```

没有悬停的设备上，提示框和跟随气泡都改成点按才出现。悬停能看到的内容，点按也必须能看到。

原帖把就地确认限定在「有锚点」的单条操作。不可逆、但是没有可以锚住的触发器时，原帖没有单列结论；先放进决策表的「拿不准」，不要擅自改成弹窗或气泡。

### 每种浮层做什么

| 种类 | 适合 | 不要用来做 |
|---|---|---|
| 提示框 Tooltip | 指针或键盘焦点下那一个东西的纯文字：被截断的长文、只有图标的按钮、控件为什么禁用 | 放按钮、链接或其他操作 |
| 跟随气泡 Cursor tip | 一整块区域里持续给出的信息，出现后贴着指针走；常见于图表和相对独立的一块图形 | 给单个按钮起名字 |
| 点击气泡 Popover | 说明里必须放可点击的内容；点击打开 | 只为了一句没有操作的说明 |
| 就地确认 Popconfirm | 不可逆的单条、且是用户自己的操作，问句锚在触发器上，路径短 | 批量、影响别人、必须停下来思考，或还能撤销的操作 |
| 弹窗 Dialog | 必须让用户停住：影响大、涉及他人、批量、还要先做另一个决定；也可以放很短、不用对照页面的表单 | 长表单，或填写时必须看见页面其余部分 |
| 抽屉 Drawer | 长表单、要对照页面上已有内容，或要更好利用横向空间。从某一边滑入，页面仍留在后面 | 任何二次确认 |
| 单独一页 | 这件事本身是当前的主任务，或表单需要一个刷新后还在、可以分享的地址 | 一句补充说明，或一次确认 |

抽屉、弹窗、独立页里的表单，有未保存的修改时，关掉或离开之前先问要不要丢掉。

### 行为清单

**提示框**

- 悬停**和**键盘焦点都会打开。
- 第一次出现前等待约 **500ms**；这一轮里已经有提示框开着、移到相邻触发器时立即切换。这一轮结束（没有提示框开着，焦点也不在触发器上）后，下一次再等约 500ms。
- 同时只开一个。
- 整块留在视口内，靠边时翻到放得下的一侧。
- 不接收指针事件，指针可以穿过去，避免挡住触发器造成闪烁。
- 用 `aria-describedby` 指到这段说明。

**跟随气泡**

- 指针进入目标区域才出现，贴在指针旁边，不盖住正在指的那个点；靠近视口边缘时翻面。
- 图表上：一条参考线跟着指针，读数吸附到最近的数据点。
- 触摸设备改为点按后显示。原帖没有给参考线的粗细和气泡与指针的间距，这些沿用图表上已有的样式，不要另造数值。

**点击气泡**

- 点击打开，里面放操作。悬停不要打开一个带按钮的层。
- 原帖没有单独写点击气泡的 Esc、点击外部和焦点归还。沿用已有 Popover 的键盘行为，不要另造一套。

**就地确认**

- 锚在触发器上，从触发器长出来。
- Esc、点击外部都是取消。
- 焦点进入气泡；关闭后焦点回到触发器。
- 确认按钮的名字是这个动作（「移除成员」），不要写「确定」。

**弹窗**

- 用设计系统里已有的模态；没有现成组件时，用原生 `<dialog>` 的 `showModal()`。
- 焦点困在弹窗里。Esc 是取消，不要接成确认。
- 文案写清谁被影响、改了什么、哪一部分收不回来。
- 按钮写出动作，不要写「确定」或 "OK"。

**抽屉**

- 从自己所在的那一条边滑入。背后的页面保持可见，正在编辑的内容即时反映到页面上。
- Esc 和一枚关闭按钮都能关掉。焦点移进抽屉，关闭后回到触发器。
- 不拿来做确认。

**撤销**

- 能撤销的操作立刻执行。撤销入口放在 toast 上，或放进回收站。两种都没有时，先问再加，不要临时造第三种入口。

### 无障碍

无障碍是上面清单的一部分，集中核对这几条：

| 浮层 | 最低要求 |
|---|---|
| 提示框 | 悬停与键盘焦点都能打开；同时只有一个；`aria-describedby`；指针穿透；触摸设备可以点按打开 |
| 跟随气泡 | 不盖住被指的点；触摸设备可以点按打开 |
| 点击气泡 | 点击打开；Esc、点击外部和焦点归还原帖未单列，沿用已有组件 |
| 就地确认 | 关闭后焦点回到触发器；Esc 与点击外部是取消；确认按钮名是动作 |
| 弹窗 | 焦点困住；Esc 取消；按钮名是动作；说清影响范围和不可逆的部分 |
| 抽屉 | 关闭按钮 + Esc；焦点进去再回来；背后页面仍看得见 |

### 改什么，留什么

只改两件事：浮层的种类，以及上面清单里页面还没有的行为。颜色、圆角、阴影、字体、间距、动效保持原样。文案也保持原样，只有两处例外：确认按钮的名字，以及那句「什么收不回来」。

### 先交表，再改

先交一张决策表，等确认之后再动代码。列：用途、出现位置、现在用的种类、建议的种类、一句理由。已经合适的标「已经正确」，吃不准的标「拿不准」。然后只改被点头的那些行，并列出动过的文件。

## 实现要点（示例实现，非原帖代码）

选型函数把决策流收成可单测的纯函数。`anchored: false` 的不可逆操作原帖没有给结论，函数返回 `unsure`，调用方应把它写进决策表，而不是直接换组件。

```ts
// overlay-choice.ts —— 本仓库整理
export type OverlayChoice =
  | 'tooltip'
  | 'cursor-tip'
  | 'popover'
  | 'do-and-undo'
  | 'popconfirm'
  | 'dialog'
  | 'drawer'
  | 'page'
  | 'unsure';

export type OverlayQuestion =
  | { task: 'explain'; scope: 'element' | 'region'; hasActions: boolean }
  | { task: 'act'; reversible: boolean; reach: 'single-own' | 'wide'; anchored: boolean }
  | { task: 'form'; length: 'short' | 'long'; needsPage: boolean; isMainTask: boolean };

export function chooseOverlay(q: OverlayQuestion): OverlayChoice {
  if (q.task === 'explain') {
    if (q.hasActions) return 'popover';
    return q.scope === 'region' ? 'cursor-tip' : 'tooltip';
  }
  if (q.task === 'act') {
    if (q.reversible) return 'do-and-undo';
    if (q.reach === 'wide') return 'dialog';
    return q.anchored ? 'popconfirm' : 'unsure';
  }
  if (q.isMainTask) return 'page';
  if (q.length === 'long' || q.needsPage) return 'drawer';
  return 'dialog';
}
```

`reach: 'wide'` 覆盖影响大、涉及他人、批量，以及必须先做另一个决定。`needsPage` 覆盖「要对照页面」和「要更宽的横向空间」。主任务优先于长短：主任务即使用很短的表单，也走单独一页。

提示框延迟。约 500ms 是原帖的量级；「相邻立即出现」按「这一轮尚未结束」来实现，原帖没有另给一个间隙时长，这里也不要加一个魔法数字。

```ts
// tooltip-delay.ts —— 本仓库整理
export const TOOLTIP_DELAY = { first: 500, next: 0 } as const;

export function tooltipShowDelay(sessionOpen: boolean): number {
  return sessionOpen ? TOOLTIP_DELAY.next : TOOLTIP_DELAY.first;
}
```

一轮从第一个提示框打开开始，直到没有任何提示框开着、焦点也不在触发器上才结束。`sessionOpen === true` 时移到相邻触发器，延迟为 0。

弹窗在没有设计系统模态时用原生对话框。Esc 和 `close()` 都是取消；确认走单独的按钮，不要绑在 `close` 事件上。

```js
const dialog = document.querySelector('#confirm-remove');
const opener = document.querySelector('#remove-member');

opener.addEventListener('click', () => {
  dialog.showModal(); // 焦点困在对话框内
});

dialog.addEventListener('close', () => {
  opener.focus();
});
```

确认按钮的可访问名称写动作，例如「移除成员」。就地确认同样在关闭时把焦点还给触发器，并从触发器的位置长出（`transform-origin` 落在锚点一侧）。已有组件做到了这些，就不要换一套。

## 可直接用的 Prompt（精简版）

> Before editing, list every overlay on the page (tooltip, hover card, popover, confirm bubble, drawer, sheet, modal) and the design-system component behind it. Reuse what exists; flag duplicates; ask before adding a missing undo toast or popover. Pick the kind from the task. Words about the single thing under the pointer or keyboard focus — clipped text, an icon-only button, why a control is disabled — are a tooltip. The same kind of note across a region, such as a chart, is a cursor-following tip. Anything clickable inside the note becomes a click-opened popover. If the action can be undone, do it and offer Undo on a toast or by trash. An irreversible change to one of the user's own items, with a trigger to anchor to, is a popconfirm; if there is no anchor, say so and wait. Use a dialog when the user must stop: large impact, other people, a bulk change, or another decision that has to happen first. A short form that does not need the rest of the page can also be a dialog. A long form, a form read against the page, or a form that needs horizontal room is a drawer, never a confirmation. The main task, or a form that needs a URL that survives reload and can be shared, is its own page. On touch, hover-only content must open on tap. Where behaviour is missing: tooltips open on hover and on focus, wait about 500ms the first time and show immediately when moving to a neighbour, stay on screen, let the pointer pass through, show one at a time, and use aria-describedby. A follow tip sits beside the pointer without covering the point and flips near the edges; on a chart a guide line tracks the pointer and the readout snaps to the nearest point. A popconfirm grows from its trigger, Escape or an outside click cancels, focus returns, and the confirm control names the verb. A dialog uses the existing modal or dialog.showModal(), traps focus, treats Escape as cancel, states who is affected and what cannot be undone, and names its buttons for the action. A drawer slides in from its edge, leaves the page visible, mirrors edits onto that page, closes with Escape and a close control, and moves focus in and back. Forms in a drawer, dialog, or page ask before discarding unsaved edits. Change only the kind and the missing behaviour. Leave color, radius, shadow, type, spacing, motion, and the rest of the copy alone. Return a table first (purpose, where it sits, current kind, recommended kind, one line of why; mark rows that are already right or unclear), wait for approval, then edit only the approved rows and list the files touched.

原版 Prompt 见[这条回复](https://x.com/Kubai087/status/2117314198430757283)。上面是改写，不是原文。

## 常见错误

- 带按钮的说明放进提示框 → 悬停层里的控件点不到，键盘也进不去。改点击气泡。
- 图表读数用一个不跟着走的提示框 → 扫数据时要反复进出。改跟随气泡，读数吸附到最近的点。
- 只有鼠标悬停、没有焦点，也没有点按 → 键盘和触摸设备看不到这段说明。
- 提示框接住指针事件 → 挡住触发器，悬停闪一下就消失。
- 同时开两个提示框，或第一次和相邻触发器都等满 500ms。
- 能撤销的删除先弹确认，页面上却没有撤销入口。
- 批量删除、或会影响到别人的操作，用一颗就地确认气泡 → 影响范围写不下，用户也停不下来。
- 用抽屉问「确定删除吗」 → 抽屉留给表单；确认要么就地，要么用弹窗。
- 长表单塞进弹窗，填写时看不见页面上要对照的内容；或者一句确认做成了整页。
- 主任务放在弹窗里，刷新或把链接发给别人就丢。
- 弹窗按钮写「确定」，Esc 被接成执行。
- 就地确认或抽屉关掉之后焦点回到 `body`。
- 抽屉打开后整页被压暗，背后的数据也不跟着改 → 失去边看边填。
- 关掉还有未保存内容的表单，不先问。
- 顺手改了圆角、阴影、颜色或无关文案。
- 没出决策表、也没等确认，就改了浮层。

## 出处

- 原帖：https://x.com/Kubai087/status/2117314098165862612
- Prompt 回复：https://x.com/Kubai087/status/2117314198430757283
- 作者：[@Kubai087](https://x.com/Kubai087)
- 本文是改写整理，不是原帖，也不是原版 Prompt。
