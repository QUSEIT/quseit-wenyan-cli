---
name: "wenyan-cli-wechat-theme-applier"
description: "Use when testing or publishing a Markdown article with a local custom CSS theme to WeChat Official Accounts via Wenyan CLI."
version: 1.0.0
author: caol64 (adapted for Hermes)
license: Apache-2.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [WeChat, Wenyan, CSS, Theme, Publish, 微信公众号]
    category: wenyan-cli
    related_skills: [wenyan-cli-generate-wechat-theme, wenyan-cli-publish-to-wechat]
---

# 微信公众号自定义主题应用工具 (WeChat Custom Theme Applier)

为 AI Agent 设计，将本地自定义 CSS 主题应用到 Markdown 文章，支持本地预览、主题注册和推送草稿箱。

## When to Use

- 用户已有本地 `.css` 主题文件和 `.md` 文章，需要测试渲染效果
- 用户已有自定义主题，要发布文章到微信公众号草稿箱
- 用户希望将自定义主题注册到 `wenyan-cli` 主题库长期使用

**不用于**：直接使用 wenyan-cli 内置主题发布（用 `wenyan-cli-publish-to-wechat`）

## Prerequisites

- 已安装：`wenyan-cli`（`@wenyan-md/cli`，version ≥ 2.0，binary 名为 `wenyan`）
- 本地已有 Markdown 文件 (`.md`) 和 CSS 主题文件 (`.css`)

### 安装/环境验证（Agent 必做）

```bash
# 检查 wenyan-cli 是否安装
wenyan --version          # 实测输出形如 "2.0.11"
# 若未安装：npm install -g @wenyan-md/cli   （npm/npx 亦可，不限于 pnpm）

# 检查凭证存储位置（凭证不是靠手设环境变量，见下方"凭证配置"）
wenyan credential -l      # 输出 credential.json 的存储路径，如 C:\Users\<u>\AppData\Roaming\wenyan-md\credential.json
```

**凭证配置（三级，任选其一）**——实测确认，wenyan-cli 的 AppID/AppSecret 来源优先级为：

1. **命令行 `--app-id <id>`**（publish 时传入，AppSecret 仍需环境变量或配置文件）
2. **环境变量 `WECHAT_APP_ID` / `WECHAT_APP_SECRET`**（实测在 `wrapper.js` 中读取，有效）
3. **配置文件 `credential.json`**：运行 `wenyan credential -s` 交互式设置（可存多组，用 alias 区分），存储位置由 `wenyan credential -l` 查看

> 也可用 `--env-file <path>` 参数从 `.env` 文件加载环境变量，或用 `--app-id` + 环境变量组合。

## 工作流 SOP

### Step 1: 测试渲染 (Render Test) — 强制

发布前必须先验证 CSS 语法和文件路径：

```bash
wenyan render -f <markdown_file_path> -c <css_file_path>
```

**实测输出样例**（`@wenyan-md/cli@2.0.11`）：

```
<section id="wenyan" style="font-family: system-ui, ...;" data-provider="WenYan">
  <h1 style="..."><span>文章标题</span></h1>
  <p>正文内容...</p>
  ...
</section>
```

**验证通过标准**：
- [ ] 命令退出码为 0（无报错）
- [ ] 输出容器是 **`<section id="wenyan">`**（不是 `<div>`），且为完整 HTML 字符串
- [ ] 自定义 CSS 已注入：检查目标元素的 style 是否出现你的定制值（如标题颜色变化）
- [ ] 无 CSS 语法错误警告

### Step 2: 推入草稿箱 (Publish to Draft)

测试通过后，推送到微信公众号**草稿箱**（wenyan 只发草稿，不自动群发）：

```bash
wenyan publish -f <markdown_file_path> -c <css_file_path>
```

**实测确认的参数**（`wenyan publish --help`）：

| 参数 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `-f, --file <path>` | Markdown 文件路径或 URL | — |
| `-c, --custom-theme <path>` | 自定义 CSS 主题文件路径 | — |
| `-t, --theme <id>` | 内置主题 ID | `default` |
| `-h, --highlight <id>` | **代码高亮**主题（注意：此处 `-h` 不是 help） | `solarized-light` |
| `--mac-style` / `--no-mac-style` | 代码块 Mac 风格窗口（**默认开启**） | `true` |
| `--footnote` / `--no-footnote` | 链接转脚注（**默认开启**） | `true` |
| `--app-id <appId>` | 指定 AppID（配合环境变量/配置文件用） | — |
| `--env-file <file>` | 从 `.env` 文件加载环境变量 | — |
| `--proxy <url>` | 代理（如 `http://127.0.0.1:1080`） | — |
| `--server <url>` / `--api-key <key>` | 远程 HTTP 服务发布模式 | — |

**成功判断标准**（实测 `draft/add` 接口）：
- [ ] 命令退出码为 0
- [ ] 输出包含微信返回的**草稿 `media_id`**（来源 `draft/add` 接口的 `data.media_id`）
- [ ] 无 `invalid credential`、`invalid ip` 等错误
- [ ] 错误提示是**中文**（如缺标题报"未能找到文章标题"、缺凭证报"未提供 AppID：请通过参数、环境变量或配置文件指定。"）

> ⚠️ **`media_id` 是草稿 ID，不是群发结果**。wenyan 只负责推入草稿箱，真正的群发仍需在公众号后台手动操作。

### Step 3: 注册主题 (Register Theme) — 按需

长期使用该主题时，注册到本地主题库（实测可用）：

```bash
wenyan theme --add --name <theme_name> --path <css_file_path>
```

**实测成功输出**：`主题 "<theme_name>" 已添加`

**参数说明**：
- `--name <theme_name>`：自定义主题名称（如 `my-cyberpunk`），后续用 `-t my-cyberpunk` 调用
- `--path <css_file_path>`：CSS 文件的绝对或相对路径

注册成功后，列出主题验证：运行 `wenyan theme -l`，输出分两区：`内置主题：`（default/orangeheart/rainbow/lapis/pie/maize/purple/phycat）和 `自定义主题：`（应含刚注册的名字）。

**注意**：自定义主题注册后存储位置是全局的（`credential.json` 同目录的配置），跨目录调用 `wenyan publish -t <theme_name>` 依然生效。

## Common Pitfalls

1. **文件不存在**：`ENOENT` 报错 → 检查 Markdown 或 CSS 路径是否正确（相对路径基于当前工作目录）
2. **样式未生效**：检查 CSS 选择器是否缺少 `#wenyan` 前缀
3. **缺凭证**：报"未提供 AppID：请通过参数、环境变量或配置文件指定" → 用 `--app-id` 参数 / `WECHAT_APP_ID`+`WECHAT_APP_SECRET` 环境变量 / `wenyan credential -s` 三者之一
4. **凭证错**：`invalid credential` → 检查 AppID/AppSecret 是否正确
5. **IP 白名单**：`invalid ip` → 将当前出口 IP 加入微信后台"IP 白名单"
6. **theme_name 记错**：注册时用的 `--name` 值（不是文件名）；调用时用 `-t <theme_name>`
7. **`-h` 歧义**：`wenyan -h` 是帮助，但 `wenyan publish -h <id>` / `wenyan render -h <id>` 里的 `-h` 是**代码高亮主题**，两处含义不同

## Verification Checklist

- [ ] `wenyan --version` 正常输出版本号
- [ ] 凭证已配置（`--app-id` / 环境变量 / `credential -s` 三者至少一种）
- [ ] Step 1 render 输出含 `<section id="wenyan">` 且自定义样式已注入、无错误
- [ ] Step 2 publish 输出含草稿 `media_id`，退出码为 0
- [ ] （如需）Step 3 theme 注册后 `wenyan theme -l` 的"自定义主题"区出现该名字
- [ ] 用户在微信公众号草稿箱确认文章样式正确