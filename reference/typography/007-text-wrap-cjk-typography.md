# 007 · 让你的文字排版更顺眼（以中英文为例）
**Cleaner text wrapping for Chinese and English**

> 分类：排版 Typography · 原帖：[2026-10-31](https://x.com/Kubai087/status/2116555064227811412) · Prompt：[回复](https://x.com/Kubai087/status/2116555520274493582) · 作者 [@Kubai087](https://x.com/Kubai087)
>
> 本文是对原帖说明与 Prompt 的改写整理，属性与数值取自原帖；「实现要点」代码为本仓库示例。只改折行、间距、对齐和截断。标了「本仓库整理」的句子是对规范或库用法的补充，不是原帖原文。

## 原理

英文和中文都会难看在三件事上：对不齐、各行长短差一截、标点被甩到不该去的一端（避头尾）。这几件事用全局文字样式里的几条 CSS 就能处理。不认识这些属性的浏览器会直接忽略，不要写 polyfill，也不要为它们拉一套脚本回退。

比 CSS 更要紧的是句子本身。做 i18n 时按**当前版面**挑文案：不要让一个词单独掉到下一行，也不要留出大片空白。折行规则补的是断行，文案补的是长度。

中文专用规则挂在 `:lang(zh)` 上。`:lang(zh)` 会命中 `zh`、`zh-Hans`、`zh-Hant`、`zh-CN`、`zh-TW`。`<html>` 或对应区块必须写上 `lang`，否则这些规则不会生效。

## 规则 / 数值

只动折行、间距、对齐、截断。不改字号、字色、字族，也不改布局结构。规则写进全局文字样式，不要散落在单个组件的内联样式里。

| 规则 | 用在 | 不用 |
|---|---|---|
| `text-wrap: balance` | `h1`–`h3`、卡片标题、Hero 标题 | 正文、列表、按钮、单行截断的标签 |
| `text-wrap: pretty` | 段落、说明、列表项 | 标题、要整行截断的标签、代码 |
| `tabular-nums` | 计数、价格、计时、统计、会变的表格单元格 | 正文和标题（保持比例数字） |
| `nowrap` 或 `flex-shrink: 0` | 横排 flex 里的按钮标签 | 窄容器里本该折行的长句按钮 |
| 省略号截断 | 宽度被限死的 flex / grid 标签 | 标题和正文（让它们折行）；没有溢出的短标签 |
| `text-align: justify` | 中文正文，行长是整数个字 | 窄栏（约少于 15 字）、短句、图注、按钮、以英文为主的文字 |
| `text-autospace: normal` | 中文及中英混排 | 代码、输入框、等宽字体；也不要手打空格 |
| `text-spacing-trim` | 中文 UI（`trim-start`）；有两字段首缩进的长文（`space-first`） | 非中文；不支持就让它被忽略 |
| BudouX + `keep-all` | 中文标题 | 正文；没有断点的标题；`Intl.Segmenter` |
| 删掉 `line-break: anywhere` | 普通 HTML 文本 | canvas、SVG、JS 自己折行的文本（要手写避头尾） |

### 标题：`text-wrap: balance`

`h1`–`h3`、卡片标题、Hero 标题：

```css
text-wrap: balance;
```

Chrome 最多把 **6 行**拉齐；更长的标题，超出的部分按普通折行。不要用在正文上。

### 正文：`text-wrap: pretty`

段落、说明、列表项用 `text-wrap: pretty`，避免段末孤词和特别难看的断行。

不该拆开的一对，用 **U+00A0**（HTML `&nbsp;`）粘住，不要用普通空格：

- 破折号和紧跟的名字：`——&nbsp;枯白`
- 数字和单位：`26&nbsp;字`、`24&nbsp;px`

### 会变的数字：`font-variant-numeric: tabular-nums`

计数器、价格、计时器、统计数字、表格里会刷新的单元格：

```css
font-variant-numeric: tabular-nums;
```

字体必须带 OpenType **`tnum`**。没有这个特性时声明无效，数字仍会随字形宽窄跳动。正文和标题保持比例数字，不要加这条。`tabular-nums` 会继承，所以写在数字元素上，不要写在 `body` 上（本仓库整理）。

### 横排按钮的标签

同一行 flex 里的按钮标签保持一行：`white-space: nowrap`，或者 `flex-shrink: 0`。

本仓库整理：二者不是同一件事。`nowrap` 阻止文案折行；`flex-shrink: 0` 阻止按钮被 flex 压扁。标签必须保持一行、同时这一行还可以横向滚动或整组换行时，两条可以一起写。窄容器里的长句按钮不要 `nowrap`，让它折行。

### 截断

省略号要出现在 **flex 子项**上时，先把默认的 `min-width: auto` 打掉，否则子项不肯缩到比内容更窄，省略号不会出现：

```css
min-width: 0;
overflow: hidden;
white-space: nowrap;
text-overflow: ellipsis;
```

只有 `scrollWidth > clientWidth` 时才给完整文本的 tooltip。没有溢出的短标签不要挂一条重复的提示。标题和正文不要截断，让它们折行。

### 中文正文两端对齐

`:lang(zh)` 的正文：

```css
text-align: justify;
max-width: 26em; /* 示例：整数个字。一个汉字约 1em，按版面改成别的整数 */
```

`26em` 是「行长等于整数个字」的例子，不是唯一合法值。换成 20、24、28 都可以，只要是整数 em。

不要两端对齐的情况：

- 窄栏，大约 **少于 15 个字**
- 短句、图注、按钮
- 以英文为主的文字（英文两端对齐会撑出河沟）。这种段落标 `lang="en"`，让它离开中文规则

### 中英之间的空隙：`text-autospace: normal`

`:lang(zh)` 上写 `text-autospace: normal`，在汉字和拉丁字母、汉字和数字之间留出间隙。不要在源码里手打空格来凑这道缝；开了自动间距之后，手打的空格会变成双倍空隙。

代码、输入框、等宽字体关掉。关掉的值是 `text-autospace: no-autospace`（本仓库整理：这是该属性的关闭值）。

### 标点贴边：`text-spacing-trim`

原帖标明 **仅 Chrome**。不支持的浏览器忽略这条声明，不要 polyfill。

- 没有段首缩进的中文 UI：`text-spacing-trim: trim-start`（行首的全角开标点贴齐左缘）
- 有两个字段首缩进的长文：`text-spacing-trim: space-first`

本仓库整理：两个字的缩进写成 `text-indent: 2em`。`space-first` 会保留段首第一行的标点间距，后面各行仍贴边，这样缩进看起来还是两个字。字体需要 OpenType `halt` 或 `chws`，否则 `text-spacing-trim` 不生效。

### 中文标题按短语折，而不是按字折

只用于中文标题，不用于正文。

用 BudouX 的 **zh-Hans** 或 **zh-Hant** 模型，在短语边界插入 `<wbr>`。静态标题在构建时插入；动态标题在渲染时插入。然后：

```css
word-break: keep-all;
overflow-wrap: anywhere;
text-wrap: balance;
```

`keep-all` 让中文只在标点和显式断点处折行。没有 `<wbr>` 就不要写 `keep-all`，否则整句变成不可断的一行，只能靠 `overflow-wrap` 硬拆。不要改用 `Intl.Segmenter`。

本仓库整理：BudouX 的 `translateHTMLString` 默认插入的是 U+200B，并带一行内联样式。本条按原帖用 `<wbr>`，样式留在全局样式表里，不要再叠那行内联样式。

### 避头尾

浏览器对 HTML 文本已经做了避头尾（行首不放句号、逗号、闭括号，行尾不放开括号）。删掉 `line-break: anywhere`，它允许在任意位置断开，避头尾就没了。超长单词或 URL 需要退路时，改用 `overflow-wrap: anywhere`。

`word-break: break-all` 同样会在任意字之间断开。默认删掉；如果某处必须留下，写明原因（例如一条不可断的长 token，且不能改用 `overflow-wrap`）。

canvas、`fillText`、SVG `<text>`、以及用 JS 自己切行的文本，不走这套浏览器规则，要自己保证断行位置。本仓库整理，常见禁则：

- 行首避免：`。，、！？；：）」』】》`
- 行尾避免：`「『【《（`

### 做完后交代

1. 列出改过的文件和选择器。
2. 列出每一处 `line-break: anywhere` 和 `word-break: break-all`：删了，还是留下，以及原因。
3. 给出五处前后对照：折行的标题、段落的收尾、会变的数字、被截断的标签、一段中英混排。

## 实现要点（示例实现，非原帖代码）

```css
/* 全局文字样式。不支持的声明会被忽略，不要加 polyfill。 */
:where(h1, h2, h3, .card-title, .hero-title) {
  text-wrap: balance;
}
:where(p, .description, li) {
  text-wrap: pretty;
}
:where(.counter, .price, .timer, .stat, td.num) {
  font-variant-numeric: tabular-nums;
}
.actions > .btn {
  white-space: nowrap;
  flex-shrink: 0;
}
.row > .label {
  min-width: 0;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

:lang(zh) {
  text-autospace: normal;
  text-spacing-trim: trim-start; /* UI，无段首缩进；仅 Chrome */
}
:lang(zh) p {
  text-align: justify;
  max-width: 26em; /* 正文行长示例。窄栏、图注、按钮、导航 li 不要套 */
}
:lang(zh) .prose {
  text-indent: 2em;
  text-spacing-trim: space-first;
}
:lang(zh) :where(h1, h2, h3, .card-title, .hero-title) {
  word-break: keep-all;       /* 元素内必须已有 <wbr> */
  overflow-wrap: anywhere;
  text-wrap: balance;
  text-indent: 0;
}
:lang(zh) :where(code, kbd, samp, pre, input, textarea, .monospace) {
  text-autospace: no-autospace;
  text-align: start;
  text-indent: 0;
}
:lang(en) :where(p, li) {
  text-align: start;
}
```

```html
<p lang="zh">这套规则改写自原帖——&nbsp;枯白，行长约 26&nbsp;字。</p>
```

```ts
import {
  loadDefaultSimplifiedChineseParser,
  loadDefaultTraditionalChineseParser,
} from 'budoux';

const hans = loadDefaultSimplifiedChineseParser();
const hant = loadDefaultTraditionalChineseParser();

/** 静态中文标题：构建时插入 <wbr>。只用于标题。 */
export function segmentHeading(text: string, script: 'hans' | 'hant' = 'hans') {
  const phrases = (script === 'hant' ? hant : hans).parse(text);
  return phrases.join('<wbr>');
}

/** 动态标题：渲染时插入 <wbr> 节点，不走 innerHTML。 */
export function fillHeading(el: HTMLElement, text: string, script: 'hans' | 'hant' = 'hans') {
  const phrases = (script === 'hant' ? hant : hans).parse(text);
  el.replaceChildren();
  phrases.forEach((phrase, i) => {
    if (i > 0) el.append(document.createElement('wbr'));
    el.append(phrase);
  });
}

/** 省略号 tooltip：没溢出就不提示。 */
export function syncTruncationTooltip(el: HTMLElement) {
  if (el.scrollWidth > el.clientWidth) el.title = el.textContent ?? '';
  else el.removeAttribute('title');
}
```

项目里已有排版 token、tooltip 或文案组件时，把这些声明并进去，不要另起一套标题组件。

## 可直接用的 Prompt（精简版）

> Touch only wrapping, spacing, alignment, and truncation. Put the rules in global text styles, scope Chinese rules with `:lang(zh)`, set `lang`, and do not add polyfills. Headings (`h1`–`h3`, card titles, hero): `text-wrap: balance` (Chrome evens at most 6 lines). Paragraphs, descriptions, list items: `text-wrap: pretty`; join an em dash to the following name, and a number to its unit, with U+00A0. Changing figures (counters, prices, timers, stats, table cells): `font-variant-numeric: tabular-nums` only where the font has `tnum`; leave prose proportional. Button labels in a flex row: `white-space: nowrap` or `flex-shrink: 0`. Truncation: `min-width: 0` on the flex item, then hidden overflow, nowrap, ellipsis; tooltip the full string only when `scrollWidth > clientWidth`. Chinese body: `text-align: justify` and a whole-character measure such as `max-width: 26em`; skip columns under about 15 characters, short text, captions, buttons, and mostly-English copy. Chinese text: `text-autospace: normal`; do not type those gaps by hand; turn it off for code, inputs, and monospace. Chinese UI with no first-line indent: `text-spacing-trim: trim-start`; long-form with a two-character indent: `space-first`. Chrome only. Chinese headings only: segment with BudouX (`zh-Hans` or `zh-Hant`), insert `<wbr>` (build time if static, render time if dynamic), then `word-break: keep-all`, `overflow-wrap: anywhere`, `text-wrap: balance`. Never `keep-all` without those breaks, never `Intl.Segmenter`, never on body text. Browsers already keep punctuation off the wrong line edge; delete `line-break: anywhere` and use `overflow-wrap: anywhere` for emergency breaks. Canvas, SVG, and JS-wrapped text need that rule by hand. When localizing, pick strings that fit the layout: no word stranded on its own line, no large gaps. Finish by listing files and selectors, every `line-break: anywhere` / `word-break: break-all` removed or kept and why, and before/after for a wrapped heading, a paragraph ending, a changing number, a truncated label, and a mixed Chinese–English paragraph.

原版 Prompt 见[这条回复](https://x.com/Kubai087/status/2116555520274493582)。上面是改写，不是原文。

## 常见错误

- 给正文写 `text-wrap: balance`，或给标题写 `pretty` → 标题该拉齐的行没拉齐，长文却被强行配平。
- 在 `body` 上写 `tabular-nums` → 正文数字变成等宽，和比例字体打架。字体没有 `tnum` 时，计数器看起来「设了但还在跳」。
- flex 子项只写了省略号三件套，没写 `min-width: 0` → 省略号不出现，把旁边的控件挤出去。
- 短标签也挂 tooltip → 提示和可见文字重复。
- 中文窄栏、图注、按钮、英文段落用 `justify` → 空隙被拉大。英文段落没标 `lang="en"` 时会吃到中文规则。
- 手打「中文 English」之间的空格，同时又开 `text-autospace: normal` → 间隙变成两份。
- 代码、输入框、等宽字体没关掉自动间距 → 源码和输入看起来被加了空格。
- 中文标题写了 `word-break: keep-all` 但没有 `<wbr>` → 整句不折行，或只能在溢出时硬断。
- 用 `Intl.Segmenter` 或把 BudouX 用在正文上 → 断点和原帖要的标题短语不一致，正文被切得过碎。
- 留下 `line-break: anywhere` 或 `word-break: break-all` → 句号、逗号、闭括号掉到行首。
- 用 `overflow-wrap: anywhere` 的地方改回 `line-break: anywhere` → 避头尾再次失效。
- canvas / SVG / JS 折行以为浏览器会避头尾 → 标点仍会落在错误的一端。
- 为 `text-spacing-trim` 或 `text-wrap` 写 polyfill → 原帖要求渐进增强，不支持就忽略。
- i18n 只翻译词、不看版面 → 一个词独占第二行，或两端对齐后空隙过大。

## 出处

- 原帖：https://x.com/Kubai087/status/2116555064227811412
- Prompt 回复：https://x.com/Kubai087/status/2116555520274493582
- 作者：[@Kubai087](https://x.com/Kubai087)
