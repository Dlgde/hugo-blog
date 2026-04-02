# hugo-blog

使用 Hugo 搭建的静态博客，支持自动部署到 GitHub Pages。

## 快速开始

```bash
# 本地预览
hugo server

# 新建文章
hugo new post/xxx.md
```

## 部署

推送到 main 分支后，GitHub Actions 会自动构建并部署到 https://dlgde.github.io

## 文章写作

1. 创建新文章：`hugo new post/标题.md`
2. 本地预览：`hugo server -D`
3. 提交发布：
```bash
git add .
git commit -m "feat: 新增文章标题"
git push origin main
```

## 访问

- 博客：https://dlgde.github.io
- 源码：https://github.com/Dlgde/hugo-blog
