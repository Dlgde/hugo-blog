# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

基于 Hugo 的个人技术博客，使用 maupassant 主题，部署到 GitHub Pages (dlgde.github.io)。

## 常用命令

```bash
# 本地预览（跳过草稿）
hugo server

# 本地预览（含草稿）
hugo server -D

# 构建到 public/ 目录
hugo

# 构建并清除缓存
hugo --gc

# 手动构建 + 推送到 Dlgde.github.io 仓库
./build

# 新建文章
hugo new post/2026/文章标题.md
```

## 架构

### 双仓库部署

- **源码仓库**: `Dlgde/hugo-blog` — 即当前仓库，存放 Hugo 源码
- **页面仓库**: `Dlgde/Dlgde.github.io` — 存放构建后的静态文件，GitHub Pages 从此仓库渲染
- 日常 push 到 `hugo-blog` 的 `main` 分支后，GitHub Actions (`.github/workflows/hugo.yaml`) 自动触发构建并推送到 `Dlgde.github.io`
- `./build` 脚本可直接本地构建后推送到 `../Dlgde.github.io`，跳过 CI

### 文章目录结构

`content/post/` 按年份分子目录：`2019/`, `2020/`, `2021/`, `2024/`, `2026/`。每篇文章路径与 URL 无关——URL 由 front matter 的 `date` 字段决定，格式为 `/post/年份/标题/`。

### 模板定制

对 maupassant 主题模板的修改均在 `themes/maupassant/layouts/` 下：

- `index.html` — 首页改为按年份分组的归档列表（日期 + 标题，无摘要）
- `_default/baseof.html` — `<html lang>` 改用 `.Site.Language.Locale`
- `_default/taxonomy.html` — 分类/标签页与首页风格统一，按年分组简约列表
- `_default/terms.html` — 分类/标签汇总页，去掉 h1 大标题
- `partials/header.html` — 导航链接用 relURL 替代 absURL
- `partials/sidebar.html` — 移除 ads、links 区域

**URL 约定**：所有模板链接统一使用 `.RelPermalink` / `relURL` / `relLangURL`（非绝对 URL），避免本地预览时跳转到线上域名。仅 `head.html` 的 `og:url` 和 RSS feed 保留绝对 URL。

### 配置

- `config.toml` — Hugo v0.162+，需使用 `locale = "zh-CN"` 而非已弃用的 `languageCode`
- 代码高亮：github 风格，`lineNos = false`（关掉 chroma 表格行号，避免多余边框线）
- 主题自定义 CSS 追加在 `themes/maupassant/static/css/style.css` 末尾
