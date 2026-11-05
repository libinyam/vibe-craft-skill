# 011 · 如何选择更合适的动画节奏？
**Choose easing and duration by motion role**

> 分类：动效 Motion · 原帖：[2026-11-04](https://x.com/Kubai087/status/2118143759984803898) · Prompt：[续 1](https://x.com/Kubai087/status/2118143923571048817) · [续 2](https://x.com/Kubai087/status/2118144019545067872) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对原帖说明与两条续帖的改写整理，不是原文，也没有照抄原版 Prompt。曲线和时长取自原帖。原帖没有英文版。原帖附约 32 秒演示视频，用残影对照缓动曲线，观感以视频为准。「实现要点」代码为本仓库示例。

## 原理

曲线越陡，动作看起来越快。节奏不是装饰，它说明这件事是进来、离开、在屏上换地方，还是跟着手走。

匀速没有加减速，起和停都是硬的。ease-out 一开始就快、停下来变慢，进场才跟手。更强的 ease-out 起步更冲、尾巴更长，小东西落定最利落。ease-in 反过来，慢慢启动再加速离开。ease-in-out 两头慢，适合已经在屏幕上的东西挪位置。过冲会冲过终点大约 10% 再收回，只适合小的、带一点玩味的确认。弹簧会晃一两下才停住，用在松手之后。

全产品同一种角色只用一条曲线。CSS 的默认值 `ease` 不是一种选择。

## 规则 / 数值

先盘点每一段过渡和动画：CSS `transition`、`@keyframes`、Framer Motion、GSAP、WAAPI、`requestAnimationFrame` 循环。每条记下组件、触发、时长、曲线。三种情况标出来：用了默认 `ease`；位移用了 `linear`；进场用了 `ease-in`。

项目里已有动效 token 就改 token，不要在组件旁边再写一套魔法数字。

### 曲线 token

| token | 值 | 角色 |
|---|---|---|
| `--ease-out` | `cubic-bezier(0, 0, .58, 1)` | 进场 |
| `--ease-out-strong` | `cubic-bezier(.22, 1, .36, 1)` | 点击之后，小部件落定 |
| `--ease-in` | `cubic-bezier(.42, 0, 1, 1)` | 只用于离场 |
| `--ease-in-out` | `cubic-bezier(.42, 0, .58, 1)` | 已在屏上的位置或尺寸变化；不用于进场 |
| `--ease-overshoot` | `cubic-bezier(.34, 1.56, .64, 1)` | 小确认。大约冲过终点 10% 再回来 |
| `--ease-spring` | `linear(0, .32, .78, 1.04, 1.09, 1.06, 1.02, 1, .99, .99, 1)` | 拖拽或滑动之后松手 |
| `linear` | `linear` | 只给旋转指示、加载、进度条、跑马灯 |

`1.56` 是贝塞尔控制点，不是「冲过 56%」。过冲的幅度按原帖的大约 10% 来理解。

拖拽、滑动松手时，优先用能吃进松手速度的库弹簧（Framer Motion、GSAP 这类）。`--ease-spring` 是没有这条速度输入时的 CSS 写法。不支持 `linear()` 的环境，弹簧退回 `--ease-out-strong`，不要退回默认 `ease`。

### 时长 token

离场比同档进场短。弹簧和过冲大约是同档时长的 1.5 倍。

| token | 进场 | 离场 |
|---|---|---|
| `--duration-sm` | **150–250ms** | **120–200ms** |
| `--duration-md` | **250–350ms** | 比进场短。原帖没有另给区间 |
| `--duration-lg` | **350–500ms**（大块或全屏） | 比进场短。原帖没有另给区间 |

悬停和按下单独用 **100–150ms** 的 `--ease-out`，不套进场时长。

小号、中号、大号按元素尺度分。原帖没有给像素分界。过冲那条的大约 64px 是「不要用过冲」的上限，不是时长分档。尺度拿不准就问，不要自己切一刀。

### 按角色怎么动

| 角色 | 曲线 | 用在 |
|---|---|---|
| 进场 | `--ease-out` | toast、菜单、对话框、抽屉出现 |
| 点击后落定 | `--ease-out-strong` | 图标、文字、选中的 pill |
| 离场 | `--ease-in` | 对话框关闭、toast 离开 |
| 屏上移动或改尺寸 | `--ease-in-out` | 标签指示条、轮播、锚点平滑滚动 |
| 小确认 | `--ease-overshoot` | 复选框、喜欢、徽章 |
| 松手 | 带速度的弹簧，否则 `--ease-spring` | 抽屉回弹、下拉刷新、排序落下 |
| 匀速 | `linear` | 旋转指示、加载、进度条、跑马灯 |

进场从它最终所在的位置附近出发，距离 **8–24px**，缩放从 **96%** 收到 1。透明度略早于位移结束。toast 可以选加大约 **8px** 的模糊，在进场过程里散掉；别的进场不要顺手加模糊。

离场走同一条路径的反方向，用 `--ease-in`，时长更短。

屏上换位只用 `--ease-in-out`。刚出现的父元素，其子元素改用 `--ease-out-strong`，并错开 **100–200ms**。原帖同时用了 stagger 和 later，没有写明是整组只晚一次，还是相邻子元素各间隔 100–200ms。间隔落在这个区间里即可；要做成逐个子元素递增之前，先按页面上已有的错开方式，拿不准就问。

跟手阶段位移与指针 **1:1**，不加缓动。松手才把速度交给弹簧。

悬停和按下只动 `transform` 和透明度，**100–150ms**、`--ease-out`。悬停不要动画会触发布局的属性。

可动画的属性是 `transform` 和透明度；`filter` 少用（toast 那约 8px 模糊是点名的例外）。不要动画 `top`、`left`、`width`、`height`。

过冲不到对话框、抽屉、大约 **64px** 以上的物件，也不到高频操作上。

### 全产品一条，以及减弱动效

一种角色全产品一条曲线，不要每个组件各写一个贝塞尔。

`prefers-reduced-motion: reduce`：透明度留下，位移和缩放去掉。弹簧和过冲不再晃，改成一段普通的 **150ms** 淡化。原帖没有指定这段淡化的曲线，不要再套过冲或弹簧。

动画不挡住输入。不要等动画结束才接受下一次操作。新的操作从当前状态打断，不要把元素送回关键帧起点再重播。

### 交付清单

1. 列出找到的每段动画：组件、触发、现在的曲线、现在的时长、判定的角色、换上的曲线和时长。
2. 加上面的 token，再把调用点迁过去。
3. 列出改过的文件。
4. 不改「什么在动、动多远、什么触发」。曲线换成该角色的 token，时长收进对应档。
5. 角色判不清就问，不要猜一条曲线塞上去。

## 实现要点（示例实现，非原帖代码）

原帖点名的时长 token 是 `--duration-sm`、`--duration-md`、`--duration-lg`。下面三个进场取值是各区间里的一个点，不是唯一合法值。小号离场和按压更短，示例另用 `--duration-sm-out`、`--duration-press` 装这两个区间，这两个名字不是原帖里的。透明度比位移短一截，用来落实「略早结束」；原帖没有给提前量，`0.75` 是示例系数。

```css
:root {
  --ease-out: cubic-bezier(0, 0, .58, 1);
  --ease-out-strong: cubic-bezier(.22, 1, .36, 1);
  --ease-in: cubic-bezier(.42, 0, 1, 1);
  --ease-in-out: cubic-bezier(.42, 0, .58, 1);
  --ease-overshoot: cubic-bezier(.34, 1.56, .64, 1);
  --ease-spring: linear(0, .32, .78, 1.04, 1.09, 1.06, 1.02, 1, .99, .99, 1);

  --duration-sm: 200ms;       /* 150–250 */
  --duration-sm-out: 160ms;   /* 120–200，示例 token 名 */
  --duration-md: 300ms;       /* 250–350 */
  --duration-lg: 400ms;       /* 350–500 */
  --duration-press: 120ms;    /* 100–150，示例 token 名 */
}

@supports not (animation-timing-function: linear(0, 1)) {
  :root { --ease-spring: var(--ease-out-strong); }
}

.enter {
  transition:
    opacity calc(var(--duration-sm) * 0.75) var(--ease-out),
    transform var(--duration-sm) var(--ease-out);
}
.leave {
  transition:
    opacity var(--duration-sm-out) var(--ease-in),
    transform var(--duration-sm-out) var(--ease-in);
}

.toast-in {
  animation: toast-in var(--duration-sm) var(--ease-out) both;
}
@keyframes toast-in {
  from {
    opacity: 0;
    transform: translateY(16px) scale(0.96); /* 16px 落在 8–24px */
    filter: blur(8px);                       /* 只给 toast 这一类 */
  }
}

.pressable {
  transition: transform var(--duration-press) var(--ease-out),
              opacity var(--duration-press) var(--ease-out);
}

.drag { transition: none; } /* 跟手 1:1，松手后再交弹簧 */

@media (prefers-reduced-motion: reduce) {
  .enter,
  .toast-in,
  .pressable {
    animation: none;
    transition-property: opacity;
    transition-duration: var(--duration-sm);
  }
  .pressable { transition-duration: var(--duration-press); }
  .leave {
    animation: none;
    transition-property: opacity;
    transition-duration: var(--duration-sm-out);
  }
  .confirm,
  .release {
    animation: none;
    transition: opacity 150ms linear; /* 曲线原帖未指定；不用过冲、不用弹簧 */
  }
}
```

松手若走库的弹簧，把松手速度传进去。没有速度输入时用 `--ease-spring`；`linear()` 不可用时退回 `--ease-out-strong`。打断时先取消当前动画，再从计算后的 `transform` 接到下一段。这段接续是本仓库示例，原帖只要求从当前状态打断，并且不挡住输入。

## 可直接用的 Prompt（精简版）

> Inventory every transition and animation (CSS transitions and keyframes, Framer Motion, GSAP, WAAPI, requestAnimationFrame). For each, record the component, trigger, duration, and curve, and flag default `ease`, `linear` on movement, and `ease-in` on entrances. Add shared tokens and use one curve per role: `--ease-out: cubic-bezier(0, 0, .58, 1)` for entrances; `--ease-out-strong: cubic-bezier(.22, 1, .36, 1)` for small parts settling after a click; `--ease-in: cubic-bezier(.42, 0, 1, 1)` for exits only; `--ease-in-out: cubic-bezier(.42, 0, .58, 1)` for on-screen position or size changes, never entrances; `--ease-overshoot: cubic-bezier(.34, 1.56, .64, 1)` for small confirmations (about 10% past, then back), never on dialogs, drawers, anything over about 64px, or high-frequency actions; `--ease-spring: linear(0, .32, .78, 1.04, 1.09, 1.06, 1.02, 1, .99, .99, 1)` for release after a drag or swipe, preferring a library spring that takes release velocity, and falling back to `--ease-out-strong` where `linear()` is unsupported. `linear` only for spinners, loaders, progress bars, and marquees. Durations: small 150–250ms in and 120–200ms out, medium 250–350ms, large or full-screen 350–500ms, springs and overshoots about 1.5×, as `--duration-sm` / `--duration-md` / `--duration-lg`. Entrances ease out from where the element belongs, 8–24px, scale from 96%, opacity slightly ahead of movement, optional ~8px blur clearing on toasts only. Exits ease in along the reversed path and finish sooner. Children of a just-appeared element follow 100–200ms later on the strong ease-out; ask whether that delay is once for the group or between siblings. While a gesture tracks the pointer, move 1:1 with no easing; spring on release. Hover and press are 100–150ms ease-out and must not animate layout. Animate transform and opacity, filter sparingly, not top, left, width, or height. Under `prefers-reduced-motion`, keep opacity, drop movement and scale, and turn springs and overshoots into a plain 150ms fade. Never block input on an animation; a new action interrupts from the current state. Deliver the inventory (current curve and duration, assigned role and curve), then the tokens, the migration, and the files touched. Do not change what animates, the distance, or the trigger. Ask when the role is unclear.

上面是改写，不是原文。完整 Prompt 见原帖下的[续 1](https://x.com/Kubai087/status/2118143923571048817)和[续 2](https://x.com/Kubai087/status/2118144019545067872)。

## 常见错误

- 把 CSS 默认的 `ease` 留在位移上，当成已经选过曲线。
- 会移动的元素用 `linear`。匀速只留给旋转指示、加载、进度条、跑马灯。
- 进场用 `ease-in`，或用 `ease-in-out`。进场是 ease-out；屏上换位才是 ease-in-out。
- 对话框、抽屉、大约 64px 以上的面板，或高频操作，用过冲。
- 把控制点 `1.56` 当成冲过终点 56%。原帖对过冲的描述是大约 10%。
- 普通进场用弹簧。弹簧用在松手，并且优先带上松手速度。
- 跟手拖拽还套着缓动，松手却不看速度。
- 离场和进场一样长。离场更短；小号离场是 120–200ms。
- 动画 `top` / `left` / `width` / `height`，或在悬停时动画会触发布局的属性。
- 每个进场都加模糊。大约 8px、随进场散掉的模糊只给 toast。
- 从屏幕外飞入，或缩放不是从 96% 起。进场只离开终点 8–24px。
- `prefers-reduced-motion` 下仍在位移、缩放或回弹；或者把透明度也去掉了。弹簧和过冲在这里是 150ms 的普通淡化。
- 动画播完才允许点击，或新操作把元素打回起点再播。
- 迁移时改了距离、被动画的属性或触发时机。
- 角色判不清仍配了一条曲线。

## 出处

- 原帖：https://x.com/Kubai087/status/2118143759984803898
- Prompt 续 1：https://x.com/Kubai087/status/2118143923571048817
- Prompt 续 2：https://x.com/Kubai087/status/2118144019545067872
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 本文是改写整理，不是原帖，也不是原版 Prompt。时长变量的具体取值、透明度提前系数、淡化曲线和打断时的接续写法是本仓库示例。
