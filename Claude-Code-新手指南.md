# Claude Code 新手指南

> 适用版本：当前环境 (DeepSeek V4 Pro 后端 + Claude Code CLI)
> 更新日期：2026-05-24

---

## 目录

**第一部分：入门与基础操作**
1. [Claude Code 是什么](#1-claude-code-是什么)
2. [安装与启动](#2-安装与启动)
3. [基本用法](#3-基本用法)
4. [CLI 命令与参数](#4-cli-命令与参数) ← 完整命令行参考
5. [内置工具 (Tools)](#5-内置工具-tools) ← Read/Write/Edit/Bash/Grep/Glob

**第二部分：核心概念**
6. [Agent（代理）](#6-核心概念agent代理) ← 50+ 专业子AI，含调用方法
7. [Skill（技能）](#7-核心概念skill技能) ← 80+ Slash 命令和知识技能
8. [Rule（规则）](#8-核心概念rule规则) ← 编码规范自动执行
9. [Hook（钩子）](#9-核心概念hook钩子) ← 事件触发自动化
10. [Memory（记忆）](#10-核心概念memory记忆) ← 持久化偏好记忆
11. [Plugin（插件市场）](#11-核心概念plugin插件市场) ← 社区扩展

**第三部分：工作流与参考**
12. [工作流门禁 G1-G7](#12-工作流门禁系统-g1-g7)
13. [完整 Slash 命令速查表](#13-完整-slash-命令速查表) ← 全部 90+ 命令
14. [快捷键](#14-快捷键)
15. [实用场景示例](#15-实用场景示例)
16. [常见问题 FAQ](#16-常见问题-faq)

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

## 4. CLI 命令与参数

### 4.1 启动交互式会话

```bash
claude                          # 在当前目录启动交互会话
claude -c                       # 继续最近的会话
claude -r                       # 从列表中选择要恢复的会话
claude -r "session-id"          # 恢复指定 ID 的会话
claude --resume "session-id"    # 同上，完整写法
```

### 4.2 一次性问答模式（非交互）

```bash
claude -p "这段代码有什么问题？"                    # 打印结果后退出
claude -p "帮我解释 src/main.py 的功能" --add-dir ./src  # 允许访问额外目录
claude -p "生成一个 React 组件" --model sonnet      # 指定模型
claude -p "..." --output-format json                 # JSON 格式输出
claude -p "..." --output-format stream-json          # 流式 JSON 输出
claude -p "..." --max-budget-usd 5                   # 设置费用上限(美元)
claude -p "..." --json-schema '{"type":"object",...}' # 结构化输出校验
```

### 4.3 会话管理

```bash
claude --name "修复登录bug"       # 给会话起个名字
claude --session-id "自定义UUID"  # 使用指定会话ID
claude --fork-session -r "xxx"   # 从旧会话分叉，不覆盖原会话
claude --from-pr                 # 从 PR 关联会话列表中选择
claude --from-pr 123             # 恢复关联到 PR #123 的会话
claude --no-session-persistence  # 不保存本次会话
```

### 4.4 模型与行为控制

```bash
claude --model sonnet              # 使用 Sonnet 模型
claude --model opus                # 使用 Opus 模型（最深推理）
claude --model haiku               # 使用 Haiku 模型（最快最便宜）
claude --effort max                # 最大努力级别
claude --effort medium             # 中等努力级别
claude --permission-mode plan      # 规划模式（先计划后执行）
claude --permission-mode acceptEdits # 自动接受编辑
claude --permission-mode auto      # 自动模式
claude --permission-mode bypassPermissions  # 跳过权限检查
```

### 4.5 工作区与隔离

```bash
claude -w                          # 创建隔离的 git worktree
claude -w "feature-x"              # 指定 worktree 名称
claude --tmux                      # 同时创建 tmux 会话（需要 -w）
claude --add-dir ./lib ./shared    # 允许访问额外目录
```

### 4.6 自定义代理和工具

```bash
# 启动时注册临时代理
claude --agents '{"reviewer":{"description":"Reviews code","prompt":"You are a code reviewer"}}'

# 限制可用工具
claude --tools "Bash,Read,Write,Edit"              # 只允许这些工具
claude --tools ""                                   # 禁用所有工具
claude --allowedTools "Bash(git *) Edit"            # 允许特定操作
claude --disallowedTools "Bash(rm *)"               # 禁止特定操作

# 指定使用哪个代理
claude --agent code-reviewer
```

### 4.7 MCP 服务器配置

```bash
claude --mcp-config ./mcp.json                    # 加载 MCP 配置文件
claude --mcp-config '{"server":{...}}'            # 直接传入 JSON
claude --strict-mcp-config --mcp-config ./mcp.json # 只用指定配置，忽略其他
```

### 4.8 系统提示与设置

```bash
claude --system-prompt "You are a Python expert"    # 自定义系统提示
claude --append-system-prompt "Always use type hints" # 追加到默认提示
claude --settings ./custom-settings.json            # 加载自定义设置
claude --setting-sources user,project               # 指定配置来源
claude --disable-slash-commands                     # 禁用所有技能
```

### 4.9 调试与诊断

```bash
claude --debug                  # 启用调试模式
claude --debug "api,hooks"      # 按类别过滤调试日志
claude --debug-file ./debug.log # 写入调试日志文件
claude --verbose                # 详细输出
claude doctor                   # 检查 Claude Code 健康状态
```

### 4.10 插件和 IDE

```bash
claude --plugin-dir ./my-plugins  # 加载额外插件目录（可重复）
claude --chrome                   # 启用 Chrome 浏览器集成
claude --ide                      # 自动连接到可用的 IDE
claude --bare                     # 最小模式：跳过 hooks/LSP/插件/记忆
```

### 4.11 管理命令

```bash
claude update              # 检查并安装更新
claude install stable      # 安装稳定版
claude install latest      # 安装最新版
claude mcp                 # 管理 MCP 服务器
claude plugin              # 管理插件
claude agents              # 列出已配置的代理
claude auth                # 管理认证
claude setup-token         # 设置长期认证令牌
```

---

## 5. 内置工具 (Tools)

Claude Code 拥有一组内置工具，能直接操作你的文件系统、搜索代码、执行命令。理解这些工具能帮你更精准地描述需求。

### 5.1 文件操作工具

| 工具 | 功能 | 典型用法场景 |
|------|------|-------------|
| **Read** | 读取文件内容 | 查看源代码、截图、PDF、Jupyter notebook |
| **Write** | 创建或完全重写文件 | 新建文件、整体替换文件内容 |
| **Edit** | 精确的字符串替换编辑 | 修改文件中的部分代码（推荐优先使用） |
| **Glob** | 按文件名模式搜索 | 查找所有 `*.tsx` 文件、搜索特定路径 |
| **Grep** | 按内容正则搜索 | 搜索函数定义、查找引用、找特定模式 |
| **Bash** | 执行 Shell 命令 | 运行测试、git 操作、npm/cargo/pip 命令 |

### 5.2 Bash 工具详解

Bash 是最常用的工具，可以执行系统命令：

```
# Claude Code 内部的 Bash 工具自动执行以下操作：
- 运行 npm install / pip install / cargo build 等包管理命令
- 执行 git 操作（status, diff, commit, push, log 等）
- 运行测试套件
- 启动开发服务器
- 任何你需要的命令行操作
```

**常用 Bash 指令示例：**

```bash
# 你在对话中说，Claude Code 会自动执行：
"帮我跑一下测试"                    → npm test / pytest / cargo test
"看看 git 状态"                     → git status
"安装 axios 依赖"                   → npm install axios
"创建一个新分支 feature-login"       → git checkout -b feature-login
"看一下最近的提交记录"               → git log --oneline -10
```

### 5.3 搜索工具详解

| 工具 | 用法 | 示例 |
|------|------|------|
| **Glob** | 文件名匹配 | `**/*.test.ts` 找所有测试文件 |
| **Grep** | 内容正则搜索 | `function\s+handleLogin` 找函数定义 |
| **Agent (Explore)** | 深入代码库探索 | "这个项目的路由是怎么组织的？" |

### 5.4 如何让 Claude Code 使用工具

你不需要手动调用工具——用自然语言描述需求即可：

```
你: 帮我找一下所有调用 deleteUser 的地方
    → Claude Code 用 Grep 搜索 "deleteUser"

你: 看看 src/components/ 下有哪些文件
    → Claude Code 用 Glob 搜索 "src/components/**/*"

你: 运行测试，看看有没有失败的
    → Claude Code 用 Bash 执行测试命令

你: 把 config.ts 里的 API_BASE_URL 改成 https://api.new.com
    → Claude Code 用 Edit 精确替换
```

---

## 6. 核心概念：Agent（代理）

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

### 6.5 代理调用方式详解

Claude Code 有三种方式调用代理：

#### 方式 1：自然语言自动触发（最常用）

```
你: 帮我审查一下刚改的代码       → 自动调用 code-reviewer
你: 这个功能怎么设计架构？       → 自动调用 architect
你: 构建报错了，帮我看看         → 自动调用 build-error-resolver
你: 帮我写单元测试               → 自动调用 tdd-guide
你: 这段代码有安全问题吗？       → 自动调用 security-reviewer
```

#### 方式 2：启动时指定代理

```bash
# 整个会话以特定代理身份运行
claude --agent planner
claude --agent code-reviewer
claude --agent tdd-guide
```

#### 方式 3：通过 Slash 命令间接调用

```
/plan          → 调用 planner 代理
/code-review   → 调用 code-reviewer 代理
/tdd           → 调用 tdd-guide 代理
/review-pr 123 → 调用 code-reviewer + security-reviewer 等多个代理
/santa-loop    → 启动 2 个独立审查代理
/feature-dev   → 调用 code-explorer + planner
/build-fix     → 调用 build-error-resolver 代理
```

### 6.6 完整代理目录（按功能分类）

#### 规划与架构类

| 代理 | 擅长模型 | 功能 |
|------|---------|------|
| **planner** | Opus | 复杂功能实现计划、任务分解 |
| **architect** | Opus | 系统架构设计、技术选型 |
| **code-architect** | - | 分析现有代码模式，输出实现蓝图 |

#### 代码审查类

| 代理 | 功能 |
|------|------|
| **code-reviewer** | 通用代码质量、模式、最佳实践 |
| **security-reviewer** | OWASP Top 10、安全漏洞检测 |
| **pr-test-analyzer** | PR 测试覆盖率和质量分析 |
| **silent-failure-hunter** | 检测被吞掉的错误、糟糕的回退逻辑 |
| **comment-analyzer** | 分析注释的准确性和维护风险 |
| **type-design-analyzer** | 分析类型设计的封装性和不变性 |

#### 测试类

| 代理 | 功能 |
|------|------|
| **tdd-guide** | 测试驱动开发全流程 |
| **e2e-runner** | 端到端测试生成和维护 |

#### 代码维护类

| 代理 | 功能 |
|------|------|
| **code-simplifier** | 简化和精炼代码 |
| **refactor-cleaner** | 死代码检测和清理 |
| **performance-optimizer** | 性能瓶颈分析和优化 |
| **code-explorer** | 追踪执行路径、映射架构层 |
| **doc-updater** | 更新文档和代码地图 |

#### 语言专用审查代理

| 代理 | 语言 |
|------|------|
| **python-reviewer** | Python (PEP 8, 类型提示, Pythonic) |
| **typescript-reviewer** | TypeScript/JavaScript (类型安全, 异步) |
| **go-reviewer** | Go (惯用模式, 并发安全) |
| **rust-reviewer** | Rust (所有权, 生命周期, unsafe) |
| **java-reviewer** | Java/Spring Boot (分层架构, JPA) |
| **kotlin-reviewer** | Kotlin (协程安全, Compose) |
| **csharp-reviewer** | C#/.NET (异步模式, 可空引用类型) |
| **cpp-reviewer** | C++ (内存安全, 现代C++惯用法) |
| **flutter-reviewer** | Flutter/Dart (Widget最佳实践) |

#### 构建修复类

| 代理 | 修复类型 |
|------|---------|
| **build-error-resolver** | 通用构建和 TypeScript 错误 |
| **go-build-resolver** | Go 构建、vet、lint |
| **rust-build-resolver** | Rust 借用检查器、Cargo |
| **java-build-resolver** | Java/Maven/Gradle |
| **kotlin-build-resolver** | Kotlin/Gradle |
| **dart-build-resolver** | Dart/Flutter 分析错误 |
| **cpp-build-resolver** | C++/CMake/链接器 |
| **pytorch-build-resolver** | PyTorch 张量形状、设备、梯度 |

#### 其他专业代理

| 代理 | 功能 |
|------|------|
| **database-reviewer** | PostgreSQL 查询优化、Schema 设计 |
| **a11y-architect** | WCAG 2.2 无障碍合规 |
| **seo-specialist** | SEO 审计、结构化数据、Core Web Vitals |
| **docs-lookup** | 查框架/库的最新官方文档 |
| **pentest-scanner** | 渗透测试和漏洞利用 |
| **healthcare-reviewer** | 医疗代码审查 (CDSS, PHI 合规) |
| **gan-planner / gan-generator / gan-evaluator** | GAN 对抗式开发（规格→实现→评估循环） |
| **loop-operator** | 自治代理循环的监控和干预 |
| **harness-optimizer** | 优化代理 harness 配置的可靠性和成本 |

#### 开源准备类

| 代理 | 功能 |
|------|------|
| **opensource-forker** | Fork 项目、剥离密钥和内部引用 |
| **opensource-sanitizer** | 扫描泄露的密钥/PII/内部引用 |
| **opensource-packager** | 生成 README、LICENSE、CONTRIBUTING 等 |

#### 沟通类

| 代理 | 功能 |
|------|------|
| **chief-of-staff** | 邮件/Slack/LINE/信使分级和草稿回复 |
| **conversation-analyzer** | 分析对话记录找出可自动化的模式 |

---

## 7. 核心概念：Skill（技能）

Skill 是**可复用的专业工作流**，存放在 `~/.claude/skills/` 和 `~/.claude/commands/` 中。每个 Skill 是一个 `.md` 文件，定义了特定场景下的专业流程。

### 7.1 Skill 的三种调用方式

#### 方式 1：Slash 命令调用（显式）

```
/plan                   → 调用 plan 技能（制定实现计划）
/code-review            → 调用 code-review 技能（代码审查）
/tdd                    → 调用 tdd 技能（测试驱动开发）
/commit                 → 调用 prp-commit 技能（智能提交）
/review-pr 123          → 调用 review-pr 技能（审查 PR）
```

#### 方式 2：自然语言自动触发（隐式）

```
你: "帮我审查代码"      → Claude Code 自动识别并调用 code-review 技能
你: "提交一下改动"      → 自动调用 prp-commit 技能
你: "看看这个 PR"       → 自动调用 review-pr 技能
```

#### 方式 3：Skill 工具调用（编程式）

Claude Code 内部通过 `Skill` 工具调用技能：
```
Skill({ skill: "python-patterns" })     # 加载 Python 最佳实践
Skill({ skill: "frontend-design" })     # 加载前端设计指南
Skill({ skill: "security-review" })     # 加载安全检查清单
```

### 7.2 日常开发技能（Slash 命令）

| 命令 | 参数 | 功能 |
|------|------|------|
| **/plan** | 无 | 需求确认 → 风险评估 → 分步计划，确认后才动手 |
| **/tdd** | 无 | RED(写测试) → GREEN(实现) → REFACTOR(重构) |
| **/code-review** | `[PR号\|空=本地]` | 审查代码，输出严重级别标注的问题列表 |
| **/review-pr** | `<PR号>` | 多代理综合 PR 审查 |
| **/commit** | 无 | 用自然语言描述要提交什么，自动分析和提交 |
| **/feature-dev** | 无 | 发现→代码库探索→计划→实现→验证 |
| **/build-fix** | 无 | 自动检测构建系统，增量修复错误 |

| 技能 | 用途 | 示例 |
|------|------|------|
| **/plan** | 制定实现计划 | `/plan` 然后描述你要做什么 |
| **/commit** | 智能提交代码 | `/commit` 自动分析变更并提交 |
| **/code-review** | 代码审查 | `/code-review` 审查未提交的改动 |
| **/review-pr** | 审查 PR | `/review-pr 123` 审查 PR #123 |
| **/tdd** | 测试驱动开发 | `/tdd` 按 TDD 流程开发 |
| **/feature-dev** | 引导式功能开发 | `/feature-dev` 按工作流一步步来 |

### 7.3 语言和框架技能

这些技能在写对应语言代码时会自动加载，也可以手动调用 `Skill({ skill: "xxx" })` 来加载：

| 技能 | 适用领域 |
|------|---------|
| **python-patterns** | Python PEP 8、类型提示、Pythonic 惯用法 |
| **golang-patterns** | Go 惯用模式、并发模式、错误处理 |
| **rust-patterns** | Rust 所有权、借用、trait、生命周期 |
| **dart-flutter-patterns** | Flutter/Dart 生产级模式（BLoC, Riverpod, Provider） |
| **kotlin-patterns** | Kotlin 惯用模式、可空安全、DSL、协程 |
| **kotlin-coroutines-flows** | Kotlin 结构化并发、Flow 操作符、测试 |
| **kotlin-ktor-patterns** | Ktor 路由 DSL、插件、Koin DI、序列化 |
| **kotlin-exposed-patterns** | Exposed ORM DSL、DAO、事务、连接池 |
| **swiftui-patterns** | SwiftUI 架构、@Observable、导航、性能 |
| **swift-concurrency-6-2** | Swift 6.2 可及性并发（单线程默认） |
| **swift-actor-persistence** | Swift Actor 线程安全数据持久化 |
| **cpp-coding-standards** | C++ Core Guidelines（isocpp.github.io） |
| **java-coding-standards** | Spring Boot 命名、不可变性、Optional |
| **dotnet-patterns** | C#/.NET DI、async/await、可空引用类型 |
| **perl-patterns** | 现代 Perl 5.36+ 惯用模式 |
| **springboot-patterns** | Spring Boot 分层架构、REST API、JPA |
| **django-patterns** | Django REST API、ORM、缓存、中间件 |
| **nestjs-patterns** | NestJS 模块/控制器/提供者/DTO 验证 |
| **laravel-patterns** | Laravel 路由、Eloquent、队列、事件 |
| **android-clean-architecture** | Android/KMP Clean Architecture |
| **compose-multiplatform-patterns** | Compose Multiplatform 状态管理、导航、主题 |
| **frontend-patterns** | React/Next.js 状态管理、性能优化 |
| **frontend-design** | 高质量前端 UI 设计（非模板化） |
| **backend-patterns** | Node.js/Express 后端模式、API 设计 |
| **api-design** | REST API 设计：命名、分页、错误、版本控制 |
| **mcp-server-patterns** | MCP 服务器构建、工具/资源/Prompts/Zod |
| **claude-api** | Anthropic Claude API 模式、流式、工具使用、缓存 |
| **agentic-engineering** | 带评估的代理工程、模型路由 |
| **agent-harness-construction** | AI 代理 action space 设计和优化 |
| **agent-introspection-debugging** | AI 代理失败的结构化调试 |

### 7.4 代码质量技能

| 命令 | 功能 |
|------|------|
| **/fix** | 增量修复构建和类型错误 |
| **/clean** | 清理和简化代码，去除重复 |
| **/refactor** | 重构优化代码结构 |
| **/verify** | 验证功能正确性 |
| **/test-coverage** | 检查测试覆盖率 |
| **/santa-loop** | 启动对抗式双审查循环（两个独立审查员都通过才放行） |
| **/quality-gate** | 运行质量门禁检查 |
| **/prune** | 删除超过 30 天的未推广 instincts |

### 7.5 自动化与运维技能

| 命令 | 参数 | 功能 |
|------|------|------|
| **/loop** | `<间隔> <任务>` | 定时循环（如 `/loop 5m 检查构建`） |
| **/loop-start** | `[模式] [--mode safe\|fast]` | 启动托管自治循环 |
| **/loop-status** | 无 | 查看循环运行状态 |
| **/hookify** | 无 | 从对话分析中创建预防性钩子 |
| **/hookify-list** | 无 | 列出所有 hookify 规则 |
| **/hookify-configure** | 无 | 交互式启用/禁用 hookify 规则 |
| **/hookify-help** | 无 | hookify 帮助 |
| **/save-session** | 无 | 保存当前会话到 `~/.claude/session-data/` |
| **/resume-session** | 无 | 从保存文件恢复最近会话 |
| **/sessions** | 无 | 管理会话历史、别名和元数据 |
| **/pm2** | 无 | PM2 初始化（进程管理） |
| **/checkpoint** | 无 | 创建检查点 |
| **/claw** | 无 | nanoclaw-repl 入口 |
| **/setup-pm** | 无 | 设置 PM 相关配置 |

### 7.6 搜索与研究技能

| 命令 | 功能 |
|------|------|
| **/search-first** | 先搜索 GitHub/包注册表/文档，再写代码 |
| **/docs** | 搜索技术文档 |
| **deep-research** | 多源深度调研，输出带引用的报告 |
| **exa-search** | Exa 神经搜索引擎，支持网页/代码/公司/人员搜索 |

### 7.7 Git 与 PR 管理

| 命令 | 功能 |
|------|------|
| **/prp-commit** | 自然语言描述要提交的内容，自动分析并提交 |
| **/prp-plan** | 带代码库分析和模式提取的实现计划 |
| **/prp-implement** | 按计划执行实现，带严格验证循环 |
| **/prp-pr** | 自动从当前分支创建 PR |
| **/prp-prd** | 交互式 PRD 生成器（问题优先，假设驱动） |

### 7.8 项目知识管理

| 命令 | 功能 |
|------|------|
| **/learn** | 从会话中提取可复用模式 |
| **/learn-eval** | 提取模式并自我评估质量 |
| **/evolve** | 分析 instincts 并建议改进 |
| **/instinct-export** | 导出 instincts 到文件 |
| **/instinct-import** | 从文件或 URL 导入 instincts |
| **/instinct-status** | 查看已学习的 instincts 和置信度 |
| **/promote** | 将项目级 instincts 提升为全局 |
| **/rules-distill** | 从代码中蒸馏规则 |

### 7.9 安全与渗透测试

| 命令                          | 功能                     |
| --------------------------- | ---------------------- |
| **/pentest-code-audit**     | 多语言代码安全审计              |
| **/pentest-web-scan**       | Web 漏洞扫描（OWASP Top 10） |
| **/security-review**        | 安全审查清单和模式（认证/输入/密钥/支付） |
| **/security-bounty-hunter** | 寻找可远程触达的赏金级漏洞          |

### 7.10 前端专项技能

| 命令/技能                     | 功能                 |
| ------------------------- | ------------------ |
| **frontend-design**       | 创建有设计感的前端界面，非模板化   |
| **frontend-patterns**     | React/Next.js 前端模式 |
| **api-connector-builder** | 按项目现有模式构建 API 连接器  |

### 7.11 其他专项技能

| 命令/技能 | 功能 |
|------|------|
| **/gan-build** | GAN 构建流程 |
| **/gan-design** | GAN 设计流程 |
| **/devfleet** | claude-devfleet 入口 |
| **/orchestrate** | dmux-workflows / autonomous-agent-harness 入口 |
| **/model-route** | 模型路由配置 |
| **/context-budget** | 上下文预算管理 |
| **/eval** | eval-harness 入口 |
| **/prompt-optimize** | Prompt 优化器入口 |
| **/update-codemaps** | 更新代码地图 |
| **/update-docs** | 更新文档 |
| **/skill-create** | 分析 git 历史提取编码模式生成 SKILL.md |
| **/skill-health** | 技能组合健康仪表盘 |
| **/projects** | 列出已知项目及其 instinct 统计 |
| **/jira** | Jira 工单检索和分析 |
| **/aside** | 快速回答副问题，不丢失主线上下文 |
| **/harness-audit** | 审查代理 harness 配置 |
| **/agent-sort** | 代理排序技能入口 |
| **/cpp-build** | C++ 构建修复 |
| **/cpp-review** | C++ 代码审查 |
| **/cpp-test** | C++ TDD 流程 |
| **/go-build** | Go 构建修复 |
| **/go-review** | Go 代码审查 |
| **/go-test** | Go TDD 流程 |
| **/rust-build** | Rust 构建修复 |
| **/rust-review** | Rust 代码审查 |
| **/rust-test** | Rust TDD 流程 |
| **/flutter-build** | Flutter 构建修复 |
| **/flutter-review** | Flutter 代码审查 |
| **/flutter-test** | Flutter 测试 |
| **/kotlin-build** | Kotlin 构建修复 |
| **/kotlin-review** | Kotlin 代码审查 |
| **/kotlin-test** | Kotlin TDD 流程 |
| **/gradle-build** | Android/KMP Gradle 构建修复 |
| **/python-review** | Python 代码审查 |
| **nutrient-document-processing** | PDF/DOCX/XLSX/PPTX 文档处理 |

### 7.12 多模型协作

| 命令 | 功能 |
|------|------|
| **/multi-plan** | 多模型协作规划 |
| **/multi-execute** | 多模型协作执行 |
| **/multi-frontend** | 前端专注的多人协作 |
| **/multi-backend** | 后端专注的多人协作 |
| **/multi-workflow** | 多模型协作开发完整流程 |

---

## 8. 核心概念：Rule（规则）

Rule 是你给 Claude Code 设定的**长期行为准则**。它们告诉 Claude Code 在写代码时要遵守什么标准。

### 8.1 规则存放位置

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

### 8.2 你当前已启用的规则

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

## 9. 核心概念：Hook（钩子）

Hook 是在**特定事件发生时自动触发的脚本**，让你的工作流自动化。

### 9.1 你当前的钩子配置

| 触发时机                   | 做什么                    |
| ---------------------- | ---------------------- |
| **SessionStart**（会话开始） | 显示 G1-G7 工作流门禁提醒       |
| **PostToolUse**（写文件后）  | 提醒进行 TDD 和 Code Review |
| **Stop**（会话结束前）        | 验证是否完成 G7（分支/PR/清理）    |

### 9.2 钩子类型

| 类型              | 触发时机  | 用途示例            |
| --------------- | ----- | --------------- |
| **PreToolUse**  | 工具执行前 | 阻止写入超大文件、验证参数   |
| **PostToolUse** | 工具执行后 | 自动格式化代码、运行 lint |
| **Stop**        | 会话结束时 | 运行构建验证          |

---

## 10. 核心概念：Memory（记忆）

Memory 是 Claude Code 的**持久化记忆系统**。它会在 `~/.claude/projects/<项目>/memory/` 下保存关于你、项目、反馈等信息，让未来的对话能更好地理解你的偏好。

### 10.1 记忆类型

| 类型            | 存什么                  | 示例                             |
| ------------- | -------------------- | ------------------------------ |
| **user**      | 你的角色、偏好、技能水平         | "我熟悉 Go，但 React 是新手"           |
| **feedback**  | 你对 Claude Code 行为的反馈 | "不要用 mock 数据库做集成测试"            |
| **project**   | 项目背景、目标、约束           | "合并冻结期 2026-03-05 起"           |
| **reference** | 外部资源链接               | "流水线 Bug 在 Linear 项目 INGEST 中" |

### 10.2 手动管理记忆

```
你: 记住：我们这个项目要兼容 Python 3.10+
你: 我刚刚说了什么来着？     → Claude Code 会检查记忆
你: 忘掉关于 Python 版本的事
```

---

## 11. 核心概念：Plugin（插件市场）

Claude Code 支持从 GitHub 仓库安装插件市场，获取社区贡献的代理、技能、钩子和命令。

### 11.1 你当前已安装的插件市场

| 市场                          | 来源                                 | 内容             |
| --------------------------- | ---------------------------------- | -------------- |
| **claude-plugins-official** | anthropics/claude-plugins-official | Anthropic 官方插件 |
| **everything-claude-code**  | affaan-m/everything-claude-code    | 综合社区插件合集       |
| **thedotmack**              | thedotmack/claude-mem              | 记忆相关功能增强       |
| **planning-with-files**     | OthmanAdi/planning-with-files      | 文件式规划工作流       |

### 11.2 管理插件

```bash
claude plugin              # 交互式插件管理
claude plugin install <name>  # 安装插件
claude plugin uninstall <name>  # 卸载插件
claude plugin list          # 列出已安装插件
claude plugin update        # 更新所有插件
```

### 11.3 插件市场配置

插件市场在 `~/.claude/settings.json` 中配置：

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  }
}
```

### 11.4 安装位置

```
~/.claude/plugins/
├── cache/                    # 已下载的插件缓存
├── marketplaces/             # 已安装的插件市场
│   ├── claude-plugins-official/
│   ├── everything-claude-code/
│   ├── thedotmack/
│   └── planning-with-files/
├── installed_plugins.json    # 已安装插件清单
└── known_marketplaces.json   # 已知市场列表
```

---

## 12. 工作流门禁系统 (G1-G7)

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

## 13. 完整 Slash 命令速查表

### 核心开发流程

| 命令               | 参数                 | 功能                             |
| ---------------- | ------------------ | ------------------------------ |
| `/plan`          | 无                  | 需求确认→风险评估→分步计划，确认前不动代码         |
| `/tdd`           | 无                  | RED(测试)→GREEN(实现)→REFACTOR(重构) |
| `/code-review`   | `[PR号\|URL\|空=本地]` | 代码审查，输出严重级别标注列表                |
| `/review-pr`     | `<PR号>`            | 多代理综合 PR 审查                    |
| `/prp-commit`    | 无                  | 自然语言描述后自动分析提交                  |
| `/prp-plan`      | 无                  | 带代码库分析+模式提取的实现计划               |
| `/prp-implement` | 无                  | 按计划带验证循环的实现                    |
| `/prp-pr`        | 无                  | 从当前分支自动创建 PR                   |
| `/prp-prd`       | 无                  | 交互式 PRD 生成器                    |
| `/feature-dev`   | 无                  | 发现→探索→计划→实现→验证                 |
| `/build-fix`     | 无                  | 自动检测构建系统→增量修复                  |

### 代码质量

| 命令 | 功能 |
|------|------|
| `/clean` | 清理和简化代码 |
| `/refactor` | 重构优化代码结构 |
| `/verify` | 验证功能正确性 |
| `/test-coverage` | 检查测试覆盖率 |
| `/santa-loop` | 对抗式双审查循环（两个独立审查员都通过才放行） |
| `/quality-gate` | 运行质量门禁检查 |

### 自动化

| 命令 | 参数 | 功能 |
|------|------|------|
| `/loop` | `<间隔> <任务>` | 定时循环（如 `/loop 5m 检查构建状态`） |
| `/loop-start` | `[模式] [--mode]` | 启动自治代理循环 |
| `/loop-status` | 无 | 查看循环状态 |
| `/hookify` | 无 | 从对话创建预防性钩子 |
| `/hookify-list` | 无 | 列出 hookify 规则 |
| `/hookify-configure` | 无 | 交互式启用/禁用规则 |
| `/hookify-help` | 无 | hookify 帮助 |

### 会话管理

| 命令                | 功能                                |
| ----------------- | --------------------------------- |
| `/save-session`   | 保存当前会话到 `~/.claude/session-data/` |
| `/resume-session` | 从保存文件恢复最近会话                       |
| `/sessions`       | 管理会话历史、别名、元数据                     |
| `/checkpoint`     | 创建检查点                             |
| `/aside`          | 快速回答副问题不丢失上下文                     |
| `/context-budget` | 管理上下文预算                           |

### 知识管理

| 命令 | 功能 |
|------|------|
| `/learn` | 从会话中提取可复用模式 |
| `/learn-eval` | 提取+自我评估质量 |
| `/evolve` | 分析 instincts 并建议改进 |
| `/instinct-export` | 导出 instincts 到文件 |
| `/instinct-import` | 从文件/URL 导入 instincts |
| `/instinct-status` | 查看已学习 instincts |
| `/promote` | 提升项目级 instincts 为全局 |
| `/prune` | 删除 30 天外的未推广 instincts |
| `/rules-distill` | 从代码蒸馏规则 |
| `/projects` | 列出已知项目 |
| `/skill-create` | 分析 git 历史生成 SKILL.md |
| `/skill-health` | 技能组合健康仪表盘 |

### 安全

| 命令 | 功能 |
|------|------|
| `/pentest-code-audit` | 多语言代码安全审计 |
| `/pentest-web-scan` | Web 漏洞扫描 |

### 文档与搜索

| 命令 | 功能 |
|------|------|
| `/docs` | 搜索技术文档 |
| `/update-codemaps` | 更新代码地图 |
| `/update-docs` | 更新文档 |
| `/search-first` | 先搜索再写代码 |

### 语言专项

| 命令 | 功能 |
|------|------|
| `/cpp-build` | C++ 构建修复 |
| `/cpp-review` | C++ 代码审查 |
| `/cpp-test` | C++ TDD |
| `/go-build` | Go 构建修复 |
| `/go-review` | Go 代码审查 |
| `/go-test` | Go TDD |
| `/rust-build` | Rust 构建修复 |
| `/rust-review` | Rust 代码审查 |
| `/rust-test` | Rust TDD |
| `/flutter-build` | Flutter 构建修复 |
| `/flutter-review` | Flutter 代码审查 |
| `/flutter-test` | Flutter 测试 |
| `/kotlin-build` | Kotlin 构建修复 |
| `/kotlin-review` | Kotlin 代码审查 |
| `/kotlin-test` | Kotlin TDD |
| `/gradle-build` | Gradle 构建修复 |
| `/python-review` | Python 代码审查 |

### 多模型协作

| 命令 | 功能 |
|------|------|
| `/multi-plan` | 多模型协作规划 |
| `/multi-execute` | 多模型协作执行 |
| `/multi-frontend` | 前端多人协作 |
| `/multi-backend` | 后端多人协作 |
| `/multi-workflow` | 多模型完整工作流 |

### 其他

| 命令 | 功能 |
|------|------|
| `/jira` | Jira 工单操作 |
| `/model-route` | 模型路由配置 |
| `/agent-sort` | 代理排序 |
| `/devfleet` | claude-devfleet |
| `/orchestrate` | 工作流编排入口 |
| `/claw` | nanoclaw-repl |
| `/eval` | eval-harness |
| `/prompt-optimize` | Prompt 优化 |
| `/harness-audit` | 审查 harness 配置 |
| `/setup-pm` | PM 配置 |
| `/pm2` | PM2 初始化 |

---

## 14. 快捷键

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

## 15. 实用场景示例

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

## 16. 常见问题 FAQ

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
┌──────────────────────────────────────────────────────────────────┐
│                     Claude Code 速查卡                            │
├──────────────────────────────────────────────────────────────────┤
│  交互式启动        │  claude                                       │
│  一次性问答        │  claude -p "问题"                             │
│  继续上次会话      │  claude -c                                    │
│  恢复会话          │  claude -r [session-id]                       │
│  指定模型          │  claude --model sonnet                        │
│  创建隔离工作区    │  claude -w [name]                             │
│  调试模式          │  claude --debug                               │
│  健康检查          │  claude doctor                                │
│  插件管理          │  claude plugin                                │
│  更新              │  claude update                                │
├──────────────────────────────────────────────────────────────────┤
│  /plan              │  制定实现计划（先计划后执行）                  │
│  /code-review       │  代码审查（可带 PR 号）                       │
│  /tdd               │  测试驱动开发（红→绿→重构）                    │
│  /commit            │  智能提交                                    │
│  /build-fix         │  自动修复构建错误                            │
│  /feature-dev       │  引导式功能开发                              │
│  /review-pr <N>     │  多代理综合 PR 审查                          │
│  /santa-loop        │  对抗式双审查                                │
│  /loop <间隔> <任务>│  定时循环执行                                │
│  /search-first      │  先搜索已有方案再写代码                      │
├──────────────────────────────────────────────────────────────────┤
│  核心代理 (自动或手动调用)                                          │
│  planner            │  复杂功能规划 (Opus)                         │
│  code-reviewer      │  通用代码审查 (写完代码自动)                   │
│  tdd-guide          │  测试驱动开发                                │
│  security-reviewer  │  安全检查 (改敏感代码后)                      │
│  architect          │  系统架构设计                                │
│  build-error-resolver│ 构建修复                                    │
├──────────────────────────────────────────────────────────────────┤
│  内置工具 (Claude Code 自动选用)                                   │
│  Read    │  读文件（代码、图片、PDF、Notebook）                     │
│  Write   │  创建/覆写文件                                         │
│  Edit    │  精确字符串替换编辑                                     │
│  Bash    │  执行 Shell 命令                                       │
│  Grep    │  代码内容正则搜索                                       │
│  Glob    │  文件名模式匹配                                         │
├──────────────────────────────────────────────────────────────────┤
│  快捷键                                                           │
│  Ctrl+O  │  查看 AI 思考过程    │  Alt+T   │  切换扩展思考           │
│  Ctrl+C  │  中断当前操作        │  Ctrl+D  │  退出                  │
│  Tab     │  自动补全路径        │  Esc Esc │  回到对话              │
├──────────────────────────────────────────────────────────────────┤
│  工作流      │  G1→G2→G3→G4→G5→G6→G7                              │
│              │  头脑风暴→隔离→计划→开发→测试→审查→完成                │
│  记忆        │  "记住：..." / "忘掉..."                            │
│  规则        │  ~/.claude/rules/ (自动生效)                        │
│  钩子        │  SessionStart → PostToolUse → Stop                │
│  插件        │  claude plugin (管理)                              │
└──────────────────────────────────────────────────────────────────┘
```

---

> **提示：** 最好的学习方式就是多用。遇到不知道怎么描述的问题，直接用大白话说出来——Claude Code 会理解你的意图。如果它做错了，直接说 "不对，应该是..." 来纠正它。
