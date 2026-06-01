# hugo-blog

使用 Hugo 搭建的静态博客，支持自动部署到 GitHub Pages。

## 快速开始

```bash
# 本地预览（跳过草稿）
hugo server

# 新建文章
hugo new post/2026/标题.md
```

## 部署

推送到 main 分支后，GitHub Actions 会自动构建并部署到 https://dlgde.github.io

## 文章写作

1. 创建新文章（按年份归档）：`hugo new post/当前年份/标题.md`
2. 编辑 front matter，将 `draft: true` 改为 `false`
3. 本地预览（含草稿）：`hugo server -D`
4. 提交发布：
```bash
git add .
git commit -m "feat: 新增文章标题"
git push origin main
```

## 文章目录

```
content/post/
  2019/  (19 篇)
  2020/  ( 1 篇)
  2021/  ( 6 篇)
  2024/  ( 1 篇)
  2026/  ( 1 篇)
```

每篇文章的 front matter 最小模板：

```yaml
---
title: "文章标题"
date: 2026-06-01T20:00:00+08:00
draft: false
tags: [golang]
categories: [golang]
description: ""
---
```

## 配置说明

- 主题：maupassant
- Hugo 版本要求：v0.162+ (Homebrew 安装)
- 代码高亮：github 风格，行号关闭
- 建站配置在 `config.toml`

## 访问

- 博客：https://dlgde.github.io
- 源码：https://github.com/Dlgde/hugo-blog
