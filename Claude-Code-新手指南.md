# Claude Code 新手指南

> 适用版本：当前环境 (DeepSeek V4 Pro 后端 + Claude Code CLI)
> 更新日期：2026-05-24

---

## 目录

1. [Claude Code 是什么](#1-claude-code-是什么)
2. [安装与启动](#2-安装与启动)
3. [基本用法](#3-基本用法)
4. [核心概念：Agent（代理）](#4-核心概念agent代理)
5. [核心概念：Skill（技能）](#5-核心概念skill技能)
6. [核心概念：Rule（规则）](#6-核心概念rule规则)
7. [核心概念：Hook（钩子）](#7-核心概念hook钩子)
8. [核心概念：Memory（记忆）](#8-核心概念memory记忆)
9. [工作流门禁系统 (G1-G7)](#9-工作流门禁系统-g1-g7)
10. [常用 Slash 命令速查](#10-常用-slash-命令速查)
11. [快捷键](#11-快捷键)
12. [实用场景示例](#12-实用场景示例)
13. [常见问题 FAQ](#13-常见问题-faq)

---

## 1. Claude Code 是什么

Claude Code 是一个**命令行 AI 编程助手**，运行在你的终端里。它不是普通的聊天机器人——它能直接读你的代码、写代码、运行命令、管理 Git、执行测试，就像一个经验丰富的结对编程搭档。

**核心能力：**
- 读写你的项目文件
- 执行 shell 命令（安装依赖、运行测试、git 操作等）
- 搜索代码库
- 创建和管理 Git 提交与 Pull Request
- 并行启动多个子代理处理复杂任务

**本环境的模型配置：**
- 主模型：DeepSeek V4 Pro（100万 token 上下文窗口）
- 快速模型：DeepSeek V4 Flash（轻量任务）
- 扩展思考：默认启用（最多 31,999 token 的内部推理）

---

## 2. 安装与启动

### 安装

```bash
# npm 全局安装
npm install -g @anthropic-ai/claude-code

# 或使用一行脚本安装
curl -fsSL https://claude.ai/code/install.sh | bash
```

### 启动

在终端中进入你的项目目录，然后：

```bash
claude
```

启动后你会看到一个交互式 REPL（Read-Eval-Print Loop）界面，直接输入中文或英文指令即可。

### 首次配置

```bash
# 查看当前配置
claude config

# 设置主题（dark/light）
claude config set theme dark

# 设置权限模式（default/acceptEdits/plan）
claude config set permissionMode default
```

---

## 3. 基本用法

### 3.1 直接对话式编程

最简单的方式：进入项目目录，启动 Claude Code，用自然语言描述你想做什么。

```
你: 帮我创建一个 React 登录组件，包含邮箱和密码输入框
你: 这段代码有什么问题？
你: 给 utils/format.ts 里的所有函数写单元测试
你: 帮我修复 main.py 第42行的类型错误
```

Claude Code 会：
1. 先读取相关文件了解上下文
2. 计划修改方案
3. 执行修改
4. 验证结果

### 3.2 两种模式

| 模式 | 适用场景 | 特点 |
|------|---------|------|
| **普通对话** | 简单编辑、问答、调试 | 直接操作，快速 |
| **Plan Mode（规划模式）** | 复杂功能、重构、架构设计 | 先设计方案，你确认后再动手 |

**进入规划模式：** 输入 `/plan` 或当你的请求比较复杂时，Claude Code 会自动建议进入。

### 3.3 权限控制

Claude Code 在执行某些操作前会请求你的许可（尤其是 Bash 命令、文件写入等）。你可以：

- **Allow** — 允许本次操作
- **Deny** — 拒绝本次操作
- **Always allow** — 记住选择，以后不再询问
- **Allow all [type]** — 批量允许同类操作

---

## 4. 核心概念：Agent（代理）

Agent 是**专门的子 AI**，各有擅长的领域。当任务匹配某个代理的专业领域时，Claude Code 会自动调用它。你也可以手动指定。

### 4.1 你最常用的代理

| 代理名 | 干什么的 | 什么时候用 |
|--------|---------|-----------|
| **planner** | 制定实现计划 | 复杂功能、重构前 |
| **architect** | 系统架构设计 | 技术选型、架构决策 |
| **code-reviewer** | 代码审查 | 写完代码后 |
| **security-reviewer** | 安全检查 | 改完认证/支付/用户数据后 |
| **tdd-guide** | 测试驱动开发 | 写新功能、修 bug |
| **build-error-resolver** | 修复构建错误 | 编译失败时 |
| **e2e-runner** | 端到端测试 | 测试关键用户流程 |

### 4.2 语言专用审查代理

| 代理名 | 语言 |
|--------|------|
| **python-reviewer** | Python |
| **typescript-reviewer** | TypeScript / JavaScript |
| **go-reviewer** | Go |
| **rust-reviewer** | Rust |
| **java-reviewer** | Java / Spring Boot |
| **kotlin-reviewer** | Kotlin |
| **csharp-reviewer** | C# / .NET |
| **cpp-reviewer** | C++ |
| **flutter-reviewer** | Flutter / Dart |

### 4.3 语言专用构建修复代理

| 代理名 | 语言 |
|--------|------|
| **go-build-resolver** | Go 构建错误 |
| **rust-build-resolver** | Rust 构建错误 |
| **java-build-resolver** | Java/Maven/Gradle |
| **kotlin-build-resolver** | Kotlin/Gradle |
| **dart-build-resolver** | Dart/Flutter |
| **cpp-build-resolver** | C++/CMake |
| **pytorch-build-resolver** | PyTorch 运行时错误 |

### 4.4 其他实用代理

| 代理名 | 用途 |
|--------|------|
| **code-explorer** | 深入分析现有代码逻辑 |
| **code-simplifier** | 简化和优化代码 |
| **refactor-cleaner** | 清理死代码和重复 |
| **performance-optimizer** | 性能瓶颈分析和优化 |
| **database-reviewer** | 数据库查询优化、Schema 设计 |
| **doc-updater** | 更新文档和代码地图 |
| **docs-lookup** | 查框架/库的官方文档 |
| **a11y-architect** | WCAG 2.2 无障碍合规 |
| **seo-specialist** | SEO 审计和优化 |
| **pentest-scanner** | 渗透测试和安全漏洞扫描 |

### 4.5 如何调用代理

```
# 自动调用：Claude Code 会根据任务自动选择合适的代理

# 手动调用（在对话中）：
你: 用 code-reviewer 审查一下我刚改的 auth.ts

# 或者直接描述需求，Claude Code 会自动分派：
你: 帮我检查这个 PR 的安全性 → 自动调用 security-reviewer
你: 构建失败了帮我看看 → 自动调用 build-error-resolver
```

---

## 5. 核心概念：Skill（技能）

Skill 是**可复用的专业工作流**，通过 `/技能名` 调用。Skill 提供深度的领域知识和标准化的工作流程。

### 5.1 日常开发技能

| 技能 | 用途 | 示例 |
|------|------|------|
| **/plan** | 制定实现计划 | `/plan` 然后描述你要做什么 |
| **/commit** | 智能提交代码 | `/commit` 自动分析变更并提交 |
| **/code-review** | 代码审查 | `/code-review` 审查未提交的改动 |
| **/review-pr** | 审查 PR | `/review-pr 123` 审查 PR #123 |
| **/tdd** | 测试驱动开发 | `/tdd` 按 TDD 流程开发 |
| **/feature-dev** | 引导式功能开发 | `/feature-dev` 按工作流一步步来 |

### 5.2 语言和框架技能

这些技能在写某种语言的代码时会自动加载，提供该语言的最佳实践指导：

| 技能 | 领域 |
|------|------|
| **python-patterns** | Python 惯用法、PEP 8 |
| **golang-patterns** | Go 惯用模式、并发 |
| **rust-patterns** | Rust 所有权、trait、错误处理 |
| **dart-flutter-patterns** | Flutter/Dart 生产级模式 |
| **frontend-patterns** | React/Next.js 前端模式 |
| **frontend-design** | 高质量前端 UI 设计 |
| **backend-patterns** | Node.js/Express 后端模式 |
| **springboot-patterns** | Spring Boot 分层架构 |
| **django-patterns** | Django REST API、ORM |
| **nestjs-patterns** | NestJS 模块化架构 |
| **swiftui-patterns** | SwiftUI 架构、状态管理 |
| **dotnet-patterns** | C#/.NET 惯用模式 |
| **laravel-patterns** | Laravel 路由、Eloquent、队列 |
| **kotlin-patterns** | Kotlin 惯用模式、协程 |
| **cpp-coding-standards** | C++ Core Guidelines |

### 5.3 搜索和研究技能

| 技能 | 用途 |
|------|------|
| **/search-first** | 先搜索已有方案再写代码 |
| **deep-research** | 多源深度调研并生成带引用报告 |
| **exa-search** | AI 神经搜索引擎搜索 |
| **api-connector-builder** | 按项目现有模式构建 API 连接器 |

### 5.4 代码质量技能

| 技能 | 用途 |
|------|------|
| **/clean** | 清理和简化代码 |
| **/refactor** | 重构优化 |
| **/fix** | 修复构建和代码问题 |
| **/verify** | 验证功能正确性 |
| **/santa-loop** | 双审查员对抗审查（两个独立审查员都通过才算通过） |

### 5.5 自动化技能

| 技能 | 用途 |
|------|------|
| **/loop** | 定时循环执行任务（如每 5 分钟检查部署状态） |
| **/hookify** | 从对话中提取模式并创建自动化钩子 |
| **/save-session** | 保存当前会话方便以后恢复 |
| **/resume-session** | 恢复上次保存的会话 |

---

## 6. 核心概念：Rule（规则）

Rule 是你给 Claude Code 设定的**长期行为准则**。它们告诉 Claude Code 在写代码时要遵守什么标准。

### 6.1 规则存放位置

规则文件存放在 `~/.claude/rules/` 下，分两个层级：

```
~/.claude/rules/
├── common/       ← 通用规则（所有项目适用）
├── zh/           ← 中文翻译版通用规则
├── typescript/   ← TypeScript 专用规则
├── python/       ← Python 专用规则
├── golang/       ← Go 专用规则
├── rust/         ← Rust 专用规则
├── web/          ← 前端专用规则
├── java/         ← Java 专用规则
├── kotlin/       ← Kotlin 专用规则
├── swift/        ← Swift 专用规则
├── php/          ← PHP 专用规则
├── cpp/          ← C++ 专用规则
├── csharp/       ← C# 专用规则
├── dart/         ← Dart 专用规则
└── perl/         ← Perl 专用规则
```

### 6.2 你当前已启用的规则

| 规则文件 | 内容 |
|---------|------|
| **coding-style.md** | 不可变性、KISS/DRY/YAGNI 原则、文件组织、命名规范 |
| **testing.md** | 最低 80% 测试覆盖率、TDD 流程、AAA 测试结构 |
| **security.md** | 提交前安全检查清单、密钥管理 |
| **git-workflow.md** | 提交消息格式、PR 工作流 |
| **code-review.md** | 审查清单、严重级别、何时必须审查 |
| **development-workflow.md** | 研究→规划→TDD→审查→提交的完整流程 |
| **agents.md** | 代理调用规则、并行执行要求 |
| **hooks.md** | 钩子系统配置和使用 |
| **patterns.md** | 仓储模式、API 响应格式等设计模式 |
| **performance.md** | 模型选择策略、上下文窗口管理 |

**语言专用规则会自动生效**——当你在 Python 项目中工作时，`rules/python/` 下的规则会自动激活。

---

## 7. 核心概念：Hook（钩子）

Hook 是在**特定事件发生时自动触发的脚本**，让你的工作流自动化。

### 7.1 你当前的钩子配置

| 触发时机 | 做什么 |
|---------|--------|
| **SessionStart**（会话开始） | 显示 G1-G7 工作流门禁提醒 |
| **PostToolUse**（写文件后） | 提醒进行 TDD 和 Code Review |
| **Stop**（会话结束前） | 验证是否完成 G7（分支/PR/清理） |

### 7.2 钩子类型

| 类型 | 触发时机 | 用途示例 |
|------|---------|---------|
| **PreToolUse** | 工具执行前 | 阻止写入超大文件、验证参数 |
| **PostToolUse** | 工具执行后 | 自动格式化代码、运行 lint |
| **Stop** | 会话结束时 | 运行构建验证 |

---

## 8. 核心概念：Memory（记忆）

Memory 是 Claude Code 的**持久化记忆系统**。它会在 `~/.claude/projects/<项目>/memory/` 下保存关于你、项目、反馈等信息，让未来的对话能更好地理解你的偏好。

### 8.1 记忆类型

| 类型 | 存什么 | 示例 |
|------|--------|------|
| **user** | 你的角色、偏好、技能水平 | "我熟悉 Go，但 React 是新手" |
| **feedback** | 你对 Claude Code 行为的反馈 | "不要用 mock 数据库做集成测试" |
| **project** | 项目背景、目标、约束 | "合并冻结期 2026-03-05 起" |
| **reference** | 外部资源链接 | "流水线 Bug 在 Linear 项目 INGEST 中" |

### 8.2 手动管理记忆

```
你: 记住：我们这个项目要兼容 Python 3.10+
你: 我刚刚说了什么来着？     → Claude Code 会检查记忆
你: 忘掉关于 Python 版本的事
```

---

## 9. 工作流门禁系统 (G1-G7)

你当前的 Claude Code 配置了一套 **7 步工作流门禁**，确保代码质量：

```
G1  头脑风暴    → 直接说需求，或使用 /brainstorm
G2  工作区隔离  → git worktree add（创建隔离工作区）
G3  编写计划    → /plan 或使用 planner 代理
G4  子代理开发  → 按计划逐步实现
G5  测试驱动    → RED(写测试)→GREEN(实现)→REFACTOR(重构)
G6  代码审查    → code-reviewer 代理审查
G7  完成分支    → 合并/创建PR/清理
```

### 典型使用流程

```
1. 你: 我想给用户模块加个注销功能
   → G1 头脑风暴：Claude Code 会和你讨论需求

2. 你: /plan
   → G3 编写计划：生成详细的实现计划

3. 确认计划后
   → G4/G5 开发和测试：自动写测试 → 实现 → 重构

4. 写完代码
   → G6 自动触发代码审查

5. 审查通过后
   → G7 创建commit和PR
```

---

## 10. 常用 Slash 命令速查

### 开发流程

| 命令 | 功能 |
|------|------|
| `/plan` | 制定实现计划（先计划，确认后再动手） |
| `/tdd` | 按测试驱动开发流程工作 |
| `/feature-dev` | 引导式功能开发 |
| `/code-review` | 审查本地未提交的改动 |
| `/review-pr <编号>` | 审查 GitHub PR |
| `/commit` | 自然语言描述要提交什么，自动提交 |
| `/build-fix` | 自动修复构建错误 |

### 代码质量

| 命令 | 功能 |
|------|------|
| `/clean` | 清理死代码和简化逻辑 |
| `/refactor` | 重构优化代码结构 |
| `/verify` | 验证功能正确性 |
| `/test-coverage` | 检查测试覆盖率 |
| `/santa-loop` | 双重审查直到两个审查员都批准 |

### 自动化和管理

| 命令 | 功能 |
|------|------|
| `/loop <间隔> <任务>` | 定时循环执行（如 `/loop 5m 检查构建状态`） |
| `/save-session` | 保存当前会话 |
| `/resume-session` | 恢复之前的会话 |
| `/hookify` | 从对话中提取模式创建自动钩子 |
| `/projects` | 列出已知项目 |

### 搜索和研究

| 命令 | 功能 |
|------|------|
| `/search-first` | 先搜索已有方案再写代码 |
| `/docs <关键词>` | 搜索文档 |
| `@<文件名>` | 让 Claude Code 读取指定文件 |

---

## 11. 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送消息 |
| `Shift + Enter` | 换行 |
| `Ctrl + C` | 中断当前操作 |
| `Ctrl + D` | 退出 Claude Code |
| `Ctrl + O` | 查看 AI 的思考过程（verbose 模式） |
| `Alt + T` | 切换扩展思考开关 |
| `Tab` | 自动补全文件路径 |
| `Esc` `Esc` | 回到对话（从代码 diff 等界面） |
| `↑` / `↓` | 浏览历史命令 |
| `Ctrl + R` | 搜索历史命令 |

### 鼠标操作

- **拖拽图片/文件** 到终端 → Claude Code 会读取它们
- **粘贴截图** → 直接分析 UI 截图和设计稿

---

## 12. 实用场景示例

### 场景 1：修复一个 Bug

```
你: pages/login.tsx 里登录按钮点不动，帮我看看

Claude Code 会自动：
1. 读取 pages/login.tsx
2. 找到相关的事件处理逻辑
3. 搜索关联的组件和 API 调用
4. 定位问题并修复
5. 建议运行测试验证
```

### 场景 2：写一个新 API 接口

```
你: /plan
我想给 /api/users 加一个搜索接口，支持按姓名和邮箱模糊搜索，带分页

Claude Code 会：
1. 分析现有代码结构
2. 制定实现计划（路由 → 控制器 → Service → 数据库查询）
3. 等你确认后逐步实现
4. 自动写测试
5. 完成后跑代码审查
```

### 场景 3：审查自己的代码再提交

```
你: /code-review

Claude Code 会：
1. 运行 git diff 获取所有改动
2. 用 code-reviewer 代理分析代码质量
3. 给出严重级别标注的问题列表
4. 修复 Critical/High 问题

你: /commit
   → 自动分析改动内容，生成规范的 commit message 并提交
```

### 场景 4：学习陌生代码

```
你: 帮我理一下 src/services/order.ts 的整个下单流程

Claude Code 会：
1. 读取目标文件
2. 追踪所有函数调用链
3. 画出数据流向
4. 用你熟悉的语言解释逻辑
```

### 场景 5：多任务并行

```
你: 同时帮我做三件事：
1. auth 模块做安全审查
2. 检查缓存系统的性能
3. utils 里的类型定义有没有问题

Claude Code 会启动 3 个代理并行工作，最后汇总结果。
```

---

## 13. 常见问题 FAQ

### Q: Claude Code 和 ChatGPT/Copilot 有什么区别？

- **Copilot**：在你编辑器里自动补全代码（被动辅助）
- **ChatGPT**：独立聊天，不会直接操作你的项目文件
- **Claude Code**：在终端里主动工作——读代码、写代码、跑命令、提交 Git，是完整的结对编程搭档

### Q: 怎么让它不要每次都问我权限？

在 `~/.claude.json` 的 `permissions.allow` 中添加允许的操作。例如：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test:*)",
      "Bash(git diff:*)",
      "Bash(git status:*)"
    ]
  }
}
```

也可以直接跟 Claude Code 说：
```
你: 以后 npm test 命令不用问我
```

### Q: 代码被改坏了怎么办？

```bash
# 用 Git 回退
git diff          # 看改了什么
git checkout -- <file>   # 恢复单个文件
git stash         # 暂存所有改动

# 或者在 Claude Code 里说：
你: 撤销刚才对 login.tsx 的修改
```

### Q: 怎么控制花费？

1. 简单任务不需要扩展思考时，按 `Alt + T` 关闭
2. 对于简单查询，可以说 "简短回答"
3. 查看当前会话消耗：Claude Code 会在退出时显示

### Q: 支持哪些语言？

全语言通用，但以下语言有专门的审查代理和规则加成：
Python、TypeScript/JavaScript、Go、Rust、Java/Kotlin、C#/.NET、C++、Swift、Dart/Flutter、PHP/Laravel

### Q: Claude Code 的项目文件 (.claude/) 要不要提交到 Git？

- `CLAUDE.md` — **建议提交**（项目级别的指令，团队共享）
- `.claude/settings.local.json` — **不要提交**（个人设置）
- `memory/` — **不要提交**（个人记忆）

### Q: 如何在团队中使用？

1. 在项目根目录创建 `CLAUDE.md`，写入项目说明和规范
2. 提交到 Git，团队成员 clone 后自动生效
3. 每个人可以有自己 `~/.claude/rules/` 下的个人规则
4. 用 `/hookify` 创建团队共享的自动化钩子

---

## 速查卡片

```
┌─────────────────────────────────────────────────────────┐
│                   Claude Code 速查卡                      │
├─────────────────────────────────────────────────────────┤
│  启动         │  claude                                  │
│  规划模式     │  /plan                                   │
│  代码审查     │  /code-review                            │
│  提交代码     │  /commit                                 │
│  修复构建     │  /build-fix                              │
│  查看思考     │  Ctrl+O                                  │
│  扩展思考     │  Alt+T                                   │
│  中断操作     │  Ctrl+C                                  │
│  退出         │  Ctrl+D                                  │
├─────────────────────────────────────────────────────────┤
│  关键代理     │                                          │
│  规划         │  planner（自动或手动调用）                 │
│  审查         │  code-reviewer（写代码后自动）            │
│  测试         │  tdd-guide（新功能/修bug时）              │
│  安全         │  security-reviewer（改敏感代码后）        │
│  构建修复     │  build-error-resolver（构建失败时）       │
├─────────────────────────────────────────────────────────┤
│  工作流       │  G1→G2→G3→G4→G5→G6→G7                   │
│               │  头脑风暴→隔离→计划→开发→测试→审查→完成    │
├─────────────────────────────────────────────────────────┤
│  记忆         │  "记住：..." / "忘掉..."                   │
│  规则         │  ~/.claude/rules/ (自动生效)              │
│  钩子         │  SessionStart → PostToolUse → Stop       │
└─────────────────────────────────────────────────────────┘
```

---

> **提示：** 最好的学习方式就是多用。遇到不知道怎么描述的问题，直接用大白话说出来——Claude Code 会理解你的意图。如果它做错了，直接说 "不对，应该是..." 来纠正它。
