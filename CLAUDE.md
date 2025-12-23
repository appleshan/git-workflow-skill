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
├── skills/
│   ├── git-workflow/              # Git 工作流 Skill
│   │   ├── SKILL.md               # 主 Skill 定义（366 行）
│   │   ├── references/            # 参考文档（5 个文件，共 3254 行）
│   │   │   ├── git-topic-workflow.md
│   │   │   ├── git-safety-mechanisms.md
│   │   │   ├── git-pr-preparation.md
│   │   │   ├── git-advanced-operations.md
│   │   │   └── git-troubleshooting.md
│   │   ├── docs/                  # 开发文档（不部署）
│   │   │   ├── README.md          # 英文开发文档
│   │   │   ├── README_zh-CN.md    # 中文开发文档
│   │   │   └── testing.md
│   │   └── examples/scenarios.md
│   └── gh-pr-create/              # GitHub PR 创建 Skill
│       ├── SKILL.md               # PR 创建 Skill 定义
│       ├── references/            # PR 相关参考文档
│       │   ├── pr-templates.md
│       │   ├── gh-integration.md
│       │   └── base-branch-detection.md
│       ├── docs/                  # 开发文档（不部署）
│       │   ├── README.md          # 英文开发文档
│       │   ├── README_zh-CN.md    # 中文开发文档
│       │   └── testing.md
│       └── examples/scenarios.md
├── git/                           # Git Aliases 配置文件
│   ├── aliases.gitconfig          # 生产级 Git aliases
│   └── Git-Aliases-Reference-Manual.md
├── README.md
├── README_zh-CN.md
└── CLAUDE.md

部署位置（外部）:
~/.claude/skills/git-workflow/     # Git 工作流 Skill 运行位置
~/.claude/skills/gh-pr-create/     # PR 创建 Skill 运行位置
~/.claude/skills/skill-rules.json  # 触发规则配置
```

## Daily Development Commands

作为文档项目，不需要传统的构建/测试命令，主要是文档编辑和验证。

### 文档编辑

```bash
# 编辑主 Skill 文档
vim skills/git-workflow/SKILL.md

# 编辑参考文档
vim skills/git-workflow/references/git-topic-workflow.md
vim skills/git-workflow/references/git-safety-mechanisms.md
vim skills/git-workflow/references/git-pr-preparation.md
vim skills/git-workflow/references/git-advanced-operations.md
vim skills/git-workflow/references/git-troubleshooting.md

# 编辑测试文档
vim docs/testing.md

# 编辑示例文档
vim examples/scenarios.md
```

### 文档验证

```bash
# 检查 Markdown 语法（可选，需要安装 markdownlint-cli）
markdownlint skills/git-workflow/**/*.md

# 检查文档长度统计
wc -l skills/git-workflow/SKILL.md
wc -l skills/git-workflow/references/*.md

# 验证触发规则 JSON 格式
python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

# 验证部署状态
ls -lh ~/.claude/skills/git-workflow/
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json
```

### 同步修改到部署位置

修改 `skills/git-workflow/` 下的文件后，需要手动同步到 `~/.claude/skills/git-workflow/`：

```bash
# 同步主文档
cp skills/git-workflow/SKILL.md ~/.claude/skills/git-workflow/

# 同步参考文档
cp -r skills/git-workflow/references/*.md ~/.claude/skills/git-workflow/references/

# 同步所有文件（批量操作，排除开发文档）
rsync -av --delete --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/

# 验证同步结果
diff skills/git-workflow/SKILL.md ~/.claude/skills/git-workflow/SKILL.md
```

### 同步 gh-pr-create Skill

修改 `skills/gh-pr-create/` 下的文件后：

```bash
# 同步所有 gh-pr-create 文件（排除开发文档）
rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/

# 验证同步结果
diff skills/gh-pr-create/SKILL.md ~/.claude/skills/gh-pr-create/SKILL.md

# 验证 gh CLI 认证
gh auth status
```

### 快速验证修改

修改后立即验证效果：

```bash
# 1. 同步 git-workflow 文档（排除开发文档）
rsync -av --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/

# 2. 同步 gh-pr-create 文档（排除开发文档）
rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/

# 3. 在 Claude Code 中测试触发（手动测试）
# Git workflow: "开始新功能 test" 或 "git workflow help"
# PR creation: "创建 PR" 或 "create pr"

# 4. 检查文档引用是否正确
grep -r "references/" skills/git-workflow/SKILL.md
grep -r "references/" skills/gh-pr-create/SKILL.md
```

## Skills Overview

本仓库包含两个相互配合的 Claude Code skills：

### 1. git-workflow Skill

**用途**: Git topic 工作流管理

**核心功能**:
- 分支生命周期管理 (tnr/tn/tmg/td)
- 执行前状态检查和安全验证
- 工作流指导和问题诊断

**部署位置**: `~/.claude/skills/git-workflow/`

**触发词**:
- 中文: "开始新功能" / "合并分支" / "git 工作流"
- 英文: "start feature" / "merge branch" / "git workflow"

**详细文档**: 见 "Core Git Aliases" 章节

### 2. gh-pr-create Skill

**用途**: GitHub Pull Request 自动化创建

**核心功能**:
- 智能收集 commits 和变更信息
- 自动生成结构化 PR 描述（Summary + Test Plan）
- Base branch 智能识别（main/master/自定义）
- 三阶段安全检查（gh 认证 + 分支状态 + 工作区状态）

**部署位置**: `~/.claude/skills/gh-pr-create/`

**依赖**: GitHub CLI (`gh`) 必须已安装并认证

**触发词**:
- 中文: "创建 PR" / "开 PR" / "提交 PR"
- 英文: "create pr" / "open pr" / "submit pr"

**基本工作流**:
```bash
# 1. 开发功能（使用 git-workflow skill）
git tnr feature/user-auth
# ... 开发和提交 ...

# 2. 创建 PR（使用 gh-pr-create skill）
# 在 Claude Code 输入: "创建 PR"
# Skill 自动: 收集信息 → 生成描述 → 推送分支 → 创建 PR
```

**验证安装**:
```bash
# 检查 gh CLI
gh auth status

# 检查 skill 文件
ls ~/.claude/skills/gh-pr-create/

# 验证触发规则
grep -A 20 "gh-pr-create" ~/.claude/skills/skill-rules.json
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

### 修改 gh-pr-create Skill

1. 编辑 `skills/gh-pr-create/SKILL.md`
2. 重点关注：PR 描述模板、base branch 检测逻辑、安全检查清单
3. 同步到部署位置：`rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/`
4. 验证：
   - `gh auth status` 检查 gh CLI 认证
   - 创建测试分支并触发 "创建 PR"
   - 参考 `skills/gh-pr-create/docs/testing.md`

### 修改触发规则

1. 编辑 `~/.claude/skills/skill-rules.json`（注意：不在本仓库）
2. 修改对应 skill 条目的 `keywords` 或 `intentPatterns`
   - `git-workflow`: 工作流触发规则
   - `gh-pr-create`: PR 创建触发规则
3. JSON 格式验证：`python3 -m json.tool ~/.claude/skills/skill-rules.json`
4. 验证：
   - git-workflow: 运行 `skills/git-workflow/docs/testing.md` 中的测试 1.x 和 4.x
   - gh-pr-create: 运行 `skills/gh-pr-create/docs/testing.md` 中的触发测试

## Documentation Style

维护文档时遵循的风格：
- 中英混合：标题和关键术语可用英文，说明用中文
- 结构化：使用表格、清单、代码块提升可读性
- 实战导向：提供具体命令和示例，避免纯理论
- 三阶段说明：执行前检查 → 执行命令 → 执行后验证
- 安全优先：明确标注破坏性操作的前置条件

## Common Workflows

### 完整功能开发 + PR 创建工作流

结合两个 skills 的端到端流程：

**1. 创建功能分支** (git-workflow)
```
输入: "开始新功能 user-auth"
执行: git tnr feature/user-auth
```

**2. 开发和提交**
```bash
# 正常开发
vim src/auth.js
git add .
git commit -m "feat: implement user authentication"

# 临时保存进度（可选）
# 输入: "临时保存"
# 执行: git save "WIP: authentication logic"
```

**3. 验证变更** (git-workflow)
```
输入: "查看改动"
执行: git bdf (查看差异)
执行: git blg (查看提交历史)
```

**4. 修改历史** (git-workflow, 可选)
```
输入: "修改之前的 commit"
执行: git fixup (交互式选择)
```

**5. 创建 PR** (gh-pr-create)
```
输入: "创建 PR"
Skill 自动执行:
  - 收集所有 commits 信息
  - 分析文件变更
  - 生成 PR 描述（Summary + Test Plan）
  - 推送分支到 remote（如需要）
  - 创建 PR 并返回 URL
```

**6. PR 合并后清理** (git-workflow)
```
# PR 在 GitHub 上合并后
输入: "完成功能"
执行: git tmg (清理本地和远程分支)
```

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
