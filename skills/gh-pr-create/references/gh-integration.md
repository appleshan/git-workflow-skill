# GitHub CLI Integration Guide

GitHub CLI (`gh`) 集成详解，包括安装、认证、命令详解和故障排查。

## GitHub CLI Overview

### 什么是 gh CLI

GitHub CLI 是 GitHub 官方的命令行工具，提供：
- PR/Issue 管理
- Workflow/Release 操作
- Repository 信息查询
- GraphQL API 调用

**优势**:
- 🔐 **认证集成**: 复用 GitHub token，无需手动管理
- 🚀 **高级功能**: 支持 draft PR、reviewer 指定、label 管理
- 📝 **格式化输出**: JSON/YAML 输出，易于脚本处理
- 🔄 **自动补全**: Shell 补全支持（bash/zsh/fish）

### 安装方式

#### macOS

```bash
# Homebrew
brew install gh

# MacPorts
sudo port install gh
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt install gh

# Fedora/RHEL
sudo dnf install gh

# Arch Linux
sudo pacman -S github-cli

# 通用方式（从源安装）
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

#### Windows

```powershell
# Winget
winget install GitHub.cli

# Chocolatey
choco install gh

# Scoop
scoop install gh
```

### 版本检查

```bash
gh --version
# 输出示例: gh version 2.40.0 (2024-01-15)
```

**最低版本要求**: `gh` v2.0+（支持 `gh pr create` 的完整功能）

## Authentication

### 认证流程

#### 交互式认证（推荐）

```bash
gh auth login

# 交互式提示:
? What account do you want to log into?
  > GitHub.com
    GitHub Enterprise Server

? What is your preferred protocol for Git operations?
  > HTTPS
    SSH

? Authenticate Git with your GitHub credentials? (Y/n)
  > Y

? How would you like to authenticate GitHub CLI?
  > Login with a web browser
    Paste an authentication token
```

**浏览器认证流程**:
1. CLI 显示一次性验证码（例如: `ABCD-1234`）
2. 自动打开浏览器到 `https://github.com/login/device`
3. 输入验证码并授权
4. CLI 自动完成认证

#### Token 认证（CI/CD）

```bash
# 使用 Personal Access Token
echo $GITHUB_TOKEN | gh auth login --with-token

# 或通过环境变量
export GH_TOKEN="ghp_xxxxxxxxxxxx"
gh auth status  # 自动使用 GH_TOKEN
```

**Token 权限要求**（创建 PR 所需）:
- `repo` (full control)
- `workflow` (如果修改 GitHub Actions)

### 认证状态检查

```bash
# 检查当前认证状态
gh auth status

# 输出示例（已认证）:
github.com
  ✓ Logged in to github.com as username (keyring)
  ✓ Git operations for github.com configured to use https protocol.
  ✓ Token: ghp_************************************
  ✓ Token scopes: gist, read:org, repo, workflow
```

**故障场景**:

```bash
# 未认证
gh auth status
# 输出: You are not logged into any GitHub hosts. Run gh auth login to authenticate.

# Token 过期
gh auth status
# 输出: ✗ authentication token: API rate limit exceeded (token may be expired)
```

### 多账号管理

```bash
# 切换到不同的 GitHub host
gh auth login -h github.com
gh auth login -h ghe.company.com

# 查看所有已登录账号
gh auth status --show-token

# 登出特定账号
gh auth logout -h github.com
```

## Core Commands for PR Creation

### 1. `gh pr create`

**基本语法**:

```bash
gh pr create \
  --title "PR 标题" \
  --body "PR 描述" \
  [OPTIONS]
```

**常用选项**:

| 选项 | 说明 | 示例 |
|------|------|------|
| `--title`, `-t` | PR 标题 | `-t "Fix login bug"` |
| `--body`, `-b` | PR 描述 | `-b "Fixes #123"` |
| `--base`, `-B` | Base branch | `-B main` |
| `--head`, `-H` | Head branch | `-H feature/auth` |
| `--draft`, `-d` | 创建草稿 PR | `-d` |
| `--reviewer`, `-r` | 指定审核者 | `-r user1,user2` |
| `--assignee`, `-a` | 指定负责人 | `-a @me` |
| `--label`, `-l` | 添加标签 | `-l bug,urgent` |
| `--milestone`, `-m` | 指定里程碑 | `-m v1.2.0` |
| `--project`, `-p` | 关联项目 | `-p "Q1 Roadmap"` |
| `--web`, `-w` | 在浏览器中打开 | `-w` |

**HEREDOC 格式化**（推荐，避免引号问题）:

```bash
gh pr create \
  --title "Feature: User Authentication" \
  --base main \
  --body "$(cat <<'EOF'
#### Summary
- [feat] Implement OAuth 2.0 login
- [test] Add auth module tests

#### Test plan
✓ Unit tests pass
✓ Integration tests pass

🤖 Generated with Claude Code
EOF
)"
```

### 2. `gh pr view`

**查看 PR 详情**:

```bash
# 查看当前分支的 PR
gh pr view

# 查看指定 PR 编号
gh pr view 123

# JSON 格式输出
gh pr view --json url,title,state,author

# 提取特定字段（使用 jq）
gh pr view --json url -q .url
# 输出: https://github.com/owner/repo/pull/123
```

### 3. `gh pr edit`

**编辑已创建的 PR**:

```bash
# 修改标题
gh pr edit 123 --title "New title"

# 修改描述
gh pr edit 123 --body "$(cat updated-description.md)"

# 添加 reviewer
gh pr edit 123 --add-reviewer user1,user2

# 添加 label
gh pr edit 123 --add-label "needs-review"
```

### 4. `gh pr list`

**列出仓库的 PR**:

```bash
# 列出所有 open PR
gh pr list

# 列出我创建的 PR
gh pr list --author @me

# 列出特定 label 的 PR
gh pr list --label bug

# JSON 格式输出
gh pr list --json number,title,url,state
```

### 5. `gh repo view`

**获取仓库信息**（用于 base branch 识别）:

```bash
# 获取默认分支名称
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
# 输出: main

# 获取仓库完整信息
gh repo view --json name,owner,defaultBranchRef,isPrivate
```

## Integration with Git Commands

### 工作流集成

**典型流程**（Skill 自动执行）:

```bash
# 1. 检查认证
gh auth status || { echo "未认证"; exit 1; }

# 2. 确定 base branch
BASE=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)

# 3. 检查当前分支
CURRENT=$(git branch --show-current)
[[ "$CURRENT" == "$BASE" ]] && { echo "不能从主分支创建 PR"; exit 1; }

# 4. 检查 remote 跟踪
if ! git rev-parse --abbrev-ref @{u} &>/dev/null; then
    echo "推送分支到 remote..."
    git push -u origin "$CURRENT"
fi

# 5. 收集信息（并行）
git log --oneline "$BASE..HEAD" > /tmp/commits.txt &
git diff "$BASE...HEAD" --stat > /tmp/diff.txt &
wait

# 6. 创建 PR
gh pr create \
    --title "Auto-generated PR" \
    --base "$BASE" \
    --body "$(generate_pr_description)"  # 调用智能生成

# 7. 获取 PR URL
PR_URL=$(gh pr view --json url -q .url)
echo "✅ PR 创建成功: $PR_URL"
```

### 错误处理

```bash
# 捕获 gh 命令错误
if ! gh pr create --title "Test" --body "Test" 2>&1 | tee /tmp/gh_error.log; then
    ERROR=$(cat /tmp/gh_error.log)

    case "$ERROR" in
        *"already exists"*)
            echo "PR 已存在，使用 gh pr view 查看"
            gh pr view
            ;;
        *"authentication"*)
            echo "认证失败，请运行: gh auth login"
            ;;
        *"not found"*)
            echo "仓库或分支未找到，检查配置"
            ;;
        *)
            echo "未知错误: $ERROR"
            ;;
    esac
fi
```

## Advanced Usage

### 1. Draft PR 工作流

**创建草稿 PR**:

```bash
gh pr create --draft --title "WIP: Feature X" --body "..."
```

**Ready for review**:

```bash
# 将草稿 PR 标记为 ready
gh pr ready 123

# 或通过 edit
gh pr edit 123 --ready
```

### 2. 模板支持

**使用仓库的 PR 模板**:

```bash
# gh 自动检测 .github/pull_request_template.md
gh pr create --fill

# 或明确指定模板
gh pr create --template .github/PULL_REQUEST_TEMPLATE/feature.md
```

**Skill 集成**:

```bash
# 如果仓库有模板，优先使用
if [[ -f .github/pull_request_template.md ]]; then
    TEMPLATE=$(cat .github/pull_request_template.md)
    # 智能填充模板的占位符
    FILLED=$(fill_template "$TEMPLATE" "$COMMITS" "$CHANGES")
    gh pr create --body "$FILLED"
else
    # 使用 Skill 生成的描述
    gh pr create --body "$(generate_pr_description)"
fi
```

### 3. Fork 工作流

**跨仓库创建 PR**:

```bash
# 从 fork 创建 PR 到 upstream
gh pr create \
    --repo upstream/repo \
    --head myusername:feature-branch \
    --base main \
    --title "..." \
    --body "..."
```

**检测 fork 场景**:

```bash
# 检查是否有 upstream remote
if git remote | grep -q upstream; then
    UPSTREAM_REPO=$(gh repo view upstream/repo --json nameWithOwner -q .nameWithOwner)
    gh pr create --repo "$UPSTREAM_REPO" --head "$(git config user.name):$CURRENT"
else
    # 常规流程
    gh pr create
fi
```

### 4. 批量操作

**为多个 PR 添加 label**:

```bash
# 获取所有 open PR
PRS=$(gh pr list --json number -q '.[].number')

for pr in $PRS; do
    gh pr edit "$pr" --add-label "needs-triage"
done
```

## Troubleshooting

### 问题 1: `gh: command not found`

**原因**: gh CLI 未安装或不在 PATH 中

**解决方案**:

```bash
# 检查安装
which gh

# 如未安装，参考上面的安装方式

# 如已安装但不在 PATH，添加到 shell 配置
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 问题 2: `gh: To get started with GitHub CLI, please run: gh auth login`

**原因**: 未认证或 token 过期

**解决方案**:

```bash
# 重新认证
gh auth login

# 或使用 token
export GH_TOKEN="ghp_xxxx"
gh auth status
```

### 问题 3: `GraphQL: A pull request already exists for...`

**原因**: 当前分支已有 PR

**解决方案**:

```bash
# 查看现有 PR
gh pr view

# 如需更新 PR 描述
gh pr edit --body "$(cat new-description.md)"

# 如需推送新 commits，直接 push
git push
```

### 问题 4: `GraphQL: Could not resolve to a PullRequestable with the name of...`

**原因**: Base branch 不存在

**解决方案**:

```bash
# 检查默认分支
gh repo view --json defaultBranchRef -q .defaultBranchRef.name

# 明确指定正确的 base branch
gh pr create --base main  # 或 master
```

### 问题 5: `API rate limit exceeded`

**原因**: 未认证或 token 权限不足

**解决方案**:

```bash
# 检查 rate limit 状态
gh api rate_limit

# 输出示例:
# {
#   "resources": {
#     "core": {
#       "limit": 5000,
#       "remaining": 4999,
#       "reset": 1234567890
#     }
#   }
# }

# 认证后 rate limit 提升（60 → 5000/hour）
gh auth login
```

### 问题 6: `fatal: No upstream configured for branch 'feature-x'`

**原因**: 分支未推送到 remote

**自动处理**（Skill 内置）:

```bash
# Skill 检测到此情况时自动执行
git push -u origin "$(git branch --show-current)"

# 然后继续创建 PR
gh pr create ...
```

### 问题 7: `HEREDOC 格式错误`

**原因**: 引号嵌套或特殊字符问题

**正确格式**:

```bash
# ✅ 正确：使用单引号包围 EOF
gh pr create --body "$(cat <<'EOF'
Content with "quotes" and $variables
EOF
)"

# ❌ 错误：双引号会导致变量展开
gh pr create --body "$(cat <<"EOF"
Content with $HOME  # 会被展开为实际路径
EOF
)"
```

## Security Considerations

### Token 管理

**最佳实践**:

1. **不要硬编码 token**:
   ```bash
   # ❌ 错误
   gh auth login --with-token ghp_xxxx

   # ✅ 正确
   echo "$GITHUB_TOKEN" | gh auth login --with-token
   ```

2. **使用 keyring 存储**（默认行为）:
   ```bash
   gh auth login  # 自动使用系统 keyring
   gh auth status  # 显示 "(keyring)"
   ```

3. **CI/CD 环境使用环境变量**:
   ```yaml
   # .github/workflows/pr.yml
   env:
     GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # 自动注入
   steps:
     - run: gh pr create ...
   ```

### 权限最小化

**创建 PR 的最小权限**:

```json
{
  "scopes": [
    "repo",          // 必需：创建 PR
    "read:org"       // 可选：组织成员验证
  ]
}
```

**避免过度权限**:
- ❌ 不需要 `admin:org`
- ❌ 不需要 `delete_repo`
- ✅ 只授予 `repo` 和 `workflow`（如果修改 Actions）

## Performance Optimization

### 1. 减少 API 调用

**批量查询**（使用 GraphQL）:

```bash
# ❌ 低效：多次 REST API 调用
gh pr view 123 --json url
gh pr view 123 --json title
gh pr view 123 --json author

# ✅ 高效：单次 GraphQL 查询
gh pr view 123 --json url,title,author
```

### 2. 缓存仓库信息

```bash
# 缓存默认分支（避免重复查询）
if [[ ! -f /tmp/gh_base_branch.cache ]]; then
    gh repo view --json defaultBranchRef -q .defaultBranchRef.name > /tmp/gh_base_branch.cache
fi
BASE=$(cat /tmp/gh_base_branch.cache)
```

### 3. 并行执行

```bash
# 并行执行独立的 gh 命令
gh pr list --json number,title > /tmp/pr_list.json &
gh repo view --json defaultBranchRef > /tmp/repo_info.json &
wait

# 使用结果
PRS=$(cat /tmp/pr_list.json)
REPO_INFO=$(cat /tmp/repo_info.json)
```

## Best Practices

### 1. 命令幂等性

```bash
# 检查 PR 是否已存在
if gh pr view &>/dev/null; then
    echo "PR 已存在，更新描述"
    gh pr edit --body "$(generate_pr_description)"
else
    echo "创建新 PR"
    gh pr create --body "$(generate_pr_description)"
fi
```

### 2. 错误日志记录

```bash
# 记录所有 gh 命令输出
GH_LOG=/tmp/gh_pr_create.log

gh pr create ... 2>&1 | tee -a "$GH_LOG"

if [[ ${PIPESTATUS[0]} -ne 0 ]]; then
    echo "PR 创建失败，日志: $GH_LOG"
    cat "$GH_LOG"
fi
```

### 3. 用户友好的错误提示

```bash
# 友好提示而非原始错误
if ! gh auth status &>/dev/null; then
    cat <<EOF
❌ GitHub CLI 未认证

请运行以下命令完成认证:
  gh auth login

认证后可以:
  ✓ 创建和管理 Pull Requests
  ✓ 自动配置 Git 凭据
  ✓ 使用 GitHub API
EOF
    exit 1
fi
```

## Future Enhancements

1. **gh extensions**: 考虑开发 `gh-pr-smart` extension
2. **GraphQL 直接调用**: 绕过 `gh pr create`，完全自定义
3. **AI 驱动的 reviewer 推荐**: 基于文件变更建议 reviewer
4. **自动化 PR checklist**: 根据变更类型生成 checkbox
5. **PR 模板智能填充**: 解析模板占位符自动填充
