---
title: "Claude Code 常用操作与最佳实践"
date: 2026-06-05T16:39:18+08:00
draft: false
tags: ["工具", "AI", "Claude Code", "效率"]
categories: ["工具配置"]
description: "记录 Claude Code 的常用操作、快捷键、Slash 命令和日常使用中的最佳实践，持续更新"
toc: true
---

## 什么是 Claude Code

Claude Code 是 Anthropic 推出的终端原生 AI 编程助手，直接在命令行中运行，深度理解整个代码库上下文，帮你完成编码、调试、代码审查、Git 操作等工作。

---

## 常用 Slash 命令

Claude Code 提供了一系列内置命令，输入 `/` 即可查看完整列表。以下是最常用的：

| 命令 | 作用 |
|------|------|
| `/help` | 查看帮助 |
| `/clear` | 清除当前会话上下文 |
| `/compact` | 压缩上下文，保留关键信息 |
| `/context` | 查看当前上下文占用情况 |
| `/cost` | 查看当前会话费用 |
| `/init` | 为项目生成 CLAUDE.md 文件 |
| `/doctor` | 诊断 Claude Code 安装和环境问题 |
| `/config` | 查看和修改设置 |
| `/resume` | 打开会话选择器，恢复历史对话 |
| `/status` | 显示当前项目和会话状态 |
| `/pr-comments` | 查看当前 PR 的评论 |
| `/review-pr` | 审查一个 PR |
| `/terminal-setup` | 配置终端 Shfit+Enter 快捷键 |
| `/upgrade` | 升级 Claude Code 到最新版本 |
| `/install-github-app` | 安装 GitHub App（用于 PR 审查等） |

---

## 快捷键

| 快捷键 | 作用 |
|--------|------|
| `Enter` | 提交消息 |
| `Shift + Enter` | 换行（需在用 `/terminal-setup` 绑定后） |
| `Esc` | 取消当前操作 |
| `Ctrl + C` | 强制中断 |
| `Ctrl + O` | 显示/切换模型选择面板 |
| `↑ / ↓` | 切换历史命令 |
| `Ctrl + R` | 搜索历史命令 |

其中 `Ctrl + O` 切换模型最常用——快速任务用 Haiku，复杂任务切到 Opus。

---

## 核心技巧与最佳实践

### 1. CLAUDE.md —— 项目级"记忆"

在项目根目录创建 `CLAUDE.md`，记录 Claude Code 需要知道的上下文：项目架构、常用命令、编码规范、注意事项。每次对话 Claude Code 会自动读取这个文件。

**好的 CLAUDE.md 应该包含：**

- 项目概述和架构说明
- 常用命令（构建、测试、部署）
- 目录结构和命名约定
- 自定义模板或修改说明
- 注意事项和陷阱

### 2. 一次性任务用 `-p` 模式

简单的一次性操作不需要进入交互模式：

```bash
# 单次执行
claude -p "帮我把所有 console.log 替换成 logger.debug"

# 配合管道
cat error.log | claude -p "分析这个错误日志，找出根本原因"

# 输出 JSON
claude -p "列出所有导出了函数的文件" --output-format json
```

### 3. Session 管理 —— 会话的保存与恢复

Claude Code 会自动将每次对话保存为 Session，存储在 `~/.claude/projects/<项目路径>/<session-id>.jsonl`。这意味着：
- 关闭终端后，对话不会丢失
- 可以随时恢复到之前的某个对话
- 可以查看历史对话内容

#### 基本命令

```bash
# 继续最近一次对话（最常用）
claude -c
claude --continue

# 打开交互式会话选择器（用方向键选，输入关键词搜索）
claude -r
claude --resume

# 恢复到指定会话（适合切换不同任务线）
claude -r <session-id>
claude --resume <session-id>

# 从之前的会话分叉新分支（不影响原会话）
claude -r <session-id> --fork-session

# 用自定义名称标记会话（显示在选择器、终端标题中）
claude -n "修复登录 Bug"

# 不保存会话（仅 --print 模式，临时任务不留下记录）
claude -p "查个东西" --no-session-persistence
```

#### 如何查找 Session ID

**方法一（推荐）：用内置的 `/resume` 或 `-r` 交互选择器**

```bash
# 打开交互式会话选择器，支持方向键浏览和关键词搜索
claude -r

# 带搜索关键词直接筛选
claude -r "修复"
```

在 Claude Code 交互界面内也可以输入 `/resume` 打开同样的选择器。

**方法二：用 `-n` 给会话命名**

```bash
# 开始新会话时指定一个名字
claude -n "重构数据库层"

# 之后在选择器中就能看到这个名字，不用猜 UUID
```

**方法三：直接浏览文件系统**

```bash
# 列出当前项目的所有会话（按时间排序）
ls -lt ~/.claude/projects/*/

# 查看最近的会话文件
ls -lt ~/.claude/projects/*/ | head -20
```

**方法四：用第三方工具**
社区有很多好用的 session 管理工具：

| 工具 | 特点 |
|------|------|
| [cc-sessions](https://github.com/chronologos/cc-sessions) | Go 写的 TUI 浏览器，支持全文搜索、跨机器同步 |
| [cc-session](https://github.com/cc-deck/cc-session) | Rust 写的极速 TUI，<500ms 加载 2000+ 会话 |
| [claude-resume](https://pypi.org/project/claude-resume/) | Python 写的轻量 picker，按项目/内容搜索 |
| [claude-recent](https://github.com/rmdes/claude-recent) | Shell 脚本 + fzf，适合多机器/WSL 场景 |
| [@i-am-nio/cc-sessions](https://www.npmjs.com/package/@i-am-nio/cc-sessions) | 支持书签命名，`/bookmark <name>` 标记重要会话 |

```bash
# 以 cc-sessions 为例
cc-sessions              # 打开交互式会话浏览器
cc-sessions --list       # 纯文本列表
cc-sessions --project myapp  # 筛选特定项目
```

#### 实用场景

- **多任务并行**：用 `--fork-session` 从同一个起点分叉，一条线做 feature A，另一条做 feature B
- **回溯检查**：用 `--resume <id>` 回到之前讨论方案的会话，确认当初的决策
- **会话复用**：如果是同一个项目的持续工作，直接用 `-c` 继续，Claude 记得所有上下文
- **会话清理**：`~/.claude/projects/` 下的 `.jsonl` 文件可以直接删除来清理旧会话

### 4. `/compact` —— 长对话的救命功能

对话长了之后，上下文会占用大量 token，响应变慢、费用变高。适时用 `/compact` 压缩上下文，Claude 会自动总结关键信息。

### 5. 选择合适的模型

| 模型 | 适用场景 |
|------|----------|
| **Haiku** | 简单问答、快速搜索、小改动 |
| **Sonnet** | 常规编码、代码审查、中等复杂度任务 |
| **Opus** | 复杂架构设计、深度调试、重重构 |

按 `Ctrl + O` 随时切换。日常开发用 Sonnet 最均衡，遇到棘手问题切到 Opus。

### 6. 善用权限模式

```bash
# 可信任的项目里跳过重复确认
claude --permission-mode accept-edits

# 完全跳过所有确认（谨慎使用）
claude --dangerously-skip-permissions
```

或者在 `settings.json` 中配置，让某些目录下的操作自动授权。

### 7. 费用控制

```bash
# 设置单次最大花费
claude -p "审查整个项目" --max-budget-usd 5
```

### 8. MCP 扩展

Claude Code 支持 MCP（Model Context Protocol），可以扩展 Claude 的能力——比如连接数据库、操作浏览器、调用外部 API 等。在 `settings.json` 中配置 MCP 服务器即可。

### 9. 版本更新

Claude Code 更新频繁，定期升级：

```bash
# 查看当前版本
claude --version

# 升级
/upgrade
```

---

## 我的工作流

1. **启动项目时**：`claude` 进入交互模式（或用 `claude -c` 继续上次），Claude 自动读取 `CLAUDE.md`
2. **切换任务时**：`claude -r` 打开会话选择器，切到另一条任务线的对话
3. **写代码时**：描述需求，Claude 生成代码，我审查并确认
4. **审查代码时**：`/review-pr <PR 编号>` 或 `claude -p "审查这次改动"`
5. **遇到 Bug**：把错误信息贴进去，Claude 定位问题并修复
6. **长对话后**：`/compact` 保持上下文清爽
7. **收尾**：先 `/cost` 看花费，再 `/clear`

---

## 进阶玩法

### Custom Slash Commands

在 `.claude/commands/` 目录下创建自定义命令文件，定义自己的 `/` 命令。

### Hooks

在 `settings.json` 中配置 hooks，实现在特定事件时自动触发操作——比如每次答复前自动跑 lint、每次保存文件时自动格式化。

### Skills

`/skill-creator` 可以创建自定义 skill，把常用操作封装成可复用的能力模块。

---

## 总结

Claude Code 的核心优势在于**理解整个代码库上下文**，而不只是单个文件。用好了，它能从"代码补全工具"升级为"真正的编程搭档"。几个最重要的习惯：

1. **写好 CLAUDE.md** —— 这是投入产出比最高的操作
2. **善用 Session** —— `-c` 继续、`-r` 切换、`-n` 命名，把对话当成持久的工作资产
3. **适时 `/compact`** —— 避免上下文膨胀
4. **选对模型** —— 别让 Haiku 干 Opus 的活，也别用 Opus 做简单搜索
5. **配置权限** —— 可信任的仓库减少确认提示，专注干活
