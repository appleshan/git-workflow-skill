# Git Safety Mechanisms 安全检查机制

本文档详细说明 Git topic 工作流的安全机制，包括三阶段检查、安全边界、错误处理策略和恢复方案。

---

## 三阶段安全检查

所有破坏性操作（创建、合并、删除分支）都遵循三阶段检查：

```
执行前 (Pre-check)
    ↓
执行中 (Runtime - Aliases 内置)
    ↓
执行后 (Post-check)
```

---

## 1. 执行前检查 (Pre-check)

### 检查维度

#### 1.1 工作区状态

**检查命令**：
```bash
git working-dir-dirty
```

**应对策略**：

| 操作 | 工作区脏 | 自动处理 |
|-----|---------|---------|
| `tnr`/`tn` | 允许 | 自动 `git save` |
| `tmg` | 拒绝 | 提示用户先提交或 save |
| `td` | 拒绝 | 提示用户先提交或 save |
| `fixup` | 允许 | 会包含在 fixup commit 中 |
| `amend` | 允许 | 会包含在 amend 中 |

#### 1.2 当前分支

**检查命令**：`git current-branch`

**安全规则**：

| 当前分支 | `tmg` | `td` | `tnr`/`tn` |
|---------|-------|------|-----------|
| main | ❌ 拒绝 | ❌ 拒绝 | ✅ 允许（创建新分支） |
| base-branch | ❌ 拒绝 | ❌ 拒绝 | ✅ 允许 |
| topic 分支 | ✅ 允许 | ✅ 允许 | ✅ 允许 |

#### 1.3 远程同步状态

**检查命令**：`git ahead-count` / `git behind-count`

**应对策略**：

| 场景 | `tnr` | `td` | `tmg` |
|-----|-------|------|-------|
| 本地领先 | - | 自动 push | 提醒未推送 |
| 本地落后 | 自动 pull base | 自动 pull | 建议先同步 |
| 同步 | - | 直接操作 | 直接操作 |

#### 1.4 Commit 是否已推送

**检查命令**：`git branch -r --contains HEAD`

**安全规则**（safe-commit-amend）：

| HEAD 状态 | `amend` | `fixup` |
|----------|---------|---------|
| 已推送 | ❌ 拒绝 | ⚠ 限制范围 |
| 未推送 | ✅ 允许 | ✅ 允许 |

---

## 2. 执行中检查 (Runtime)

由 Git aliases 内置的安全机制处理，Skill 不重复实现。

### Aliases 内置安全机制

#### 2.1 自动 Stash
触发条件：工作区脏 + 执行 `tnr`/`tn`

#### 2.2 自动同步
- `tnr` 时 base 落后 → 自动 pull
- `td` 时同步 topic 分支 → 自动 push/pull

#### 2.3 分支保护
不能删除 main/base 分支

---

## 3. 执行后检查 (Post-check)

### 验证项

#### 3.1 分支切换验证
- `tnr`/`tn` 后：应在新创建的 topic 分支上
- `tmg` 后：应在 base 分支上
- `td` 后：应在 base 分支上

#### 3.2 分支存在性验证
- `td` 后：本地分支应已删除
- `tmg` 后：topic 分支应已删除

#### 3.3 Merge Commit 验证
- `tmg` 后：最新 commit 应为 merge commit

---

## 4. 错误处理分层

### L1: 预测性阻止（执行前）
**原则**：能预测的问题，直接拒绝 + 清晰提示

示例：
- 在 main 上执行 tmg → 拒绝
- 工作区脏时执行 tmg → 拒绝
- amend 已推送的 commit → 拒绝

### L2: 警告性提醒（执行前）
**原则**：有风险但可接受，提醒用户 + 继续执行

示例：
- base 分支落后 → 提醒 + 自动 pull
- 有未推送 commits → 提醒
- 删除未合并的分支 → 警告 + 要求确认

### L3: 恢复性指导（执行后出错）
**原则**：出错后提供明确的恢复路径

示例：
- Merge 冲突 → 提供 `edit-unmerged` → `add-unmerged` → `mgc` 工作流
- Rebase 失败 → 提供 `git ra` (abort) 或继续步骤
- 误删分支 → 提供 reflog 恢复方法

---

## 5. 关键操作前建议快照

### Snapshot 机制
```bash
git snapshot "message"
# 创建 stash 但不清空工作区
```

### 建议快照的场景
1. 不可逆的 merge：`git snapshot "before-merge-feature-X"`
2. 修改历史：`git snapshot "before-fixup"`
3. 批量删除分支：`git snapshot "before-branch-clean"`

---

## 6. 安全边界定义

### 6.1 可修改历史的边界
计算命令：`git oldest-changeable-commit`

决策树：
- 当前分支有远程追踪 → 边界 = origin/current_branch
- 否则 base 有远程追踪 → 边界 = origin/base_branch
- 否则 → 边界 = merge-base(base, current)

### 6.2 可删除分支的边界
保护规则：永远不能删除 main/master/develop/base-branch

### 6.3 可 amend 的边界
规则：HEAD 未推送到远程

---

## 7. 常见安全陷阱

1. **Force Push（不推荐）**：破坏协作者历史
2. **在脏工作区执行破坏性操作**：可能丢失修改
3. **删除未合并的分支**：丢失工作内容
4. **Rebase 公共分支**：重写共享历史

---

## 8. 安全最佳实践

### 8.1 操作前三问
1. 工作区干净吗？`git s`
2. 当前分支是哪个？`git current-branch`
3. 远程同步了吗？`git ahead-count` / `git behind-count`

### 8.2 养成快照习惯
```bash
# 重要操作前
git snapshot "before-<operation>"

# 每天下班前
git snapshot "end-of-day-$(date +%Y%m%d)"
```

### 8.3 定期备份 reflog
```bash
git reflog > ~/git-reflog-backup-$(date +%Y%m%d).txt
```

---

## 9. 故障恢复速查表

| 问题 | 恢复命令 | 说明 |
|-----|---------|-----|
| 误删本地分支 | `git reflog` → `git checkout -b recovered <hash>` | 从 reflog 找回 |
| 误删远程分支 | `git push origin branch-name` | 如果本地还在 |
| 错误 merge | `git reset --hard ORIG_HEAD` | 撤销最近的 merge |
| 错误 rebase | `git reflog` → `git reset --hard <hash>` | 找到 rebase 前的状态 |
| 丢失 stash | `git stash-history` → `git stash apply <hash>` | 从 fsck 找回 |
| Commit 太大 | `git reset --soft HEAD~1` → 重新拆分提交 | 拆分 commit |
| Commit 消息错误 | `git commit --amend` | 修改最近 commit 消息 |

---

## 参考资料

- Git Aliases 配置：[aliases.gitconfig](https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig)
- Git Aliases 参考手册：[Git-Aliases-Reference-Manual.md](https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md)
- Git Flight Rules：https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!：https://ohshitgit.com/
