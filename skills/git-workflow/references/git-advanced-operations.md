# Git 高级操作指南

本文档详细说明 Git 历史修改的高级技巧，包括 fixup、amend、rebase 和 cherry-pick 的安全使用方法。

---

## 核心原则

### 历史修改的黄金法则

```
永远不改写已推送的 commit
```

**原因**：
- 破坏协作者的历史
- 可能导致别人的提交丢失
- 引发 merge 冲突

**例外**：
- 个人分支（无其他人协作）
- 团队明确同意（如 force-with-lease）
- 紧急修复敏感信息泄露

---

## 1. Fixup - 优雅地修改历史 Commit

### 1.1 什么是 Fixup

**Fixup** 是 Git 提供的一种安全修改历史 commit 的方式：
1. 创建一个临时的"fixup commit"
2. 自动 rebase 并合并到目标 commit
3. 保持历史清晰（不留"fix typo"垃圾 commit）

### 1.2 使用场景

**场景 1：发现之前 commit 的 bug**
```bash
# 3 commits 前添加了登录功能，现在发现一个 bug
git log --oneline
# abc123 feat: add signup
# def456 feat: add login     ← 这个有 bug
# ghi789 feat: add homepage

# 修复 bug
vim src/auth.ts
git add src/auth.ts
git fixup
# → fzf 选择 def456
# → 自动创建 fixup commit 并 rebase
```

**场景 2：忘记添加测试**
```bash
git log --oneline
# abc123 feat: add user API   ← 忘记加测试了

# 添加测试
vim tests/test_user.py
git add tests/test_user.py
git fixup
# → 选择 abc123
# → 测试会合并到 abc123
```

**场景 3：代码审查反馈**
```bash
# PR 审查建议修改变量名
vim src/utils.ts  # 修改 foo → bar
git add src/utils.ts
git fixup
# → 选择引入 foo 的 commit
# → 修改会合并进去，就像一开始就用了 bar
```

---

### 1.3 Fixup 执行流程

```bash
git fixup
```

**内部步骤**：
1. 计算安全边界：`git oldest-changeable-commit`
2. 展示可修改的 commit 列表（fzf）
3. 用户选择目标 commit
4. 创建 fixup commit：`git commit --fixup=<target>`
5. 自动 rebase：`git rebase -i --autosquash <target>~`

**等价手动操作**：
```bash
# 1. 创建 fixup commit
git add <files>
git commit --fixup=abc123  # 目标 commit 的 hash

# 2. 交互式 rebase（fixup commit 会自动排序到目标下方）
git rebase -i --autosquash abc123~
# 编辑器中会显示：
# pick abc123 feat: add login
# fixup def456 fixup! feat: add login  ← 自动标记为 fixup
```

---

### 1.4 安全边界

**Fixup 只能修改未推送的 commit**。

**检查边界**：
```bash
git oldest-changeable-commit
```

**边界决策树**：
```
当前分支有远程追踪？
├─ 是 → 边界 = origin/current_branch
│       只能修改本地新增的 commits
└─ 否 → base 分支有远程追踪？
    ├─ 是 → 边界 = origin/base_branch
    │       只能修改从 base 分叉后的 commits
    └─ 否 → 边界 = merge-base(base, current)
            可以修改到分叉点
```

**示例**：
```bash
# 场景 1：topic 分支已推送
git log --oneline
# abc123 (origin/feature/X, feature/X) feat: login  ← 已推送
# def456 (local) feat: signup                       ← 未推送

git oldest-changeable-commit
# → origin/feature/X

git fixup
# → 只能选择 def456，不能选择 abc123
```

---

### 1.5 Fixup 最佳实践

#### 1. 频繁 fixup，保持历史清晰

**坏习惯**：
```bash
git commit -m "feat: add login"
# ... 发现 typo ...
git commit -m "fix typo"
# ... 又发现bug ...
git commit -m "fix bug"
# ... 忘记测试 ...
git commit -m "add tests"
```

**结果**：历史混乱，4 个 commits 其实只做了一件事。

**好习惯**：
```bash
git commit -m "feat: add login"
# ... 发现 typo ...
git add <files>
git fixup  # 选择"feat: add login"
# ... 又发现 bug ...
git fixup  # 选择"feat: add login"
# ... 添加测试 ...
git fixup  # 选择"feat: add login"
```

**结果**：历史清晰，只有 1 个 commit："feat: add login"（包含修复和测试）

---

#### 2. Fixup 前先快照

```bash
git snapshot "before-fixup"
git fixup
# 如果出错：
git stash list
git stash apply stash@{0}
```

---

#### 3. 分步 fixup 大改动

**如果修改涉及多个历史 commit**：

```bash
# 修改文件 A（属于 commit1）
git add A
git fixup  # 选择 commit1

# 修改文件 B（属于 commit2）
git add B
git fixup  # 选择 commit2
```

不要一次性 add 所有修改然后 fixup，这样会混乱。

---

## 2. Amend - 修改最近的 Commit

### 2.1 基本用法

**Amend** 修改最近的 commit（HEAD）。

**使用场景**：
- 忘记添加文件
- 发现 typo
- 修改 commit 消息

### 2.2 Safe Amend 系列

**传统 `git commit --amend` 的问题**：
```bash
git commit -m "feat: add login"
git push

# 后来发现 typo
git commit --amend  # ⚠ 危险！改写了已推送的 commit
git push --force    # ⚠ 破坏协作者历史
```

**Safe Amend 解决方案**：
```bash
git cma  # 或 git cmae
# → 自动检查 HEAD 是否已推送
# → 如果已推送，拒绝执行并建议使用 fixup
```

---

### 2.3 Amend 命令对比

| 命令 | 功能 | 编辑消息 | 包含文件 |
|-----|------|---------|---------|
| `git cma` | 安全 amend | ❌ 不编辑 | 所有文件 (add --all) |
| `git cmae` | 安全 amend | ✅ 可编辑 | 所有文件 (add --all) |
| `git cmu` | 安全 amend | ❌ 不编辑 | 已追踪文件 (add --update) |
| `git cmue` | 安全 amend | ✅ 可编辑 | 已追踪文件 (add --update) |

---

### 2.4 使用场景详解

#### 场景 1：忘记添加文件

```bash
git commit -m "feat: add login"
# 糟糕，忘记添加 login.css

git add src/login.css
git cma
# ✓ login.css 合并到"feat: add login"
```

---

#### 场景 2：发现 typo

```bash
git commit -m "feat: add login"
# 发现 login.ts 有 typo

vim src/login.ts  # 修复 typo
git cmu  # 只包含已追踪文件
# ✓ typo 修复合并到"feat: add login"
```

---

#### 场景 3：修改 commit 消息

```bash
git commit -m "feat: add lonig"  # 打错字了

git cmae
# 编辑器打开，修改为 "feat: add login"
```

---

#### 场景 4：添加遗漏的测试

```bash
git commit -m "feat: add user API"
# 忘记添加测试

vim tests/test_user.py
git add tests/test_user.py
git cma
# ✓ 测试合并到"feat: add user API"
```

---

### 2.5 Amend 安全机制

**检查 HEAD 是否已推送**：
```bash
HEAD_commit_exists_in_remote=$(git branch -r --contains HEAD)
if [ -n "$HEAD_commit_exists_in_remote" ]; then
    echo "HEAD commit exists in remote and you should not amend it."
    echo "Please make another commit or use git fixup."
    exit 1
fi
```

**如果 HEAD 已推送**：
```bash
git cma
# 输出：
# HEAD commit exists in remote and you should not amend it.
# Please make another commit or use git fixup.

# 正确做法：
git commit -m "fix: correct login logic"
# 或
git fixup  # 选择更早的未推送 commit
```

---

## 3. Interactive Rebase - 重写历史的瑞士军刀

### 3.1 基本概念

**Interactive Rebase** 允许你：
- 合并 commits (squash/fixup)
- 重排 commits
- 修改 commit 消息
- 删除 commits
- 拆分 commits

### 3.2 Branch Rebase (ri)

**Skill 提供的智能 rebase**：
```bash
git ri  # git branch-rebase
```

**自动决定边界**：
- 在 topic 分支 → rebase 到与 base 的分叉点
- 在 base 分支 → rebase 到未推送的部分

**等价操作**：
```bash
# 在 topic 分支上
common_ancestor=$(git merge-base $(git base-branch) $(git current-branch))
git rebase -i $common_ancestor

# 在 base 分支上
git rebase -i origin/$(git current-branch)
```

---

### 3.3 Rebase 操作命令

编辑器中可用的命令：

| 命令 | 缩写 | 功能 | 示例 |
|-----|------|-----|------|
| `pick` | `p` | 保留 commit | `pick abc123 feat: login` |
| `reword` | `r` | 修改消息 | `reword abc123 feat: login` |
| `edit` | `e` | 暂停编辑 | `edit abc123 feat: login` |
| `squash` | `s` | 合并（保留消息） | `squash abc123 fix typo` |
| `fixup` | `f` | 合并（丢弃消息） | `fixup abc123 fix typo` |
| `drop` | `d` | 删除 commit | `drop abc123 temp commit` |

---

### 3.4 常见 Rebase 场景

#### 场景 1：合并多个 WIP commits

**原始历史**：
```bash
pick abc123 feat: add login
pick def456 WIP: fix bug
pick ghi789 WIP: add tests
pick jkl012 WIP: fix typo
```

**操作**：
```bash
git ri
# 编辑为：
pick abc123 feat: add login
fixup def456 WIP: fix bug      # 合并到 abc123
fixup ghi789 WIP: add tests    # 合并到 abc123
fixup jkl012 WIP: fix typo     # 合并到 abc123
```

**结果**：
```bash
pick abc123 feat: add login (包含所有修复和测试)
```

---

#### 场景 2：重排 commits

**原始历史**：
```bash
pick abc123 feat: add login
pick def456 feat: add signup
pick ghi789 test: add login tests   # 应该在 abc123 后面
```

**操作**：
```bash
git ri
# 重排为：
pick abc123 feat: add login
pick ghi789 test: add login tests   # 移到这里
pick def456 feat: add signup
```

---

#### 场景 3：拆分大 commit

**原始历史**：
```bash
pick abc123 feat: add login and signup  # 太大，应该拆分
```

**操作**：
```bash
git ri
# 编辑为：
edit abc123 feat: add login and signup  # 标记为 edit

# Rebase 会暂停在 abc123
git reset HEAD~  # 撤销 commit，保留修改
git add src/login.ts
git commit -m "feat: add login"
git add src/signup.ts
git commit -m "feat: add signup"
git rebase --continue
```

---

#### 场景 4：删除 commits

**原始历史**：
```bash
pick abc123 feat: add login
pick def456 temp: debug code   # 应该删除
pick ghi789 feat: add signup
```

**操作**：
```bash
git ri
# 编辑为：
pick abc123 feat: add login
drop def456 temp: debug code   # 删除
pick ghi789 feat: add signup
```

---

### 3.5 Rebase 最佳实践

#### 1. Rebase 前快照

```bash
git snapshot "before-rebase"
git ri
```

#### 2. 小步前进

不要一次性 rebase 几十个 commits，容易出错。分批处理：
```bash
# 先 rebase 最近 5 个
git rebase -i HEAD~5

# 没问题后，再 rebase 更多
git rebase -i HEAD~10
```

#### 3. 遇到冲突不要慌

```bash
# Rebase 冲突
git status  # 查看冲突文件

# 解决冲突
vim <conflict-file>
git add <conflict-file>

# 继续 rebase
git rebase --continue

# 如果实在解决不了，放弃
git rebase --abort
```

---

## 4. Cherry-Pick - 精确移植 Commits

### 4.1 基本用法

**Cherry-Pick** 将特定 commit 应用到当前分支。

```bash
git cherry-pick <commit-hash>
```

**别名**：
```bash
git pi <commit-hash>         # pick
git pif                      # pick 多个（fzf 选择）
```

---

### 4.2 使用场景

#### 场景 1：将 hotfix 应用到多个分支

```bash
# 在 main 分支修复紧急 bug
git checkout main
git commit -m "fix: critical security bug"
# commit hash: abc123

# 将修复应用到 release 分支
git checkout release/v1.0
git pi abc123  # cherry-pick abc123

# 将修复应用到 develop 分支
git checkout develop
git pi abc123
```

---

#### 场景 2：从实验分支提取有用的 commit

```bash
# 在 experiment/new-algo 分支尝试新算法
git checkout experiment/new-algo
# 5 个 commits，其中 2 个有用

git checkout main
git pif  # fzf 多选
# → 选择 2 个有用的 commits
# → 应用到 main
```

---

#### 场景 3：解决 merge 冲突后继续

```bash
git pi abc123
# → 冲突

# 解决冲突
git edit-unmerged
git add-unmerged

# 继续 cherry-pick
git cherry-pick --continue

# 或放弃
git cherry-pick --abort
```

---

### 4.3 Cherry-Pick 选项

| 选项 | 功能 | 使用场景 |
|-----|------|---------|
| `-n` / `--no-commit` | 不自动 commit | 需要修改后再 commit |
| `-x` | 记录原 commit | 标注来源（用于追踪） |
| `-e` | 编辑 commit 消息 | 修改消息更清晰 |
| `--continue` | 继续 cherry-pick | 冲突解决后 |
| `--abort` | 放弃 cherry-pick | 取消操作 |

**示例**：
```bash
# 不自动 commit，可以修改后再 commit
git pi-nx abc123  # --no-commit -x

# 编辑文件
vim src/auth.ts

# 手动 commit
git add -A
git commit -m "fix: apply auth fix from main (cherry-picked from abc123)"
```

---

## 5. 历史修改工作流对比

| 场景 | 推荐工具 | 原因 |
|-----|---------|-----|
| 修改最近的 commit | `amend` | 最简单 |
| 修改历史中的单个 commit | `fixup` | 自动化 rebase |
| 整理多个 commits | `rebase -i` | 灵活控制 |
| 从其他分支提取 commit | `cherry-pick` | 精确移植 |
| 拆分大 commit | `rebase -i` + `edit` | 逐步拆分 |
| 删除垃圾 commit | `rebase -i` + `drop` | 清理历史 |

---

## 6. 安全检查清单

### 执行历史修改前

- [ ] **检查是否已推送**：`git branch -r --contains HEAD`
- [ ] **创建快照**：`git snapshot "before-<operation>"`
- [ ] **确认边界**：`git oldest-changeable-commit`
- [ ] **确认工作区干净**：`git s`

### 执行历史修改后

- [ ] **验证历史**：`git log --oneline`
- [ ] **运行测试**：`npm test` 或等效命令
- [ ] **检查差异**：`git bdf`（确保没有意外变更）

---

## 7. 常见错误和恢复

### 错误 1：Rebase 后发现改错了

**恢复方法**：
```bash
# 查看 reflog
git reflog

# 找到 rebase 前的状态
# 示例：abc123 HEAD@{5}: rebase started

# 重置到 rebase 前
git reset --hard HEAD@{5}
```

---

### 错误 2：Cherry-pick 应用到错误的分支

**恢复方法**：
```bash
# 撤销最近的 commit
git reset --hard HEAD~1

# 或使用 revert（如果已推送）
git revert HEAD
```

---

### 错误 3：Fixup 选错了 commit

**恢复方法**：
```bash
# Rebase 会在冲突时暂停
git rebase --abort

# 或从快照恢复
git stash list
git reset --hard HEAD
git stash apply stash@{0}
```

---

## 8. 高级技巧

### 技巧 1：批量 fixup

```bash
# 修改多个文件后，逐个 fixup
git add file1.ts
git fixup  # 选择 commit1

git add file2.ts
git fixup  # 选择 commit2
```

---

### 技巧 2：Amend 时部分暂存

```bash
# 只想 amend 部分修改
git add -p  # 交互式选择要暂存的块
git cmu     # amend 暂存的部分
```

---

### 技巧 3：Rebase 时自动解决冲突

**配置 rerere（重复冲突解决）**：
```bash
git config --global rerere.enabled true
```

**效果**：Git 会记住你如何解决冲突，下次遇到相同冲突自动应用解决方案。

---

### 技巧 4：交互式 Cherry-pick

```bash
# 使用 fzf 批量 cherry-pick
git pif
# → fzf 界面，按 Tab 多选
# → 选中的 commits 会依次应用
```

---

## 9. 最佳实践总结

### 黄金法则

1. **永远不改写已推送的 commit**（除非团队明确同意）
2. **操作前快照**：`git snapshot`
3. **小步前进**：一次只改少量 commits
4. **频繁验证**：每步操作后检查历史和测试

### 推荐工作流

```bash
# 1. 开发中频繁 commit（不用担心历史混乱）
git commit -m "WIP: add feature"
git commit -m "fix typo"
git commit -m "add tests"

# 2. Push 前整理历史
git ri  # 交互式 rebase
# → squash/fixup 垃圾 commits
# → 重排 commits 使其逻辑清晰
# → 修改 commit 消息

# 3. 最终推送
git ps
```

---

## 参考资料

- Git Aliases 配置：[aliases.gitconfig](https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig)
- Git Workflow Skill：`~/.claude/skills/git-workflow/SKILL.md`
- Pro Git Book - Rewriting History：https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
- Atlassian Git Tutorials - Rewriting History：https://www.atlassian.com/git/tutorials/rewriting-history
