# 022 · 空状态的三种画法
**Three ways to draw empty states**

> 分类：组件 Components · 原帖：[2026-11-21](https://x.com/Kubai087/status/2124170395825873348) · Prompt：[作者回复](https://x.com/Kubai087/status/2124170782716912111) · 作者 [@Kubai087](https://x.com/Kubai087)（枯白）
>
> 本文是对介绍帖、作者站点同一条技巧和附带 Prompt 的改写整理，不是原文，也没有照抄原版 Prompt。三种画法、基本结构、按空间选尺寸、文字空状态，以及翻译报错、统一标题语义、空间够时加说明，来自介绍帖。五种空的原因、四种尺寸的默认值、图示画法、动效、无障碍，以及先列表再改，来自作者站点同一条和官方 Prompt，下面会标明。只在演示视频里的文案不当成新规则。介绍帖时间为 2026-11-21 15:57:42 UTC，Prompt 回复为 2026-11-21 15:59:14 UTC，按本仓库约定把介绍帖记为 America/Los_Angeles 的 2026-11-21。没有英文原帖。原帖附约 52 秒演示视频。X 接口只返回了 Prompt 回复的前 280 字，完整 Prompt 以作者站点同一条为准：[空状态的诊断与改造](https://vibedesign.tips/zh/022-empty-states/)（英文页题为 Diagnosing and fixing empty states）。本文中文标题用介绍帖，英文标题用 Three ways to draw empty states。「实现要点」代码为本仓库示例。

## 原理

列表还没有内容、搜索没有结果、页面加载失败时，界面上留下来的那块就是空状态。同一类空会出现在整页、一张卡片或列表的一行里，可用空间差很多。一律按整页尺寸来做，卡片会被撑开；一页里同时出现几个大号空状态，也分不清主次。所以先按位置选画法和尺寸，再写文案和按钮。

Prompt 要求先诊断，列出每一处空，等确认再改。这是改产品里已经有的空状态，不是做一页演示。在现有代码、技术栈、组件、图标、token 和文案口气里改。没有问过，不加依赖，也不新造一套视觉样式。

## 规则 / 数值

先判断，再建议。点头的那些才改。

### 三种画法

这三条来自介绍帖。

1. **线框版。** 把当前页面收成缩小的线框，画出填满后的样子。人一眼能看出这里会放什么、从哪里开始。适合首次使用、收件箱、无权限这类需要交代页面结构的地方。
2. **图标版。** 用一个双色图标代表这里的内容，比如文件、评论、成员。比线框更轻，也更好画。适合对象明确的场景，以及加载失败、离线这类不必交代结构的情况。
3. **迷你版。** 小图标、标题和按钮排成一行，不放说明。适合侧边栏、仪表盘卡片这类空间有限的位置。

### 基本结构

完整的空状态依次是：图示、标题、说明、按钮。

| 部分 | 作用 |
|---|---|
| 图示 | 表示这里是什么 |
| 标题 | 说明现在的状态。空间再紧也不删 |
| 说明 | 补充原因或下一步 |
| 按钮 | 给出可以执行的操作 |

空间不够时，先去掉说明，再缩小图示，标题始终保留。

### 按空间选尺寸

介绍帖的分法：整页用完整结构，卡片用迷你版，列表行和菜单只留一行文字。尺寸跟空间走，不跟「空的原因」走：侧边栏卡片里的首次使用，仍是卡片尺寸；整页加载失败，才用整页尺寸。

官方 Prompt 把四种尺寸写成了默认值。产品里已经有尺寸，用产品的；还没有时，用这组作起点。

| 尺寸 | 用在哪 | 结构 | Prompt 给出的默认 |
|---|---|---|---|
| 整页 Page | 一页或一块大面板 | 图示、标题、说明、按钮，在区域内水平居中 | 整组垂直中点落在区域自上往下约 40% 处，略高于正中 |
| 卡片 Card | 仪表盘卡片、侧栏面板 | 一行：图示、标题，行尾一个 Ghost 按钮；不放说明 | 40×40 的图示槽，里面是 22px 双色图标 |
| 行 Row | 列表里某一组是空的，比如成员下面没有待处理邀请 | 一行，保持列表原来的行高：一行灰字，行尾可以跟链接或 Ghost 按钮 | 不放图示 |
| 菜单 Menu | 菜单、选择器、搜索建议 | 一行灰字 | 不放图示，也不放按钮 |

一页里有一处空列表、一处空的侧栏卡片时，分别用整页尺寸和卡片尺寸，不要做成两个整页空状态。页头、标签、搜索、筛选和卡片标题留在原位，只换内容区。

### 五种空的原因

介绍帖没有按原因分类。下面六种来自官方 Prompt，用来决定图示、文案和按钮，不用来决定尺寸。

**首次使用。** 还什么都没做过：新项目、新账号、第一个文件夹。画出填满后的样子，用线框版，或用这里会放的东西的双色图标。标题用几句话说清状态，例如「还没有任务」。说明用一句话说怎么开始。按钮：一个主要按钮做出第一件东西，例如「新建任务」；最多再给一种进入方式，例如「使用模板」或「导入」。第一件东西做出来之后，立刻占据空状态原来的位置，页面其余部分不要动。

**没有结果。** 搜索或筛选什么都没找到。产品并不是空的，不要写成空。搜索词和已生效的筛选保持原样，留在视野里。标题点出搜的是什么，例如「没有与 “invoice q3” 匹配的任务」。说明给一个改法，例如检查拼写、扩大搜索范围。按钮：「清除搜索」或「清除筛选」用次要按钮；如果还有范围更大的搜索，用 Ghost 按钮。图示要小，Prompt 写的是最多 80×60，也可以不放。搜索下拉或选择器里，只留一行灰字、不放图示，并随输入更新，例如「没有与 “invo” 匹配的任务」。

**都处理完了。** 收件箱清空、队列走完、没有通知、回收站是空的。直接说清，例如「都看过了」。说明里写这里接下来会出现什么。没有可做的事，所以不放按钮，图示里也不放强调色。

**加载失败。** 只出现在失败的那张卡片、那份列表或那块面板上。其余部分已经加载出来时，不要把整页换成空状态。加载中，这一块先显示和真实内容同形状的骨架屏（真实的行、真实图表的柱）；失败后，空状态出现在同一位置、同一尺寸。图示用双色警示图标，不用线框。标题说清什么没加载出来，例如「这张图表加载失败」。说明写发生了什么、内容是否还在，例如「连接中断，内容没有丢失」。按钮是「重试」，用次要按钮；重试时按钮上的刷新图标自己转；成功后，内容出现在空状态原来的位置。离线也归这一类：说明连接恢复后会同步，并提供「重试」。

**没有权限。** 人打开了自己可能看不到的东西。图示用线框画出门后面的内容，加上一把锁，不放强调色——进不进得去，是拥有者的决定。标题点出打不开的是什么，用它的名字。说明写谁拥有、怎么申请。按钮：「申请权限」作主要按钮，一个 Ghost 按钮回到来的地方。申请送出后，按钮就地变成「已申请」并带一个勾，说明改成接下来会发生什么。其余部分不要动。

**不必做空状态的地方。** 空了就收起来或隐藏的区域，例如关上的通知托盘、直接不渲染的区块。在表里标明这些位置，并保持原样。

### 按钮

介绍帖已经写下按尺寸选按钮：

- **大尺寸（整页为空）。** 空状态就是这一页的全部内容，下一步应当一目了然。一个主要按钮给最常用的操作，例如「新建任务」；一个次要按钮给另一种开始方式，例如「使用模板」或「导入」。最多两个，多了反而难选。
- **迷你版。** 通常出现在内容已经很多的页面里，页面本身已经有主要按钮。卡片里再用实心按钮，会和页面主操作抢注意力。改用链接或 Ghost 按钮。

Prompt 补了几条。按钮用动词，例如「新建任务」「清除搜索」「重试」。行和菜单里如果要操作，同样用链接或 Ghost，不要用实心主按钮。「都处理完了」不放按钮。

### 文字空状态

有些位置只需要一行字，不放图示。介绍帖的例子：

- **表格或列表。** 在原来的位置显示一行灰字，例如「暂无待处理的邀请」，末尾可以跟一个链接。
- **搜索下拉。** 显示「没有与 “invoice” 匹配的结果」，随输入更新。

保持原来的行高和位置，布局不要跳动。这和上面的「行」「菜单」尺寸是同一件事。

### 文案

介绍帖写了三条，Prompt 又补了几条。一起用。

1. **翻译后端报错。** 不要把接口返回的原文甩给用户，例如 `500 Internal Server Error`、`timeout`、`ECONNRESET`、堆栈。换成能看懂的话，说明哪里出了问题、内容有没有丢失、接下来怎么做，例如「连接中断，内容没有丢失」。原始细节留在日志里，或放在「详情」后面。产品有多语言时，这些文案要一起翻译。
2. **统一语义。** 同一个产品里的标题，要么都讲问题，例如「暂无任务」「没有匹配结果」；要么都讲解决方法，例如「创建第一个任务」「试试其他关键词」。选定产品已经更接近的那一种，全产品统一。不要混用。Prompt 还写了：不要道歉（不要「Oops」「哎呀」），也不要责怪用户。
3. **空间够就加说明。** 可以解释现状、安抚情绪（例如「内容没有丢失」），或直接给出解决方法。需要几步时，可以列出步骤。Prompt 把步骤上限写成三条。卡片、行和菜单尺寸不放说明。

另外，Prompt 要求：该出现空状态的地方，不要留白，不要只写一个「暂无数据」，也不要让转圈一直转下去。

### 图示

这些默认值来自官方 Prompt。产品里已有图标和颜色，用产品的；还没有时，用这组作起点。

- **两种画法，都从产品里来。** 图标：一个双色图标，整页 48px，卡片 22px。适合对象明确的位置（文件、评论、成员、日历、回收站），以及加载失败和离线。线框：当前这个界面填满后的缩小版。适合整页和大面板，界面的形状本身就能说明这里会放什么（第一份任务列表、收件箱、上了锁的项目）。不要用库存插画、人物、吉祥物或 emoji。
- **颜色。** 用比产品主色更安静的强调色：把主色往灰里拉，或用沉静的石板灰（浅色 `#7a869d`，深色 `#9aa4b8`）。图标的笔画用这个颜色，块面用同一颜色、大约 38% 叠在表面上。产品真正的强调色留给真正的按钮。
- **图标来源。** 先用产品图标库里的双色或填充样式。没有的话，可以建议 [Kubai Icons](https://icons.kubai.design)（MIT）：16 网格的双色图标，颜色跟 `currentColor`，有 React、Vue、原生 JavaScript 和静态 SVG。问过再加依赖。它的图标会在悬停时做出含义相关的动作；只在指针停在图标上时播放，不要自己动。
- **线框默认值。** 整页 `viewBox 0 0 120 90`；没有结果时 80×60。没有底、没有背景、没有斜线，直接坐在页面上。一个场景，两到三个物件，居中，四周至少留 14 的空隙。物件是表面色的卡片，圆角 10 到 11，1px 描边用线条色，阴影 `0 2px 3px`、6% 不透明。文字变成圆头占位条：正文高 4，标题高 5；空位是槽。这些灰色都是混进表面色里的浅灰：槽 14%，正文条 28%，标题条 52%。线形图标也用这层浅灰，线宽 1.5。最多一个强调色元素，而且只表示人在这里能做的下一步（添加、上传、重试）：一颗安静强调色的小圆钮，上面是白色标记。没有下一步，或这件事不该由当前这个人做：不放强调色。线框做成一份自包含的内联 SVG，颜色用产品 token 的 CSS 变量，以便跟随亮色和暗色；不要外部资源，不要 `<image>` 或 `<use>`，每个 id 带自己的前缀；阴影用 `feGaussianBlur` 加 `feOffset`，或 CSS `drop-shadow`，不要用 `feDropShadow`。暗色模式下改用暗色 token，检查槽和条仍然看得见，阴影不要变成一圈光晕。

加载失败用警示图标，不用线框。「都处理完了」和「没有权限」的图示不放强调色。

### 动效与无障碍

这些来自官方 Prompt，介绍帖没有写。

- **动效。** 空状态约 300ms 淡入，并上移 6px；内容到来时约 180ms 淡出，内容占据原来的位置。开启减弱动效时，不上移，只保留淡化。
- **无障碍。** 图示是装饰，加 `aria-hidden`。标题和说明是真实文字，不要做成图。搜索或筛选没有结果时，用礼貌的 live region 读出来。焦点留在原来的位置；如果是新打开的整页，焦点移到主要按钮。

### 先看产品，再动手

用几行把发现说清楚，再出表。不要先改代码。

找出每一处可能什么都没有的地方：列表和表格，看板和日历，收件箱和通知，搜索和筛选结果，仪表盘和图表卡片，文件夹和上传，评论和动态，成员和邀请，回收站和归档，没有选项的菜单和选择器，可能加载失败的页面，以及人可能没有权限打开的页面。

每一处记下：这里平时放什么；为什么会空（还没做过、搜索或筛选、都处理完了、加载失败、没有权限、离线）；空的区域有多大（整页、卡片、一行、菜单里的一行）；现在空的时候显示什么（空白、「暂无数据」、库存插画或 emoji、转个不停的圈）。

再找到共用的部分：有没有现成的 EmptyState，图标库是否有双色或填充样式，按钮样式，骨架屏和错误组件，亮色 / 暗色怎么切换。

表的每一行写：在哪、现在空的时候是什么、属于哪种空、用哪种尺寸、图示用图标还是线框（或没有）、建议的标题、说明和按钮文案。按空的原因分组。然后列出要新增或改动的共用部分（EmptyState 组件和尺寸、双色图标、先骨架再失败的模式），以及可能要加的依赖。等人选完再改。

改的时候：做一个带四种尺寸的 EmptyState，图示、标题、说明和最多两个操作做成 props，各处复用，不要各写一套标记。原来的加载、错误和数据逻辑留下，只改空的时候显示什么。文案用产品自己的口气和语言，并随产品一起翻译。

## 实现要点（示例实现，非原帖代码）

下面把官方 Prompt 的四种尺寸收成一组起点 token。项目已有组件库时，不要新建一套：扩展现有的空状态组件。还没有时，再做一个带 `size` 的可复用组件。数值来自 Prompt；CSS 变量名和结构是本仓库的。

```css
.empty {
  --empty-fade-in: 300ms;
  --empty-fade-out: 180ms;
  --empty-rise: 6px;
  --empty-icon-page: 48px;
  --empty-icon-card: 22px;
  --empty-tile: 40px;
  --empty-accent: #7a869d;          /* 暗色改用 #9aa4b8；优先换成产品 token */
  --empty-accent-body: color-mix(in srgb, var(--empty-accent) 38%, transparent);
}

.empty--page {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  /* 整组中点约在区域 40% 处：本仓库用 padding 近似，不是 Prompt 原文 */
  padding-top: 20%;
}
.empty--card,
.empty--row {
  display: flex;
  align-items: center;
  gap: 12px;
}
.empty--card .empty__tile {
  width: var(--empty-tile);
  height: var(--empty-tile);
  display: grid;
  place-items: center;
}
.empty--menu,
.empty--row .empty__title {
  color: var(--empty-muted, #6b7280); /* 「浅灰」的示例色，优先用产品 token */
}

@media (prefers-reduced-motion: no-preference) {
  .empty.is-in { animation: empty-in var(--empty-fade-in) ease both; }
  .empty.is-out { animation: empty-out var(--empty-fade-out) ease both; }
}
@keyframes empty-in {
  from { opacity: 0; transform: translateY(var(--empty-rise)); }
  to { opacity: 1; transform: none; }
}
@keyframes empty-out {
  to { opacity: 0; }
}
@media (prefers-reduced-motion: reduce) {
  .empty.is-in { animation: empty-fade var(--empty-fade-in) ease both; }
  .empty.is-out { animation: empty-out var(--empty-fade-out) ease both; }
}
@keyframes empty-fade { from { opacity: 0; } to { opacity: 1; } }
```

骨架屏失败后换空状态时，占同一块区域、同一尺寸，不要把整页掀掉。搜索无结果时，用 `aria-live="polite"` 读出标题；图示加 `aria-hidden="true"`。

Web 上可用 React、Vue 或 Svelte 做一个 `size` 属性；图示做成读取 CSS 变量的内联 SVG。若项目用 React Query 或 SWR，空的原因由查询状态决定（`isError`、空数组、正在搜索）。iOS 整页可用 `ContentUnavailableView`（搜索无结果用它的 search 样式），卡片和行用小号 `HStack`；分层渲染的 SF Symbols 可当双色图标。Android 用 Compose 的 `EmptyState(size, picture, title, description, actions)`。Flutter 用带尺寸枚举的 widget。这些平台写法是 Prompt 的建议，先核对本产品的技术栈和已有控件。

## 可直接用的 Prompt（精简版）

> Review every place in my product that can be empty. For each one, decide what kind of empty it is, how big it should be, and what it should say and show. Diagnose first: give me a table, and change only what I approve. Work in my code, on my stack, with my components, tokens, icons and copy. Do not add a dependency or a new visual pattern without asking. Values below are defaults to start from.
>
> Kind, not size. First use (nothing made yet): show a miniature wireframe of this screen filled in, or a two-tone icon of what lives here; title the state (No tasks yet); one sentence on how to start; one main button that makes the first thing (New task) and at most one other way in (Use a template, Import). When the first item is made, it takes the empty state's place at once. No results: the product is not empty, so do not say it is; keep the search text and filters in view; title what was searched (No tasks match "invoice q3"); offer one fix; Clear search or Clear filters as a secondary button, and a wider search as a ghost button if it exists; picture at most 80×60, or none; in a search menu or picker, one grey line that updates as the person types. All done: say so (You're all caught up), say what will appear next, no button, no accent in the picture. Failed: only in the card, list or panel that failed, never the whole page when the rest loaded; a same-shape skeleton first, then the empty state in the same place and size; a two-tone alert icon, not a wireframe; title what failed; say what happened and what is safe (The connection dropped. Nothing was lost.); Try again as a secondary button whose reload icon turns; offline is this kind too. No access: a wireframe of what is behind the door, with a lock and no accent; title it by name; say who owns it and how to get in; Request access as the main action, a ghost button back; after the request, the button becomes Request sent in place. Some areas just close or hide when empty — list those and leave them.
>
> Size follows the space, not the kind. Page or large panel: picture, title, description, buttons, centred, the group's middle about 40% of the way down. Card: one row, a 40×40 tile with a 22px two-tone icon, the title only, a ghost button at the end. Row: the list's own height, grey text, a link or ghost at the end, no picture. Menu or picker: one grey line, no picture, no button. Keep the page frame. One empty state per area, at that area's size. Never leave a place blank, never a bare "No data", never a spinner with no end.
>
> Structure is picture, title, description, buttons, in that order; leave out what is not needed, never add more. Titles follow one logic everywhere: all name the state, or all name the way forward. No apology, no blame. Description, when there is room: what happened, a reassurance, or the most direct fix, at most three steps. Map every backend error (500, timeout, ECONNRESET, a stack trace) to words that say what failed, whether anything was lost, and what to do; keep the raw detail in the logs or behind Details; translate these strings. Buttons are verbs, at most one main and one other; page sizes use primary then secondary; card, row and menu use a link or ghost.
>
> Pictures come from the product. Icon: 48px on a page, 22px in a card, for a clear object and for failed or offline. Wireframe: a small plain copy of this screen, for pages and large panels. No stock illustration, person, mascot or emoji. A quiet accent, quieter than the product's (muted slate #7a869d light, #9aa4b8 dark; the icon body at about 38%); the product's real accent stays on real buttons. Use my two-tone or filled icons; if I have none, recommend Kubai Icons (icons.kubai.design, MIT) and ask before adding it; play its hover motion only while the pointer is on the icon. Wireframe defaults: viewBox 0 0 120 90 on a page, 80×60 for no results; one scene, 2 or 3 objects, 14 clear on every side; cards in the surface colour, corner 10–11, 1px line, shadow 0 2px 3px at 6%; rounded placeholder bars (body 4, titles 5) and slots mixed into the surface (slots 14%, bars 28%, title bars 52%); line icons 1.5 wide in that grey; at most one accent mark, and only for the next step the person can take; one self-contained inline SVG on my tokens, no external resources, no image or use, prefixed ids, blur and offset instead of feDropShadow. In dark mode, the slots and bars stay visible and the shadow does not become a halo.
>
> Motion: fade in about 300ms with a 6px rise; fade out about 180ms when content arrives. Reduced motion: fade only. The picture is decorative (aria-hidden); title and description are real text; no-results is announced in a polite live region; focus stays put, or moves to the main button on a new page.
>
> Give me the table first. Then, for what I approve, one EmptyState with the four sizes and props for picture, title, description and up to two actions; keep each place's loading, error and data logic; write copy in the product's voice and languages. Check first use, a nonsense search, everything cleared, a failed and a slow load, and a link without access; nothing blank, nothing only "No data", no endless spinner, search text and filters survive; making the first item, clearing a search, a successful retry and a sent request each change the area in place; light and dark, a narrow phone, long translated strings, a screen reader, reduced motion; existing tests still pass.

上面是改写，不是原文。完整 Prompt 在[作者回复](https://x.com/Kubai087/status/2124170782716912111)里。X 接口只返回了这条回复的前 280 字，全文以作者站点同一条为准：https://vibedesign.tips/zh/022-empty-states/

## 常见错误

- 一律按整页尺寸做空状态，把卡片撑开，或一页里同时出现几个大号空状态。
- 侧边栏里的首次使用做成了整页尺寸；尺寸没有跟空间走。
- 迷你版也用实心主按钮，和页面已有的主操作抢注意力。
- 整页空状态放了两个以上的按钮，或按钮不是动词。
- 把接口原文（500、timeout、堆栈）直接显示给用户。
- 标题有的讲问题、有的讲解决方法，语义不统一。出现「Oops」「哎呀」，或责怪用户。
- 文字空状态改了行高或位置，布局跟着跳。
- 空间不够时删掉了标题。
- 搜索没有结果时写成「这里是空的」，或清掉了搜索词和筛选。
- 一块面板加载失败，却把整页换成空状态。加载中没有同形状的骨架屏，失败后尺寸对不上。
- 该出现空状态的地方是空白、只写「暂无数据」，或转圈一直不结束。
- 「都处理完了」还放了按钮或强调色。「没有权限」的线框里加了强调色。
- 用了库存插画、人物、吉祥物或 emoji。
- 没问过就加上 Kubai Icons 或其他依赖，或让图标自己循环动画。
- 表还没给人看就开始改。一次改掉原来的加载和数据逻辑。
- 给参数表和 Prompt 没写的颜色、字体或间距补上一套。`#7a869d` / `#9aa4b8` 只是没有产品色时的起点。
- 把视频里的示例文案或产品名写成必须出现的句子。

## 出处

- 介绍帖：https://x.com/Kubai087/status/2124170395825873348 （2026-11-21 15:57:42 UTC，按本仓库约定记为 America/Los_Angeles 的 2026-11-21）
- Prompt 回复：https://x.com/Kubai087/status/2124170782716912111 （2026-11-21 15:59:14 UTC。X 接口只返回前 280 字）
- 作者站点同一条（完整 Prompt）：https://vibedesign.tips/zh/022-empty-states/
- 英文页同一条：https://vibedesign.tips/022-empty-states/ （标题 Diagnosing and fixing empty states）
- 作者：[@Kubai087](https://x.com/Kubai087)（枯白）
- 没有英文原帖。原帖附约 52 秒演示视频。视频里的产品名和示例文案，没有写成新规则。
- 本文是改写整理，不是原帖，也不是原版 Prompt。示例里的 CSS 变量名、`padding-top: 20%` 的整页垂直位置近似，以及 `color-mix` 写法，是本仓库的。
