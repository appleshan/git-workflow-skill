# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Git-Workflow Skill 是一个 **文档项目**，为 Claude Code 提供智能 Git topic 工作流助手。通过包装生产级 Git aliases，提供自然语言命令映射、执行前安全检查和工作流指导。

**关键理解**：
- 这不是传统代码项目，主要是 Markdown 文档
- Skill 的实际部署位置在 `~/.claude/skills/git-workflow/`
- 本仓库用于开发和维护 Skill 文档

## Repository Structure

```
git-workflow-skill/
├── skills/git-workflow/          # Skill 源文件（需同步到 ~/.claude/skills/）
│   ├── SKILL.md                  # 主 Skill 定义（366 行）
│   └── references/               # 参考文档（5 个文件，共 3254 行）
│       ├── git-topic-workflow.md
│       ├── git-safety-mechanisms.md
│       ├── git-pr-preparation.md
│       ├── git-advanced-operations.md
│       └── git-troubleshooting.md
├── docs/testing.md               # 测试验证清单
├── examples/scenarios.md         # 使用场景示例
└── README.md                     # 项目说明文档

部署位置（外部）:
~/.claude/skills/git-workflow/    # Skill 实际运行位置
~/.claude/skills/skill-rules.json # 触发规则配置
```

## Validation and Deployment

### 验证 Skill 部署状态

```bash
# 检查 Skill 文件
ls ~/.claude/skills/git-workflow/

# 验证触发规则
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# JSON 格式检查
python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null
```

### 同步修改到部署位置

修改 `skills/git-workflow/` 下的文件后，需要手动同步到 `~/.claude/skills/git-workflow/`：

```bash
# 同步主文档
cp skills/git-workflow/SKILL.md ~/.claude/skills/git-workflow/

# 同步参考文档
cp -r skills/git-workflow/references/*.md ~/.claude/skills/git-workflow/references/
```

## Architecture Principles

### 职责边界（关键设计原则）

```
用户自然语言
    ↓ (意图识别)
Git-Workflow Skill
    ↓ (状态检查 + 命令生成)
Production Git Aliases
    ↓ (执行 + 内置安全机制)
Git 操作
```

**Skill 负责**：
- 意图理解和命令映射
- 执行前状态检查
- 工作流指导和问题诊断

**不做的事（YAGNI）**：
- ❌ 不重写 aliases 逻辑
- ❌ 不修改 dotfiles 配置
- ❌ 不添加新的 shell 脚本

### 核心概念

- **Topic 工作流**：短生命周期分支，快速合并，`--no-ff` 保留历史
- **三阶段安全检查**：执行前 (工作区/分支/同步状态) → 执行中 (aliases 内置) → 执行后 (结果验证)
- **上下文感知**：根据当前分支状态、工作区状态推荐合适操作

## Core Git Aliases (External Dependency)

Skill 依赖的生产级 Git aliases 位于：
`~/projects/private/dotfiles/git/.config/git/conf/aliases.gitconfig`

核心命令：
- `tnr` (topic-new-remote): 创建分支并推送到远程
- `tn` (topic-new): 创建仅本地分支
- `tmg` (topic-merge): `--no-ff` merge 并删除分支
- `td` (topic-delete): 智能同步删除本地+远程分支
- `bdf` (branch-diff): 与 base 分支对比
- `blg` (branch-log): 查看 topic 分支 commits
- `fixup`: 交互式修改历史 commit
- `save`/`snapshot`: 保存工作进度

## Testing and Verification

运行测试验证：参考 `docs/testing.md`

核心测试类别：
1. **触发测试**：中文/英文/混合关键词触发准确性
2. **功能测试**：创建分支、合并、PR 准备、冲突解决、误操作恢复
3. **状态感知测试**：工作区状态、分支状态的正确识别
4. **边界测试**：不相关输入、模糊输入、部分匹配

## Modification Guidelines

### 修改主 Skill 文档

1. 编辑 `skills/git-workflow/SKILL.md`
2. 重点关注：意图识别逻辑、命令映射表、安全检查清单
3. 同步到部署位置：`cp skills/git-workflow/SKILL.md ~/.claude/skills/git-workflow/`
4. 验证：运行 docs/testing.md 中的测试 2.x (功能测试)

### 修改参考文档

1. 编辑 `skills/git-workflow/references/<document>.md`
2. 关注点：工作流详解、安全机制、高级操作、故障排查
3. 同步到部署位置
4. 验证：相关功能测试 + 交叉引用检查

### 修改触发规则

1. 编辑 `~/.claude/skills/skill-rules.json`（注意：不在本仓库）
2. 修改 `git-workflow` 条目的 `keywords` 或 `intentPatterns`
3. JSON 格式验证：`python3 -m json.tool ~/.claude/skills/skill-rules.json`
4. 验证：运行 docs/testing.md 中的测试 1.x (触发测试) 和 4.x (边界测试)

## Documentation Style

维护文档时遵循的风格：
- 中英混合：标题和关键术语可用英文，说明用中文
- 结构化：使用表格、清单、代码块提升可读性
- 实战导向：提供具体命令和示例，避免纯理论
- 三阶段说明：执行前检查 → 执行命令 → 执行后验证
- 安全优先：明确标注破坏性操作的前置条件

## Common Workflows

### 开发新功能到 Skill

1. 在 `skills/git-workflow/references/` 中补充文档
2. 在 `SKILL.md` 中添加命令映射表条目
3. 在 `docs/testing.md` 中添加测试用例
4. 同步到 `~/.claude/skills/git-workflow/`
5. 运行测试验证

### 修复触发问题

1. 复现问题：记录触发/未触发的具体输入
2. 检查 `skill-rules.json` 的 keywords 和 intentPatterns
3. 调整正则表达式或关键词列表
4. JSON 格式验证
5. 回归测试：docs/testing.md 测试 1.x 和 4.x

### 更新外部依赖的 Aliases

当 `~/projects/private/dotfiles/git/.config/git/conf/aliases.gitconfig` 变更时：
1. 在 `SKILL.md` 和相关参考文档中同步命令说明
2. 更新 README.md 的命令映射表
3. 验证所有涉及该命令的测试用例
