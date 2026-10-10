# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 仓库概述

这是一个 Obsidian 知识库（Vault），用于存储个人计算机技术学习笔记。笔记语言以中文为主。

- 纯笔记仓库：没有构建、测试或 lint 工具链，不需要（也不应）引入这类配置。
- 笔记主要由用户通过 Obsidian 编辑，Codex 协助整理、编写和修改笔记内容。

## 目录结构

```
计算机技术学习/
├── C++学习/            —— C++ 基础入门笔记
├── python学习/          —— Python 零基础语法笔记
├── web渗透测试/         —— Web 安全与渗透测试笔记（含 assets 附件）
├── 计算机网络/          —— 计算机网络体系结构笔记（含 assets）
├── 漏洞报告/            —— 实际项目安全测试报告（按项目/公司分目录）
├── 刷题/               —— 刷题记录
└── 智能计算平台应用开发/ —— 智能计算平台相关
```

- 根目录下的 `Codex-新手指南.md` 是 Codex 使用教程，`Markdown基础语法.md` 是 Markdown 参考。
- `漏洞报告/` 下每个项目一个目录（如 `大连（林）精密铸造有限公司/`），报告正文为 `.md`，同目录还可能存在 `.docx`/`.html` 版本、`.zip` 证明包，截图直接放在项目目录下或 `assets/` 子目录中。这些是实际项目测试报告，内容敏感，编辑时保持原有信息结构。

## Obsidian 配置

- 已安装插件：
  - `obsidian-git`：文件变更后自动提交并推送，提交信息 `vault backup: YYYY-MM-DD HH:mm:ss`，启动时自动 pull。
  - `obsidian-custom-attachment-location`：粘贴附件自动归档并重命名。
  - `realclaudian`：个人 Codex 集成，已被 .gitignore 完全排除。
- `.obsidian/` 下的核心配置（app.json、community-plugins.json 等）随仓库推送，`workspace.json`（个人工作区状态）被排除。

## 关键约定

- **所有笔记和文件路径均使用中文命名。** 创建新内容时遵循此规范。
- **Markdown 相对链接**: 笔记间使用标准 Markdown 相对路径链接 `[文本](./路径/文件.md)`，而非 Obsidian Wiki 链接语法（`useMarkdownLinks: true`）。
- **附件存放**: 每个笔记的附件存放在其所在目录下 `assets/笔记文件名/` 子目录中，文件名由插件自动生成（`file-YYYYMMDDHHmmssSSS.扩展名`）。附件不限于图片，也常见 `.xmind`、`.pdf`、`.7z`、`.zip` 等格式。
- **Git 规范**:
  - 提交由 obsidian-git 插件自动完成，手动提交时遵循 `type: description` 格式。
  - `.gitignore` 已排除：AGENTS.md、插件编译代码（main.js/styles.css/obsidian_askpass.sh）、个人配置（.claudian/、realclaudian、.Codex/settings.local.json）、.obsidian/workspace.json、activity.log、*.base、OS 临时文件。
  - 会被推送的内容：笔记（.md）、笔记附件（assets/）、漏洞报告证明文件（.zip/.docx/.html 等）、Obsidian 核心配置与插件设置（data.json/manifest.json）。
- **不使用 emoji**, 除非用户明确要求。

## 笔记编写风格

- 章节用数字编号的多级标题组织（如 `## 1.原理及分类`），层次分明
- 原理说明常用引用块（`>`），要点用有序/无序列表
- 代码块标注语言类型（```cpp, ```python, ```bash 等），行内代码用反引号
- 关键概念使用 **加粗** 强调
