# Base Branch Detection Algorithm

Base branch 智能识别算法，支持多种配置方式和自动回退机制。

## Problem Statement

创建 PR 时需要明确 **base branch**（目标分支），但用户通常不会明确指定，需要智能识别：

**挑战**:
1. 不同仓库使用不同的默认分支（`main` vs `master` vs `develop`）
2. 单体仓库 vs monorepo（可能有多个主分支）
3. Fork 工作流（需要识别 upstream）
4. Release 分支工作流（可能 base 是 `release/*` 而非 `main`）

**目标**: 自动且准确地识别 base branch，减少用户手动输入。

## Detection Strategy

### 优先级策略（从高到低）

```
1. 用户明确指定（--base 参数）
   ↓ 未指定
2. Git 配置（branch.<current>.base）
   ↓ 未配置
3. GitHub 默认分支（gh repo view）
   ↓ API 失败
4. Git 配置的远程跟踪分支
   ↓ 未配置
5. 常见惯例（main → master → develop）
   ↓ 都不存在
6. 询问用户
```

### 算法伪代码

```python
def detect_base_branch(user_input: Optional[str], current_branch: str) -> str:
    # 1. 用户明确指定
    if user_input:
        validate_branch_exists(user_input)
        return user_input

    # 2. Git 配置
    base = git_config(f"branch.{current_branch}.base")
    if base:
        return base

    # 3. GitHub 默认分支
    try:
        default = gh_default_branch()
        return default
    except APIError:
        pass

    # 4. 远程跟踪分支的 base
    upstream = get_upstream_branch(current_branch)
    if upstream:
        # 例如: origin/feature-x → origin/main
        return extract_base_from_upstream(upstream)

    # 5. 常见惯例
    for candidate in ['main', 'master', 'develop', 'dev']:
        if branch_exists(candidate):
            return candidate

    # 6. 询问用户
    return ask_user_for_base_branch()
```

## Implementation Details

### 方式 1: 用户明确指定

**命令行参数**:

```bash
# Skill 接受用户输入
gh pr create --base develop

# 或通过自然语言
"创建 PR，base 是 develop 分支"
```

**验证逻辑**:

```bash
BASE="$1"  # 用户指定的 base

# 检查 base 分支是否存在
if ! git rev-parse --verify "origin/$BASE" &>/dev/null; then
    echo "❌ 错误: 分支 '$BASE' 不存在于 remote"
    echo "可用分支:"
    git branch -r | grep -v HEAD | sed 's/origin\//  - /'
    exit 1
fi
```

### 方式 2: Git 配置

**设置自定义 base branch**:

```bash
# 为当前分支配置 base
git config branch.$(git branch --show-current).base main

# 或为特定分支配置
git config branch.feature-x.base develop
```

**读取配置**:

```bash
CURRENT=$(git branch --show-current)
BASE=$(git config --get "branch.$CURRENT.base")

if [[ -n "$BASE" ]]; then
    echo "✓ 使用配置的 base branch: $BASE"
else
    echo "未配置 base branch，继续下一步检测..."
fi
```

**使用场景**:
- 长期 feature 分支（base 是固定的）
- Monorepo 中的子项目分支（例如 `frontend/main`）
- Release 分支工作流（base 是 `release/v1.2`）

### 方式 3: GitHub 默认分支

**使用 gh CLI 查询**:

```bash
# 查询仓库默认分支
DEFAULT=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)

if [[ -n "$DEFAULT" ]]; then
    echo "✓ GitHub 默认分支: $DEFAULT"
    BASE="$DEFAULT"
else
    echo "⚠️  无法查询 GitHub 默认分支（可能未认证）"
fi
```

**错误处理**:

```bash
# 处理 gh 命令失败
if ! DEFAULT=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null); then
    case "$?" in
        1) echo "gh CLI 未认证" ;;
        2) echo "网络错误" ;;
        *) echo "未知错误" ;;
    esac

    # 回退到下一个策略
    DEFAULT=""
fi
```

**优势**:
- ✅ 最准确（直接从 GitHub 获取）
- ✅ 支持自定义默认分支（非 main/master）
- ✅ 适用于 fork 场景（自动获取 upstream 默认分支）

### 方式 4: 远程跟踪分支

**提取 base 逻辑**:

```bash
# 获取当前分支的 upstream
UPSTREAM=$(git rev-parse --abbrev-ref @{u} 2>/dev/null)

if [[ -n "$UPSTREAM" ]]; then
    # 例如: origin/feature-x
    REMOTE=$(echo "$UPSTREAM" | cut -d/ -f1)  # origin
    BRANCH=$(echo "$UPSTREAM" | cut -d/ -f2-) # feature-x

    # 尝试推断 base（假设从 main 创建）
    for candidate in main master develop; do
        if git rev-parse --verify "$REMOTE/$candidate" &>/dev/null; then
            BASE="$candidate"
            echo "✓ 推断 base branch: $BASE（基于 upstream）"
            break
        fi
    done
fi
```

**局限性**:
- 仅适用于已推送的分支
- 无法处理复杂的分支拓扑（例如 `feature-x` 从 `develop` 创建，但 `develop` 从 `main` 创建）

### 方式 5: 常见惯例

**检查顺序**:

```bash
# 按使用频率排序
for candidate in main master develop dev trunk; do
    # 检查本地和 remote 分支
    if git show-ref --verify --quiet "refs/heads/$candidate" || \
       git show-ref --verify --quiet "refs/remotes/origin/$candidate"; then
        BASE="$candidate"
        echo "✓ 使用常见惯例分支: $BASE"
        break
    fi
done

if [[ -z "$BASE" ]]; then
    echo "❌ 未找到常见的 base 分支"
fi
```

**优势**:
- ✅ 无需外部依赖（纯 git 命令）
- ✅ 快速（本地检查）
- ✅ 覆盖 95% 的场景

**局限性**:
- ❌ 无法处理自定义分支名（例如 `production`）

### 方式 6: 询问用户

**交互式选择**:

```bash
echo "无法自动识别 base branch，请选择:"

# 列出所有 remote 分支
BRANCHES=($(git branch -r | grep -v HEAD | sed 's/origin\///' | sort -u))

select branch in "${BRANCHES[@]}"; do
    if [[ -n "$branch" ]]; then
        BASE="$branch"
        echo "✓ 用户选择 base branch: $BASE"
        break
    fi
done
```

**非交互式回退**（CI/CD 场景）:

```bash
if [[ -z "$BASE" ]]; then
    echo "❌ 错误: 无法识别 base branch 且无法交互式询问"
    echo "请通过以下方式之一指定 base branch:"
    echo "  1. 使用 --base 参数"
    echo "  2. 配置 git config branch.<current>.base"
    echo "  3. 确保 gh CLI 已认证（自动获取 GitHub 默认分支）"
    exit 1
fi
```

## Special Cases

### 场景 1: Fork 工作流

**检测逻辑**:

```bash
# 检查是否有 upstream remote
if git remote | grep -q upstream; then
    echo "检测到 fork 工作流"

    # Base 应该是 upstream 的默认分支
    if command -v gh &>/dev/null; then
        UPSTREAM_REPO=$(git remote get-url upstream | sed 's/.*github.com[:/]\(.*\).git/\1/')
        BASE=$(gh repo view "$UPSTREAM_REPO" --json defaultBranchRef -q .defaultBranchRef.name)
        echo "✓ Upstream base branch: $BASE"
    else
        # 回退到常见惯例
        BASE="main"
        echo "⚠️  假设 upstream base 是 'main'（gh CLI 不可用）"
    fi
else
    # 常规工作流
    BASE=$(detect_base_branch_normal)
fi
```

**PR 创建时的差异**:

```bash
if git remote | grep -q upstream; then
    # Fork: 需要指定完整的 repo 路径
    UPSTREAM_OWNER=$(git remote get-url upstream | sed 's/.*github.com[:/]\(.*\)\/.*\.git/\1/')
    UPSTREAM_REPO=$(git remote get-url upstream | sed 's/.*github.com[:/].*\/\(.*\)\.git/\1/')

    gh pr create \
        --repo "$UPSTREAM_OWNER/$UPSTREAM_REPO" \
        --base "$BASE" \
        --head "$(git config user.name):$(git branch --show-current)"
else
    # 常规: 直接创建
    gh pr create --base "$BASE"
fi
```

### 场景 2: Monorepo

**检测 monorepo**:

```bash
# 检查是否有多个顶级目录的 package.json/pyproject.toml
PROJECTS=$(find . -maxdepth 2 -name "package.json" -o -name "pyproject.toml" | wc -l)

if [[ $PROJECTS -gt 1 ]]; then
    echo "检测到 monorepo"

    # 根据当前分支路径推断 base
    CURRENT=$(git branch --show-current)

    case "$CURRENT" in
        frontend/*) BASE="frontend/main" ;;
        backend/*)  BASE="backend/main" ;;
        *)          BASE="main" ;;
    esac

    echo "✓ Monorepo base branch: $BASE"
fi
```

### 场景 3: Release 分支工作流

**检测 release 分支**:

```bash
# 检查是否从 release 分支创建
MERGE_BASE=$(git merge-base HEAD origin/main 2>/dev/null)
RELEASE_BASE=$(git merge-base HEAD origin/release/* 2>/dev/null)

if [[ -n "$RELEASE_BASE" ]] && [[ "$RELEASE_BASE" != "$MERGE_BASE" ]]; then
    # 当前分支更接近 release 分支
    RELEASE_BRANCH=$(git branch -r --contains "$RELEASE_BASE" | grep release | head -1 | sed 's/origin\///')
    BASE="$RELEASE_BRANCH"
    echo "✓ 检测到 release 工作流，base: $BASE"
else
    BASE="main"
fi
```

### 场景 4: 多 Remote

**优先级**:

```bash
# 检查所有 remotes
REMOTES=($(git remote))

# 优先级: upstream > origin > 其他
for remote in upstream origin "${REMOTES[@]}"; do
    if git remote | grep -q "^$remote$"; then
        DEFAULT=$(gh repo view "$(git remote get-url $remote)" --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null)
        if [[ -n "$DEFAULT" ]]; then
            BASE="$DEFAULT"
            echo "✓ Base branch from $remote: $BASE"
            break
        fi
    fi
done
```

## Validation and Safety

### 验证 Base Branch 存在性

```bash
validate_base_branch() {
    local base=$1

    # 检查本地分支
    if git show-ref --verify --quiet "refs/heads/$base"; then
        return 0
    fi

    # 检查 remote 分支
    if git show-ref --verify --quiet "refs/remotes/origin/$base"; then
        return 0
    fi

    echo "❌ 错误: Base branch '$base' 不存在"
    return 1
}

# 使用
if ! validate_base_branch "$BASE"; then
    exit 1
fi
```

### 防止常见错误

**错误 1: Base 和 Current 相同**:

```bash
CURRENT=$(git branch --show-current)

if [[ "$BASE" == "$CURRENT" ]]; then
    echo "❌ 错误: Base branch 不能与当前分支相同"
    echo "当前分支: $CURRENT"
    echo "建议: 切换到 feature 分支后再创建 PR"
    exit 1
fi
```

**错误 2: 从主分支创建 PR**:

```bash
MAIN_BRANCHES=("main" "master" "develop" "production")

if [[ " ${MAIN_BRANCHES[@]} " =~ " ${CURRENT} " ]]; then
    echo "⚠️  警告: 当前在主分支 '$CURRENT'"
    echo "通常不应从主分支创建 PR"
    read -p "确认继续？[y/N] " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi
```

**错误 3: Base 分支落后**:

```bash
# 检查 base 分支是否是最新的
git fetch origin "$BASE" &>/dev/null

LOCAL_BASE=$(git rev-parse "refs/heads/$BASE" 2>/dev/null || echo "")
REMOTE_BASE=$(git rev-parse "refs/remotes/origin/$BASE" 2>/dev/null || echo "")

if [[ -n "$LOCAL_BASE" ]] && [[ -n "$REMOTE_BASE" ]] && [[ "$LOCAL_BASE" != "$REMOTE_BASE" ]]; then
    echo "⚠️  警告: 本地 '$BASE' 分支与 remote 不同步"
    echo "建议运行: git pull origin $BASE"
fi
```

## Performance Considerations

### 缓存策略

**缓存 GitHub 默认分支**（减少 API 调用）:

```bash
CACHE_DIR="$HOME/.cache/gh-pr-create"
CACHE_FILE="$CACHE_DIR/$(git config --get remote.origin.url | md5sum | cut -d' ' -f1).base"

mkdir -p "$CACHE_DIR"

# 读取缓存（24 小时有效）
if [[ -f "$CACHE_FILE" ]] && [[ $(($(date +%s) - $(stat -c %Y "$CACHE_FILE"))) -lt 86400 ]]; then
    BASE=$(cat "$CACHE_FILE")
    echo "✓ 使用缓存的 base branch: $BASE"
else
    # 查询并缓存
    BASE=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)
    echo "$BASE" > "$CACHE_FILE"
    echo "✓ 缓存 base branch: $BASE"
fi
```

### 并行检测

```bash
# 并行执行多个检测方式
{
    gh repo view --json defaultBranchRef -q .defaultBranchRef.name > /tmp/gh_base.txt 2>/dev/null &
    PID_GH=$!

    git config --get "branch.$(git branch --show-current).base" > /tmp/git_base.txt 2>/dev/null &
    PID_GIT=$!

    # 等待任意一个完成
    while kill -0 $PID_GH 2>/dev/null || kill -0 $PID_GIT 2>/dev/null; do
        if [[ -s /tmp/gh_base.txt ]]; then
            BASE=$(cat /tmp/gh_base.txt)
            kill $PID_GIT 2>/dev/null
            break
        elif [[ -s /tmp/git_base.txt ]]; then
            BASE=$(cat /tmp/git_base.txt)
            kill $PID_GH 2>/dev/null
            break
        fi
        sleep 0.1
    done
}

echo "✓ 检测到 base branch: $BASE"
```

## Configuration Reference

### 推荐配置方式

**全局默认**（所有仓库）:

```bash
# 不推荐，因为不同仓库可能使用不同的默认分支
```

**仓库级默认**:

```bash
# 为整个仓库设置默认 base（适用于非标准分支名）
git config pr.defaultBase develop
```

**分支级配置**:

```bash
# 为特定分支设置 base（最灵活）
git config branch.feature-x.base develop
git config branch.hotfix-123.base release/v1.2
```

### 读取优先级

```bash
read_config_base() {
    local current=$1

    # 1. 分支级配置
    local branch_base=$(git config --get "branch.$current.base")
    if [[ -n "$branch_base" ]]; then
        echo "$branch_base"
        return
    fi

    # 2. 仓库级配置
    local repo_base=$(git config --get pr.defaultBase)
    if [[ -n "$repo_base" ]]; then
        echo "$repo_base"
        return
    fi

    # 3. 无配置
    echo ""
}
```

## Testing and Debugging

### 测试不同场景

```bash
# 测试脚本
test_base_detection() {
    local scenario=$1

    case "$scenario" in
        "normal")
            # 常规仓库，main 分支
            ;;
        "master")
            # 使用 master 的旧仓库
            ;;
        "fork")
            # Fork 工作流
            git remote add upstream https://github.com/original/repo.git
            ;;
        "monorepo")
            # Monorepo，多个主分支
            git checkout -b frontend/feature-x
            ;;
        "release")
            # Release 分支工作流
            git checkout -b hotfix/123 origin/release/v1.2
            ;;
    esac

    # 运行检测
    BASE=$(detect_base_branch)
    echo "Scenario: $scenario -> Base: $BASE"
}

# 运行所有测试
for scenario in normal master fork monorepo release; do
    test_base_detection "$scenario"
done
```

### Debug 模式

```bash
# 启用详细输出
export GH_PR_DEBUG=1

detect_base_branch_debug() {
    [[ "$GH_PR_DEBUG" == "1" ]] && set -x

    echo "=== Base Branch Detection Debug ==="

    echo "1. 用户输入: ${USER_BASE:-<none>}"

    echo "2. Git 配置:"
    git config --get "branch.$(git branch --show-current).base" || echo "  <not set>"

    echo "3. GitHub 默认分支:"
    gh repo view --json defaultBranchRef -q .defaultBranchRef.name || echo "  <API error>"

    echo "4. 常见惯例检查:"
    for branch in main master develop; do
        if git show-ref --verify --quiet "refs/heads/$branch"; then
            echo "  ✓ $branch (exists)"
        else
            echo "  ✗ $branch (not found)"
        fi
    done

    [[ "$GH_PR_DEBUG" == "1" ]] && set +x
}
```

## Best Practices

1. **优先使用 GitHub API**: 最准确，适用于标准工作流
2. **配置长期分支**: 对于复杂项目，使用 `git config branch.<name>.base`
3. **验证后再使用**: 始终验证 base 分支存在
4. **提供清晰的错误提示**: 检测失败时，给出明确的操作建议
5. **缓存结果**: 减少重复的 API 调用和 git 命令
6. **支持用户覆盖**: 始终允许用户通过 `--base` 参数覆盖自动检测

## References

- Git branching model: https://nvie.com/posts/a-successful-git-branching-model/
- GitHub flow: https://docs.github.com/en/get-started/quickstart/github-flow
- gh CLI reference: https://cli.github.com/manual/gh_pr_create
