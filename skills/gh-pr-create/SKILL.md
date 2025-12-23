# GitHub PR Create Skill

智能 GitHub Pull Request 创建助手，基于 GitHub CLI (gh) 实现自动化 PR 创建流程。

## Skill Overview

**Purpose**: 自动化 GitHub PR 创建的标准流程，包括信息收集、智能描述生成、分支推送和 PR 创建。

**核心价值**:
- ✅ 自动收集所有 commits 和变更信息（避免遗漏）
- ✅ 智能生成结构化 PR 描述（Summary + Test Plan）
- ✅ 三阶段安全检查（执行前状态验证 + 执行中命令安全 + 执行后结果确认）
- ✅ 上下文感知（识别项目类型和测试策略）

**Dependencies**:
- GitHub CLI (`gh`) - 必须已安装并认证
- Git - 标准 git 命令
- 可选: `git-workflow` skill（用于 topic 分支预检查）

## Trigger Patterns

### 直接触发关键词

**中文触发词**:
- "创建 PR"
- "开 PR"
- "提交 PR"
- "发 PR"
- "准备 PR"
- "新建 pull request"

**英文触发词**:
- "create pr"
- "create pull request"
- "open pr"
- "submit pr"
- "make pr"
- "new pr"

**上下文触发**（在 git 相关对话中）:
- "准备好了，可以创建了"
- "现在可以提交了"
- "已经完成，创建 PR 吧"

### 不触发场景（避免误触发）

- "查看 PR" / "view pr" → 使用 `gh pr view`
- "列出 PR" / "list pr" → 使用 `gh pr list`
- "更新 PR" / "update pr" → 不属于此 skill

## Core Workflow

### 三阶段流程

```
阶段 1: 状态预检查 (Pre-flight Checks)
    ├── 检查 gh CLI 认证状态
    ├── 检查当前分支状态
    ├── 检查工作区清洁度
    └── 识别 base branch

阶段 2: 信息收集 (Information Gathering) - 并行执行
    ├── git status           # 工作区状态
    ├── git diff            # staged + unstaged 变更
    ├── git log base..HEAD  # commit 历史
    └── git diff base...HEAD # 完整变更对比

阶段 3: 智能分析与执行 (Analysis & Execution)
    ├── 分析所有 commits（提取类型/范围/原因）
    ├── 生成 PR 描述（Summary + Test Plan）
    ├── 推送分支到 remote（如需要）
    └── 执行 gh pr create
```

### 详细步骤

#### 步骤 1: 状态预检查

**必须检查项**:

| 检查项 | 命令 | 期望结果 | 失败处理 |
|--------|------|---------|---------|
| gh 认证 | `gh auth status` | Logged in | 提示运行 `gh auth login` |
| 当前分支 | `git branch --show-current` | 非 main/master | 警告：不应从主分支创建 PR |
| 工作区状态 | `git status --porcelain` | 空或仅未跟踪文件 | 提示先 commit 或 stash |
| Remote 跟踪 | `git rev-parse --abbrev-ref @{u}` | 有输出或报错 | 准备 `git push -u origin <branch>` |

**base branch 识别逻辑**（优先级递减）:

1. 用户明确指定: `--base <branch>`
2. Git 配置: `git config --get branch.<current>.base`
3. GitHub 默认分支: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
4. 常见惯例: `main` 或 `master`（检查存在性）

#### 步骤 2: 并行信息收集

**并行执行的命令组**（单次 Bash 工具调用，多个命令）:

```bash
# 在单个 message 中并行调用多个 Bash 工具
git status
git diff HEAD  # 查看未 staged 的变更
git diff --cached  # 查看已 staged 的变更
git log --oneline <base>..HEAD  # commits 列表
git diff <base>...HEAD --stat  # 文件变更统计
```

**收集信息的结构化存储**:

```markdown
收集到的信息:
- 工作区状态: clean / has changes
- Commits 数量: N
- 变更文件: M files (+X, -Y)
- 主要文件类型: [前端/后端/测试/文档]
- Commit 类型分布: [feat: X, fix: Y, docs: Z]
```

#### 步骤 3: 智能 PR 描述生成

**Summary 生成逻辑**:

```python
# 伪代码示意
commits = parse_git_log(base_branch, 'HEAD')
commit_types = group_by_prefix(commits)  # feat:, fix:, docs:, etc.

summary_bullets = []
for type, commits in commit_types:
    # 合并同类型 commits
    scope = extract_scope(commits)  # 从文件路径提取
    why = extract_reason(commits)   # 从 commit message 提取
    summary_bullets.append(f"[{type}] {scope}: {why}")

# 限制 3 个 bullet points
summary = "\n".join(summary_bullets[:3])
```

**Test Plan 生成逻辑**:

```python
# 根据变更类型和文件生成测试清单
changed_files = parse_git_diff(base_branch, 'HEAD')

test_plan = []
if has_frontend_files(changed_files):
    test_plan.append("✓ UI 功能测试（浏览器手动验证）")
    test_plan.append("✓ 响应式布局检查（移动端/桌面端）")

if has_backend_files(changed_files):
    test_plan.append("✓ API 端点测试（Postman/curl 验证）")
    test_plan.append("✓ 单元测试通过（运行 test suite）")

if has_db_migrations(changed_files):
    test_plan.append("✓ 数据库迁移测试（migrate + rollback）")

if is_bug_fix(commits):
    test_plan.append("✓ 回归测试（验证 bug 已修复）")
    test_plan.append("✓ 原始问题场景重现测试")
```

**最终 PR 描述模板**:

```markdown
#### Summary
- [type] scope: reason
- [type] scope: reason
- [type] scope: reason

#### Test plan
✓ Test item 1
✓ Test item 2
✓ Test item 3

#### Additional notes
<!-- 可选：自动添加的额外信息 -->
- Base branch: {base_branch}
- Commits included: {commit_count}
- Files changed: {files_count}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

#### 步骤 4: 执行 PR 创建

**命令执行顺序**（条件执行）:

```bash
# 1. 如果分支未推送到 remote，先推送
if ! git rev-parse --abbrev-ref @{u} &>/dev/null; then
    git push -u origin $(git branch --show-current)
fi

# 2. 创建 PR（使用 HEREDOC 确保格式正确）
gh pr create --title "PR标题" --base <base_branch> --body "$(cat <<'EOF'
#### Summary
- [feat] 新功能描述

#### Test plan
✓ 测试项 1
✓ 测试项 2

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**PR 标题生成规则**:

- 单个 commit: 使用该 commit message
- 多个同类型 commits: `[type] scope: summary`
- 混合类型: `Feature: <主要功能描述>`

## Safety Checks

### 执行前检查清单

| 检查项 | 风险 | 处理方式 |
|--------|------|---------|
| 当前分支是 main/master | 误操作 | **阻止**，提示切换到 feature 分支 |
| 工作区有未提交变更 | 遗漏变更 | **警告**，建议先 commit |
| gh CLI 未认证 | 命令失败 | **阻止**，提供认证指导 |
| Remote 不存在 | 推送失败 | **警告**，自动添加 `-u origin` |
| Base branch 不存在 | PR 创建失败 | **阻止**，提示检查 base branch 名称 |

### 执行中安全机制

- **gh CLI 内置保护**: `gh pr create` 会检查是否已存在同名 PR
- **Git 推送保护**: 使用 `git push -u` 而非 `--force`
- **HEREDOC 格式化**: 避免引号和特殊字符导致的命令注入

### 执行后验证

```bash
# 检查 PR 是否创建成功
if gh pr view --json url -q .url; then
    echo "✅ PR 创建成功: $(gh pr view --json url -q .url)"
else
    echo "❌ PR 创建失败，请检查错误信息"
fi
```

## Command Reference

### 核心命令映射

| 用户意图 | 执行命令 | 说明 |
|---------|---------|------|
| 检查 gh 认证 | `gh auth status` | 确认 GitHub CLI 已登录 |
| 获取 base branch | `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` | 识别默认分支 |
| 收集 commits | `git log --oneline <base>..HEAD` | 获取所有待合并的 commits |
| 分析变更 | `git diff <base>...HEAD --stat` | 统计文件变更 |
| 推送分支 | `git push -u origin <branch>` | 首次推送到 remote |
| 创建 PR | `gh pr create --title "..." --base <base> --body "..."` | 执行 PR 创建 |
| 查看 PR URL | `gh pr view --json url -q .url` | 获取创建的 PR 链接 |

### 高级选项

**自定义 PR 参数**:

```bash
# Draft PR（草稿模式）
gh pr create --draft --title "..." --body "..."

# 指定 reviewer
gh pr create --reviewer @user1,@user2 --title "..." --body "..."

# 添加 labels
gh pr create --label "bug" --label "priority:high" --title "..." --body "..."

# 指定 milestone
gh pr create --milestone "v1.2.0" --title "..." --body "..."
```

**用户可选参数**（Skill 应支持）:

- `--draft`: 创建草稿 PR
- `--base <branch>`: 明确指定 base branch
- `--reviewer <users>`: 指定审核者
- `--label <labels>`: 添加标签

## Integration with git-workflow Skill

### 协同工作场景

```
用户工作流:
1. git-workflow: 创建 topic 分支 (tnr/tn)
2. [开发和提交...]
3. git-workflow: 整理 commits (fixup/amend)
4. gh-pr-create: 创建 PR ← 此 skill
5. git-workflow: 合并后删除分支 (tmg/td)
```

### 预检查集成（可选）

**gh-pr-create 在执行前可调用 git-workflow 的检查**:

```bash
# 检查是否有未整理的 fixup commits
if git log --oneline <base>..HEAD | grep -q "fixup!"; then
    echo "⚠️  检测到 fixup commits，建议先运行:"
    echo "   git rebase -i --autosquash <base>"
    echo ""
    echo "或使用 git-workflow skill 的 fixup 命令整理。"
    # 询问用户是否继续
fi
```

### 数据共享

- **base branch 识别**: 两个 skills 使用相同的逻辑
- **分支状态**: gh-pr-create 可复用 git-workflow 的状态检查函数
- **commit 分析**: 可共享 commit message 解析逻辑

## Error Handling

### 常见错误场景

#### 1. gh CLI 未安装

```
错误信息: command not found: gh

解决方案:
1. 访问 https://cli.github.com/ 安装 GitHub CLI
2. macOS: brew install gh
3. Ubuntu/Debian: apt install gh
4. Windows: winget install GitHub.cli
```

#### 2. gh CLI 未认证

```
错误信息: gh: To get started with GitHub CLI, please run: gh auth login

解决方案:
1. 运行 gh auth login
2. 选择认证方式（browser/token）
3. 完成授权流程
```

#### 3. 当前分支未跟踪 remote

```
错误信息: fatal: no upstream configured for branch 'feature-x'

自动处理: 执行 git push -u origin feature-x
```

#### 4. Base branch 不存在

```
错误信息: GraphQL: Could not resolve to a PullRequestable with the name of ...

解决方案:
1. 检查 base branch 名称（main vs master）
2. 使用 --base 参数明确指定
3. 运行 gh repo view --json defaultBranchRef 确认默认分支
```

#### 5. PR 已存在

```
错误信息: GraphQL: A pull request already exists for ...

解决方案:
1. 运行 gh pr view 查看现有 PR
2. 如需更新 PR 描述，使用 gh pr edit
3. 如需推送新 commits，直接 git push
```

## Advanced Features

### 1. 项目类型检测

**检测逻辑**:

```bash
# 前端项目特征
if [[ -f "package.json" ]] && grep -q "react\|vue\|angular" package.json; then
    project_type="frontend"
fi

# 后端项目特征
if [[ -f "requirements.txt" ]] || [[ -f "go.mod" ]] || [[ -f "Cargo.toml" ]]; then
    project_type="backend"
fi

# 全栈项目
if [[ -d "frontend" ]] && [[ -d "backend" ]]; then
    project_type="fullstack"
fi
```

**影响**:
- 调整 Test Plan 中的测试项
- 优化 Summary 中的描述重点

### 2. Conventional Commits 支持

**自动识别 commit 类型前缀**:

```
feat:     → 新功能
fix:      → Bug 修复
docs:     → 文档变更
style:    → 代码格式
refactor: → 重构
perf:     → 性能优化
test:     → 测试相关
chore:    → 构建/工具变更
```

**生成的 Summary 自动分组**:

```markdown
#### Summary
- [feat] 添加用户认证功能
- [fix] 修复登录页面响应式布局问题
- [docs] 更新 API 文档
```

### 3. 大型 PR 处理

**当 commits > 10 时**:

- Summary: 只显示前 3 个最重要的变更
- 添加折叠详情: "<!-- 10+ commits, see full log -->"
- 提供 GitHub compare link: `https://github.com/<repo>/compare/<base>...<head>`

### 4. 多 Remote 场景

**检测 fork 工作流**:

```bash
# 检查是否有 upstream remote
if git remote | grep -q upstream; then
    echo "检测到 fork 工作流"
    echo "将推送到: origin"
    echo "PR base: upstream/main"
fi
```

## Troubleshooting

详见 `references/gh-integration.md` 的故障排查章节。

## Best Practices

1. **PR 大小控制**: 建议单个 PR 不超过 20 个 commits / 500 行变更
2. **描述质量**: 自动生成后，建议人工 review 并补充业务背景
3. **测试覆盖**: 确保 Test Plan 中的项目实际执行后再 approve
4. **及时创建**: topic 分支完成后立即创建 PR，避免积压
5. **草稿模式**: 未完成的功能使用 `--draft` 选项

## References

- `references/pr-templates.md`: PR 描述模板库和生成策略
- `references/gh-integration.md`: GitHub CLI 集成详解
- `references/base-branch-detection.md`: Base branch 识别算法

## Skill Metadata

- **Version**: 1.0.0
- **Author**: Claude Code Workflow Team
- **Last Updated**: 2025-12-22
- **Compatibility**: gh CLI v2.0+, git 2.20+
