# Git-Workflow Skills for Claude Code

两个智能 Git 工作流助手，简化你的开发流程：

- **git-workflow**：基于 topic 的分支管理，提供安全检查和工作流指导
- **gh-pr-create**：自动化 GitHub Pull Request 创建，智能变更分析

## 特性

### 🎯 核心能力

- **意图识别**：自然语言 → Git 命令的智能映射
- **安全护栏**：三阶段检查（执行前/中/后）预防误操作
- **上下文感知**：根据当前分支状态推荐合适的操作
- **学习辅助**：提供工作流指导和问题诊断

### 📦 Skills 概览

#### [git-workflow](skills/git-workflow/docs/README_zh-CN.md)

基于 topic 的分支管理，提供安全检查和工作流指导。

**核心功能**：tnr/tn/tmg/td 生命周期 | 三阶段安全检查 | PR 准备 | 高级操作（fixup/amend）| 冲突解决

**[→ 完整文档](skills/git-workflow/docs/README_zh-CN.md)**

#### [gh-pr-create](skills/gh-pr-create/docs/README_zh-CN.md)

自动化 GitHub Pull Request 创建，智能变更分析。

**核心功能**：自动生成 PR 描述 | 智能 base 分支检测 | gh CLI 集成 | 结构化模板

**[→ 完整文档](skills/gh-pr-create/docs/README_zh-CN.md)**

---

## 快速开始

### 前置要求

1. **Git Aliases 配置**（git-workflow 必需）：
   ```bash
   # 位置: ./git/aliases.gitconfig
   # 包含 tnr, tn, tmg, td, fixup, bdf, blg 等核心命令
   ```

2. **GitHub CLI**（gh-pr-create 必需）：
   ```bash
   # 安装 gh CLI
   # macOS: brew install gh
   # Linux: 见 https://github.com/cli/cli#installation

   # 认证
   gh auth login
   ```

3. **Claude Code**（必需）：
   - 版本：支持 Skills 功能的版本
   - 配置：`~/.claude/skills/` 目录已存在

4. **可选依赖**：
   - `fzf`：交互式选择（fixup、blf、pif）
   - `ripgrep`：仓库搜索（rg、rg-all）

---

### 安装

Skills 部署位置：
```
~/.claude/skills/git-workflow/
├── SKILL.md
└── references/
    ├── git-topic-workflow.md
    ├── git-safety-mechanisms.md
    ├── git-pr-preparation.md
    ├── git-advanced-operations.md
    └── git-troubleshooting.md

~/.claude/skills/gh-pr-create/
├── SKILL.md
└── references/
    ├── pr-templates.md
    ├── gh-integration.md
    └── base-branch-detection.md
```

触发规则添加到：
```
~/.claude/skills/skill-rules.json
```

**验证安装**：
```bash
# 检查 git-workflow 文件
ls ~/.claude/skills/git-workflow/

# 检查 gh-pr-create 文件
ls ~/.claude/skills/gh-pr-create/

# 验证触发规则
grep -A 20 "git-workflow" ~/.claude/skills/skill-rules.json
grep -A 20 "gh-pr-create" ~/.claude/skills/skill-rules.json

# 验证 gh CLI 认证（gh-pr-create 使用）
gh auth status
```

---

## 使用方法

### 快速示例

**git-workflow**：分支管理的自然语言命令
```
"开始新功能 user-auth" → 创建并推送 feature 分支
"完成功能" → 合并并清理分支
"查看 branch diff" → 显示与 base 分支的差异
```

**gh-pr-create**：自动化 PR 创建
```
"创建 PR" → 分析 commits，生成描述，创建 PR
"create pull request" → 同上，必要时自动推送
```

**[→ git-workflow 完整使用指南](skills/git-workflow/docs/README_zh-CN.md)**
**[→ gh-pr-create 完整使用指南](skills/gh-pr-create/docs/README_zh-CN.md)**

---

## 典型工作流

### 完整功能开发（端到端）

结合两个 skills 完成完整开发周期：

```bash
# 1. 创建分支（git-workflow）
你: "开始新功能 user-auth"
→ git tnr feature/user-auth

# 2. 开发中保存进度（git-workflow）
你: "临时保存"
→ git save "WIP: implementing login"

# 3. 查看进度（git-workflow）
你: "查看我改了什么"
→ git bdf  # 差异
→ git blg  # 日志

# 4. 修改历史（git-workflow）
你: "修改之前的 commit"
→ git fixup  # fzf 选择

# 5. 创建 PR（gh-pr-create）
你: "创建 PR"
→ 分析所有 commits 和文件变更
→ 生成结构化 PR 描述（Summary + Test Plan）
→ 必要时推送分支
→ 创建 PR: https://github.com/user/repo/pull/123

# 6. PR 在 GitHub 合并后，清理（git-workflow）
你: "完成功能"
→ git tmg  # merge 并删除分支
```

**更多场景**：
- **[git-workflow 场景](skills/git-workflow/docs/README_zh-CN.md#典型工作流)**：冲突解决、误操作恢复
- **[gh-pr-create 场景](skills/gh-pr-create/docs/README_zh-CN.md#支持的场景)**：PR 模板、base 分支检测

---

## 架构

### 设计原则

**关注点分离**：
- **Skills 层**：意图识别、安全检查、指导
- **Aliases 层**：Git 操作、运行时安全、错误处理

**YAGNI 方法**：
- 不重写 aliases 逻辑
- 专注智能包装和上下文感知

**[→ 完整架构文档](skills/git-workflow/docs/README_zh-CN.md#架构设计)**

---

## 触发规则

触发规则定义在 `~/.claude/skills/skill-rules.json`。

### git-workflow

**示例关键词**："开始新功能"、"完成功能"、"git workflow"、"branch diff"、"fixup"

**[→ 完整触发规则](skills/git-workflow/docs/README_zh-CN.md#触发规则)**

### gh-pr-create

**示例关键词**："创建 PR"、"create pr"、"open pull request"

**[→ 完整触发规则](skills/gh-pr-create/docs/README_zh-CN.md#触发规则)**

---

## 项目统计

| 指标 | 数值 |
|-----|------|
| Skills 总数 | 2 个 |
| 文档总数 | 10 个 |
| 总行数 | 6029 行 |

**详细数据**：
- **[git-workflow](skills/git-workflow/docs/README_zh-CN.md#统计数据)**：6 个文档，3620 行
- **[gh-pr-create](skills/gh-pr-create/docs/README_zh-CN.md#统计数据)**：4 个文档，2409 行

---

## 开发历程

### Phase 1: MVP（已完成）

**目标**：验证核心价值 - 意图识别 + 安全检查

**交付物**：
- ✅ SKILL.md
- ✅ git-topic-workflow.md
- ✅ git-safety-mechanisms.md
- ✅ skill-rules.json 触发规则

**验证标准**：
- ✅ 能识别 5 种用户意图
- ✅ 状态检查覆盖 4 个维度
- ✅ 生成正确且安全的命令

---

### Phase 2: 增强（已完成）

**目标**：添加 PR 准备、历史修改、恢复指导

**交付物**：
- ✅ git-pr-preparation.md
- ✅ git-advanced-operations.md
- ✅ git-troubleshooting.md

**验证标准**：
- ✅ PR 准备检查清单完整
- ✅ Fixup/amend 引导清晰
- ✅ 冲突解决工作流可操作
- ✅ 恢复方案覆盖常见误操作

---

### Phase 3: 优化（可选）

**目标**：根据实际使用反馈改进

**计划**：
- 调整触发规则（基于效果）
- 添加更多常见场景
- 完善错误提示
- 工作流可视化（如有价值）

---

## 贡献指南

### 文件结构

```
git-workflow-skill/
├── README.md                    # 英文版本
├── README_zh-CN.md             # 本文件（中文版）
├── skills/
│   ├── git-workflow/           # Git 工作流 skill
│   │   ├── SKILL.md
│   │   ├── references/
│   │   ├── docs/
│   │   │   ├── README.md       # 开发文档（英文）
│   │   │   ├── README_zh-CN.md # 开发文档（中文）
│   │   │   └── testing.md
│   │   └── examples/scenarios.md
│   └── gh-pr-create/           # GitHub PR 创建 skill
│       ├── SKILL.md
│       ├── references/
│       ├── docs/
│       │   ├── README.md       # 开发文档（英文）
│       │   ├── README_zh-CN.md # 开发文档（中文）
│       │   └── testing.md
│       └── examples/scenarios.md
└── git/                         # Git aliases 配置
    ├── aliases.gitconfig
    └── Git-Aliases-Reference-Manual.md
```

### 修改 Skills

**git-workflow Skill**：

1. **修改主文档**：
   ```bash
   vim skills/git-workflow/SKILL.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/
   ```

2. **修改参考文档**：
   ```bash
   vim skills/git-workflow/references/<document>.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/
   ```

**gh-pr-create Skill**：

1. **修改主文档**：
   ```bash
   vim skills/gh-pr-create/SKILL.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/
   ```

2. **修改参考文档**：
   ```bash
   vim skills/gh-pr-create/references/<document>.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/
   ```

3. **验证 gh CLI**：
   ```bash
   gh auth status
   ```

**触发规则**：

1. **修改触发规则**：
   ```bash
   vim ~/.claude/skills/skill-rules.json
   # 修改 git-workflow 或 gh-pr-create 条目的 keywords 或 intentPatterns
   ```

2. **验证修改**：
   ```bash
   # JSON 格式检查
   python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

   # 在 Claude Code 中测试触发
   ```

---

## 故障排查

### 常见问题

**Skills 未触发**：
```bash
# 验证触发规则
grep -E "git-workflow|gh-pr-create" ~/.claude/skills/skill-rules.json

# 验证 skill 文件存在
ls ~/.claude/skills/git-workflow/
ls ~/.claude/skills/gh-pr-create/
```

**Git Aliases 未找到**：
```bash
# 检查 aliases 是否加载
git config --get-regexp alias.tnr

# 验证 aliases 文件
ls ./git/aliases.gitconfig
```

**gh CLI 问题**：
```bash
# 认证 GitHub
gh auth login

# 验证认证
gh auth status
```

**详细故障排查**：
- **[git-workflow 故障排查](skills/git-workflow/docs/README_zh-CN.md#故障排查)**：触发问题、命令错误、状态检查
- **[gh-pr-create 故障排查](skills/gh-pr-create/docs/README_zh-CN.md#故障排查)**：认证、base 分支检测、PR 创建

---

## 许可证

MIT License

---

## 参考资料

### 相关文档

- Git Aliases：
  Local: `git/aliases.gitconfig`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- Git Aliases Reference Manual:
  Local: `git/Git-Aliases-Reference-Manual.md`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md

### 外部资源

- Feature Branch Workflow：https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- Git Flight Rules：https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!：https://ohshitgit.com/

---

## 联系方式

如有问题或建议，请：
1. 查阅本文档的"故障排查"章节
2. 参考 Skill 文档：`~/.claude/skills/git-workflow/SKILL.md`
3. 查看 Git Aliases Reference Manual：`git/Git-Aliases-Reference-Manual.md`

---

**祝开发愉快！** 🚀
