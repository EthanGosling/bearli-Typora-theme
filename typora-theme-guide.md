# Typora 主题开发指南（官方文档整理）

- 原文：<https://theme.typora.io/doc/Write-Custom-Theme/>
- 原文发布：2016-07-16，作者 typora.io
- 整理时间：2026-11-20

## 一、核心结论

主题就是一个 CSS 文件。

1. **文件名规则**：只用小写字母，空格换成 `-`。Typora 会把文件名自动转成菜单标签，例如 `my-first-typora-theme.css` → 菜单里的 "My First Typora Theme"。
2. **改字体、颜色、背景时，优先覆盖 `:root` 里的 CSS 变量**，比直接重写元素样式简单得多。
3. **从零写**：用官方 toolkit 里的 `template.less`，填空即可。
4. **改现成 CSS**（WordPress / Jekyll 主题等）：直接粘进来，再补上原样式没覆盖的部分，比如 TOC 和界面组件。
5. **调试**：
   - macOS：在 Help 菜单打开 debug mode，然后右键 → "Inspect Element"
   - Windows / Linux：按键 F12，或在 View 菜单选 Toggle DevTools
   - 也可以把 CSS 放到 toolkit 的 `html-preview/theme/test.css`，用 Safari 或 Chrome 预览
6. **发布**：fork Typora Theme Gallery，然后提 PR。

## 二、CSS 变量

官方（后续更新）推荐的写法。早期 macOS/Safari 不支持，但用起来省事很多。

```css
:root {
  --bg-color: #ffffff;              /* 背景 */
  --text-color: #333333;            /* 正文颜色 */
  --md-char-color: #C7C5C5;         /* Markdown 符号（如 *）的颜色 */
  --meta-content-color: #5b808d;    /* 图片描述、链接地址等元内容的颜色 */

  --primary-color: #428bca;         /* 主按钮颜色 */
  --primary-btn-border-color: #285e8e;
  --primary-btn-text-color: #fff;

  --window-border: 1px solid #eee;  /* 侧边栏等边框 */

  --active-file-bg-color: #eee;     /* 文件树中选中项的背景 */
  --active-file-text-color: inherit;
  --active-file-border-color: #777;

  --side-bar-bg-color: var(--bg-color);       /* 侧边栏背景 */
  --item-hover-bg-color: rgba(229, 229, 229, 0.59); /* 菜单项悬停背景 */
  --item-hover-text-color: inherit;
  --monospace: monospace;           /* 代码用的等宽字体 */
}
```

> 这些变量未来可能变化，必要时用 DevTools 确认。

## 三、基本规则

1. **字号**：默认字号写在 `html` 上，`h1`、`p` 等元素用 `rem` 设置 `font-size`。否则偏好设置里的自定义字号会失效。
2. **选择器**：尽量用标签选择器。`### heading 3` 会渲染成 `h3`，所以写 `h3`，不写 `h3.md-header`。如果只想作用于编辑区，加 `#write` 前缀（例如 `#write h3`），避免影响弹窗里的同名标签。
3. **兼容性**：macOS 版基于 WebKit，Windows / Linux 版基于 Chromium。只用这两者支持的 CSS 属性。
4. **少覆盖默认样式**：有些覆盖会让 Typora 行为异常。例如给 `#write` 加 `white-space: pre-wrap` 后，按 Tab 就插不进 `\t` 了。改完一定实测。

## 四、选择器速查

### 通用

```css
html, body {
  background-color: #fefefe;   /* 窗口和标题栏背景 */
  font-family: helvetica, sans-serif;
}

html {
  font-size: 14px;             /* 默认字号 */
}

#write {
  max-width: 90%;              /* 编辑区宽度 */
  font-size: 1rem;             /* 正文字号 */
  color: #555;                 /* 正文颜色 */
}
```

注意：`html` 上的属性会影响整个窗口，包括插入表格对话框这类 UI 组件。只想改写作内容时，把规则放在 `#write` 下。

macOS 使用 seamless 窗口样式时，工具栏背景色取自 `html` 的 `background-color`。

### 块级元素（`mdtype` 属性对照）

| mdtype | 选择器 | 说明 |
|---|---|---|
| paragraph | `p` | 段落 |
| line | `.md-line` | 一个段落可包含多个 `.md-line` |
| heading | `h1` ~ `h6` | 标题 |
| blockquote | `blockquote` | 引用 |
| list（无序） | `ul li` | 无序列表 |
| list（有序） | `ol li` | 有序列表 |
| list（任务） | `ul.task-list li.task-list-item` | 任务列表 |
| toc | `.md-toc` | 目录 |
| fences（初始化前） | `pre.md-fences.mock-cm` | 代码块未初始化状态 |
| fences | `pre.md-fences` | 代码块，高亮主题类为 `.cm-s-inner` |
| diagrams | `pre[lang='mermaid']`、`pre[lang='flow']`、`pre[lang='sequence']` | 图表 |
| hr | `hr` | 分割线 |
| def_link | `.md-def-link` | 子元素 `.md-def-name` / `.md-def-content` / `.md-def-title` |
| def_footnote | `.md-def-footnote` | 子元素 `.md-def-name` / `.md-def-content` |
| meta_block | `pre.md-meta-block` | YAML front matter |
| math_block | `[mdtype="math_block"]` | 预览部分为 `.mathjax-block`，TeX 编辑器由 CodeMirror 驱动 |

所有块级元素都有 `mdtype` 属性，可以按属性选，但多数情况下标签选择器就够了。

### 行内元素

```css
strong { font-weight: bold; }
em { ... }
code { ... }
a { ... }
img { ... }
mark { ... }   /* ==高亮== */
```

完整结构示例（`**strong**`）：

```html
<span md-inline="strong" class="">
  <span class="md-meta md-before">**</span>
  <strong><span md-inline="plain">strong</span></strong>
  <span class="md-meta md-after">**</span>
</span>
```

`md-inline` 属性和输出标签对照：

| md-inline | 语法 | 输出标签 |
|---|---|---|
| plain | 纯文本 | `span` |
| strong | `**strong**` | `strong` |
| em | `*em*` | `em` |
| code | `` `code` `` | `code` |
| underline | `<u>underline</u>` | `u` |
| escape | `\(` | `span` |
| del | `~~del~~` | `del` |
| footnote | `^1` | `sup` |
| emoji | `:smile:` | `span` |
| inline_math | `$x^2$` | `span` |
| subscript | `~sub~` | `sub` |
| superscript | `^sup^` | `sup` |
| highlight | `==highlight==` | `mark` |
| url / autolink / link / reflink | — | `a` |
| image / refimg | — | `img` |

### Markdown 符号（元语法）

- `**`、`==` 这类符号在导出 HTML 时会消失，包在 `.md-meta` 里，默认 `display:none`。
- 图片语法等内容包在 `.md-content` 里，默认隐藏。
- 光标进入某个行内元素时，该元素会加上 `.md-expand`，此时 `.md-meta` 和 `.md-content` 变为可见。

要改这些符号的样式，就作用于 `.md-meta` 和 `.md-content`。

### 源码模式

源码模式同样由 CodeMirror 驱动，语法高亮类名与代码块一致，但主题类不同：代码块用 `.cm-s-inner`，源码模式用 `.cm-s-typora-default`。

```css
.cm-s-typora-default .cm-header {
  /* 源码模式下 h1~h6 的样式 */
}
```

### 打印 / 导出 PDF

```css
@media print {
  .typora-export * {
    -webkit-print-color-adjust: exact;
  }
  /* 只作用于打印和导出 PDF 的样式 */
}
```

## 五、界面组件

tooltip、对话框、按钮等多数 UI 组件也是 HTML 画的。写完正文样式后如果发现界面和主题不搭，再调这些部分。toolkit 里的 HTML 文件已包含常见 UI 组件，方便调试。

Windows / Linux 版用 HTML 画的部分比 macOS 多，包括右键菜单、偏好设置面板，以及在 Windows 上使用 unibody 窗口样式时的窗口边框。这些也要检查。

## 六、本文档未展开的部分

以下主题在这篇指南里只有链接入口，需要看各自的独立文档：

- 代码块语法高亮（CodeMirror）
- Mermaid 图表（示例：`mermaid.css`、`mermaid.dark.css`、`mermaid.forest.css`）
- 专注模式
- 自定义字体
- 背景图
- TOC 样式
- 主题安装方法
