# Git Topic Workflow 核心命令详解

本文档详细说明 topic 工作流的核心命令：`tnr`/`tn`/`tmg`/`td`，以及相关的状态检查命令。

## Topic Branch 生命周期

```
创建        开发        合并        清理
tnr/tn  →  编码  →    tmg    →  (自动删除)
          ↓
       fixup/amend
       save/snapshot
```

---

## 1. topic-new-remote (tnr)

### 功能
创建新的 topic 分支并推送到远程，适用于需要团队协作的功能开发。

### 语法
```bash
git tnr <topic-branch> [base-branch]
```

### 参数
- `topic-branch` (必需): 新分支名称，推荐命名规范：
  - `feature/user-auth` - 新功能
  - `hotfix/login-bug` - 紧急修复
  - `refactor/api-cleanup` - 重构
- `base-branch` (可选): 基础分支，默认从 `git base-branch` 读取（通常是 main）

### 执行流程

```bash
1. 接受参数 topic_branch 和 base_branch
2. 检查远程是否存在 base_branch
3. 检查当前工作区是否干净
   ↓ 如果脏 →  自动执行 git save "Autosave on topic-new-remote: $topic_branch"
4. 如果 base_branch 在远程存在且本地落后
   ↓ 执行 git pull origin $base_branch
5. 从 base_branch 创建 topic_branch
6. 推送到远程并设置追踪
   ↓ git push -u origin $topic_branch
```

### 使用场景

**场景 1：标准功能开发**
```bash
# 从 main 创建功能分支并推送
git tnr feature/user-auth

# 等价于：
# git checkout -b feature/user-auth main
# git push -u origin feature/user-auth
```

**场景 2：指定不同的 base 分支**
```bash
# 从 develop 分支创建
git tnr feature/new-api develop
```

**场景 3：工作区有未提交修改**
```bash
# 当前在 main 分支，有未提交修改
git s
# M  src/app.js
# ?? src/temp.js

git tnr feature/cleanup
# → 输出：============= Autosaving current working directory in branch: main ============
# → 自动 stash
# → 创建分支
# → 推送到远程
```

### 安全机制

1. **自动保存工作进度**：工作区脏时自动 `git save`，不会丢失修改
2. **自动同步 base**：base 分支落后时自动 pull，避免基于过时代码开发
3. **设置远程追踪**：`-u` 参数让本地分支追踪远程，后续 `git ps`/`git pl` 无需指定分支名

### 注意事项

- ⚠ 需要网络连接（会推送到远程）
- ⚠ 如果远程已存在同名分支，会失败（Git 保护机制）
- ✅ 自动 stash 的内容可通过 `git pop` 或 `git stash list` 找回

---

## 2. topic-new-local (tn)

### 功能
创建仅本地的 topic 分支，适用于实验性开发、本地重构等不需要立即推送的场景。

### 语法
```bash
git tn <topic-branch> [base-branch]
```

### 参数
同 `tnr`。

### 执行流程

```bash
1. 接受参数 topic_branch 和 base_branch
2. 检查当前工作区是否干净
   ↓ 如果脏 → 自动执行 git save "Autosave on topic-new-local: $topic_branch"
3. 从 base_branch 创建 topic_branch
   ↓ git checkout -b $topic_branch $base_branch
```

### 使用场景

**场景 1：实验性开发**
```bash
# 尝试新的实现方案，不确定是否会采用
git tn experiment/new-algorithm
```

**场景 2：本地重构**
```bash
# 重构代码，确认无问题后再推送
git tn refactor/cleanup-utils
# ... 重构 ...
# ... 测试通过 ...
git publish  # 推送并设置追踪（如果需要）
```

**场景 3：临时分支**
```bash
# 临时尝试某个 idea
git tn temp/try-websocket
```

### tnr vs tn 决策树

```
需要团队协作？
├─ 是 → git tnr (推送到远程)
└─ 否 → git tn (仅本地)
    ├─ 稍后需要推送？
    │  └─ git publish (追溯推送)
    └─ 实验失败？
       └─ git branch -D experiment/X (直接删除)
```

### 优势

- ✅ 快速（无网络操作）
- ✅ 灵活（可随意删除而不影响远程）
- ✅ 安全（实验性修改不会污染远程仓库）

---

## 3. topic-merge (tmg)

### 功能
将当前 topic 分支合并到 base 分支，使用 `--no-ff`（no fast-forward）保留分支历史，合并后自动删除 topic 分支。

### 语法
```bash
git tmg [base-branch]
```

### 参数
- `base-branch` (可选): 目标分支，默认从 `git base-branch` 读取

### 执行流程

```bash
1. 获取当前分支名 topic_branch
2. 获取目标分支 base_branch
3. 安全检查
   ├─ 当前分支是 main/base？→ 拒绝执行，输出错误提示
   └─ 工作区脏？→ 拒绝执行，输出错误提示
4. 切换到 base_branch
   ↓ git checkout $base_branch
5. 合并 topic_branch（强制创建 merge commit）
   ↓ git merge --no-ff $topic_branch
6. 删除 topic_branch
   ↓ git topic-delete $topic_branch
```

### --no-ff 的意义

**使用 --no-ff (推荐)**：
```
* commit M (merge commit)
|\
| * commit C (topic 分支)
| * commit B
|/
* commit A (base 分支)
```
- ✅ 保留分支历史，可追溯完整的开发过程
- ✅ merge commit 作为功能完成的标记
- ✅ 可通过 `git revert M` 快速撤销整个功能

**不使用 --no-ff (fast-forward)**：
```
* commit C (看起来像在 base 上直接提交)
* commit B
* commit A
```
- ❌ 丢失分支历史，无法区分哪些 commit 属于同一功能
- ❌ 难以撤销整个功能（需要逐个 revert）

### 使用场景

**场景 1：功能开发完成**
```bash
# 当前在 feature/user-auth 分支
git blg           # 确认要合并的 commits
git bdf           # 查看全部差异
git tmg           # 合并到 base 分支并删除 feature/user-auth
```

**场景 2：指定目标分支**
```bash
# 合并到 develop 分支（而非默认的 main）
git tmg develop
```

### 安全机制

1. **防止误操作**：不能在 main/base 分支上执行（防止"merge main to main"）
2. **工作区检查**：工作区必须干净（防止混入未提交的修改）
3. **自动清理**：merge 后立即删除分支（减少分支混乱）

### 为何 merge 后立即删除分支？

这是**安全的**，因为：
1. **merge commit 记录了分支历史**：可通过 `git log --graph` 看到分支结构
2. **可随时恢复分支**：
   ```bash
   # 找到 merge commit
   git log --oneline --grep="Merge branch 'feature/user-auth'"

   # 恢复分支（指向 merge 前的最后一个 commit）
   git zb feature/user-auth <commit-before-merge>
   ```
3. **减少分支污染**：已合并的分支留着没有价值，反而增加 `git branch` 的噪音

### 注意事项

- ⚠ 合并前确保测试通过、代码审查完成
- ⚠ 如果有冲突，`git merge` 会暂停，进入冲突解决流程
- ✅ 合并前建议 `git snapshot "before-merge-feature-X"` 保险备份

---

## 4. topic-delete (td)

### 功能
智能删除 topic 分支，自动处理远程同步，删除本地和远程分支。

### 语法
```bash
git td [topic-branch]
```

### 参数
- `topic-branch` (可选): 要删除的分支名，默认为当前分支

### 执行流程

```bash
1. 获取要删除的分支 topic_branch（默认当前分支）
2. 获取 base_branch
3. 安全检查
   ├─ topic_branch 是 main/base？→ 拒绝删除
   └─ 工作区脏？→ 拒绝删除
4. 检查远程是否存在 topic_branch
5. 如果远程存在：
   ├─ 本地落后远程？→ git pull origin $topic_branch
   ├─ 本地领先远程？→ git push origin $topic_branch
   └─ 删除远程分支 → git push origin --delete $topic_branch
6. 清理本地追踪引用
   ↓ git remote-prune-all
7. 切换到 base_branch
   ↓ git checkout $base_branch
8. 删除本地分支
   ↓ git branch --delete $topic_branch
```

### 智能同步逻辑

**场景 1：本地领先远程（有未推送 commits）**
```bash
git td
# → 自动 push
# → 删除远程分支
# → 删除本地分支
```

**场景 2：本地落后远程（协作者推送了新 commits）**
```bash
git td
# → 自动 pull
# → 删除远程分支
# → 删除本地分支
```

**场景 3：本地和远程同步**
```bash
git td
# → 直接删除远程分支
# → 删除本地分支
```

**场景 4：仅本地分支（远程不存在）**
```bash
git td
# → 跳过远程操作
# → 删除本地分支
```

### 使用场景

**场景 1：清理已合并的分支**
```bash
# 在 feature/user-auth 上
git tmg           # 合并到 main
# → 已自动删除 feature/user-auth

# 或手动删除其他已合并分支
git td feature/old-api
```

**场景 2：放弃未完成的功能**
```bash
# 决定不继续开发某个功能
git td experiment/failed-approach
```

**场景 3：删除指定分支（不切换）**
```bash
# 当前在 main，删除 feature/X
git td feature/X
```

### 安全机制

1. **分支保护**：不能删除 main/dev/base-branch
2. **工作区检查**：工作区脏时拒绝操作（防止丢失修改）
3. **自动同步**：确保本地和远程一致后再删除
4. **追踪清理**：删除远程分支后自动清理本地追踪引用

### 注意事项

- ⚠ 删除远程分支需要权限（某些仓库可能受保护）
- ⚠ 删除后无法直接恢复远程分支（但本地可通过 reflog 恢复）
- ✅ 如果误删，可通过 `git reflog` 找回：
  ```bash
  git reflog
  # 找到分支的最后一个 commit
  git checkout -b recovered-branch <commit-hash>
  ```

---

## 状态检查命令

这些命令是 topic 工作流的基础，用于安全检查和状态分析。

### working-dir-dirty

**功能**：检查工作区是否有未提交的修改。

**语法**：
```bash
git working-dir-dirty
```

**返回值**：
- 工作区干净：空字符串（长度为 0）
- 工作区脏：非空字符串（包含修改摘要）

**实现**：
```bash
git diff --stat | head -n -1
```

**使用**：
```bash
if [ -n "$(git working-dir-dirty)" ]; then
    echo "工作区有未提交修改"
    git save "Autosave"
fi
```

---

### current-branch

**功能**：获取当前分支名。

**语法**：
```bash
git current-branch
```

**返回值**：当前分支名（如 `feature/user-auth`、`main`）

**实现**：
```bash
git rev-parse --abbrev-ref HEAD
```

---

### base-branch

**功能**：获取配置的 base 分支名。

**语法**：
```bash
git base-branch  # 或 git bb
```

**返回值**：base 分支名（本地配置 > 全局配置 > 默认值 `main`）

**配置**：
```bash
# 设置当前仓库的 base 分支
git set-bb develop

# 设置全局 base 分支
git set-global-base-branch main
```

**实现**：
```bash
git config --get gitalias.topic.base.branch.name || printf '%s\n' main
```

---

### remote-branch

**功能**：检查本地分支在远程是否存在。

**语法**：
```bash
git remote-branch [local-branch]
```

**参数**：
- `local-branch` (可选): 本地分支名，默认当前分支

**返回值**：
- 存在：远程分支名（如 `feature/user-auth`）
- 不存在：空字符串

**实现**：
```bash
current_branch=${1:-$(git current-branch)}
git branch -r | awk '{print $1}' | awk -F '/' '{if($2~/'$current_branch'/)print $2}'
```

---

### ahead-count / behind-count

**功能**：计算本地分支与远程分支的领先/落后 commit 数。

**语法**：
```bash
git ahead-count [local-branch]
git behind-count [local-branch]
```

**参数**：
- `local-branch` (可选): 本地分支名，默认当前分支

**返回值**：整数（commit 数量）

**实现**：
```bash
# 领先数量
git rev-list --count origin/$local_branch..$local_branch

# 落后数量
git rev-list --count $local_branch..origin/$local_branch
```

**使用场景**：
```bash
# 检查是否需要 push
if [ $(git ahead-count) -gt 0 ]; then
    echo "有 $(git ahead-count) 个 commits 未推送"
fi

# 检查是否需要 pull
if [ $(git behind-count) -gt 0 ]; then
    echo "落后远程 $(git behind-count) 个 commits"
fi
```

---

### oldest-changeable-commit

**功能**：计算最远的可以安全修改的 commit（用于 fixup/rebase）。

**语法**：
```bash
git oldest-changeable-commit [base-branch]
```

**逻辑**：
1. 如果当前分支有远程追踪 → 返回 `origin/current_branch`（不能改已推送的）
2. 否则如果 base 分支有远程追踪 → 返回 `origin/base_branch`（不能改 base 的已推送部分）
3. 否则 → 返回与 base 的公共祖先（可以改到分叉点）

**实现**：
```bash
common_ancestor=$(git merge-base $base_branch $current_branch)
current_exists_in_remote=$(git remote-branch)
base_exists_in_remote=$(git remote-branch $base_branch)

if [ -n "$current_exists_in_remote" ]; then
    oldest_commit=origin/$current_branch
elif [ -n "$base_exists_in_remote" ]; then
    oldest_commit=origin/$base_branch
else
    oldest_commit=$common_ancestor
fi
```

---

## 典型工作流示例

### 完整功能开发流程

```bash
# 1. 创建功能分支
git tnr feature/user-auth
# → 自动从 main 创建并推送

# 2. 开发过程中
git save "WIP: implementing login"  # 临时保存
git cmu                             # 修改上次 commit（已追踪文件）
git cma                             # 修改上次 commit（所有文件）

# 3. 查看进度
git blg                             # 查看 topic 分支的 commits
git bdf                             # 查看与 main 的全部差异
git ahead                           # 查看未推送的 commits

# 4. 修改历史（如果需要）
git fixup                           # 交互式选择要修改的 commit

# 5. 合并前准备
git zz && git pl                    # 同步 base 分支
git z -                             # 切回 topic 分支
git blg                             # 最后确认
git bdf | less                      # 查看完整差异

# 6. 合并
git tmg
# → 自动 merge --no-ff
# → 自动删除 feature/user-auth
```

### 实验性开发流程

```bash
# 1. 创建本地实验分支
git tn experiment/new-approach

# 2. 尝试新实现
# ... 编码 ...

# 3. 失败了？直接删除
git z main
git branch -D experiment/new-approach

# 4. 成功了？推送并合并
git publish                         # 推送到远程
git tmg
```

### 协作开发流程

```bash
# 1. 拉取队友的分支
git fetch
git checkout feature/shared-work

# 2. 协作开发
# ... 编码 ...
git ps                              # 推送自己的 commits

# 3. 同步队友的更新
git pl                              # 拉取队友的新 commits

# 4. 合并前最后同步
git ps                              # 确保自己的修改已推送
git tmg                             # 合并
```

---

## 最佳实践

### 1. 分支命名规范

```
feature/    - 新功能
hotfix/     - 紧急修复
bugfix/     - 常规 bug 修复
refactor/   - 重构
experiment/ - 实验性修改
```

### 2. Commit 消息规范

```
feat: 添加用户登录功能
fix: 修复登录页面样式错误
refactor: 重构 API 请求逻辑
docs: 更新 README
test: 添加登录功能测试
```

### 3. 操作前检查

```bash
# 始终先确认状态
git s               # 工作区状态
git bb              # base 分支
git current-branch  # 当前分支
```

### 4. 重要操作前快照

```bash
git snapshot "before-merge-feature-X"
git tmg
# 如果出错：
git stash apply stash@{0}  # 恢复快照
```

### 5. 定期清理

```bash
# 每周清理已合并的分支
git branch-clean-local
```

---

## 故障排查

### 问题 1：tnr 提示"远程已存在同名分支"

**原因**：之前创建过同名分支但未删除。

**解决**：
```bash
# 方案 1：使用不同的分支名
git tnr feature/user-auth-v2

# 方案 2：删除远程旧分支（如果确认不需要）
git push origin --delete feature/user-auth
git tnr feature/user-auth
```

### 问题 2：tmg 提示"工作区脏"

**原因**：有未提交的修改。

**解决**：
```bash
# 方案 1：提交修改
git add -A && git commit -m "..."

# 方案 2：临时保存
git save "WIP before merge"
git tmg
git pop  # 恢复修改
```

### 问题 3：td 提示"权限不足，无法删除远程分支"

**原因**：远程分支受保护或无删除权限。

**解决**：
```bash
# 只删除本地分支
git checkout main
git branch -d feature/X

# 或请求仓库管理员删除远程分支
```

### 问题 4：误删分支

**解决**：
```bash
# 找到分支最后的 commit
git reflog | grep "feature/user-auth"

# 恢复分支
git checkout -b feature/user-auth <commit-hash>

# 如果需要恢复远程分支
git push -u origin feature/user-auth
```

---

## 参考资料

- Git Aliases 配置：[aliases.gitconfig](https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig)
- Git Aliases 参考手册：[Git-Aliases-Reference-Manual.md](https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md)
- Feature Branch Workflow：https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
