# Git-Workflow Skill 使用场景演示

本文档提供真实场景下的完整使用演示，展示 skill 如何在实际开发中提供智能指导。

---

## 场景 1：完整功能开发流程

### 背景

**项目**: Web 应用
**需求**: 实现用户认证功能
**当前状态**: 在 main 分支，工作区干净

---

### Step 1: 创建功能分支

**用户输入**:
```
开始新功能 user-authentication
```

**Skill 响应**:
```
✓ Git Workflow - 创建 Topic 分支

当前状态:
  • 分支: main
  • 工作区: 干净 ✓
  • 远程同步: 已同步 ✓

推荐命令:
  git tnr feature/user-authentication

说明:
  - tnr (topic-new-remote): 创建分支并推送到远程
  - 适用于团队协作场景，立即建立远程跟踪
  - 如果只是本地实验，使用 `git tn` (不推送)

执行流程:
  1. 从 main 创建新分支
  2. 推送到远程: git push -u origin feature/user-authentication
  3. 自动设置跟踪关系

安全检查:
  ✓ 工作区干净 (如有未提交修改，自动 stash)
  ✓ Base 分支同步 (如落后，自动 pull)
```

**实际执行**:
```bash
git tnr feature/user-authentication
# Branch 'feature/user-authentication' set up to track remote branch 'feature/user-authentication' from 'origin'.
# Switched to a new branch 'feature/user-authentication'
```

---

### Step 2: 开发中保存进度

**场景**: 实现了一半，需要临时切换到别的任务

**用户输入**:
```
临时保存进度
```

**Skill 响应**:
```
✓ Git Workflow - 保存工作进度

推荐命令:
  git save "WIP: implementing login API"

说明:
  - save: 快速保存当前进度 (add + commit)
  - WIP (Work In Progress): 标记未完成的工作
  - 后续可用 `git fixup` 修改或合并此 commit

替代方案:
  - 如需切换分支但不提交: git stash
  - 查看 stash 列表: git stash-history
```

**实际执行**:
```bash
git save "WIP: implementing login API"
# [feature/user-authentication a1b2c3d] WIP: implementing login API
# 3 files changed, 45 insertions(+)
```

---

### Step 3: 继续开发并提交

**场景**: 完成了登录功能，准备提交

**用户输入**:
```
提交代码
```

**Skill 响应**:
```
✓ Git Workflow - 提交代码

基础提交:
  git commit -m "feat: implement user login endpoint"

或使用快捷命令:
  - git c "message": 提交暂存区
  - git ca "message": 提交所有修改 (add + commit)
  - git save "message": 添加并提交 (等同 ca)

提交规范 (Conventional Commits):
  - feat: 新功能
  - fix: bug 修复
  - refactor: 重构
  - test: 测试
  - docs: 文档
  - chore: 配置/工具

下一步:
  - 继续开发: 重复开发-提交循环
  - 推送到远程: git push
  - 查看进度: git blg (branch log)
```

**实际执行**:
```bash
git ca "feat: implement user login endpoint"
# [feature/user-authentication b2c3d4e] feat: implement user login endpoint
# 5 files changed, 120 insertions(+), 8 deletions(-)
```

---

### Step 4: 查看开发进度

**用户输入**:
```
查看我改了什么
```

**Skill 响应**:
```
✓ Git Workflow - 查看修改内容

当前分支: feature/user-authentication

查看差异 (与 main 对比):
  git bdf                    # 显示完整差异
  git bdf --stat             # 仅显示统计信息
  git bdf --name-only        # 仅显示文件名

查看日志 (topic 分支独有的 commits):
  git blg                    # 简洁日志
  git blg --oneline          # 单行格式
  git blog                   # 详细日志

查看当前状态:
  git status                 # 工作区状态
  git ahead-count            # 领先 base 分支的 commits
  git behind-count           # 落后 base 分支的 commits

推荐工作流:
  1. git blg -> 查看提交历史
  2. git bdf --stat -> 快速了解改动范围
  3. git bdf -> 详细审查代码差异
```

**实际执行**:
```bash
git blg
# * b2c3d4e (HEAD -> feature/user-authentication) feat: implement user login endpoint
# * a1b2c3d WIP: implementing login API
# * 9z8y7x6 feat: add authentication middleware

git bdf --stat
# src/api/auth.ts          | 45 +++++++++++++++++++++
# src/middleware/auth.ts   | 32 +++++++++++++++
# tests/api/auth.test.ts   | 58 +++++++++++++++++++++++++
# 3 files changed, 135 insertions(+)
```

---

### Step 5: 修改历史 Commit

**场景**: 发现 WIP commit 应该合并到后面的 commit

**用户输入**:
```
修改之前的 commit
```

**Skill 响应**:
```
✓ Git Workflow - 修改历史 Commit

推荐命令:
  git fixup

说明:
  - 交互式选择要修改的 commit (需要 fzf)
  - 自动合并到选中的 commit
  - 保持 commit 历史清晰

工作流:
  1. 暂存要修正的修改: git add <files>
  2. 运行: git fixup
  3. 选择目标 commit (fzf 界面)
  4. 自动 rebase 并合并

安全边界:
  ⚠️  只能修改未推送的 commits
  ⚠️  如已推送，需使用 force-push (风险操作)

替代方案:
  - 修改最近一次 commit: git cmae (amend with editor)
  - 修改消息: git commit --amend
  - 交互式 rebase: git rebase -i HEAD~N

检查是否可以安全修改:
  git safe-commit-amend <commit-hash>
  # 返回 0: 可以修改
  # 返回 1: 已推送，不建议修改
```

**实际执行**:
```bash
# 已有修改暂存
git fixup
# (fzf 界面弹出，显示最近的 commits)
# 选择: b2c3d4e feat: implement user login endpoint
# Successfully rebased and updated refs/heads/feature/user-authentication.
```

---

### Step 6: 准备 Pull Request

**用户输入**:
```
准备 PR
```

**Skill 响应**:
```
✓ Git Workflow - PR 准备检查清单

当前分支: feature/user-authentication
Base 分支: main

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 代码审查

  [ ] 查看 commits 列表
      git blg

  [ ] 查看完整差异
      git bdf

  [ ] 确认修改范围
      git bdf --stat

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 质量检查

  [ ] 测试通过
      npm test / pytest / cargo test

  [ ] Lint 通过
      npm run lint / ruff check

  [ ] 类型检查通过
      tsc --noEmit / mypy

  [ ] 构建成功
      npm run build / cargo build

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔄 分支同步

  [ ] 与 base 分支同步
      git fetch origin main
      git merge origin/main  (或 git rebase)

  [ ] 推送到远程
      git push

  [ ] 检查远程分支状态
      git remote-branch feature/user-authentication

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 PR 描述模板

## Summary
[简要描述此 PR 的目的和改动]

## Changes
- 实现了用户登录 API 端点
- 添加了 JWT 认证中间件
- 完善了单元测试覆盖

## Testing
- [ ] 单元测试通过
- [ ] 手动测试登录流程
- [ ] 边界条件测试

## Related Issues
Closes #123

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

快速检查命令:
  # 一键检查脚本 (复制到 shell)
  echo "=== Commits ===" && git blg && \
  echo "=== Changes ===" && git bdf --stat && \
  echo "=== Status ===" && git status

完成检查后:
  - 在 GitHub/GitLab 创建 Pull Request
  - 填写 PR 描述
  - 请求 Code Review
```

**实际执行**:
```bash
# 运行测试
npm test
# ✓ All tests passed

# 运行 lint
npm run lint
# ✓ No issues found

# 推送到远程
git push
# To github.com:user/project.git
#    9z8y7x6..b2c3d4e  feature/user-authentication -> feature/user-authentication
```

---

### Step 7: 合并功能分支

**场景**: PR 已通过审查，准备合并

**用户输入**:
```
完成功能
```

**Skill 响应**:
```
✓ Git Workflow - 合并 Topic 分支

当前状态:
  • 分支: feature/user-authentication
  • 工作区: 干净 ✓
  • 待合并 commits: 2 个
  • 远程同步: 已同步 ✓

推荐命令:
  git tmg

说明:
  - tmg (topic-merge): 合并当前分支到 base 分支
  - 使用 --no-ff: 保留分支历史
  - 自动删除本地和远程分支 (可选)

执行流程:
  1. 切换到 main 分支
  2. Merge --no-ff feature/user-authentication
  3. 推送到远程
  4. 删除本地分支: git branch -d feature/user-authentication
  5. 删除远程分支: git push origin :feature/user-authentication

安全检查:
  ⚠️  确保不在 main/master 分支执行
  ⚠️  确保工作区干净 (自动 stash)
  ⚠️  确认与远程同步 (自动 push/pull)

⚠️  执行前建议:
  git snapshot "before-merge-user-auth"

确认无误后执行:
  git tmg
```

**实际执行**:
```bash
git tmg
# Switched to branch 'main'
# Updating 9z8y7x6..b2c3d4e
# Merge made by the 'recursive' strategy.
# (merge commit created with --no-ff)
# Deleted branch feature/user-authentication (was b2c3d4e).
# To github.com:user/project.git
#  - [deleted]         feature/user-authentication
```

---

## 场景 2: 冲突解决流程

### 背景

**场景**: 在合并分支时遇到冲突
**冲突文件**: `src/api/auth.ts`

---

### Step 1: 识别冲突

**系统输出**:
```bash
git merge feature/oauth
# Auto-merging src/api/auth.ts
# CONFLICT (content): Merge conflict in src/api/auth.ts
# Automatic merge failed; fix conflicts and then commit the result.
```

**用户输入**:
```
merge 冲突怎么办
```

**Skill 响应**:
```
✓ Git Workflow - 解决 Merge 冲突

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 查看冲突原因

  git merge-log
  # 显示两条分支的提交历史，理解为何冲突

  git conflict-log
  # 只显示导致冲突的提交

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🛠️  解决冲突工作流

  Step 1: 编辑冲突文件
    git edit-unmerged
    # 自动打开所有冲突文件 ($EDITOR)

  Step 2: 手动解决冲突
    # 在编辑器中:
    # 1. 找到冲突标记: <<<<<<<, =======, >>>>>>>
    # 2. 决定保留哪部分代码
    # 3. 删除冲突标记
    # 4. 保存文件

  Step 3: 标记为已解决
    git add-unmerged
    # 自动 add 所有冲突文件

  Step 4: 继续 merge
    git mgc  (merge continue)
    # 或 git merge --continue

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ 放弃 Merge

  如果决定不继续:
    git mga  (merge abort)
    # 或 git merge --abort

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

当前冲突文件:
  src/api/auth.ts

推荐步骤:
  1. git edit-unmerged
  2. 解决冲突
  3. git add-unmerged
  4. git mgc
```

---

### Step 2: 查看冲突详情

**实际执行**:
```bash
git merge-log
# < feature/oauth: c4d5e6f feat: add OAuth provider
# < feature/oauth: b3c4d5e feat: implement OAuth flow
# > main: a2b3c4d feat: add JWT validation
# > main: 9a0b1c2 feat: update auth middleware

git conflict-log
# < feature/oauth: c4d5e6f feat: add OAuth provider (conflicts in: src/api/auth.ts)
# > main: a2b3c4d feat: add JWT validation (conflicts in: src/api/auth.ts)
```

---

### Step 3: 解决冲突

**实际执行**:
```bash
git edit-unmerged
# Opening: src/api/auth.ts
```

**文件内容 (解决前)**:
```typescript
<<<<<<< HEAD
function authenticate(token: string) {
  return validateJWT(token);
}
=======
function authenticate(provider: string, code: string) {
  return validateOAuth(provider, code);
}
>>>>>>> feature/oauth
```

**解决后**:
```typescript
function authenticate(token: string): Promise<User>;
function authenticate(provider: string, code: string): Promise<User>;
function authenticate(...args: any[]): Promise<User> {
  if (args.length === 1) {
    return validateJWT(args[0]);
  } else {
    return validateOAuth(args[0], args[1]);
  }
}
```

---

### Step 4: 完成 Merge

**实际执行**:
```bash
git add-unmerged
# Adding: src/api/auth.ts

git mgc
# [main d5e6f7g] Merge branch 'feature/oauth'
```

---

## 场景 3: 误操作恢复

### 场景 3.1: 误删分支

**背景**: 不小心删除了还在开发的分支

**用户输入**:
```
误删分支怎么办
```

**Skill 响应**:
```
✓ Git Workflow - 恢复已删除分支

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 查找分支最后的位置

  git reflog | grep "branch-name"
  # 或
  git reflog | grep -E "(checkout|commit)" | head -20

示例输出:
  a1b2c3d HEAD@{0}: checkout: moving from feature/user-auth to main
  b2c3d4e HEAD@{1}: commit: feat: add login endpoint
  c3d4e5f HEAD@{2}: checkout: moving from main to feature/user-auth

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🛠️  恢复分支

  Step 1: 找到分支最后的 commit hash
    # 从 reflog 中找到: b2c3d4e

  Step 2: 重建分支
    git checkout -b feature/user-auth-recovered b2c3d4e
    # 或
    git branch feature/user-auth b2c3d4e
    git checkout feature/user-auth

  Step 3: 验证恢复
    git log --oneline -5
    # 确认 commits 都在

  Step 4: 推送到远程 (如需要)
    git push -u origin feature/user-auth

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⏱️ 时间限制

  reflog 默认保留 90 天
  超过此时间，commit 可能被垃圾回收

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

远程分支恢复:

  如果本地分支还在:
    git push -u origin branch-name

  如果本地也删除:
    1. 联系有权限的管理员
    2. 或从其他协作者克隆分支
```

**实际执行**:
```bash
git reflog | grep "user-auth"
# a1b2c3d HEAD@{0}: checkout: moving from feature/user-auth to main
# b2c3d4e HEAD@{1}: commit: feat: add login endpoint

git checkout -b feature/user-auth-recovered b2c3d4e
# Switched to a new branch 'feature/user-auth-recovered'

git log --oneline -3
# b2c3d4e feat: add login endpoint
# a1b2c3d WIP: implementing login API
# 9z8y7x6 feat: add authentication middleware
```

---

### 场景 3.2: 误操作后回滚

**背景**: 执行了错误的 rebase

**用户输入**:
```
rebase 搞乱了怎么办
```

**Skill 响应**:
```
✓ Git Workflow - Rebase 回滚

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔙 紧急回滚

  git reset --hard ORIG_HEAD
  # ORIG_HEAD: 危险操作前的自动备份

  ⚠️  仅在 rebase/merge/reset 后立即有效
  ⚠️  如已有新 commit，ORIG_HEAD 会被覆盖

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 使用 reflog 精确回滚

  Step 1: 查看 reflog
    git reflog

  示例输出:
    a1b2c3d HEAD@{0}: rebase finished: returning to refs/heads/feature
    b2c3d4e HEAD@{1}: rebase: commit: fix bug
    c3d4e5f HEAD@{2}: rebase: checkout main
    d4e5f6g HEAD@{3}: commit: original state  ← 回滚到这里

  Step 2: 回滚到 rebase 前
    git reset --hard HEAD@{3}
    # 或
    git reset --hard d4e5f6g

  Step 3: 验证
    git log --oneline -5

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🛡️  预防措施

  执行危险操作前创建快照:
    git snapshot "before-rebase-feature"
    # 或
    git tag backup-$(date +%Y%m%d-%H%M%S)

  恢复快照:
    git reset --hard $(git rev-parse backup-20250115-143022)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

如果已推送:

  ⚠️  不建议强制推送，会影响其他协作者

  安全方案:
    1. 创建新分支: git checkout -b feature-fixed
    2. Cherry-pick 需要的 commits
    3. 提交新 PR，废弃旧分支
```

**实际执行**:
```bash
# 紧急回滚
git reset --hard ORIG_HEAD
# HEAD is now at d4e5f6g original state

# 或使用 reflog
git reflog
git reset --hard HEAD@{3}
# HEAD is now at d4e5f6g original state
```

---

## 场景 4: 多人协作场景

### 场景 4.1: 同步远程更新

**背景**: 其他人更新了 main 分支，需要同步到当前分支

**用户输入**:
```
同步 main 分支的更新
```

**Skill 响应**:
```
✓ Git Workflow - 同步远程更新

当前分支: feature/new-ui
Base 分支: main

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔄 同步方式选择

方式 1: Merge (推荐)
  git fetch origin main
  git merge origin/main

  优点: 保留完整历史，安全
  缺点: 会产生 merge commit

方式 2: Rebase
  git fetch origin main
  git rebase origin/main

  优点: 线性历史，干净
  缺点: 修改历史，需 force-push

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 检查差异

  查看落后多少:
    git behind-count

  查看新增的 commits:
    git log HEAD..origin/main

  查看会引入的差异:
    git diff HEAD...origin/main

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

推荐流程 (Merge):

  Step 1: 保存当前工作
    git status
    # 如有未提交修改: git save "WIP"

  Step 2: 获取远程更新
    git fetch origin main

  Step 3: 合并
    git merge origin/main

  Step 4: 解决冲突 (如有)
    git edit-unmerged
    git add-unmerged
    git mgc

  Step 5: 推送
    git push

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  使用 Rebase 的注意事项:

  仅在以下情况使用:
    - 分支只有你一人开发
    - 未推送到远程
    - 想保持线性历史

  执行后需要:
    git push --force-with-lease
    # 而非 --force (更安全)
```

**实际执行 (Merge 方式)**:
```bash
git fetch origin main
# From github.com:user/project
#  * branch            main       -> FETCH_HEAD

git behind-count
# 3

git merge origin/main
# Merge made by the 'recursive' strategy.
# src/api/users.ts | 15 +++++++++++----
# 1 file changed, 11 insertions(+), 4 deletions(-)

git push
# To github.com:user/project.git
#    a1b2c3d..d4e5f6g  feature/new-ui -> feature/new-ui
```

---

### 场景 4.2: 协作者推送了更新

**背景**: 推送失败，提示远程有更新

**系统输出**:
```bash
git push
# To github.com:user/project.git
#  ! [rejected]        feature/api -> feature/api (fetch first)
# error: failed to push some refs
# hint: Updates were rejected because the remote contains work that you do
# hint: not have locally. This is usually caused by another repository pushing
```

**用户输入**:
```
推送失败怎么办
```

**Skill 响应**:
```
✓ Git Workflow - 处理推送冲突

原因: 远程分支有新的 commits (协作者推送了更新)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔄 解决步骤

  Step 1: 拉取远程更新
    git fetch origin feature/api

  Step 2: 查看差异
    git log HEAD..origin/feature/api
    # 查看远程有哪些新 commits

  Step 3: 合并或 Rebase

    方式 A: Merge (保险)
      git merge origin/feature/api
      # 或 git pull (相当于 fetch + merge)

    方式 B: Rebase (线性历史)
      git rebase origin/feature/api
      # 或 git pull --rebase

  Step 4: 推送
    git push

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  何时使用 Force Push

  仅在以下情况:
    - 确认远程更新是错误的
    - 你和协作者已沟通确认
    - 分支只有你一人开发

  使用安全的 force push:
    git push --force-with-lease
    # 而非 --force

  --force-with-lease 会:
    - 检查远程分支是否被其他人更新
    - 如被更新，拒绝推送
    - 避免覆盖他人工作

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

推荐流程:

  # 简单情况 (只需合并)
  git pull
  git push

  # 需要查看差异
  git fetch origin
  git log HEAD..origin/feature/api
  git pull
  git push

  # 想要线性历史
  git pull --rebase
  git push
```

**实际执行**:
```bash
git pull
# From github.com:user/project
#  * branch            feature/api -> FETCH_HEAD
# Merge made by the 'recursive' strategy.
# src/api/posts.ts | 8 ++++++++
# 1 file changed, 8 insertions(+)

git push
# To github.com:user/project.git
#    a1b2c3d..e5f6g7h  feature/api -> feature/api
```

---

## 场景 5: 高级历史修改

### 场景 5.1: 交互式 Rebase

**背景**: 提交历史很乱，需要整理

**用户输入**:
```
整理 commit 历史
```

**Skill 响应**:
```
✓ Git Workflow - 交互式 Rebase

当前分支: feature/complex-ui
待整理 commits: 8 个

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔧 交互式 Rebase

  git rebase -i HEAD~8
  # 或指定 base commit
  git rebase -i <base-commit-hash>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

可用操作:

  pick:   保留 commit (默认)
  reword: 修改 commit 消息
  edit:   修改 commit 内容
  squash: 合并到上一个 commit (保留消息)
  fixup:  合并到上一个 commit (丢弃消息)
  drop:   删除 commit

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

示例场景:

原始历史:
  pick a1b2c3d feat: add button
  pick b2c3d4e fix: typo
  pick c3d4e5f WIP: testing
  pick d4e5f6g feat: add styling
  pick e5f6g7h fix: button color
  pick f6g7h8i refactor: extract component
  pick g7h8i9j feat: add hover effect
  pick h8i9j0k docs: add comments

整理后:
  pick a1b2c3d feat: add button
  fixup e5f6g7h fix: button color      # 合并到 button commit
  fixup d4e5f6g feat: add styling      # 合并到 button commit
  reword g7h8i9j feat: add hover effect # 改消息
  pick f6g7h8i refactor: extract component
  pick h8i9j0k docs: add comments
  drop b2c3d4e fix: typo               # 删除
  drop c3d4e5f WIP: testing            # 删除

最终历史 (4 个 commits):
  - feat: add button with styling
  - feat: add button hover animation
  - refactor: extract button component
  - docs: add button documentation

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  安全检查

  执行前:
    1. 确保已推送或创建备份:
       git snapshot "before-rebase"

    2. 确认只修改未推送的 commits:
       git log --oneline origin/feature/complex-ui..HEAD

  如果已推送:
    - Rebase 后需要 force-push
    - 确认分支只有你一人开发
    - 与协作者确认

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

简化方案:

  自动合并所有 fixup commits:
    git rebase -i --autosquash HEAD~8

  需配合:
    git commit --fixup=<target-commit>
    # 创建时标记为 fixup
```

**实际执行**:
```bash
git snapshot "before-rebase"
# Created tag: snapshot-2025-01-15-143022

git rebase -i HEAD~8
# (编辑器打开，修改 pick -> fixup/squash/drop/reword)
# (保存并退出)
# Successfully rebased and updated refs/heads/feature/complex-ui.

git log --oneline -5
# h8i9j0k docs: add button documentation
# f6g7h8i refactor: extract button component
# g7h8i9j feat: add button hover animation
# a1b2c3d feat: add button with styling
```

---

## 场景 6: 紧急救援

### 场景 6.1: 提交到错误分支

**背景**: 在 main 分支上提交了应该在 feature 分支的代码

**用户输入**:
```
提交到错误分支了
```

**Skill 响应**:
```
✓ Git Workflow - 移动 Commits 到正确分支

当前状态: 在 main 分支，有 2 个新 commits

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔧 解决方案

  Step 1: 创建新分支保存 commits
    git branch feature/correct-branch
    # 不切换，只创建分支指向当前 HEAD

  Step 2: 重置 main 到正确位置
    git reset --hard origin/main
    # 或
    git reset --hard HEAD~2  (如果知道 commits 数量)

  Step 3: 切换到新分支继续工作
    git checkout feature/correct-branch

  Step 4: 推送新分支
    git push -u origin feature/correct-branch

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  如果已推送到 main

  ⚠️  不能直接重置远程 main (受保护)

  方案 A: Revert (安全)
    git revert HEAD~2..HEAD
    # 创建新 commits 撤销错误 commits
    git push

  方案 B: 联系管理员
    - 如果仓库有保护规则，需要管理员权限
    - 或通过 PR 的方式回滚

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

完整示例:

  # 当前在 main，有 2 个错误 commits
  git log --oneline -3
  # c3d4e5f (HEAD -> main) feat: add new feature
  # b2c3d4e fix: update config
  # a1b2c3d (origin/main) previous work

  # 保存 commits 到新分支
  git branch feature/new-feature

  # 重置 main
  git reset --hard origin/main

  # 验证 main 已恢复
  git log --oneline -3
  # a1b2c3d (HEAD -> main, origin/main) previous work

  # 切换到新分支
  git checkout feature/new-feature
  git log --oneline -3
  # c3d4e5f (HEAD -> feature/new-feature) feat: add new feature
  # b2c3d4e fix: update config
  # a1b2c3d (origin/main, main) previous work
```

**实际执行**:
```bash
git branch feature/new-feature
# Created branch 'feature/new-feature'

git reset --hard origin/main
# HEAD is now at a1b2c3d previous work

git checkout feature/new-feature
# Switched to branch 'feature/new-feature'

git push -u origin feature/new-feature
# To github.com:user/project.git
#  * [new branch]      feature/new-feature -> feature/new-feature
```

---

## 总结

这些场景涵盖了 Git-Workflow Skill 在实际开发中的典型使用方式：

1. **完整功能开发**: 从创建分支到合并的全流程
2. **冲突解决**: 系统化的冲突处理工作流
3. **误操作恢复**: reflog 和备份策略
4. **多人协作**: 同步和推送冲突处理
5. **高级历史修改**: 交互式 rebase 整理历史
6. **紧急救援**: 常见错误的快速修复

每个场景都展示了：
- ✅ Skill 如何识别用户意图
- ✅ 提供的具体命令和工作流
- ✅ 安全检查和注意事项
- ✅ 实际执行效果

**关键价值**:
- 🎯 自然语言 → Git 命令的智能映射
- 🛡️ 执行前的安全检查和警告
- 📚 学习辅助和操作指导
- 🔄 状态感知的上下文推荐

---

**下一步**: 参考 [docs/testing.md](../docs/testing.md) 进行完整的触发和功能测试。
