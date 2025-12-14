# Git 故障排查与恢复指南

本文档提供 Git 常见问题的诊断方法和恢复方案，涵盖冲突解决、误操作恢复和紧急救援。

---

## 快速索引

| 问题类型 | 跳转 |
|---------|------|
| Merge 冲突 | [→ 1. 冲突解决](#1-冲突解决工作流) |
| Rebase 冲突 | [→ 2. Rebase 故障](#2-rebase-故障排查) |
| 误删分支 | [→ 3.1 恢复删除的分支](#31-恢复删除的分支) |
| 误删 commit | [→ 3.2 恢复删除的 commit](#32-恢复删除的-commit) |
| 丢失 stash | [→ 3.3 找回丢失的 stash](#33-找回丢失的-stash) |
| 错误 push | [→ 4. 撤销已推送的 commit](#4-撤销已推送的-commit) |
| 工作区污染 | [→ 5. 工作区问题](#5-工作区问题) |
| 性能问题 | [→ 6. 性能优化](#6-性能优化) |

---

## 1. 冲突解决工作流

### 1.1 识别冲突

**检查冲突状态**：
```bash
git status
```

**输出示例**：
```
On branch feature/user-auth
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   src/auth.ts
	both modified:   src/user.ts
```

---

### 1.2 使用 edit-unmerged 快捷工作流

**Skill 提供的便捷命令**：

**步骤 1：编辑冲突文件**
```bash
git edit-unmerged
```
→ 自动在编辑器中打开所有冲突文件

**步骤 2：解决冲突**
手动编辑，删除冲突标记：
```typescript
<<<<<<< HEAD
// 当前分支的代码
function login(user) {
  return authenticate(user);
}
=======
// 要合并的分支的代码
function login(username, password) {
  return auth.verify(username, password);
}
>>>>>>> feature/new-auth
```

选择保留哪个版本，或合并两者：
```typescript
// 合并后的代码
function login(username, password) {
  return authenticate({ username, password });
}
```

**步骤 3：标记已解决**
```bash
git add-unmerged
```
→ 自动将所有冲突文件添加到暂存区

**步骤 4：继续操作**
```bash
git mgc   # 如果是 merge
# 或
git rc    # 如果是 rebase
```

---

### 1.3 手动解决冲突

**查看冲突文件列表**：
```bash
git diff --name-only --diff-filter=U
```

**查看冲突详情**：
```bash
git diff <conflicted-file>
```

**使用 mergetool（图形界面）**：
```bash
git mgt  # git mergetool
```

---

### 1.4 查看冲突原因

**查看两条分支的提交**：
```bash
git merge-log  # git log --oneline --left-right HEAD...MERGE_HEAD
```

**只查看冲突的提交**：
```bash
git conflict-log  # git log --oneline --left-right --merge
```

**示例输出**：
```
< abc123 feat: refactor auth (HEAD)
> def456 feat: new auth system (MERGE_HEAD)
```
→ 两个 commit 都修改了相同文件，导致冲突

---

### 1.5 放弃 merge

**如果冲突太复杂，放弃本次 merge**：
```bash
git merge --abort
```

**恢复到 merge 前的状态**。

---

## 2. Rebase 故障排查

### 2.1 Rebase 冲突

**场景**：
```bash
git rebase main
# 输出：
# CONFLICT (content): Merge conflict in src/auth.ts
# error: could not apply abc123... feat: add login
```

**解决流程**：

**步骤 1：查看冲突**
```bash
git status
```

**步骤 2：解决冲突**
```bash
git edit-unmerged
# 编辑冲突文件
```

**步骤 3：标记已解决**
```bash
git add-unmerged
```

**步骤 4：继续 rebase**
```bash
git rc  # git rebase --continue
```

**如果还有冲突，重复步骤 1-4**。

---

### 2.2 放弃 Rebase

**如果 rebase 失败，想回到 rebase 前**：
```bash
git ra  # git rebase --abort
```

---

### 2.3 Rebase 后发现错误

**场景**：rebase 完成后，发现改错了。

**恢复方法**：
```bash
# 1. 查看 reflog
git reflog

# 输出示例：
# abc123 HEAD@{0}: rebase finished: ...
# def456 HEAD@{1}: rebase: ...
# ghi789 HEAD@{5}: rebase started     ← 这是 rebase 前的状态

# 2. 重置到 rebase 前
git reset --hard ghi789  # 或 HEAD@{5}
```

---

### 2.4 Rebase 卡在某个 commit

**场景**：rebase 时某个 commit 无法自动应用。

**跳过该 commit**：
```bash
git rebase --skip
```

**注意**：跳过后，该 commit 的修改会丢失。确保这是你想要的。

---

## 3. 误操作恢复

### 3.1 恢复删除的分支

#### 场景 1：刚删除分支，还没关闭终端

**查看终端历史**，找到分支最后的 commit hash。

**恢复分支**：
```bash
git checkout -b recovered-branch <commit-hash>
```

---

#### 场景 2：终端历史已清除

**使用 reflog 查找**：
```bash
git reflog --all | grep "branch-name"
```

**输出示例**：
```
abc123 refs/heads/feature/user-auth@{0}: commit: feat: add login
def456 refs/heads/feature/user-auth@{1}: commit: feat: add signup
```

**恢复分支**：
```bash
git checkout -b recovered-branch abc123
```

---

#### 场景 3：远程分支被删除

**如果本地分支还在**：
```bash
git push -u origin feature/user-auth
```

**如果本地分支也被删除**：
```bash
# 查看远程删除记录（如果 GitHub/GitLab 有备份）
gh api repos/:owner/:repo/events | grep "delete"

# 联系仓库管理员恢复
```

---

### 3.2 恢复删除的 Commit

#### 场景 1：使用 `git reset --hard` 删除了 commits

**使用 reflog 查找**：
```bash
git reflog
```

**输出示例**：
```
abc123 HEAD@{0}: reset: moving to HEAD~3
def456 HEAD@{1}: commit: feat: add login    ← 被删除的 commit
ghi789 HEAD@{2}: commit: feat: add signup   ← 被删除的 commit
```

**恢复 commits**：
```bash
git reset --hard def456  # 恢复到 def456
```

---

#### 场景 2：Rebase 时删除了 commit（drop）

**查找被 drop 的 commit**：
```bash
git reflog | grep "drop"
```

**恢复**：
```bash
git cherry-pick <commit-hash>
```

---

### 3.3 找回丢失的 Stash

#### 场景 1：`git stash drop` 后想找回

**使用 stash-history**：
```bash
git sh  # git stash-history
```

**输出示例**（显示最近 15 个 stash，包括已 drop 的）：
```
abc123 WIP on main: 2024-01-15 12:30:45 - Autosave
def456 WIP on feature: 2024-01-14 10:20:30 - Snapshot before merge
```

**恢复 stash**：
```bash
git stash apply abc123
```

---

#### 场景 2：完全不记得 stash 的信息

**使用 fsck 查找所有悬空 commits**：
```bash
git fsck --unreachable | grep commit
```

**输出**：
```
unreachable commit abc123
unreachable commit def456
```

**查看 commit 内容**：
```bash
git show abc123
```

**如果是你要的 stash，恢复**：
```bash
git stash apply abc123
# 或创建分支
git branch recovered-stash abc123
```

---

### 3.4 恢复未提交的修改

#### 场景 1：误执行 `git reset --hard`

**如果之前有 stash 或 snapshot**：
```bash
git stash list
git stash apply stash@{0}
```

**如果没有备份**：
- ❌ 未提交的修改**无法恢复**（Git 不追踪未提交的内容）
- ✅ 教训：重要修改前先 `git save` 或 `git snapshot`

---

#### 场景 2：误执行 `git checkout --` 丢弃了修改

**如果编辑器（如 VSCode）有本地历史**：
- 查看编辑器的本地历史功能
- 恢复文件的历史版本

**如果没有**：
- ❌ 无法恢复

---

## 4. 撤销已推送的 Commit

### 4.1 使用 Revert（推荐）

**原理**：创建新 commit 撤销旧 commit，不改写历史。

**撤销最近的 commit**：
```bash
git revert HEAD
git push
```

**撤销多个 commits**：
```bash
git revert abc123 def456 ghi789
git push
```

**优点**：
- ✅ 安全，不破坏协作者历史
- ✅ 保留完整的历史记录

**缺点**：
- ❌ 历史中会留下"revert"commits

---

### 4.2 使用 Reset + Force Push（谨慎）

**原理**：删除 commit，强制推送新历史。

**⚠ 警告**：只在以下情况使用：
- 个人分支（无其他人协作）
- 团队明确同意
- 紧急修复敏感信息泄露

**步骤**：
```bash
# 1. 重置到目标 commit
git reset --hard abc123  # 删除 abc123 之后的所有 commits

# 2. 强制推送（--force-with-lease 更安全）
git push --force-with-lease origin feature/my-branch
```

**`--force-with-lease` vs `--force`**：
- `--force-with-lease`：如果远程有其他人推送的新 commit，拒绝强制推送（保护协作者）
- `--force`：无条件强制推送（危险，可能覆盖别人的工作）

---

### 4.3 敏感信息泄露紧急处理

**场景**：不小心提交了密码、API key 到公开仓库。

**紧急步骤**：

**步骤 1：立即更换泄露的凭证**
```bash
# 立即更换密码或重新生成 API key
```

**步骤 2：从历史中彻底删除敏感文件**
```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch config/secrets.json" \
  --prune-empty --tag-name-filter cat -- --all
```

**或使用 BFG Repo-Cleaner（更快）**：
```bash
# 安装 BFG
brew install bfg  # macOS
# 或从 https://rtyley.github.io/bfg-repo-cleaner/ 下载

# 删除敏感文件
bfg --delete-files secrets.json

# 清理和强制推送
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force --all
git push --force --tags
```

**步骤 3：通知协作者**
```
所有协作者需要重新 clone 仓库，不能 pull。
```

---

## 5. 工作区问题

### 5.1 工作区污染

**场景**：工作区有大量未追踪文件或修改，想清理。

**预览将要删除的文件**：
```bash
git dry  # git clean -d --dry-run
```

**交互式清理**：
```bash
git cl   # git clean -d -i
```

**清理包括 ignored 文件**：
```bash
git clx  # git clean -d -x -i
```

---

### 5.2 忽略所有未追踪文件

**将未追踪文件加入 .gitignore**：
```bash
git ignore-untracked
```

**原理**：
```bash
git status | grep -P "^\t" | grep -vF .gitignore | sed "s/^\t//" >> .gitignore
```

---

### 5.3 文件模式问题（Windows/Mac/Linux 差异）

**场景**：文件模式（权限）改变导致 Git 认为文件修改了。

**禁用文件模式检查**：
```bash
git config core.fileMode false
```

---

## 6. 性能优化

### 6.1 仓库膨胀

**检查仓库大小**：
```bash
du -sh .git
```

**压缩仓库**：
```bash
git gc --aggressive --prune=now
```

---

### 6.2 Reflog 清理

**Reflog 会记录所有 HEAD 移动，占用空间。**

**查看 reflog**：
```bash
git reflog
```

**清理旧的 reflog 记录**：
```bash
git reflog expire --expire=30.days.ago --all
git gc --prune=now
```

**注意**：清理后，无法通过 reflog 恢复超过 30 天的操作。

---

### 6.3 大文件问题

**查找大文件**：
```bash
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sed -n 's/^blob //p' | \
  sort --numeric-sort --key=2 | \
  tail -20
```

**从历史中删除大文件**：
```bash
bfg --strip-blobs-bigger-than 10M
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

---

## 7. 特殊场景救援

### 7.1 Detached HEAD

**场景**：执行 `git checkout <commit-hash>` 后进入 detached HEAD 状态。

**识别**：
```bash
git status
# 输出：HEAD detached at abc123
```

**如果做了修改并想保留**：
```bash
git branch temp-branch
git checkout main
git merge temp-branch
```

**如果不想保留修改**：
```bash
git checkout main  # 直接切回分支，修改会丢失
```

---

### 7.2 Git 仓库损坏

**场景**：`.git` 目录损坏，无法执行 Git 命令。

**尝试修复**：
```bash
git fsck --full
```

**如果修复失败，从备份或远程恢复**：
```bash
cd ..
rm -rf broken-repo
git clone <remote-url> recovered-repo
```

---

### 7.3 提交了超大文件，push 失败

**场景**：
```bash
git push
# 输出：error: remote: File too large (>100MB)
```

**解决方法**：

**如果文件在最近的 commit**：
```bash
git reset --soft HEAD~1  # 撤销 commit，保留修改
git rm --cached large-file.zip  # 从暂存区移除
echo "large-file.zip" >> .gitignore
git commit -m "remove large file"
git push
```

**如果文件在历史中**：
```bash
bfg --delete-files large-file.zip
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force-with-lease
```

---

## 8. 诊断工具

### 8.1 查看 Git 状态

```bash
git status          # 工作区状态
git log --oneline   # 提交历史
git reflog          # HEAD 移动历史
git branch -av      # 所有分支
```

---

### 8.2 查看远程状态

```bash
git remote -v               # 远程仓库列表
git remote show origin      # 远程仓库详情
git fetch --dry-run         # 预览 fetch 结果
```

---

### 8.3 查看冲突详情

```bash
git diff --name-only --diff-filter=U  # 冲突文件列表
git diff <file>                       # 冲突详情
git log --merge --oneline             # 冲突的 commits
```

---

## 9. 预防措施

### 9.1 日常备份

**每天下班前快照**：
```bash
git snapshot "end-of-day-$(date +%Y%m%d)"
```

**每周导出 reflog**：
```bash
git reflog > ~/git-reflog-backup-$(date +%Y%m%d).txt
```

---

### 9.2 重要操作前快照

```bash
# Merge 前
git snapshot "before-merge-$(git current-branch)"

# Rebase 前
git snapshot "before-rebase"

# Reset 前
git snapshot "before-reset"
```

---

### 9.3 使用受保护的分支

在 GitHub/GitLab 设置：
- ✅ 保护 main/develop 分支
- ✅ 要求 PR 审查
- ✅ 禁止 force push
- ✅ 要求 CI 通过

---

### 9.4 配置 Git Hooks

**Pre-commit hook（防止提交敏感信息）**：
```bash
# .git/hooks/pre-commit
#!/bin/bash

if git diff --cached | grep -i "password\|api_key\|secret"; then
    echo "⚠ 检测到敏感信息，拒绝提交"
    exit 1
fi
```

**Pre-push hook（防止 push 到错误分支）**：
```bash
# .git/hooks/pre-push
#!/bin/bash

current_branch=$(git current-branch)
if [ "$current_branch" = "main" ]; then
    echo "⚠ 不能直接 push 到 main，请创建 PR"
    exit 1
fi
```

---

## 10. 紧急救援检查清单

**当遇到严重问题时，按此清单逐步排查**：

### 步骤 1：不要慌，停止操作
- [ ] 停止执行任何 Git 命令
- [ ] 截图或记录当前错误信息

### 步骤 2：检查状态
- [ ] `git status` - 查看工作区状态
- [ ] `git log --oneline` - 查看提交历史
- [ ] `git reflog` - 查看 HEAD 移动历史

### 步骤 3：尝试恢复
- [ ] 查看是否有 snapshot：`git stash list`
- [ ] 查看 reflog 是否有恢复点
- [ ] 尝试 `git rebase --abort` 或 `git merge --abort`

### 步骤 4：求助
- [ ] 查看本文档相关章节
- [ ] 搜索错误信息
- [ ] 在团队群或 Stack Overflow 求助

### 步骤 5：最后手段
- [ ] 备份当前 `.git` 目录：`cp -r .git .git.backup`
- [ ] 从远程重新 clone
- [ ] 手动合并本地修改

---

## 参考资料

- Git Aliases 配置：`/home/alecshan/projects/private/dotfiles/git/.config/git/conf/aliases.gitconfig`
- Git Workflow Skill：`~/.claude/skills/git-workflow/SKILL.md`
- Git Flight Rules：https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!：https://ohshitgit.com/
- Pro Git Book：https://git-scm.com/book
