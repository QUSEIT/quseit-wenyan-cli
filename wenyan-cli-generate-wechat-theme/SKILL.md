---
name: "wenyan-cli-wechat-theme-generator"
description: "Use when designing a custom CSS theme for WeChat Official Account articles. Generates #wenyan-namespaced CSS with gradient, pseudo-elements, and inline SVG."
version: 1.0.0
author: caol64 (adapted for Hermes)
license: Apache-2.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [WeChat, CSS, Theme, Wenyan, 微信公众号]
    category: wenyan-cli
    related_skills: [wenyan-cli-apply-wechat-custom-theme, wenyan-cli-publish-to-wechat]
---

# 微信公众号自定义主题 CSS 生成器 (WeChat CSS Theme Generator)

为 AI Agent 设计，根据自然语言需求生成符合微信公众号排版规范的高度定制化 CSS 样式表。

## When to Use

- 用户要求"设计一个 xxx 风格的主题" / "生成微信公众号 CSS 主题"
- 用户描述视觉需求（如"赛博朋克风"、"带可爱表情的引用块"、"深色代码块"、"渐变标题"、"代码块行号"）
- 需要生成符合 `#wenyan` 命名空间约束的 CSS 代码

**不用于**：已有现成 CSS 只需应用和发布（用 `wenyan-cli-apply-wechat-custom-theme` 或 `wenyan-cli-publish-to-wechat`）

## Output Contract

生成的 CSS 必须：

1. **写入本地文件**：默认路径 `./theme.css`（用户可指定其他路径）
2. **完整可用**：包含全局、标题、段落、引用块、代码块、链接、分割线等常用元素的样式，不得只生成片段
3. **通过语法检查**：无 CSS 语法错误，选择器均以 `#wenyan` 开头

文件写入完成标准：`write_file` 返回 `verified: true`，且文件大小 > 500 字节。

## CSS 生成规则

### 1. 强制命名空间约束 (最重要！)

所有 CSS 选择器 **必须** 以 `#wenyan` 开头，否则样式在微信公众号中失效。

- ✅ `#wenyan h1 { color: red; }`
- ❌ `h1 { color: red; }`

2. **license 注意**：wenyan-cli 实际为 Apache-2.0 许可证（非 MIT）。若复用其主题/CSS 片段请留意署名与许可。（2026-08 实测 @wenyan-md/cli@2.0.11）

### 3. 字体与字号约束

- **禁止在全局/正文选择器主动设置 `font-family`**：渲染器已经输出 `system-ui, "Apple Color Emoji", "Segoe UI", "Noto Sans", "Roboto", sans-serif` 系统字体栈，覆盖会破坏公众号编辑器的字体适配。
- **唯一例外**：代码块内 `#wenyan pre code` 可设置等宽字体（如 `"SF Mono", "Fira Code", monospace`），代码阅读需要等宽。
- **字号范围**：`12px - 18px`，避免排版溢出（代码块可略小，最低 11px）。

### 3. 支持的 CSS 选择器速查

> 实测确认（@wenyan-md/cli@2.0.11）：渲染容器标签是 `<section id="wenyan">`（非 `<div>`），标题文字包裹在 `#wenyan h1 span` 内，代码块外层 `#wenyan pre`、内容 `#wenyan pre code`。

| 目标元素 | CSS 选择器 | 常用定制属性 |
| :--- | :--- | :--- |
| 全局默认样式 | `#wenyan` | `background-image`, `line-height`, `color` |
| 标题 (H1-H6) | `#wenyan h1` ~ `#wenyan h6` | `font-size`, `text-align`, `border-bottom` |
| 标题文字本身 | `#wenyan h1 span` | `color`, `font-weight`, `background` |
| 标题装饰 | `#wenyan h1::before` | `content`, `display`, `width`, `height` |
| 段落文本 | `#wenyan p` | `text-indent`, `letter-spacing`, `color` |
| 引用块 | `#wenyan blockquote` | `border-left`, `background-color`, `padding` |
| 代码块外层 | `#wenyan pre` | `background-color`, `border-radius`, `padding` |
| 代码块内容 | `#wenyan pre code` | `color`, `font-family`（此处允许等宽） |
| 分割线 | `#wenyan hr` | `border`, `border-top-style`, `border-color` |
| 超链接 | `#wenyan a` | `color`, `text-decoration`, `border-bottom` |

### 4. 外部资源引用限制

- **禁止本地路径**：`url("./bg.png")` 无效
- **合法方式**：
  - Data URI（推荐）：`url("data:image/svg+xml;utf8,<svg>...</svg>")`
  - HTTPS 地址：`url(https://example.com/bg.jpg)`
- **禁止 Web 字体**：`@font-face` 不支持

## 参考模板

生成新主题时，参考以下基础结构：

```css
/* 全局属性 */
#wenyan {
    line-height: 1.75;
    font-size: 16px;
}
#wenyan h1,
#wenyan h2,
#wenyan h3,
#wenyan h4,
#wenyan h5,
#wenyan h6,
#wenyan p {
    margin: 1em 0;
}
#wenyan h1 {
    text-align: center;
    font-size: 1.5em;
}
#wenyan h2 {
    text-align: center;
    font-size: 1.2em;
    border-bottom: 1px solid #f7f7f7;
    font-weight: bold;
}
#wenyan blockquote {
    background: #afb8c133;
    border-left: 0.5em solid #ccc;
    margin: 1.5em 0;
    padding: 0.5em 10px;
    font-style: italic;
    font-size: 0.9em;
}
#wenyan pre {
    border-radius: 5px;
    line-height: 2;
    margin: 1em 0.5em;
    padding: .5em;
    box-shadow: rgba(0, 0, 0, 0.55) 0px 1px 5px;
    font-size: 12px;
}
#wenyan a {
    word-wrap: break-word;
    color: #0069c2;
}
```

## 执行步骤

1. **分析需求**：提取关键词（风格、主色调、特殊效果、是否需要深色模式）
2. **生成 CSS**：严格遵循 `#wenyan` 前缀约束，生成完整样式表
3. **保存文件**：写入本地 `.css` 文件（默认 `./theme.css`，用户可指定路径）
4. **验证输出**：
   - [ ] 文件存在且大小 > 500 字节（辅助判据，仅防空文件）
   - [ ] **主判据：包含 6 类基础选择器** —— `#wenyan`（全局）、`#wenyan h1`~`h6`（标题）、`#wenyan p`（段落）、`#wenyan blockquote`（引用）、`#wenyan pre`（代码块）、`#wenyan a`（链接）。用 `grep` 逐个确认，缺一类即为不完整
   - [ ] 所有选择器以 `#wenyan` 开头（grep 验证，排除 `@media`/`@font-face` 等 at-rule）
   - [ ] 无明显 CSS 语法错误（如缺少分号、括号不匹配）
5. **实测渲染验证（推荐）**：若环境已装 `wenyan`，用 `wenyan render -f <任一md> -c <生成的css>` 跑一次，确认退出码 0、输出 `<section id="wenyan">` 且自定义样式已注入（如标题颜色变化）。这一步能当场揪出选择器写错、语法错误等问题。
6. **引导后续**：提示用户使用 `wenyan-cli-apply-wechat-custom-theme` 技能测试渲染 / 发布草稿

## 高级效果参考代码片段

### 赛博朋克风渐变标题

```css
#wenyan h1 {
    background: linear-gradient(90deg, #ff006e, #8338ec, #3a86ff);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-align: center;
    font-size: 2em;
    text-shadow: 0 0 30px rgba(255, 0, 110, 0.5);
}
#wenyan h1::before {
    content: "";
    display: block;
    width: 60px;
    height: 4px;
    margin: 0 auto 0.5em;
    background: linear-gradient(90deg, #ff006e, #3a86ff);
    border-radius: 2px;
}
```

### 可爱表情引用块

```css
#wenyan blockquote {
    position: relative;
    background: #fff0f5;
    border-left: 4px solid #ff69b4;
    border-radius: 12px;
    padding: 1em 1.5em;
    margin: 1.5em 0;
}
#wenyan blockquote::before {
    content: "💭";
    position: absolute;
    top: -12px;
    left: 16px;
    background: white;
    padding: 0 6px;
    font-size: 1.2em;
}
```

### 深色代码块

```css
#wenyan pre {
    background: #1e1e1e;
    border-radius: 8px;
    padding: 1em;
    overflow-x: auto;
}
#wenyan pre code {
    color: #d4d4d4;
    font-family: "SF Mono", "Fira Code", monospace;
    font-size: 13px;
    line-height: 1.6;
    display: block;
}
```

> ⚠️ **行号不可用纯 CSS 实现**：wenyan 渲染的 `<pre>` 内没有按行拆分的 `.line` 元素，`counter` 行号技巧（`#wenyan pre code .line::before { counter-increment: line; ... }`）不会生效。如需行号，需在 Markdown 源里手写，或改用其他支持行号的渲染方案。

## Verification Checklist

- [ ] 生成的 CSS 文件写入指定路径且 `verified: true`
- [ ] 所有选择器以 `#wenyan` 开头
- [ ] 文件大小 > 500 字节（辅助判据，防空文件）
- [ ] **主判据：6 类基础选择器齐全**（全局/标题/段落/引用块/代码块/链接）
- [ ] 无 CSS 语法错误（括号匹配、分号完整）
- [ ] 未使用禁止项：本地路径 url()、@font-face、正文 font-family
- [ ] 用户确认可进入下一步（测试渲染）

## Common Pitfalls

1. **忘记 `#wenyan` 前缀**：生成的样式在公众号中完全不生效
2. **使用本地路径**：微信公众号无法访问本地图片/字体
3. **字号过大/过小**：超出 `12-18px` 范围容易排版异常
4. **使用 `@font-face`**：微信公众号环境不支持
5. **在全局选择器设 `font-family`**：覆盖渲染器的系统字体栈，破坏公众号字体适配（代码块 `#wenyan pre code` 设等宽字体除外）
6. **用 `counter` 伪元素做行号**：wenyan 渲染的 `<pre>` 内没有 `.line` 元素，行号技巧不生效，属无效代码