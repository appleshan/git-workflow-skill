# gh-pr-create Skill

智能 GitHub Pull Request 创建助手，基于 GitHub CLI 实现自动化 PR 创建流程。

## 概述

gh-pr-create 是一个 Claude Code skill，用于简化和自动化 GitHub Pull Request 的创建过程。通过智能分析 commits 和文件变更，自动生成高质量的 PR 描述。

### 核心功能

- 🤖 **智能 PR 描述生成**: 基于 commits 和文件变更自动生成 Summary 和 Test Plan
- 🔍 **Base Branch 识别**: 自动识别目标分支（支持 main/master/自定义）
- ✅ **安全检查**: 执行前验证 gh 认证、分支状态、工作区状态
- 🎯 **上下文感知**: 根据项目类型（前端/后端/全栈）调整 PR 描述
- 🔄 **工作流集成**: 与 git-workflow skill 无缝配合
- 📝 **多种模板**: 支持 feature/bugfix/refactor/docs/perf 等类型

## 快速开始

### 安装

1. **安装 GitHub CLI**:

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install GitHub.cli
```

2. **认证 GitHub CLI**:

```bash
gh auth login
```

3. **部署 Skill**:

```bash
# 复制 skill 文件到 Claude Code skills 目录
cp -r skills/gh-pr-create ~/.claude/skills/

# 配置触发规则（添加到 ~/.claude/skills/skill-rules.json）
```

### 基本用法

```bash
# 1. 开发功能并提交
git checkout -b feature/user-auth
echo "auth code" >> src/auth.js
git add . && git commit -m "feat: implement user authentication"

# 2. 在 Claude Code 中触发 skill
# 输入: "创建 PR"
# 或: "create pr"

# 3. Skill 自动执行:
#   - 检查 gh 认证和分支状态
#   - 收集 commits 和文件变更信息
#   - 生成 PR 描述（Summary + Test Plan）
#   - 推送分支到 remote（如需要）
#   - 创建 PR 并返回 URL
```

## 触发方式

### 直接触发

**中文**:
- "创建 PR"
- "开 PR"
- "提交 PR"
- "发 PR"

**英文**:
- "create pr"
- "open pr"
- "submit pr"
- "make pr"

### 上下文触发

在 git 相关对话中：
- "准备好了，可以创建了"
- "现在可以提交了"
- "已经完成，创建 PR 吧"

### 高级用法

```bash
# 创建草稿 PR
"创建草稿 PR"
"create draft pr"

# 指定 base branch
"创建 PR，base 是 develop"
"create pr with base develop"

# 指定 reviewer
"创建 PR，reviewer 是 @alice 和 @bob"

# 添加 labels
"创建 PR，添加 bug 和 urgent 标签"
```

## 工作流程

### 三阶段流程

```
┌─────────────────────────────────────────────────────────────┐
│ 阶段 1: 状态预检查 (Pre-flight Checks)                      │
├─────────────────────────────────────────────────────────────┤
│ ✓ 检查 gh CLI 认证状态                                       │
│ ✓ 检查当前分支（避免从 main/master 创建）                    │
│ ✓ 检查工作区清洁度                                          │
│ ✓ 识别 base branch                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 阶段 2: 信息收集 (Information Gathering) - 并行执行          │
├─────────────────────────────────────────────────────────────┤
│ ⚡ git status            # 工作区状态                        │
│ ⚡ git diff              # staged + unstaged 变更            │
│ ⚡ git log base..HEAD    # commit 历史                       │
│ ⚡ git diff base...HEAD  # 完整变更对比                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 阶段 3: 智能分析与执行 (Analysis & Execution)                │
├─────────────────────────────────────────────────────────────┤
│ 🤖 分析所有 commits（提取类型/范围/原因）                    │
│ 📝 生成 PR 描述（Summary + Test Plan）                       │
│ 🚀 推送分支到 remote（如需要）                               │
│ ✅ 执行 gh pr create                                         │
│ 🔗 返回 PR URL                                               │
└─────────────────────────────────────────────────────────────┘
```

### PR 描述生成逻辑

#### Summary 生成

```markdown
#### Summary
- [type] scope: 变更描述（聚焦 WHY 而非 WHAT）

示例:
- [feat] 用户认证: 实现 OAuth 2.0 登录和 JWT token 管理
- [fix] 购物车: 修复折扣计算错误（关联 issue #456）
- [perf] 数据库: 添加查询缓存，提升 65%
```

**特性**:
- 自动识别 Conventional Commits 前缀（feat/fix/docs/等）
- 提取 scope（从 commit message 或文件路径）
- 合并相似 commits（避免冗长）
- 限制 3 个 bullet points（保持简洁）

#### Test Plan 生成

```markdown
#### Test plan
✓ 功能测试
  - [ ] 核心功能按需求工作
  - [ ] 边界条件处理正确

✓ 回归测试
  - [ ] 现有功能未受影响
  - [ ] 所有测试通过
```

**智能调整**:
- **Frontend 项目**: 添加 UI 测试、响应式布局检查
- **Backend 项目**: 添加 API 测试、单元测试覆盖率
- **Bug Fix**: 强调回归测试和 bug 验证
- **Refactoring**: 强调行为一致性

## 与 git-workflow Skill 集成

### 协同工作流

```
用户开发流程:
1. git-workflow: 创建 topic 分支 (tnr/tn)
2. [开发和提交...]
3. git-workflow: 整理 commits (fixup/amend)
4. gh-pr-create: 创建 PR ← 此 skill
5. [Code review + CI checks]
6. git-workflow: 合并后删除分支 (tmg/td)
```

### 职责划分

| Skill | 职责范围 |
|-------|---------|
| **git-workflow** | 本地 Git 操作<br>- 分支管理（创建/合并/删除）<br>- Commit 整理（fixup/amend）<br>- 冲突解决<br>- 本地工作区管理 |
| **gh-pr-create** | GitHub 远程操作<br>- PR 创建和描述生成<br>- Base branch 识别<br>- gh CLI 集成<br>- Fork 工作流支持 |

### 数据共享

两个 skills 共享：
- Base branch 识别逻辑
- 分支状态检查函数
- Commit message 解析逻辑

## 支持的场景

### 1. 标准 Feature 开发

```bash
git checkout -b feature/user-profile
# 开发...
git commit -m "feat(profile): add user profile component"
# 触发 skill: "创建 PR"
```

### 2. Bug Fix（关联 Issue）

```bash
git checkout -b hotfix/cart-calculation
git commit -m "fix: correct discount calculation (closes #456)"
# 触发 skill: "创建 PR"
```

### 3. 大规模 Refactoring

```bash
git checkout -b refactor/auth-module
# 多个 refactor commits...
# 触发 skill: "创建 PR"
# 自动生成行为一致性测试清单
```

### 4. Fork 贡献

```bash
git remote add upstream https://github.com/original/repo.git
git checkout -b feature/dark-mode
# 触发 skill: "创建 PR 到 upstream"
# 自动识别 fork 场景并创建到 upstream
```

### 5. Monorepo 多 Package 变更

```bash
# packages/frontend/, packages/backend/, packages/shared/
# 触发 skill
# 自动识别 affected packages 并生成对应测试
```

更多场景参见 [scenarios.md](../examples/scenarios.md)

## 配置

### Base Branch 配置

**方式 1: Git 配置**（推荐用于长期分支）:

```bash
# 为当前分支配置 base
git config branch.$(git branch --show-current).base develop

# 为特定分支配置
git config branch.feature-x.base release/v1.2
```

**方式 2: 用户明确指定**:

```
输入: "创建 PR，base 是 develop"
```

**方式 3: 自动检测**（默认）:

优先级：用户指定 > Git 配置 > GitHub 默认分支 > 常见惯例（main/master/develop）

### 触发规则配置

编辑 `~/.claude/skills/skill-rules.json`:

```json
{
  "gh-pr-create": {
    "keywords": [
      "创建 PR", "开 PR", "提交 PR",
      "create pr", "open pr", "submit pr"
    ],
    "intentPatterns": [
      ".*创建.*[Pp][Rr].*",
      ".*[Cc]reate.*[Pp][Rr].*"
    ]
  }
}
```

## 文档

### 核心文档

- **[SKILL.md](../SKILL.md)**: 主 Skill 定义和核心流程
- **[testing.md](testing.md)**: 完整测试验证清单
- **[scenarios.md](../examples/scenarios.md)**: 10+ 实际使用场景

### 参考文档

- **[pr-templates.md](../references/pr-templates.md)**: PR 描述模板库和生成策略
- **[gh-integration.md](../references/gh-integration.md)**: GitHub CLI 集成详解
- **[base-branch-detection.md](../references/base-branch-detection.md)**: Base branch 识别算法

---

## 统计数据

| 指标 | 数值 |
|-----|------|
| 文档数量 | 4 个 |
| 总代码行数 | 2409 行 |
| 主文档 | 468 行 |
| PR 模板 | 619 行 |
| GitHub 集成 | 695 行 |
| Base 分支检测 | 627 行 |

---

## 故障排查

### 问题 1: gh: command not found

**原因**: gh CLI 未安装

**解决方案**:

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh
```

### 问题 2: gh: To get started with GitHub CLI, please run: gh auth login

**原因**: 未认证

**解决方案**:

```bash
gh auth login
```

### 问题 3: GraphQL: A pull request already exists for...

**原因**: 当前分支已有 PR

**解决方案**:

```bash
# 查看现有 PR
gh pr view

# 如需更新描述
gh pr edit --body "$(cat new-description.md)"
```

### 问题 4: Base branch 识别错误

**原因**: 自动识别可能不准确

**解决方案**:

```bash
# 方式 1: 配置 git config
git config branch.$(git branch --show-current).base correct-base

# 方式 2: 明确指定
# 输入: "创建 PR，base 是 correct-base"
```

更多故障排查参见 [gh-integration.md#troubleshooting](../references/gh-integration.md#troubleshooting)

## 最佳实践

### 1. Commit Message 规范

使用 Conventional Commits 格式：

```
type(scope): description

type: feat, fix, docs, style, refactor, perf, test, chore
scope: 变更的模块或功能
description: 简短描述（50 字符内）
```

### 2. Commit 原子性

每个 commit 解决单一问题：

```bash
# ✅ 好
git commit -m "feat: add user login"
git commit -m "feat: add JWT token management"

# ❌ 差
git commit -m "add login and token stuff"
```

### 3. PR 大小控制

单个 PR 建议：
- ≤ 20 commits
- ≤ 500 lines changed
- 聚焦单一功能或 bug

大型变更拆分为多个 PR：

```bash
# PR 1: 基础架构
# PR 2: 核心功能
# PR 3: UI 集成
```

### 4. 及时创建 PR

Feature 分支完成后立即创建 PR：
- 避免积压
- 早期反馈
- 减少 merge conflicts

### 5. 使用 Draft PR

未完成的功能使用 Draft PR：

```
输入: "创建草稿 PR"
```

完成后标记为 ready:

```bash
gh pr ready <PR-number>
```

## 贡献

欢迎贡献！主要贡献方式：

1. **添加新的 PR 模板**: 编辑 `../references/pr-templates.md`
2. **改进 base branch 识别**: 编辑 `../references/base-branch-detection.md`
3. **添加使用场景**: 编辑 `../examples/scenarios.md`
4. **报告 bug 或建议**: 创建 issue

## License

MIT License

## 相关资源

- [GitHub CLI 官方文档](https://cli.github.com/manual/)
- [Conventional Commits 规范](https://www.conventionalcommits.org/)
- [Pull Request 最佳实践](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)
- [git-workflow Skill](../../git-workflow/docs/README_zh-CN.md)

## 联系方式

- GitHub Issues: [创建 issue](https://github.com/your-org/git-workflow-skill/issues)
- 文档更新: 参见各文档的 Last Updated 时间戳

---

**Version**: 1.0.0
**Last Updated**: 2025-12-22
**Compatibility**: gh CLI v2.0+, git 2.20+
