---
title: "Git 常用命令与最佳实践"
date: 2026-06-12T10:00:00+08:00
draft: false
tags: ["Git", "工具", "最佳实践", "版本控制"]
categories: ["工具配置"]
description: "整理 Git 日常高频命令、分支管理策略、提交规范以及实际工作中的踩坑经验"
toc: true
---

## 引言

Git 是日常开发中最常用的工具之一，但很多人只停留在 `add` / `commit` / `push` 三步走。这篇文章整理了我日常使用的高频命令、分支管理策略和提交规范, 希望能帮你更高效地使用 Git。

---

## 基础配置

装好 Git 后第一件事——设置用户名和邮箱, 每次提交都会带上这些信息:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

几个推荐的全局配置:

```bash
# 换行符统一为 LF（跨平台协作必备）
git config --global core.autocrlf input

# 设置默认分支名为 main
git config --global init.defaultBranch main

# 使用更直观的冲突标记
git config --global merge.conflictStyle diff3

# 彩色输出
git config --global color.ui auto

# 别名（高频命令缩短）
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all --decorate -20"
```

---

## 日常高频工作流

### 拉代码 → 改代码 → 提交 → 推送

```bash
git pull                  # 拉最新代码
# ... 改代码 ...
git status                # 看改了什么
git diff                  # 看具体改动内容
git add <file>            # 暂存
git commit -m "xxx"       # 提交
git push                  # 推送
```

### 提交前最后一次审视

`git add` 之后再跑一遍 `git diff --staged` 看看将要提交的 diff, 避免把调试代码、临时注释带上去。

```bash
git diff --staged
```

### 查看提交历史

```bash
# 简洁的一行日志（配合别名 lg 更好用）
git log --oneline --graph --all --decorate -20

# 查看某文件的修改历史
git log -p -- <file>

# 看某次提交的详细内容
git show <commit-hash>
```

---

## 分支操作

### 创建和切换

```bash
# 创建分支并切换过去
git checkout -b feature/xxx

# 等价于新版 Git 的
git switch -c feature/xxx
```

### 分支命名惯例

| 前缀 | 用途 |
|------|------|
| `feature/` | 新功能 |
| `fix/` | Bug 修复 |
| `hotfix/` | 紧急线上修复 |
| `refactor/` | 重构 |
| `docs/` | 文档 |
| `chore/` | 杂项（依赖更新、配置调整等） |

### 同步分支

```bash
# 把 main 的最新代码合并到当前分支
git fetch origin
git rebase origin/main

# 或者用 merge
git merge origin/main
```

### 删除分支

```bash
# 删除本地分支
git branch -d feature/xxx

# 删除远程分支
git push origin --delete feature/xxx
```

---

## 撤销与回退

这是最让人心跳加速的部分, 搞清楚了能减少很多焦虑。

### 还没 commit：撤销暂存或修改

```bash
# 取消暂存（把文件从 staging area 拿出来）
git restore --staged <file>

# 丢弃工作区的修改（⚠️ 不可恢复）
git restore <file>
```

### 已经 commit 但没 push

```bash
# 修改最近一次提交（追加改动、改 commit message）
git commit --amend

# 只改 commit message, 不改内容
git commit --amend -m "新的 message"
```

### 已经 push 了

```bash
# 回退到指定 commit, 保留改动在工作区
git reset --soft HEAD~1
git push --force-with-lease

# 新建一个反向提交来撤销（更安全, 适合公共分支）
git revert <commit-hash>
```

> **关键区分**: `git revert` 不会改写历史, 而是生成一个新 commit 来反向操作, 更适合多人协作的主分支。`git reset` + `--force` 会改写历史, 只应在自己的分支上用。

---

## 合并与 Rebase

### Merge vs Rebase 怎么选

| 场景 | 推荐 |
|------|------|
| 把 main 同步到 feature 分支 | `rebase`——保持历史线性 |
| 把 feature 合入 main | `merge`——保留分支合并点 |
| 合入前清理 feature 分支的提交记录 | `rebase -i` 整理后再 merge |

### 交互式 Rebase（整理提交历史）

```bash
# 整理最近 3 条 commit
git rebase -i HEAD~3
```

常用操作:
- `pick` — 保留这个 commit
- `squash` — 合并到上一个 commit
- `reword` — 改 commit message
- `drop` — 删掉这个 commit

### 解决冲突

```bash
# rebase 过程中冲突了
# 1. 手动解决冲突文件
# 2. 标记为已解决
git add <resolved-file>
# 3. 继续 rebase
git rebase --continue

# 想放弃这次 rebase
git rebase --abort
```

---

## Stash：临时保存工作现场

写了一半代码, 突然要切分支处理紧急问题：

```bash
# 保存当前改动
git stash

# 保存时加说明
git stash save "写了一半的登录功能"

# 查看 stash 列表
git stash list

# 恢复最近一次 stash（不删除 stash 记录）
git stash apply

# 恢复最近一次 stash（删除记录）
git stash pop

# 恢复指定 stash
git stash apply stash@{2}
```

---

## Tag：版本标记

Tag 用于标记重要的版本节点, 最常见的就是发布版本号。

### 创建 Tag

```bash
# 轻量标签（只指向某个 commit）
git tag v1.0.0

# 附注标签（含 tagger、日期、说明信息，推荐）
git tag -a v1.0.0 -m "正式发布 v1.0.0"

# 给过去的某个 commit 补打 tag
git tag -a v1.0.0 <commit-hash> -m "补标记"
```

> **轻量 vs 附注**: 附注标签是一个完整的 Git 对象, 带有作者、日期和 message, `git show v1.0.0` 能看到详细信息。轻量标签只是一个引用, 没有额外信息。发布版本推荐用附注标签。

### 查看 Tag

```bash
# 列出所有标签
git tag

# 按版本号模式过滤
git tag -l "v1.*"

# 查看某个标签的详细信息
git show v1.0.0
```

### 推送 Tag

```bash
# 推送单个 tag
git push origin v1.0.0

# 推送所有本地 tag
git push origin --tags
```

> `git push` 默认不会推送 tag, 需要显式指定。常见的做法是推代码时顺手跟上 tag: `git push && git push origin --tags`。

### 删除 Tag

```bash
# 删除本地 tag
git tag -d v1.0.0

# 删除远程 tag
git push origin --delete v1.0.0
```

### 基于 Tag 切分支

```bash
# 从某个 tag 切出分支做 hotfix
git checkout -b hotfix/v1.0.1 v1.0.0
```

### 语义化版本 (Semantic Versioning)

推荐按 [SemVer](https://semver.org/lang/zh-CN/) 规范给 tag 命名：

| 版本号 | 含义 | 示例场景 |
|--------|------|----------|
| 主版本号 | 不兼容的 API 修改 | `v1.0.0` → `v2.0.0` |
| 次版本号 | 向下兼容的功能新增 | `v1.0.0` → `v1.1.0` |
| 修订号 | 向下兼容的 Bug 修复 | `v1.0.0` → `v1.0.1` |

---

## 查看与对比

```bash
# 看某个 commit 和它的父 commit 的差异
git show <commit>

# 两个分支的差异
git diff main..feature/xxx

# 看谁改了这个文件的每一行
git blame <file>

# 看某一行是谁什么时候改的
git blame -L 10,20 <file>
```

---

## 实用小技巧

### 只 add 已跟踪文件的修改（不加新文件）

```bash
git add -u
```

### 交互式暂存（只 add 文件的一部分）

```bash
git add -p <file>
```

### 找回误删的分支

```bash
# 先看 reflog 找分支的最后一个 commit
git reflog
# 从那个 commit 重建分支
git checkout -b recovered-branch <commit-hash>
```

### 查看某个文件在两个分支间的差异

```bash
git diff main feature/xxx -- path/to/file
```

### cherry-pick：只拿某一个 commit

```bash
git cherry-pick <commit-hash>
```

---

## 提交信息规范

推荐使用 [Conventional Commits](https://www.conventionalcommits.org/) 格式:

```
<type>(<scope>): <subject>

<body>

<footer>
```

type 常用值:

| type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `refactor` | 重构 |
| `docs` | 文档 |
| `style` | 格式（不影响代码逻辑） |
| `test` | 测试 |
| `chore` | 构建/工具/依赖 |

示例:

```
feat(auth): 添加 JWT 登录支持

实现了基于 JWT 的用户认证流程，包括 token 签发和验证中间件。

Closes #42
```

---

## 工作流建议

### 日常开发节奏

1. 从 `main` 拉一个 feature 分支
2. 小步提交, 每个 commit 只做一件事
3. commit message 清晰描述**做了什么**和**为什么**
4. 每天结束时 push 到远程（备份 + 方便别人提前看）
5. 开发完成后, `rebase -i` 整理提交历史
6. 发起 PR, 经过 Review 后合入 main

### 几个原则

- **一个 commit 只做一件事**——方便 review, 方便 revert
- **不要混入无关改动**——比如格式化代码和功能修改分开两次 commit
- **push 前先 pull/rebase**——减少不必要的 merge commit
- **`--force-with-lease` 而不是 `--force`**——前者会检查远程有没有别人的新提交, 减少误覆盖
- **不要 `commit --amend` 已经 push 的提交**——除非只有你在用这个分支
- **定期 `git fetch --prune`**——清理本地已不存在的远程分支引用

---

## 小结

| 场景 | 命令 |
|------|------|
| 撤销未 commit 的暂存 | `git restore --staged <file>` |
| 丢弃未 commit 的修改 | `git restore <file>` |
| 改最近一次 commit | `git commit --amend` |
| 回退已 push 的 commit（自己的分支） | `git reset --soft HEAD~1 && git push --force-with-lease` |
| 回退已 push 的 commit（公共分支） | `git revert <hash>` |
| 临时切走保存现场 | `git stash` |
| 整理提交历史 | `git rebase -i HEAD~N` |
| 取某个 commit 到当前分支 | `git cherry-pick <hash>` |
| 找回误删分支 | `git reflog` + `git checkout -b 分支名 <hash>` |
| 创建附注标签 | `git tag -a v1.0.0 -m "说明"` |
| 推送 tag 到远程 | `git push origin v1.0.0` 或 `git push origin --tags` |
| 删除远程 tag | `git push origin --delete v1.0.0` |
| 从 tag 切分支 | `git checkout -b hotfix/xxx v1.0.0` |
| 看某行代码是谁写的 | `git blame <file>` |

Git 不是学一次就放下的工具, 多在实践中积累手感, 慢慢地从"不敢乱动"到"随便怎么玩都能救回来"。
