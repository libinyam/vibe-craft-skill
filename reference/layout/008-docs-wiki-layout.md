# 008 · 如何设计 Wiki 的框架和布局，兼顾 PC 和移动端体验
**A docs / wiki frame that holds up on desktop and on a phone**

> 分类：布局 Layout · 原帖：[2026-11-01](https://x.com/Kubai087/status/2116911391437443471) · Prompt：[回复](https://x.com/Kubai087/status/2116917025348231369) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，数值取自原帖；「实现要点」代码为本仓库示例。原帖附演示视频。只改这些区域怎么摆、怎么滚、怎么响应。文案、颜色、字体保持原样。原帖没写死的部分（断点像素、离场关键帧、阴影具体数值、完全透明带的宽度）在下文标明「本仓库整理」，不要当成原帖数值。

## 原理

产品一复杂，或者官网要堆 SEO 长文，就需要一套稳定的 Wiki / 帮助中心：读者能顺着目录把问题解决掉，页面本身也承担搜索流量。质感不来自再做一套皮肤，而来自**三层各滚各的**。

1. **正文和页脚跟着页面滚。** 它们是文档本身。
2. **页面列表和文内大纲是 `position: sticky`。** 吸顶偏移就落在它们静止时已经待着的位置上，所以一滚动不会先跳一下；滚到布局尽头，包含块会把它们顶上去，目录给页脚让路。
3. **顶栏先属于页面。** 它跟开头的 banner / hero 一起滚出屏幕；等这块首屏离开视口，顶栏才作为浮条回到画面上方。

窄屏以直板手机为主，侧栏塞不进去。章节切换改成底部一条常驻栏，点开后从底边升起一张清单，一行一条，拇指点得中。

文末按这个顺序收：先一张帮助卡（人还没解决，就近给一个真实渠道），再上一篇 / 下一篇，最后是一块**高于半屏**、里面真有用的页脚。帮助卡的样子贴着正文组件，读起来像文章的一部分，而不是插进来的横幅。

## 规则 / 数值

改共享的文档布局（顶栏、页面列表、大纲、页脚），让每一页继承。用 CSS 加少量原生 JS。不要引入滚动库或动画库。已有布局的左右列保持不动，只改滚动和响应。

### 顶栏：先滚走，再浮回来

信息结构三块：**Logo（回到官网）**、**关键页面的下拉**、**一个核心 CTA**。滚动过程中这三块都留在顶栏里。

- 停在页顶时顶栏是透明的，没有底色，跟首屏连在一起。
- 首屏（banner 或 hero）的底边到达或越过视口顶边时浮出。判定：`getBoundingClientRect().bottom <= 0`。
- 浮条：`position: fixed`；底为半透明白；`backdrop-filter: blur(18px)`；再加一层柔和的阴影。
- 进场 **320ms `ease-out`**：从 `translateY(-24px)`、`blur(8px)`、`opacity: 0` 进入。
- 读者滚回首屏时，**120ms** 内离场。离场的关键帧原帖没写（本仓库整理：示例做成进场的逆过程）。
- 离场进行中如果又往下滚、首屏再次离开视口：取消离场，浮条保持显示，不要把 320ms 进场再放一遍。
- 标题加 `scroll-margin-top`（至少等于浮条高度），锚点不要落在浮条底下。
- 页面没有 banner / hero 时，原帖没写以哪一块为准。先确认测量对象，不要另造一条出现规则。
- Safari 等浏览器会按顶栏背景给地址栏着色。浮条的底色要和品牌放在一起仍然协调；原帖的浮条底是半透明白，品牌若不是白，就改成同明度、能跟品牌共处的底，不要让地址栏被染成一块无关的白。

### 页面列表：自己滚，并标出「外面还有」

列表单独占一段高度，和正文的滚动分开。藏掉滚动条，让它在自己的盒子里滚。

- 某一边外面还有内容时，才在那一边做渐隐。滚动和 resize 时切换 `data-more-above` / `data-more-below`，用它们驱动 `mask-image: linear-gradient(...)`。
- 渐隐带大约 **顶部 40px、底部 56px**。每一边的最外沿还有一条很短的**全透明**带，避免被切到一半的行剩下一道浅线。全透明带的宽度原帖没写（本仓库整理：示例取 8px）。
- 有剩余内容的那一边，放一枚小圆箭头，相对列表水平居中。点击让列表滚动**自身可视高度的 60%**，平滑滚动。
- 箭头只给鼠标用：`tabindex="-1"`、`aria-hidden="true"`。键盘焦点在列表里移动时，列表自己会跟着滚。
- 加载时把当前页带进可视区。同时把列表的 `scrollTop` 记在 `sessionStorage`，换一篇时不要回到顶部。两件事一起做时（本仓库整理）：先恢复上次的滚动位置；当前页不在可视范围内，再把它滚进来。
- 文档壳上如果有搜索框，按 `/` 聚焦它。焦点已经在输入框、文本域或可编辑区域时，不要抢走。

渐隐、箭头和 `sessionStorage` 是页面列表的规则。大纲只要求 sticky 和当前节高亮，不要把箭头和遮罩照搬过去。

### 大纲：标出正在读的那一节

当前节 = 视口顶部往下 **20%** 那条线之上、最后一个标题。

- 对应链接设 `aria-current="location"`。
- 样式只动两处：`border-left-color` 和文字颜色，都用强调色。左边框先占好宽度，高亮时只改颜色，文字不会横跳（本仓库整理）。
- 在滚动时更新，**每帧最多一次**（`requestAnimationFrame`）。
- 底部栏右半边显示的「正在读的标题」用同一次计算，两处不要各算各的。

### 文末：帮助卡，然后上一篇 / 下一篇，然后页脚

顺序固定：

1. **帮助卡。** 读者可能还没解决。给一到两个真实渠道（issue、社群、社媒、联系入口里页面上确实有的）。卡片样式贴近正文已有组件。没有的渠道不要做占位按钮。
2. **上一篇 / 下一篇**，链上各自的标题。第一篇的「上一篇」回到首页。
3. **页脚。** 高度**超过半个视口**。用背景图把这块撑起来，让人愿意滑到底并停一下；里面放能帮读者解决问题的信息。只有大图、没有信息，高度会变成负担。

### 窄屏

- **平板**：藏起侧栏里的大纲。**手机**：再藏起侧栏里的页面列表。原帖没给断点像素（本仓库整理：示例用 960px / 640px，按项目已有断点替换）。
- 这两档都用一条 `position: fixed` 的底部栏：左半边打开页面列表，右半边打开大纲。右半边的文案是正在读的节名，过长用省略号截断。
- 点左半边或右半边，用 `<dialog>` 的 `showModal()` 从底边升起对应清单。进场是向上 **24px** 的同时淡入，背后不要压暗。进场时长原帖没写（本仓库整理：示例 200ms）。
- 打开时标出当前项，并 `scrollIntoView({ block: "center" })`。原帖这里没有写 smooth。
- 点一条、点关闭、点清单外部、或按 Esc，都关闭。点到某一节的链接后，滚到那个标题。
- 清单行的点击热区比桌面列表大一号。原帖没给像素，跟着桌面行高升一档即可。
- 一行一条，铺满清单宽度。

### 减弱动效

`prefers-reduced-motion: reduce` 时：关掉平滑滚动（含列表箭头和锚点），关掉顶栏的进场和离场动画。浮条改为瞬间出现、瞬间离开。原帖没有要求关掉清单的 24px 进场。

### 做完核对

列出改过的每个文件和选择器。在真实页面上看四个状态：页顶、读到中段、文末、手机宽度。原帖还要求每个状态一张截图。

## 实现要点（示例实现，非原帖代码）

断点、阴影、白色透明度、离场关键帧、透明带宽度、存储 key、进场时长都是示例，见各处注释。项目里已有文档布局就改那一套，不要另起一个页面。

```html
<header class="docs-header">
  <a class="docs-logo" href="/">Logo</a>
  <nav class="docs-keypages" aria-label="关键页面"><!-- 下拉菜单 --></nav>
  <a class="docs-cta" href="/start">开始使用</a>
</header>
<div class="docs-hero"><!-- 已有的 banner / hero，没有就不要造 --></div>

<div class="docs-frame">
  <nav class="page-list" aria-label="页面">
    <button class="page-list-jump" data-dir="-1" tabindex="-1" aria-hidden="true">↑</button>
    <div class="page-list-scroll">
      <a href="/docs/a" aria-current="page">当前页</a>
      <a href="/docs/b">下一页</a>
    </div>
    <button class="page-list-jump" data-dir="1" tabindex="-1" aria-hidden="true">↓</button>
  </nav>

  <article class="docs-article">
    <h2 id="install">安装</h2>
    <!-- 正文 -->
    <aside class="docs-help">
      <p>还没解决？</p>
      <a href="/community">去社群问</a>
    </aside>
    <nav class="docs-pager" aria-label="上下篇">
      <a href="/">上一篇 · 首页</a>
      <a href="/docs/b">下一篇 · 配置</a>
    </nav>
  </article>

  <nav class="outline" aria-label="本页大纲">
    <a href="#install">安装</a>
  </nav>
</div>

<footer class="docs-footer"><!-- 有用的链接和说明，高度靠背景图撑开 --></footer>

<div class="docs-mobile-bar">
  <button type="button" data-sheet="pages">全部页面</button>
  <button type="button" data-sheet="outline"><span class="docs-mobile-section">安装</span></button>
</div>
<dialog class="docs-sheet" id="sheet-pages"></dialog>
<dialog class="docs-sheet" id="sheet-outline"></dialog>
```

```css
.docs-header {
  /* 页顶：透明，留在文档流里，跟 hero 一起走 */
  background: none;
  box-shadow: none;
}
.docs-header.is-floating {
  position: fixed;
  inset: 0 0 auto 0;
  z-index: 20; /* 本仓库整理 */
  background: rgb(255 255 255 / 72%); /* 本仓库整理：原帖只说半透明白 */
  backdrop-filter: blur(18px);
  box-shadow: 0 8px 24px rgb(0 0 0 / 8%); /* 本仓库整理：原帖只说柔和阴影 */
}
.docs-header.is-entering {
  animation: docs-header-in 320ms ease-out;
}
.docs-header.is-leaving {
  animation: docs-header-out 120ms ease-out forwards;
}
@keyframes docs-header-in {
  from { transform: translateY(-24px); filter: blur(8px); opacity: 0; }
}
@keyframes docs-header-out {
  to { transform: translateY(-24px); filter: blur(8px); opacity: 0; }
}

.docs-article :is(h2, h3, h4) {
  scroll-margin-top: var(--docs-header-h, 64px); /* 64px 为浮条高度的示例回退 */
}

.page-list,
.outline {
  position: sticky;
  /* 等于它们静止时已经在的位置，开始滚动才不会跳 */
  top: var(--docs-sticky-top, 0px);
  align-self: start;
}
.page-list { display: grid; }
.page-list-scroll,
.page-list-jump { grid-area: 1 / 1; }
.page-list-scroll {
  --fade-clear: 8px;   /* 本仓库整理 */
  --fade-top: 40px;
  --fade-bottom: 56px;
  max-height: calc(100vh - var(--docs-sticky-top, 0px) - 32px); /* 32px 余量为本仓库整理 */
  overflow: auto;
  scrollbar-width: none;
  mask-image: none;
}
.page-list-scroll::-webkit-scrollbar { display: none; }
.page-list-scroll[data-more-above] {
  mask-image: linear-gradient(
    to bottom,
    transparent 0,
    transparent var(--fade-clear),
    #000 var(--fade-top)
  );
}
.page-list-scroll[data-more-below] {
  mask-image: linear-gradient(
    to bottom,
    #000 calc(100% - var(--fade-bottom)),
    transparent calc(100% - var(--fade-clear)),
    transparent 100%
  );
}
.page-list-scroll[data-more-above][data-more-below] {
  mask-image: linear-gradient(
    to bottom,
    transparent 0,
    transparent var(--fade-clear),
    #000 var(--fade-top),
    #000 calc(100% - var(--fade-bottom)),
    transparent calc(100% - var(--fade-clear)),
    transparent 100%
  );
}
.page-list-jump {
  display: none;
  z-index: 1;
  justify-self: center;
  align-self: start;
  width: 28px;
  height: 28px;
  border: 0;
  border-radius: 999px;
  background: var(--bg, #fff); /* 本仓库整理：原帖只说小圆箭头 */
}
.page-list-jump[data-dir="1"] { align-self: end; }
.page-list:has(.page-list-scroll[data-more-above]) .page-list-jump[data-dir="-1"],
.page-list:has(.page-list-scroll[data-more-below]) .page-list-jump[data-dir="1"] {
  display: grid;
  place-items: center;
}

.outline a {
  border-left: 2px solid transparent; /* 先占宽度，高亮不横跳 */
}
.outline a[aria-current="location"] {
  color: var(--accent);
  border-left-color: var(--accent);
}

.docs-help {
  /* 贴近正文已有卡片：圆角、边框、内边距走同一套 token */
}
.docs-footer {
  min-height: 70vh; /* 本仓库整理：规则是「超过半屏」，70vh 只是示例 */
  background: url("/images/footer-bg.webp") center / cover no-repeat; /* 换成页面已有的图 */
}

.docs-mobile-bar { display: none; }
.docs-mobile-section {
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 本仓库整理：原帖只区分平板和手机，没有给像素 */
@media (max-width: 960px) {
  .outline { display: none; }
  .docs-mobile-bar {
    display: grid;
    grid-template-columns: 1fr 1fr;
    position: fixed;
    inset: auto 0 0 0;
  }
}
@media (max-width: 640px) {
  .page-list { display: none; }
  .docs-sheet a { padding-block: 14px; } /* 比桌面行大一档的示例 */
}

.docs-sheet {
  margin: auto 0 0 0;
  width: 100%;
  max-height: 80vh; /* 本仓库整理：原帖只说从底边升起，没有写高度 */
  border: 0;
}
.docs-sheet::backdrop { background: none; }
.docs-sheet[open] { animation: docs-sheet-in 200ms ease-out; } /* 200ms 为本仓库整理 */
@keyframes docs-sheet-in {
  from { transform: translateY(24px); opacity: 0; }
}

@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  .docs-header.is-entering,
  .docs-header.is-leaving { animation: none; }
}
```

```js
const header = document.querySelector('.docs-header');
const hero = document.querySelector('.docs-hero');
const list = document.querySelector('.page-list-scroll');
const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
const SCROLL_KEY = 'docs-page-list-scroll'; // 本仓库整理：原帖没指定 key

function heroGone() {
  return !!hero && hero.getBoundingClientRect().bottom <= 0;
}

let floating = false;
let leaving = false;

function onLeaveEnd(event) {
  if (event.animationName !== 'docs-header-out') return;
  header.removeEventListener('animationend', onLeaveEnd);
  if (!leaving) return;
  floating = false;
  leaving = false;
  header.classList.remove('is-floating', 'is-leaving');
}

function showHeader() {
  leaving = false;
  header.classList.remove('is-leaving');
  if (floating) return;
  floating = true;
  header.classList.add('is-floating');
  if (!reduce) header.classList.add('is-entering');
}

function hideHeader() {
  if (!floating || leaving) return;
  if (reduce) {
    floating = false;
    header.classList.remove('is-floating', 'is-entering', 'is-leaving');
    return;
  }
  leaving = true;
  header.classList.remove('is-entering');
  header.classList.add('is-leaving');
  header.addEventListener('animationend', onLeaveEnd);
}

function syncHeaderBox() {
  if (!header) return;
  document.documentElement.style.setProperty('--docs-header-h', `${header.offsetHeight}px`);
}

function syncHeader() {
  if (!header) return;
  if (heroGone()) showHeader();
  else hideHeader();
}

addEventListener('scroll', syncHeader, { passive: true });
addEventListener('resize', syncHeaderBox);
syncHeaderBox();
syncHeader();

if (list) {
  function syncListEdges() {
    const above = list.scrollTop > 1;
    const below = list.scrollTop + list.clientHeight < list.scrollHeight - 1;
    list.toggleAttribute('data-more-above', above);
    list.toggleAttribute('data-more-below', below);
    sessionStorage.setItem(SCROLL_KEY, String(list.scrollTop));
  }

  const saved = sessionStorage.getItem(SCROLL_KEY);
  if (saved != null) list.scrollTop = Number(saved);
  const currentPage = list.querySelector('[aria-current="page"]');
  if (currentPage) {
    const top = currentPage.offsetTop;
    const bottom = top + currentPage.offsetHeight;
    const inView = top >= list.scrollTop && bottom <= list.scrollTop + list.clientHeight;
    if (!inView) currentPage.scrollIntoView({ block: 'nearest' });
  }
  syncListEdges();
  list.addEventListener('scroll', syncListEdges, { passive: true });
  addEventListener('resize', syncListEdges);

  document.querySelectorAll('.page-list-jump').forEach((button) => {
    button.addEventListener('click', () => {
      list.scrollBy({
        top: list.clientHeight * 0.6 * Number(button.dataset.dir),
        behavior: reduce ? 'auto' : 'smooth',
      });
    });
  });
}

document.addEventListener('keydown', (event) => {
  if (event.key !== '/' || event.metaKey || event.ctrlKey || event.altKey) return;
  const target = event.target;
  if (target instanceof HTMLElement && target.closest('input, textarea, select, [contenteditable="true"]')) return;
  const search = document.querySelector('.docs-search');
  if (!search) return;
  event.preventDefault();
  search.focus();
});

const headings = [...document.querySelectorAll('.docs-article h2, .docs-article h3')];
const outlineLinks = new Map(
  [...document.querySelectorAll('.outline a')].map((link) => [link.hash.slice(1), link])
);
const mobileSection = document.querySelector('.docs-mobile-section');
let ticking = false;

function syncOutline() {
  ticking = false;
  const line = innerHeight * 0.2;
  let current = headings[0];
  for (const heading of headings) {
    if (heading.getBoundingClientRect().top <= line) current = heading;
  }
  for (const link of outlineLinks.values()) link.removeAttribute('aria-current');
  const active = current && outlineLinks.get(current.id);
  if (active) active.setAttribute('aria-current', 'location');
  if (mobileSection && current) mobileSection.textContent = current.textContent;
}

addEventListener('scroll', () => {
  if (ticking) return;
  ticking = true;
  requestAnimationFrame(syncOutline);
}, { passive: true });
syncOutline();

document.querySelectorAll('.docs-mobile-bar button').forEach((button) => {
  button.addEventListener('click', () => {
    const sheet = document.getElementById(`sheet-${button.dataset.sheet}`);
    if (!sheet) return;
    sheet.showModal();
    sheet.querySelector('[aria-current]')?.scrollIntoView({ block: 'center' });
  });
});

document.querySelectorAll('.docs-sheet').forEach((sheet) => {
  sheet.addEventListener('click', (event) => {
    if (event.target === sheet) sheet.close();
  });
  sheet.addEventListener('click', (event) => {
    const link = event.target.closest('a');
    if (!link) return;
    sheet.close();
  });
  sheet.querySelectorAll('[data-close]').forEach((btn) => {
    btn.addEventListener('click', () => sheet.close());
  });
});
```

两个 `<dialog>` 里放和侧栏同一批链接（页面列表或大纲），示例把它们留空，避免同一份链接写两遍。`aria-current="page"` 用来在页面列表里认出当前页（本仓库整理：原帖只要求把当前页滚进可视区，没有指定这个属性）。大纲的当前节必须是 `aria-current="location"`，不要和页级标记混用。

## 可直接用的 Prompt（精简版）

> Adjust only the shared docs or help-center frame: how the header, page list, outline, article ending, and footer sit, scroll, and respond. Leave the copy, colors, fonts, and which side each column is on. CSS plus a little plain JS; no scroll or animation library. The article and footer scroll with the page. Sticky-pin the page list and the outline at the offset they already occupy so they do not jump, and let their container push them away at the bottom. Keep the header in flow, transparent, until the hero's bottom edge reaches the top of the viewport; then pin it as a translucent white bar with an 18px backdrop blur and a soft shadow. Enter that bar in 320ms ease-out from 24px up, 8px of blur, and opacity 0; leave in 120ms when the hero returns. A downward scroll during the exit cancels the exit and keeps the bar up, without replaying the entrance. Offset heading anchors with scroll-margin-top. The bar still holds a logo back to the marketing site, menus for the key pages, and one primary action; its fill has to look right in the address-bar tint Safari takes from it. Let the page list scroll inside itself, scrollbar hidden. Fade an edge only while more rows sit past it — about 40px on top and 56px on the bottom, with a short fully clear strip at the outer edge — using data-more-above and data-more-below on scroll and resize. A small round, pointer-only arrow on such an edge scrolls the list by 60% of its visible height. Restore the list scroll from sessionStorage, and still bring the current page into view. "/" focuses search when a search field exists and the reader is not already typing. Mark the outline link for the last heading above a line 20% down the viewport with aria-current="location" and the accent color on the text and left border, at most once per frame. End the article with a native-looking help card (one or two real channels), then previous and next titles (the first page's previous link goes home), then a footer taller than half the viewport: a background image plus information that actually helps. From tablet width, drop the sidebar outline; on a phone, drop the sidebar page list too. A fixed bottom bar opens each as a showModal() sheet from the bottom edge — page list on the left, the live section name (ellipsis) on the right — rising 24px as it fades in, with no dimmed backdrop. Center the current row, and dismiss on a row, the close control, an outside click, or Esc; a section link then scrolls to its heading. Make those rows one size easier to hit than on desktop. When the reader prefers reduced motion, drop smooth scrolling and the header enter/exit. Then list every file and selector, and check the top, the middle, the end, and a phone-width window.

原版 Prompt 见[这条回复](https://x.com/Kubai087/status/2116917025348231369)。上面是改写，不是原文。

## 常见错误

- 顶栏从第一像素就 `fixed` → 首屏被一条底栏压住，少了「先离开、再浮回」。
- 浮条离场做到一半又向下滚，却把进场动画重放一遍 → 顶栏闪一下，而不是停住。
- 标题没有 `scroll-margin-top` → 锚点钻进浮条底下。
- 页面列表跟正文共用一个滚动条，或用 `fixed` 钉死 → 滚到页脚时目录还盖在上面；sticky 才会在布局结束时被顶走。
- 上下都做渐隐，不管外面还有没有内容 → 列表尽头也像被切掉一截。渐隐一直淡到边缘、没有一条全透明带 → 半行文字剩一道浅线。
- 箭头能被 Tab 聚到 → 键盘用户多出两个没有名字的控件；列表本身跟着焦点滚就够了。
- 换页时把列表 `scrollTop` 清零 → 长目录每次都跳回第一项。
- 大纲用视口正中或标题顶边贴住 0 来决定当前节 → 和「20% 那条线之上的最后一个标题」不是同一节。
- 每次 `scroll` 都同步改 DOM，不合并到帧 → 长文滚动时大纲抖动。
- 高亮时才加上 `border-left` → 当前行的文字横向跳一下。
- 帮助卡做成一条异质横幅，或排在上一篇 / 下一篇后面 → 文末顺序乱了，也不像正文的一部分。
- 页脚只靠空白凑过半屏，里面没有能用的信息 → 滑到底只觉得被拖时间。
- 手机上把两列侧栏挤进窄屏，或底部清单背后压一层暗色 → 窄屏仍旧难扫，弹层也不符合「不压暗」。
- `prefers-reduced-motion` 下箭头仍 `behavior: 'smooth'`、顶栏仍做 320ms 进场 → 减弱动效没有生效。
- 为这一页单独写一套顶栏，共享布局没改 → 下一篇又回到旧框架。

## 出处

- 原帖：https://x.com/Kubai087/status/2116911391437443471
- Prompt 回复：https://x.com/Kubai087/status/2116917025348231369
- 作者：[@Kubai087](https://x.com/Kubai087)
