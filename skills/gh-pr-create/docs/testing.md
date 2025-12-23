# Testing Guide for gh-pr-create Skill

测试验证清单，确保 gh-pr-create skill 在各种场景下正确工作。

## Test Environment Setup

### 前置条件

```bash
# 1. 安装 GitHub CLI
gh --version  # 应显示 v2.0+

# 2. 完成认证
gh auth status  # 应显示 "Logged in"

# 3. 准备测试仓库
git clone https://github.com/your-username/test-repo.git
cd test-repo

# 4. 部署 skill（如尚未部署）
cp -r skills/gh-pr-create ~/.claude/skills/
```

### 测试仓库结构

建议使用以下结构的测试仓库：

```
test-repo/
├── .github/
│   └── pull_request_template.md  # 可选：PR 模板
├── frontend/
│   ├── package.json
│   └── src/
├── backend/
│   ├── requirements.txt
│   └── api/
├── README.md
└── .gitignore
```

## Test Categories

### 1. Trigger Tests（触发测试）

验证 skill 能被正确触发。

#### 1.1 中文触发词

| 输入 | 期望结果 | 通过 |
|------|---------|------|
| "创建 PR" | ✅ 触发 skill | [ ] |
| "开 PR" | ✅ 触发 skill | [ ] |
| "提交 PR" | ✅ 触发 skill | [ ] |
| "发 PR" | ✅ 触发 skill | [ ] |
| "准备 PR" | ✅ 触发 skill | [ ] |
| "新建 pull request" | ✅ 触发 skill | [ ] |

#### 1.2 英文触发词

| 输入 | 期望结果 | 通过 |
|------|---------|------|
| "create pr" | ✅ 触发 skill | [ ] |
| "create pull request" | ✅ 触发 skill | [ ] |
| "open pr" | ✅ 触发 skill | [ ] |
| "submit pr" | ✅ 触发 skill | [ ] |
| "make pr" | ✅ 触发 skill | [ ] |

#### 1.3 上下文触发

| 输入 | 上下文 | 期望结果 | 通过 |
|------|--------|---------|------|
| "准备好了，可以创建了" | Git 操作对话中 | ✅ 触发 skill | [ ] |
| "现在可以提交了" | Commit 完成后 | ✅ 触发 skill | [ ] |
| "已经完成，创建 PR 吧" | Feature 开发完成后 | ✅ 触发 skill | [ ] |

#### 1.4 不应触发的场景

| 输入 | 期望结果 | 通过 |
|------|---------|------|
| "查看 PR" | ❌ 不触发（应建议 `gh pr view`） | [ ] |
| "列出 PR" | ❌ 不触发（应建议 `gh pr list`） | [ ] |
| "更新 PR" | ❌ 不触发（应建议 `gh pr edit`） | [ ] |
| "合并 PR" | ❌ 不触发（应建议 `gh pr merge`） | [ ] |

### 2. Pre-flight Checks（预检查测试）

验证执行前的状态检查。

#### 2.1 gh CLI 认证状态

**测试步骤**:

```bash
# 测试 1: 未认证场景
gh auth logout
# 触发 skill："创建 PR"

# 期望输出:
# ❌ GitHub CLI 未认证
# 请运行以下命令完成认证:
#   gh auth login

# 测试 2: 已认证场景
gh auth login
# 触发 skill："创建 PR"

# 期望: 继续执行后续步骤
```

**验证清单**:

- [ ] 未认证时阻止执行并提供明确指导
- [ ] 已认证时正常继续
- [ ] Token 过期时能识别并提示重新认证

#### 2.2 当前分支检查

**测试场景**:

```bash
# 场景 1: 从 main 分支创建 PR（应警告）
git checkout main
# 触发 skill

# 期望: 警告用户不应从主分支创建 PR

# 场景 2: 从 feature 分支创建 PR（正常）
git checkout -b feature/test-pr
git commit --allow-empty -m "test: empty commit"
# 触发 skill

# 期望: 正常执行
```

**验证清单**:

- [ ] 在 main/master 分支时显示警告
- [ ] 在 feature 分支时正常执行
- [ ] 在 develop 分支时根据项目惯例决定（可配置）

#### 2.3 工作区状态检查

**测试场景**:

```bash
# 场景 1: 工作区有未提交变更
echo "test" >> README.md
# 触发 skill

# 期望: 警告有未提交的变更，建议先 commit

# 场景 2: 工作区干净
git add README.md && git commit -m "test: update readme"
# 触发 skill

# 期望: 正常执行
```

**验证清单**:

- [ ] 有 unstaged 变更时警告
- [ ] 有 staged 但未 commit 的变更时警告
- [ ] 工作区干净时正常执行
- [ ] 仅有 untracked 文件时可选择忽略

#### 2.4 Remote 跟踪状态

**测试场景**:

```bash
# 场景 1: 分支未推送到 remote
git checkout -b feature/new-branch
git commit --allow-empty -m "test"
# 触发 skill

# 期望: 自动执行 git push -u origin feature/new-branch

# 场景 2: 分支已推送
# （分支已在 remote）
# 触发 skill

# 期望: 直接创建 PR
```

**验证清单**:

- [ ] 未推送时自动 push -u
- [ ] 已推送时跳过 push
- [ ] Push 失败时能捕获错误并提示

### 3. Information Gathering（信息收集测试）

验证能正确收集 commits 和变更信息。

#### 3.1 单个 Commit

**测试步骤**:

```bash
git checkout -b test/single-commit
git commit --allow-empty -m "feat: add user login feature"
# 触发 skill

# 期望 Summary:
# - [feat] add user login feature
```

**验证清单**:

- [ ] 正确提取 commit message
- [ ] 识别 conventional commit 前缀（feat）
- [ ] Summary 长度合理（不截断）

#### 3.2 多个 Commits（同类型）

**测试步骤**:

```bash
git checkout -b test/multi-commits-same-type
git commit --allow-empty -m "feat: implement OAuth login"
git commit --allow-empty -m "feat: add JWT token management"
git commit --allow-empty -m "feat: implement role-based auth"
# 触发 skill

# 期望 Summary:
# - [feat] 实现 OAuth 登录、JWT token 管理和基于角色的认证
```

**验证清单**:

- [ ] 合并相似 commits
- [ ] 保留关键信息
- [ ] Summary 不超过 3 个 bullet points

#### 3.3 多个 Commits（混合类型）

**测试步骤**:

```bash
git checkout -b test/multi-commits-mixed
git commit --allow-empty -m "feat: add product recommendation"
git commit --allow-empty -m "fix: cart price calculation bug"
git commit --allow-empty -m "perf: optimize homepage load time"
git commit --allow-empty -m "docs: update API documentation"
# 触发 skill

# 期望 Summary（按优先级排序）:
# - [feat] 新增产品推荐功能
# - [fix] 修复购物车价格计算错误
# - [perf] 优化首页加载速度
```

**验证清单**:

- [ ] 按优先级排序（feat > fix > perf > docs）
- [ ] 每种类型只保留最重要的信息
- [ ] 合并不重要的 commits（例如 docs 和 chore）

#### 3.4 文件变更统计

**测试步骤**:

```bash
git checkout -b test/file-changes
echo "test" >> frontend/src/App.jsx
echo "test" >> backend/api/views.py
git add . && git commit -m "feat: full-stack feature"
# 触发 skill

# 期望: 识别前端和后端变更，生成对应的 Test Plan
```

**验证清单**:

- [ ] 正确统计变更文件数量
- [ ] 识别文件类型（前端/后端/测试/文档）
- [ ] 根据文件类型调整 Test Plan

### 4. Base Branch Detection（分支识别测试）

验证 base branch 的智能识别。

#### 4.1 标准场景（main/master）

**测试步骤**:

```bash
# 场景 1: main 分支
git checkout -b test/base-main
# 触发 skill

# 期望: 自动识别 base 为 "main"

# 场景 2: master 分支（旧仓库）
# (在使用 master 的仓库中测试)
# 期望: 自动识别 base 为 "master"
```

**验证清单**:

- [ ] main 优先于 master
- [ ] 自动检测仓库使用的默认分支
- [ ] 使用 `gh repo view` 获取 GitHub 默认分支

#### 4.2 自定义 Base Branch

**测试步骤**:

```bash
# 配置自定义 base
git config branch.test-feature.base develop
git checkout -b test-feature
# 触发 skill

# 期望: 使用配置的 "develop" 作为 base
```

**验证清单**:

- [ ] 读取 `git config branch.<name>.base`
- [ ] 优先使用配置的 base
- [ ] 配置不存在时回退到自动检测

#### 4.3 用户明确指定

**测试步骤**:

```bash
# 用户输入："创建 PR，base 是 develop"
# 期望: 使用 develop 作为 base
```

**验证清单**:

- [ ] 解析用户输入中的 base branch
- [ ] 验证 base branch 存在
- [ ] 不存在时提示用户并列出可用分支

#### 4.4 Fork 工作流

**测试步骤**:

```bash
# 添加 upstream remote
git remote add upstream https://github.com/original/repo.git
git checkout -b test/fork-workflow
# 触发 skill

# 期望:
# - 识别 fork 场景
# - Base branch 为 upstream 的默认分支
# - PR 创建到 upstream 仓库
```

**验证清单**:

- [ ] 检测 upstream remote
- [ ] 使用 upstream 的默认分支
- [ ] gh pr create 指定正确的 --repo 和 --head

### 5. PR Description Generation（描述生成测试）

验证智能 PR 描述生成。

#### 5.1 Feature 类型 PR

**测试场景**:

```bash
git checkout -b test/feature-pr
echo "login" >> frontend/src/Login.jsx
echo "auth" >> backend/api/auth.py
git add . && git commit -m "feat(auth): implement OAuth 2.0 login"
# 触发 skill

# 期望 PR 描述:
# #### Summary
# - [feat] 认证模块: 实现 OAuth 2.0 登录
#
# #### Test plan
# ✓ 前端测试
#   - [ ] 登录组件渲染正常
#   - [ ] OAuth 流程工作正常
# ✓ 后端测试
#   - [ ] /api/auth/login 端点返回正确
#   - [ ] Token 生成和验证正确
```

**验证清单**:

- [ ] Summary 准确反映功能
- [ ] Test Plan 包含前端和后端测试
- [ ] 格式符合模板结构

#### 5.2 Bug Fix 类型 PR

**测试场景**:

```bash
git checkout -b test/bugfix-pr
echo "fix" >> backend/api/cart.py
git add . && git commit -m "fix: cart price calculation error (closes #123)"
# 触发 skill

# 期望 PR 描述:
# #### Summary
# - [fix] 修复购物车价格计算错误（关联 issue #123）
#
# #### Test plan
# ✓ Bug 修复验证
#   - [ ] 复现原始 bug 场景（已修复）
#   - [ ] 相关边界条件测试通过
# ✓ 回归测试
#   - [ ] 购物车其他功能未受影响
```

**验证清单**:

- [ ] 识别 issue 编号（#123）
- [ ] Test Plan 包含回归测试
- [ ] 强调 bug 验证

#### 5.3 Refactoring 类型 PR

**测试场景**:

```bash
git checkout -b test/refactor-pr
echo "refactor" >> backend/api/utils.py
git add . && git commit -m "refactor: simplify auth logic"
# 触发 skill

# 期望 PR 描述:
# #### Summary
# - [refactor] 简化认证逻辑
#
# #### Test plan
# ✓ 行为一致性验证
#   - [ ] 所有现有测试通过（无测试修改）
#   - [ ] 输出结果与重构前一致
```

**验证清单**:

- [ ] 强调"无行为变更"
- [ ] Test Plan 聚焦行为一致性
- [ ] 不生成新功能测试项

#### 5.4 Documentation 类型 PR

**测试场景**:

```bash
git checkout -b test/docs-pr
echo "update" >> README.md
git add . && git commit -m "docs: update installation guide"
# 触发 skill

# 期望 PR 描述:
# #### Summary
# - [docs] 更新安装指南
#
# #### Test plan
# ✓ 文档质量
#   - [ ] Markdown 格式正确
#   - [ ] 代码示例可运行
```

**验证清单**:

- [ ] 识别纯文档变更
- [ ] Test Plan 聚焦文档质量
- [ ] 不包含功能测试

### 6. PR Creation Execution（创建执行测试）

验证 PR 实际创建过程。

#### 6.1 标准创建流程

**测试步骤**:

```bash
git checkout -b test/pr-creation
git commit --allow-empty -m "test: standard PR creation"
# 触发 skill

# 期望:
# 1. 自动 push 分支（如未推送）
# 2. 执行 gh pr create
# 3. 返回 PR URL
```

**验证清单**:

- [ ] Push 成功（如需要）
- [ ] PR 创建成功
- [ ] 返回可点击的 PR URL
- [ ] PR 描述格式正确（在 GitHub 上查看）

#### 6.2 Draft PR 创建

**测试步骤**:

```bash
# 用户输入："创建草稿 PR"
# 或 "create draft pr"

# 期望: gh pr create --draft
```

**验证清单**:

- [ ] 识别"草稿"/"draft"关键词
- [ ] 使用 --draft 参数
- [ ] PR 在 GitHub 显示为 Draft 状态

#### 6.3 指定 Reviewer

**测试步骤**:

```bash
# 用户输入："创建 PR，reviewer 是 @user1 和 @user2"

# 期望: gh pr create --reviewer user1,user2
```

**验证清单**:

- [ ] 解析 reviewer 列表
- [ ] 使用 --reviewer 参数
- [ ] Reviewers 正确添加到 PR

#### 6.4 添加 Labels

**测试步骤**:

```bash
# 用户输入："创建 PR，添加 bug 和 urgent 标签"

# 期望: gh pr create --label "bug,urgent"
```

**验证清单**:

- [ ] 解析 label 列表
- [ ] 使用 --label 参数
- [ ] Labels 正确添加到 PR

### 7. Error Handling（错误处理测试）

验证各种错误场景的处理。

#### 7.1 gh CLI 未安装

**测试步骤**:

```bash
# 临时重命名 gh（模拟未安装）
sudo mv /usr/bin/gh /usr/bin/gh.bak
# 触发 skill

# 期望:
# ❌ 错误: gh CLI 未安装
# 请访问 https://cli.github.com/ 安装

# 恢复
sudo mv /usr/bin/gh.bak /usr/bin/gh
```

**验证清单**:

- [ ] 检测 gh 不存在
- [ ] 提供安装指导（含链接）
- [ ] 阻止后续执行

#### 7.2 PR 已存在

**测试步骤**:

```bash
# 创建 PR
gh pr create --title "Test" --body "Test"

# 再次触发 skill（同一分支）

# 期望:
# ⚠️  PR 已存在: https://github.com/.../pull/123
# 如需更新描述，使用: gh pr edit
```

**验证清单**:

- [ ] 检测已存在的 PR
- [ ] 显示现有 PR URL
- [ ] 提示使用 `gh pr edit` 更新

#### 7.3 Base Branch 不存在

**测试步骤**:

```bash
# 用户输入："创建 PR，base 是 nonexistent"

# 期望:
# ❌ 错误: Base branch 'nonexistent' 不存在
# 可用分支:
#   - main
#   - develop
```

**验证清单**:

- [ ] 验证 base branch 存在性
- [ ] 列出可用分支
- [ ] 阻止错误的 PR 创建

#### 7.4 Push 失败

**测试步骤**:

```bash
# 模拟 push 失败（例如网络问题或权限问题）

# 期望:
# ❌ 错误: 无法推送分支到 remote
# 错误信息: [原始 git 错误]
# 请检查网络连接和权限
```

**验证清单**:

- [ ] 捕获 push 错误
- [ ] 显示原始错误信息
- [ ] 提供故障排查建议

### 8. Integration Tests（集成测试）

验证与其他 skills 和工具的集成。

#### 8.1 与 git-workflow Skill 集成

**测试步骤**:

```bash
# 1. 使用 git-workflow 创建分支
# 输入: "开始新功能 test-integration"

# 2. 开发和提交
echo "code" >> src/feature.js
git add . && git commit -m "feat: test integration"

# 3. 使用 gh-pr-create 创建 PR
# 输入: "创建 PR"

# 期望: 整个流程无缝衔接
```

**验证清单**:

- [ ] 识别 git-workflow 创建的分支
- [ ] 正确识别 base branch
- [ ] 复用 git-workflow 的安全检查

#### 8.2 与 PR 模板集成

**测试步骤**:

```bash
# 1. 创建 PR 模板
cat > .github/pull_request_template.md <<EOF
## Summary
<!-- 描述变更 -->

## Test plan
- [ ] 测试项 1
- [ ] 测试项 2
EOF

# 2. 触发 skill
# 期望: 检测到模板，询问是否使用模板还是智能生成
```

**验证清单**:

- [ ] 检测 PR 模板存在
- [ ] 询问用户选择
- [ ] 支持填充模板占位符

### 9. Performance Tests（性能测试）

验证执行性能。

#### 9.1 大型 PR（多 Commits）

**测试步骤**:

```bash
git checkout -b test/large-pr

# 创建 20 个 commits
for i in {1..20}; do
    git commit --allow-empty -m "feat: commit $i"
done

# 触发 skill
# 期望: 在 < 10 秒内完成（包括 push 和 PR 创建）
```

**验证清单**:

- [ ] 处理 20+ commits 不卡顿
- [ ] Summary 正确合并 commits
- [ ] 总耗时 < 10 秒

#### 9.2 大量文件变更

**测试步骤**:

```bash
git checkout -b test/many-files

# 创建 100 个文件
for i in {1..100}; do
    echo "test" > "file$i.txt"
done

git add . && git commit -m "feat: add 100 files"
# 触发 skill

# 期望: 正常处理，不超时
```

**验证清单**:

- [ ] 处理 100+ 文件变更
- [ ] git diff 命令不超时
- [ ] PR 描述生成正常

### 10. Edge Cases（边界测试）

验证边界和异常场景。

#### 10.1 空 Commit

**测试步骤**:

```bash
git checkout -b test/empty-commit
# 触发 skill（分支无 commits）

# 期望:
# ⚠️  警告: 当前分支无新 commits
# 无法创建 PR
```

**验证清单**:

- [ ] 检测无 commits 场景
- [ ] 阻止创建空 PR
- [ ] 提示用户先 commit

#### 10.2 非常长的 Commit Message

**测试步骤**:

```bash
LONG_MSG="feat: $(printf 'a%.0s' {1..500})"
git commit --allow-empty -m "$LONG_MSG"
# 触发 skill

# 期望: Summary 截断到合理长度（<= 100 chars）
```

**验证清单**:

- [ ] 截断过长的 commit message
- [ ] 保留关键信息（前缀和主要内容）
- [ ] 添加 "..." 表示截断

#### 10.3 特殊字符和表情符号

**测试步骤**:

```bash
git commit --allow-empty -m "feat: 🚀 add \"quoted\" feature & <tag>"
# 触发 skill

# 期望: 正确处理特殊字符，不导致命令注入
```

**验证清单**:

- [ ] 特殊字符正确转义
- [ ] 表情符号正确显示
- [ ] 无命令注入风险
