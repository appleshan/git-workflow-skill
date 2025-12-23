# Git-Workflow Skill for Claude Code

智能 Git topic 工作流助手，基于生产级 aliases，提供上下文感知的分支管理、安全检查和 PR 准备指导。

## 概述

Git-Workflow Skill 包装生产级 Git aliases，提供三层价值：

1. **意图识别层**：自然语言 → Git 命令的智能映射
2. **安全护栏层**：执行前状态检查，预防误操作
3. **学习辅助层**：工作流指导和问题诊断

**定位**：不重写 aliases 逻辑，专注智能包装和上下文感知。

**与 Aliases 的关系**：
```
用户自然语言
    ↓ (意图识别)
Git-Workflow Skill
    ↓ (状态检查 + 命令生成)
Production Git Aliases
    ↓ (执行 + 内置安全机制)
Git 操作
```

---

## 特性

### 📦 功能模块

| 模块 | 功能 | 文档 |
|-----|------|------|
| **核心工作流** | tnr/tn/tmg/td 分支生命周期管理 | [git-topic-workflow.md](references/git-topic-workflow.md) |
| **安全机制** | 三阶段检查、安全边界、错误处理 | [git-safety-mechanisms.md](references/git-safety-mechanisms.md) |
| **PR 准备** | 检查清单、质量保证、PR 描述模板 | [git-pr-preparation.md](references/git-pr-preparation.md) |
| **高级操作** | fixup/amend/rebase/cherry-pick | [git-advanced-operations.md](references/git-advanced-operations.md) |
| **故障排查** | 冲突解决、误操作恢复、紧急救援 | [git-troubleshooting.md](references/git-troubleshooting.md) |

---

## 前置要求

### 必需：Git Aliases 配置

```bash
# 位置: ../../git/aliases.gitconfig
# 包含核心命令: tnr, tn, tmg, td, fixup, bdf, blg 等

# 验证 aliases 已加载
git config --get-regexp alias.tnr
git config --get-regexp alias.tmg
```

**参考**：
- 本地：`../../git/aliases.gitconfig`
- GitHub：https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- 手册：`../../git/Git-Aliases-Reference-Manual.md`

### 可选依赖

- **fzf**：交互式选择（fixup、blf、pif）
- **ripgrep**：仓库搜索（rg、rg-all）

---

## 安装

### 部署位置

```
~/.claude/skills/git-workflow/
├── SKILL.md
└── references/
    ├── git-topic-workflow.md
    ├── git-safety-mechanisms.md
    ├── git-pr-preparation.md
    ├── git-advanced-operations.md
    └── git-troubleshooting.md
```

### 从仓库部署

```bash
# 同步所有文件（排除开发文档）
rsync -av --exclude 'docs/' --exclude 'examples/' \
  skills/git-workflow/ ~/.claude/skills/git-workflow/

# 验证部署
ls ~/.claude/skills/git-workflow/
```

### 验证安装

```bash
# 检查 skill 文件
ls ~/.claude/skills/git-workflow/SKILL.md
ls ~/.claude/skills/git-workflow/references/

# 验证触发规则
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# 测试 Git aliases
git tnr --help  # 应显示用法
```

---

## 使用方法

### 基础命令映射

| 用户意图 | 推荐命令 | 说明 |
|---------|---------|-----|
| "开始新功能 X" | `git tnr feature/X` | 创建并推送到远程 |
| "本地实验" | `git tn experiment/X` | 仅本地分支 |
| "完成功能" | `git tmg` | Merge 并删除分支 |
| "删除分支" | `git td [branch]` | 智能同步删除 |
| "查看差异" | `git bdf` | 与 base 分支对比 |
| "查看日志" | `git blg` | Topic 分支 commits |
| "准备 PR" | 检查清单 + 流程 | PR 准备指导 |
| "修改历史" | `git fixup` | 交互式修改 commit |
| "解决冲突" | 冲突解决工作流 | edit-unmerged 流程 |

### 触发示例

#### 中文触发

```
你："开始新功能 user-authentication"
Skill 响应：
  ✓ 当前分支: main
  ✓ 工作区: 干净
  推荐命令: git tnr feature/user-authentication
  说明: 将创建并推送分支到远程
```

#### 英文触发

```
你："finish this feature"
Skill 响应：
  ✓ 当前分支: feature/user-auth
  ✓ 工作区: 干净
  ✓ 3 commits 待合并
  推荐命令: git tmg
  说明: 将 merge --no-ff 并自动删除分支
```

#### 混合触发

```
你："查看 branch diff"
Skill 响应：
  ✓ 当前分支: feature/user-auth
  推荐命令: git bdf
  说明: 显示与 main 的全部差异
```

---

## 典型工作流

### 场景 1：冲突解决

```bash
# Merge 冲突
你: "merge 冲突怎么办"
Skill 提供：
  1. git edit-unmerged  # 编辑冲突
  2. 手动解决冲突
  3. git add-unmerged   # 标记解决
  4. git mgc            # 继续 merge
```

### 场景 2：误操作恢复

```bash
# 误删分支
你: "误删分支怎么办"
Skill 提供：
  1. git reflog | grep "branch-name"
  2. 找到分支最后的 commit
  3. git checkout -b recovered <hash>
```

### 场景 3：功能开发（仅 Git-Workflow）

```bash
# 1. 创建分支
你: "开始新功能 user-auth"
→ git tnr feature/user-auth

# 2. 开发中保存进度
你: "临时保存"
→ git save "WIP: implementing login"

# 3. 查看进度
你: "查看我改了什么"
→ git bdf  # 差异
→ git blg  # 日志

# 4. 修改历史
你: "修改之前的 commit"
→ git fixup  # fzf 选择

# 5. 最终合并（PR 在 GitHub 合并后）
你: "完成功能"
→ git tmg  # merge 并删除分支
```

**注意**：PR 创建请参考 [gh-pr-create skill](../../gh-pr-create/docs/README_zh-CN.md)。

---

## 架构设计

### 职责边界

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
- ✅ 意图理解和命令映射
- ✅ 执行前状态检查
- ✅ 工作流指导
- ✅ 问题诊断

**Aliases 负责**：
- ✅ 实际 Git 操作
- ✅ 运行时安全检查（自动 stash、同步、保护）
- ✅ 错误处理

**不做的事**（YAGNI）：
- ❌ 不重写 aliases 逻辑
- ❌ 不修改 dotfiles 配置
- ❌ 不添加新的 shell 脚本

---

### 三阶段安全检查

```
执行前 (Pre-check)
├─ 工作区状态: git working-dir-dirty
├─ 当前分支: git current-branch
├─ 远程同步: git ahead-count / behind-count
└─ 分支存在: git remote-branch

执行中 (Runtime)
└─ Aliases 内置安全机制

执行后 (Post-check)
├─ 结果验证: git status / current-branch
└─ 预期确认: 分支切换/删除/merge commit
```

**状态检查命令**（来自 aliases）：
- `git working-dir-dirty` → 检查工作区是否有未提交修改
- `git current-branch` → 获取当前分支名
- `git base-branch` → 获取 base 分支（通常是 main/master）
- `git ahead-count` → 领先远程的 commits 数
- `git behind-count` → 落后远程的 commits 数
- `git remote-branch` → 检查分支是否存在于远程

---

## 触发规则

### 关键词

**中文**：
- topic分支、功能分支、合并分支、删除分支
- 修改提交、分支差异、准备PR、git工作流
- tnr、tmg

**英文**：
- topic branch、feature branch、git workflow
- merge branch、delete branch、fixup
- branch diff、branch log

### 意图模式

```regex
(start|create|new|开始|创建).*(feature|topic|branch|功能|分支)
(merge|finish|complete|合并|完成).*(branch|feature|topic|分支|功能)
(delete|remove|clean|删除|清理).*(branch|topic|分支)
(show|view|diff|log|查看|显示).*(branch|changes|差异|修改)
(prepare|ready|check|准备|检查).*(pr|pull request)
\bfixup\b|修改.*提交|amend
(conflict|resolve|冲突|解决)
git.*(workflow|工作流)
```

**配置**：
触发规则定义在 `~/.claude/skills/skill-rules.json`

---

## 统计数据

| 指标 | 数值 |
|-----|------|
| 文档数量 | 6 个 |
| 总代码行数 | 3620 行 |
| 主文档 | 366 行 |
| 核心工作流 | 724 行 |
| 安全机制 | 227 行 |
| PR 准备 | 740 行 |
| 高级操作 | 766 行 |
| 故障排查 | 797 行 |

---

## 开发指南

### 修改 Skill

**1. 修改主文档**：
```bash
vim SKILL.md
rsync -av --exclude 'docs/' --exclude 'examples/' . ~/.claude/skills/git-workflow/
```

**2. 修改参考文档**：
```bash
vim references/<document>.md
rsync -av --exclude 'docs/' --exclude 'examples/' . ~/.claude/skills/git-workflow/
```

**3. 修改触发规则**：
```bash
vim ~/.claude/skills/skill-rules.json
# 修改 "git-workflow" 条目: keywords 或 intentPatterns

# 验证 JSON 格式
python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

# 在 Claude Code 中测试触发
```

### 测试

参考 [docs/testing.md](docs/testing.md) 获取完整测试用例：

**核心测试类别**：
1. **触发测试**：中文/英文/混合关键词准确性
2. **功能测试**：分支创建、合并、PR 准备、冲突解决
3. **状态感知测试**：工作区和分支状态识别
4. **边界测试**：不相关输入、模糊输入、部分匹配

---

## 故障排查

### Skill 未触发

**可能原因**：
1. 关键词不匹配
2. 意图模式不匹配
3. skill-rules.json 格式错误

**排查步骤**：
```bash
# 1. 检查 JSON 格式
python3 -m json.tool ~/.claude/skills/skill-rules.json

# 2. 查看触发规则
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# 3. 尝试精确关键词
# 输入: "tnr" 或 "git workflow"
```

---

### 命令不存在

**可能原因**：
Git aliases 未配置或路径不正确

**排查步骤**：
```bash
# 1. 检查 aliases 是否加载
git config --get-regexp alias.tnr
git config --get-regexp alias.tmg

# 2. 检查 aliases 文件路径
ls ../../git/aliases.gitconfig

# 3. 确认 Git 配置引用
git config --get include.path
```

---

### 状态检查失败

**可能原因**：
辅助命令（working-dir-dirty、current-branch 等）不存在

**排查步骤**：
```bash
# 测试辅助命令
git working-dir-dirty
git current-branch
git base-branch

# 如果失败，检查 aliases 配置
git config --get-regexp alias | grep "working-dir-dirty"
```

---

## 参考资料

### 相关文档

- **Git Aliases**：
  - 本地：`../../git/aliases.gitconfig`
  - GitHub：https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- **Git Aliases Reference Manual**：
  - 本地：`../../git/Git-Aliases-Reference-Manual.md`
  - GitHub：https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md
- **测试指南**：[docs/testing.md](docs/testing.md)
- **示例场景**：[examples/scenarios.md](examples/scenarios.md)

### 外部资源

- Feature Branch Workflow：https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- Git Flight Rules：https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!：https://ohshitgit.com/

---

## 相关 Skills

- **gh-pr-create**：GitHub Pull Request 创建 skill - [README](../../gh-pr-create/docs/README_zh-CN.md)
- 与 git-workflow 无缝配合，完成完整功能开发周期

---

## 许可证

MIT License

---

## 联系方式

如有问题或建议：
1. 查阅本文档的"故障排查"章节
2. 参考 Skill 文档：`~/.claude/skills/git-workflow/SKILL.md`
3. 查看 Git Aliases Reference Manual：`../../git/Git-Aliases-Reference-Manual.md`

---

**祝开发愉快！** 🚀
