---
name: "wenyan-cli-wechat-publisher"
description: "Use when publishing a Markdown article to WeChat Official Accounts with built-in themes, code highlighting, and auto image upload via Wenyan CLI."
version: 1.0.0
author: caol64 (adapted for Hermes)
license: Apache-2.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [WeChat, Wenyan, Publish, Markdown, 微信公众号]
    category: wenyan-cli
    related_skills: [generate-wechat-theme, apply-wechat-custom-theme]
---

# 微信公众号文章发布工具 (WeChat Publisher)

为 AI Agent 设计，将 Markdown 文档转换为微信公众号富文本并直接发布，支持内置主题、代码高亮和素材图片自动上传。

## When to Use

- 用户有一篇 Markdown 文章要发布到微信公众号
- 用户想使用 wenyan-cli 内置主题（如 `orangeheart`）进行排版
- 用户需要自动上传 Markdown 中的本地/网络图片至微信素材库

**不用于**：使用自定义 CSS 主题（用 `apply-wechat-custom-theme`）

**选用逻辑**：
- 有自定义 `.css` 文件 → 用 `apply-wechat-custom-theme`
- 只想用内置主题 + 自动图片上传 → 用 `publish-to-wechat`（本技能）

## Prerequisites

- 已安装：`wenyan-cli`（`@wenyan-md/cli`，version ≥ 2.0，binary 名为 `wenyan`）

### 安装/环境验证（Agent 必做）

```bash
# 检查 wenyan-cli 是否安装
wenyan --version          # 实测输出形如 "2.0.11"
# 若未安装：npm install -g @wenyan-md/cli

# 查看凭证存储位置
wenyan credential -l
```

**凭证配置（三级，任选其一）**：
1. `wenyan credential -s` 交互式设置（存到 `credential.json`，可多组 + alias）
2. 环境变量 `WECHAT_APP_ID` / `WECHAT_APP_SECRET`
3. `publish` 时用 `--app-id <id>` 指定（AppSecret 仍需环境变量或配置文件）
4. 或 `--env-file <path>` 从 `.env` 加载

## Frontmatter 约束 (必须包含)

文章开头 **必须** 包含以下 YAML 块：

```yaml
---
title: 文章标题
cover: ./cover.jpg   # 若缺省则自动取正文第一张图
author: 作者名称     # 可选
source_url: https://example.com/original-article   # 可选，原文链接
---
```

**缺 `title` 时报错为中文"未能找到文章标题"**，`cover` 建议显式指定（缺省时取正文第一张图；封面图会走微信素材上传接口，本地路径需真实存在）。

## 核心参数（实测 `wenyan publish --help`）

| 参数 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `-f, --file <path>` | **必填**，Markdown 文件路径或 URL | — |
| `-t, --theme <id>` | 内置主题 ID | `default` |
| `-h, --highlight <id>` | 代码高亮主题（**此处 `-h` 非 help**） | `solarized-light` |
| `-c, --custom-theme <path>` | 自定义 CSS 主题文件 | — |
| `--mac-style` / `--no-mac-style` | 代码块 Mac 风格（**默认开启**） | `true` |
| `--footnote` / `--no-footnote` | 链接转脚注（**默认开启**） | `true` |
| `--app-id <appId>` | 指定 AppID | — |
| `--env-file <file>` | 从 `.env` 加载变量 | — |
| `--proxy <url>` | 代理 | — |
| `--server <url>` / `--api-key <key>` | 远程 HTTP 服务发布模式 | — |

**实测内置主题**（`wenyan theme -l`）：`default`、`orangeheart`、`rainbow`、`lapis`、`pie`、`maize`、`purple`、`phycat`。另有 `自定义主题` 分区（`wenyan theme --add` 注册）。

## 常用命令

```bash
# 标准发布草稿（默认 default 主题）
wenyan publish -f my-article.md

# 指定内置主题与高亮
wenyan publish -f article.md -t orangeheart -h solarized-light

# 列出所有主题（内置 + 自定义）
wenyan theme -l

# 仅渲染预览（不发草稿）
wenyan render -f my-article.md

# 设置凭证
wenyan credential -s

# 本地 HTTP API 服务模式
wenyan serve
```

## 成功判断标准

发布命令执行后（**实测确认：wanan 推入的是「草稿箱」，走 `draft/add` 接口，不自动群发**）：

- [ ] 退出码为 0（无报错）
- [ ] 输出包含草稿 `media_id`（来自 `draft/add` 的 `data.media_id`）
- [ ] 无 `invalid credential`、`invalid ip`、`ENOENT` 等错误
- [ ] 用户在微信公众号后台草稿箱可见新文章

> ⚠️ **"发布"= 推入草稿箱**。wenyan-cli 不负责群发，需在公众号后台手动完成正式群发。

## Common Pitfalls

1. **IP 限制错误 (`invalid ip`)**：将当前出口 IP 加入微信后台"IP 白名单"
2. **AppID/Secret 错误**：优先用 `wenyan credential -s` 交互式核对，或用环境变量 `WECHAT_APP_ID`/`WECHAT_APP_SECRET`、`--app-id` 参数
3. **图片上传失败**：Markdown 中的本地图片路径需在当前目录真实存在；网络图片需可公网访问（图片走素材上传接口，失败常因路径/公网可达性）
4. **排版不符预期**：检查 YAML Frontmatter，缺 `title` 报"未能找到文章标题"
5. **主题不存在**：`-t` 指定的主题需在 `wenyan theme -l` 列表中（内置无 `github`/`vue`）；自定义主题用 `wenyan theme --add` 注册或 `-c <css>` 直接传入
6. **`-h` 歧义**：`wenyan -h` 是帮助，但 `wenyan publish -h <id>` 是代码高亮主题

## Verification Checklist

- [ ] `wenyan --version` 正常输出版本号
- [ ] 凭证已配置（`credential -s` / 环境变量 / `--app-id` 至少一种）
- [ ] Markdown 文件存在且包含合法 Frontmatter（必填 `title`）
- [ ] `wenyan render -f <file>` 退出码 0，输出 `<section id="wenyan">`（发布前预检）
- [ ] `wenyan publish -f <file>` 退出码 0，输出含草稿 `media_id`
- [ ] 用户在微信公众号草稿箱确认文章样式、图片、代码高亮均正确