---
name: git-workflow
description: 智能 Git topic 工作流助手，基于生产级 aliases 提供上下文感知的分支管理、安全检查和 PR 准备。支持 tnr/tn/tmg/td 核心命令，集成 fixup/amend/conflict 高级场景，自动状态检查和恢复指导。
version: 1.0.0
license: MIT
---

# Git Workflow Skill

## Overview

智能 Git topic 工作流助手，包装生产级 Git aliases，提供三层价值：

1. **意图识别层**：自然语言 → Git 命令的智能映射
2. **安全护栏层**：执行前状态检查，预防误操作
3. **学习辅助层**：工作流指导和问题诊断

**定位**：不重写 aliases，专注智能包装和上下文感知。

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

## When to Use

- 创建/合并/删除 topic 分支（`tnr`/`tn`/`tmg`/`td`）
- 查看分支差异和日志（`bdf`/`blg`）
- 准备 PR（检查清单 + 差异预览）
- 修改历史 commit（`fixup`/`amend`）
- 解决 merge/rebase 冲突
- 恢复误操作（snapshot/reflog）

## Quick Command Mapping

| 用户意图 | 推荐命令 | 场景说明 |
|---------|---------|---------|
| 开始新功能 | `git tnr feature/X` | 需要推送到远程协作 |
| 本地实验 | `git tn experiment/X` | 不推送，仅本地开发 |
| 完成功能 | `git tmg` | `--no-ff` merge 并自动删除分支 |
| 删除分支 | `git td [branch]` | 智能同步远程+本地 |
| 查看差异 | `git bdf` | 与 base 分支的全部差异 |
| 查看日志 | `git blg` | 只看 topic 分支的 commits |
| 查看未推送 | `git ahead` | 列出未推送的 commits |
| 修改历史 | `git fixup` | 交互式选择要修改的 commit |
| 修改上次（全部文件） | `git cma` | 安全 amend（检查是否已推送） |
| 修改上次（已追踪文件） | `git cmu` | 同上，只包含已追踪文件 |
| 修改上次（可编辑消息） | `git cmae`/`cmue` | 同上，可修改 commit message |

## Safety Checklist

执行破坏性操作前，Skill 会自动检查：

**✓ 工作区状态**
- 检查：`git working-dir-dirty`
- 脏工作区时：`tnr`/`tn` 会自动 stash，`tmg` 会拒绝执行

**✓ 当前分支**
- 检查：`git current-branch`
- 在 main/base 分支上：拒绝 `tmg`/`td`

**✓ 远程同步状态**
- 检查：`git ahead-count` / `git behind-count`
- 领先：`td` 会自动推送
- 落后：`tnr` 会自动 pull base 分支

**✓ 分支存在性**
- 检查：`git remote-branch`
- 判断分支在远程是否存在，决定操作策略

**✓ Commit 是否已推送**
- 检查：`git branch -r --contains HEAD`
- 已推送的 commit：拒绝 amend，建议 fixup

## Workflow Decision Tree

### 当前状态：工作区有未提交修改

**可执行操作**：
- `git save "message"` → 保存工作进度（带时间戳和 base commit）
- `git snapshot "message"` → 创建快照但不清空工作区（保险备份）
- `git diff` → 查看未暂存修改
- `git dc` (diff --cached) → 查看已暂存修改
- `git tnr/tn feature/X` → 会自动 stash，然后创建分支

**不可执行**：
- ❌ `git tmg` → 工作区必须干净

### 当前状态：在 topic 分支上，工作区干净

**可执行操作**：
- `git bdf` → 查看与 base 分支的全部差异
- `git blg` → 查看 topic 分支独有的 commits
- `git ahead` → 查看未推送的 commits（如果远程分支存在）
- `git fixup` → 修改历史 commit（未推送的部分）
- `git tmg` → 合并到 base 分支并删除 topic 分支
- `git td` → 删除 topic 分支（会智能同步远程）

**不可执行**：
- ⚠ `git cma/cmu` → 如果 HEAD 已推送，拒绝并建议 fixup

### 当前状态：在 main/base 分支上

**可执行操作**：
- `git tnr/tn feature/X` → 创建新的 topic 分支
- `git blf` → 用 fzf 选择分支切换
- `git branch-clean-local` → 清理已合并的本地分支

**不可执行**：
- ❌ `git tmg` → 不能 merge main 自己
- ❌ `git td` → 不能删除 main/base 分支

### 当前状态：Merge/Rebase 冲突中

**可执行操作**：
- `git edit-unmerged` → 编辑所有冲突文件
- `git add-unmerged` → 标记所有冲突已解决
- `git mgc` / `git rc` → 继续 merge/rebase
- `git refix <file>` → 恢复某个文件到冲突前状态
- `git merge-log` → 查看两条分支的提交
- `git conflict-log` → 只查看冲突的提交

**不可执行**：
- ⚠ 大部分正常操作都会被阻止，必须先解决冲突

## Reference Navigation

**核心工作流**（P0）：
- `git-topic-workflow.md` - tnr/tn/tmg/td 详细说明、状态检查命令、典型工作流示例
- `git-safety-mechanisms.md` - 三阶段检查机制、安全边界、错误处理策略

**增强功能**（P1）：
- `git-pr-preparation.md` - PR 准备检查清单、代码审查步骤、质量保证
- `git-advanced-operations.md` - fixup/amend 指导、历史修改最佳实践

**问题诊断**（P2）：
- `git-troubleshooting.md` - 冲突解决、误操作恢复、stash 找回

## Key Concepts

### Topic 工作流核心原则

1. **短生命周期分支**：feature 分支应该快速合并，避免长期存在
2. **保留 merge 历史**：使用 `--no-ff` 保留分支历史，可追溯
3. **merge 后立即删除**：有 merge commit 记录，可随时恢复
4. **自动化安全检查**：执行前验证状态，减少人为错误

### Base Branch 配置

**查询当前 base 分支**：
```bash
git bb  # 或 git base-branch
```

**设置 base 分支**：
```bash
# 全局设置（所有仓库）
git set-global-base-branch main

# 当前仓库设置
git set-bb develop  # 或 git set-local-base-branch develop
```

**优先级**：本地设置 > 全局设置 > 默认值（main）

### 安全的历史修改

**原则**：永远不改写已推送的 commit

**推荐方式**：
- 未推送的 commit → `fixup` 或 `amend`
- 已推送的 commit → 新建 commit 修复

**检查边界**：
```bash
git oldest-changeable-commit  # 返回最远可修改的 commit
```

### Stash vs Save vs Snapshot

| 命令 | 用途 | 保留工作区 | 包含信息 |
|-----|------|-----------|---------|
| `git stash` | 临时保存 | ❌ 清空 | 基本信息 |
| `git save "msg"` | 带标签保存 | ❌ 清空 | 时间戳 + base commit + 自定义消息 |
| `git snapshot "msg"` | 保险备份 | ✅ 保留 | 同 save，但 stash 后立即 apply |

**推荐**：用 `save` 替代原生 `stash`，信息更丰富。

## Best Practices

1. **创建分支前先检查状态**
   ```bash
   git s  # git status --short --branch
   git bb # 确认 base 分支
   ```

2. **定期同步 base 分支**
   ```bash
   git zz  # 切换到 base 分支
   git pl  # git pull
   ```

3. **Merge 前预览**
   ```bash
   git blg  # 查看将要合并的 commits
   git bdf  # 查看全部差异
   ```

4. **重要操作前快照**
   ```bash
   git snapshot "before-merge-feature-X"
   git tmg  # 即使出错，也能恢复
   ```

5. **定期清理已合并分支**
   ```bash
   git branch-clean-local   # 清理本地
   git branch-clean-remote  # 清理远程（需要权限）
   ```

6. **使用 fzf 交互选择**
   ```bash
   git blf    # 选择分支切换
   git fixup  # 选择要修改的 commit
   git pif    # 选择多个 commit 进行 cherry-pick
   ```

## Common Workflows

### 典型开发流程

```bash
# 1. 创建新功能分支
git tnr feature/user-auth
# → 自动检查工作区 → 自动 stash（如需要） → 创建并推送分支

# 2. 开发中临时保存
git save "WIP: implementing login logic"
# → 保存工作进度，可随时恢复

# 3. 查看进度
git blg           # 看 commits
git bdf           # 看差异

# 4. 修改之前的 commit
git fixup
# → fzf 选择 commit → 自动 fixup + rebase

# 5. 合并回主分支
git tmg
# → 安全检查 → merge --no-ff → 自动删除分支
```

### PR 准备流程

```bash
# 1. 确保与 base 同步
git zz && git pl  # 切换到 base 并 pull
git z -           # 切回之前的分支

# 2. 查看变更
git blg           # Commits 列表
git bdf | less    # 完整差异

# 3. 自检
- 测试通过？
- Lint 通过？
- Commit 消息清晰？
- 文档更新？

# 4. 推送（如果有未推送 commits）
git ps            # git push

# 5. 创建 PR
gh pr create
```

### 冲突解决流程

```bash
# 1. 查看冲突文件
git edit-unmerged
# → 编辑器打开所有冲突文件

# 2. 解决冲突后标记
git add-unmerged
# → 将所有冲突文件添加到暂存区

# 3. 继续操作
git mgc   # 如果是 merge
git rc    # 如果是 rebase
```

### 误操作恢复流程

```bash
# 误删本地分支
git ref           # git reflog --no-abbrev
# → 找到分支的最后一个 commit
git zb recovered-branch <commit-hash>

# 找回丢失的 stash
git sh            # git stash-history
# → 显示最近 15 个 stash（包括已 drop 的）
git stash apply <hash>
# 或
git branch recovered <hash>

# 错误 merge
git reset --hard ORIG_HEAD

# 错误 rebase
git ref
git reset --hard <commit-before-rebase>
```

## Integration with Codeagent

根据用户全局配置，Skill 负责**规划和验证**，Codeagent 负责**执行**：

```
Git-Workflow Skill:
├─ 分析当前仓库状态
├─ 设计操作步骤
├─ 生成 git 命令序列
└─ 验证执行结果

Codeagent:
└─ 执行 git 命令（如需要）
```

**示例**：
```
用户："完成这个功能"
Skill 分析：
  1. 当前分支：feature/user-auth
  2. 工作区状态：干净
  3. 与 base 无冲突
  4. 有 3 个 commits 待合并
Skill 生成：git tmg
Codeagent：执行命令
Skill 验证：确认已在 base 分支，topic 分支已删除
```

## Dependencies

**硬依赖**（必需）：
- Git aliases 配置：`/home/alecshan/projects/private/dotfiles/git/.config/git/conf/aliases.gitconfig`
- 关键命令：`tnr`, `tn`, `tmg`, `td`, `fixup`, `bdf`, `blg`, `current-branch`, `base-branch`, `working-dir-dirty`, `remote-branch`, `ahead-count`, `behind-count`, `safe-commit-amend`

**软依赖**（增强）：
- fzf：交互式选择（fixup、blf、pif）
- ripgrep：仓库搜索（rg、rg-all）

## Resources

- Git Aliases:
  Local: `git/aliases.gitconfig`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- Git Aliases Reference Manual:
  Local: `git/Git-Aliases-Reference-Manual.md`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md
