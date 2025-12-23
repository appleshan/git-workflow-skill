# Usage Scenarios for gh-pr-create Skill

实际使用场景示例，展示 gh-pr-create skill 的各种用法和最佳实践。

## Scenario 1: Standard Feature Development

**场景描述**: 开发新功能后创建标准 PR

### 工作流

```bash
# 1. 创建 feature 分支
git checkout -b feature/user-profile

# 2. 开发和提交
echo "ProfileComponent" >> frontend/src/Profile.jsx
git add . && git commit -m "feat(profile): add user profile component"

echo "profile API" >> backend/api/profile.py
git add . && git commit -m "feat(profile): implement profile API endpoint"

# 3. 触发 skill
# 用户输入: "创建 PR"
```

### 预期输出

```markdown
✅ 执行前检查:
- gh CLI 已认证
- 当前分支: feature/user-profile
- Base branch: main
- 工作区干净

📊 收集信息:
- 2 commits 自 main
- 5 files changed (+120, -0)
- 变更类型: Frontend + Backend

🤖 生成 PR 描述...

#### Summary
- [feat] 用户资料模块: 新增个人资料组件和 API 端点

#### Test plan
✓ 前端测试
  - [ ] Profile 组件渲染正常
  - [ ] 数据展示正确
  - [ ] 响应式布局工作

✓ 后端测试
  - [ ] /api/profile 端点返回正确
  - [ ] 数据验证工作正常

✓ 集成测试
  - [ ] 前后端数据流正确
  - [ ] 错误处理符合预期

🚀 推送分支到 origin...
✅ PR 创建成功: https://github.com/owner/repo/pull/123
```

---

## Scenario 2: Bug Fix with Issue Reference

**场景描述**: 修复已报告的 bug，关联 issue

### 工作流

```bash
# 1. 创建 hotfix 分支
git checkout -b hotfix/cart-calculation

# 2. 修复 bug
echo "fix calculation" >> backend/cart.py
git add . && git commit -m "fix: correct discount calculation in cart (closes #456)"

# 3. 添加测试
echo "test case" >> tests/test_cart.py
git add . && git commit -m "test: add test for discount calculation"

# 4. 触发 skill
# 用户输入: "创建 PR 修复 #456"
```

### 预期输出

```markdown
✅ 执行前检查通过

📊 收集信息:
- 2 commits
- 识别关联 issue: #456

🤖 生成 PR 描述...

#### Summary
- [fix] 修复购物车折扣计算错误（关联 issue #456）
- [test] 添加折扣计算的测试用例

#### Context
**Bug 原因**: 折扣金额计算时未考虑商品数量
**影响范围**: 购物车结算功能

#### Test plan
✓ Bug 修复验证
  - [ ] 复现原始 bug 场景（已修复）
  - [ ] 多商品折扣计算正确
  - [ ] 边界条件测试通过

✓ 回归测试
  - [ ] 购物车其他功能未受影响
  - [ ] 新增测试通过

🚀 PR 创建成功: https://github.com/owner/repo/pull/124
```

---

## Scenario 3: Draft PR for Work in Progress

**场景描述**: 创建草稿 PR 以便早期反馈

### 工作流

```bash
# 1. 创建分支并部分完成工作
git checkout -b wip/payment-integration

echo "partial implementation" >> src/payment.js
git add . && git commit -m "wip: payment integration (incomplete)"

# 2. 触发 skill 创建草稿 PR
# 用户输入: "创建草稿 PR"
```

### 预期输出

```markdown
✅ 检测到草稿 PR 请求

🤖 生成 PR 描述...

#### Summary
- [WIP] 支付集成（进行中）

#### Current Status
- ✅ 基础架构搭建完成
- 🚧 API 集成进行中
- ⏳ 测试待添加

#### Next Steps
- [ ] 完成 Stripe API 集成
- [ ] 添加错误处理
- [ ] 编写单元测试

🚀 创建草稿 PR...
✅ Draft PR 创建成功: https://github.com/owner/repo/pull/125
💡 完成后运行 `gh pr ready 125` 标记为 ready
```

---

## Scenario 4: Large Refactoring PR

**场景描述**: 大规模重构代码，无功能变更

### 工作流

```bash
# 1. 创建 refactor 分支
git checkout -b refactor/auth-module

# 2. 重构代码（多个 commits）
git commit -am "refactor: extract auth logic to separate module"
git commit -am "refactor: simplify token validation"
git commit -am "refactor: improve error handling"
git commit -am "style: apply linter rules"

# 3. 触发 skill
# 用户输入: "创建 PR"
```

### 预期输出

```markdown
📊 分析 4 个 commits...
🎯 检测到重构类型 PR

#### Summary
- [refactor] 认证模块重构: 提取逻辑、简化验证、改进错误处理

#### Goals
- 提升代码可维护性（模块化架构）
- 降低代码复杂度（圈复杂度从 15 → 8）
- 无行为变更（功能保持一致）

#### Test plan
✓ 行为一致性
  - [ ] 所有现有测试通过（0 修改）
  - [ ] 输出结果与重构前一致
  - [ ] 性能无退化

✓ 代码质量
  - [ ] Linter 无新增警告
  - [ ] 代码复杂度降低（verified）
  - [ ] 重复代码减少

🚀 PR 创建成功: https://github.com/owner/repo/pull/126
```

---

## Scenario 5: Fork Contribution

**场景描述**: 向开源项目贡献代码（fork 工作流）

### 工作流

```bash
# 1. Fork 项目并克隆
git clone https://github.com/your-username/open-source-project.git
cd open-source-project

# 2. 添加 upstream remote
git remote add upstream https://github.com/original/open-source-project.git

# 3. 创建 feature 分支
git checkout -b feature/add-dark-mode

# 4. 实现功能并提交
git commit -am "feat: add dark mode support"

# 5. 触发 skill
# 用户输入: "创建 PR 到 upstream"
```

### 预期输出

```markdown
✅ 检测到 fork 工作流
🔍 Upstream: original/open-source-project
🎯 Base branch: main (from upstream)

#### Summary
- [feat] 添加暗色模式支持

#### Implementation
- 新增主题切换组件
- 支持系统偏好设置检测
- 持久化用户选择

#### Test plan
✓ 功能测试
  - [ ] 主题切换正常工作
  - [ ] 系统偏好设置检测正确
  - [ ] 刷新后主题保持

✓ 浏览器兼容性
  - [ ] Chrome/Edge 显示正常
  - [ ] Firefox 显示正常
  - [ ] Safari 显示正常

🚀 创建 PR 到 upstream...
✅ PR 创建成功: https://github.com/original/open-source-project/pull/789
```

---

## Scenario 6: Documentation Update

**场景描述**: 更新项目文档

### 工作流

```bash
# 1. 创建 docs 分支
git checkout -b docs/update-api-docs

# 2. 更新文档
echo "updated" >> docs/API.md
git add . && git commit -m "docs: update API documentation with new endpoints"

echo "examples" >> docs/examples/auth.md
git add . && git commit -m "docs: add authentication examples"

# 3. 触发 skill
# 用户输入: "创建 PR"
```

### 预期输出

```markdown
🎯 检测到文档变更 PR

#### Summary
- [docs] 更新 API 文档: 新增端点说明和认证示例

#### Changes
- 新增: /api/v2/users 端点文档
- 新增: OAuth 认证示例
- 修订: 错误码说明

#### Test plan
✓ 文档质量
  - [ ] Markdown 格式正确（无 broken links）
  - [ ] 代码示例可运行
  - [ ] 截图清晰

✓ 准确性验证
  - [ ] API 端点信息与代码一致
  - [ ] 认证流程描述准确

🚀 PR 创建成功: https://github.com/owner/repo/pull/127
```

---

## Scenario 7: Performance Optimization

**场景描述**: 性能优化 PR

### 工作流

```bash
# 1. 创建 perf 分支
git checkout -b perf/optimize-db-queries

# 2. 优化代码
git commit -am "perf: add database query caching"
git commit -am "perf: optimize N+1 queries with eager loading"

# 3. 收集性能数据
echo "benchmark results" >> benchmarks/results.txt
git add . && git commit -m "docs: add benchmark results"

# 4. 触发 skill
# 用户输入: "创建 PR"
```

### 预期输出

```markdown
#### Summary
- [perf] 数据库查询优化: 添加缓存和预加载，提升 65%

#### Optimizations
1. 数据库查询缓存: Redis 缓存热点数据
2. N+1 查询优化: 使用 eager loading 减少查询次数

#### Metrics
- **Before**: 平均响应时间 850ms
- **After**: 平均响应时间 295ms
- **Improvement**: 65% 提升

#### Test plan
✓ 性能验证
  - [ ] Benchmark 测试显示 65% 提升
  - [ ] 真实场景测试符合预期
  - [ ] 缓存命中率 > 80%

✓ 功能验证
  - [ ] 功能行为无变化
  - [ ] 所有测试通过

🚀 PR 创建成功: https://github.com/owner/repo/pull/128
```

---

## Scenario 8: Monorepo Multi-Package Change

**场景描述**: Monorepo 中修改多个 packages

### 工作流

```bash
# 1. Monorepo 结构
# packages/frontend/
# packages/backend/
# packages/shared/

# 2. 创建 feature 分支
git checkout -b feature/shared-types

# 3. 修改多个 packages
echo "types" >> packages/shared/src/types.ts
git add . && git commit -m "feat(shared): add common types"

echo "import types" >> packages/frontend/src/App.tsx
git add . && git commit -m "feat(frontend): use shared types"

echo "import types" >> packages/backend/src/server.ts
git add . && git commit -m "feat(backend): use shared types"

# 4. 触发 skill
# 用户输入: "创建 PR"
```

### 预期输出

```markdown
🎯 检测到 monorepo 变更（3 packages）

#### Summary
- [feat] 添加共享类型定义并在前后端使用

#### Affected Packages
- **@monorepo/shared**: 新增通用类型定义
- **@monorepo/frontend**: 使用共享类型
- **@monorepo/backend**: 使用共享类型

#### Test plan
✓ Shared package 测试
  - [ ] 类型定义正确
  - [ ] 导出配置正确

✓ Frontend 集成测试
  - [ ] 类型导入成功
  - [ ] TypeScript 编译通过

✓ Backend 集成测试
  - [ ] 类型导入成功
  - [ ] 运行时验证正确

✓ Monorepo 整体测试
  - [ ] 所有 packages 测试通过
  - [ ] 跨 package 依赖正确

🚀 PR 创建成功: https://github.com/owner/repo/pull/129
```

---

## Scenario 9: Security Fix (Private PR)

**场景描述**: 修复安全漏洞，避免公开细节

### 工作流

```bash
# 1. 创建 security 分支
git checkout -b security/fix-sql-injection

# 2. 修复漏洞
git commit -am "fix: sanitize user input (security)"

# 3. 触发 skill（简化描述模式）
# 用户输入: "创建安全修复 PR，minimal description"
```

### 预期输出

```markdown
🔒 检测到安全修复 PR
⚠️  使用简化描述模式（避免公开漏洞细节）

#### Summary
- [security] 改进输入验证和数据清理

#### Test plan
✓ 安全验证
  - [ ] 所有输入正确清理
  - [ ] 安全扫描通过

✓ 功能验证
  - [ ] 现有功能正常工作
  - [ ] 所有测试通过

🚀 创建 PR（限制可见性）...
✅ PR 创建成功: https://github.com/owner/repo/pull/130
💡 详细漏洞信息已通过私有渠道共享
```

---

## Scenario 10: Emergency Hotfix

**场景描述**: 紧急修复生产 bug

### 工作流

```bash
# 1. 从 production 分支创建 hotfix
git checkout production
git checkout -b hotfix/critical-api-error

# 2. 快速修复
git commit -am "fix: critical API endpoint error"

# 3. 触发 skill（快速模式）
# 用户输入: "创建紧急 PR，target production"
```

### 预期输出

```markdown
🚨 检测到紧急 hotfix PR
🎯 Target: production branch

#### Summary
- [CRITICAL] 修复 API 端点严重错误

#### Urgency
- **Severity**: Critical
- **Impact**: API 端点返回 500 错误
- **Users Affected**: All users

#### Test plan
✓ 快速验证
  - [ ] API 端点返回正确
  - [ ] 错误不再重现

✓ 关键路径测试
  - [ ] 主要用户流程工作正常
  - [ ] 无新增错误

⚠️  警告: 目标分支是 production
建议:
1. 代码审查优先
2. 测试环境验证
3. 生产部署后监控

🚀 创建紧急 PR...
✅ PR 创建成功: https://github.com/owner/repo/pull/131
📢 通知相关团队进行快速审查
```

---

## Advanced Usage Patterns

### Pattern 1: Custom PR Template Integration

**场景**: 使用自定义 PR 模板

```bash
# 用户输入: "创建 PR，使用 feature 模板"

# Skill 行为:
# 1. 检测 .github/PULL_REQUEST_TEMPLATE/feature.md
# 2. 解析模板占位符
# 3. 智能填充模板内容
# 4. 保留模板结构
```

### Pattern 2: Batch PR Operations

**场景**: 为多个相关分支创建 PR

```bash
# 用户输入: "为所有 feature/* 分支创建 PR"

# Skill 行为:
# 1. 列出所有匹配的分支
# 2. 逐个分析和生成 PR 描述
# 3. 批量创建 PR
# 4. 返回所有 PR URLs
```

### Pattern 3: Intelligent Reviewer Suggestion

**场景**: 基于文件变更建议 reviewer

```bash
# 用户输入: "创建 PR 并自动分配 reviewer"

# Skill 行为:
# 1. 分析变更文件路径
# 2. 查询 CODEOWNERS 文件
# 3. 识别负责人
# 4. 自动添加 --reviewer 参数
```

### Pattern 4: PR Dependency Chain

**场景**: 创建依赖关系的多个 PR

```bash
# 用户输入: "创建 PR chain: feature-1 → feature-2 → feature-3"

# Skill 行为:
# 1. feature-1 PR base: main
# 2. feature-2 PR base: feature-1
# 3. feature-3 PR base: feature-2
# 4. 在 PR 描述中标注依赖关系
```

---

## Best Practices Demonstrated

### 1. Clear Commit Messages
- 使用 Conventional Commits 格式
- 包含 scope 和清晰的描述
- 引用相关 issue 编号

### 2. Atomic Commits
- 每个 commit 解决单一问题
- 便于 code review
- 方便 cherry-pick

### 3. Comprehensive Test Plans
- 覆盖功能测试、回归测试
- 包含手动和自动化测试
- 明确验收标准

### 4. Context-Rich Descriptions
- 解释 "why"（业务原因）
- 提供 "how"（技术实现）
- 包含性能指标（如适用）

### 5. Appropriate PR Size
- 单个 PR 不超过 500 行（推荐）
- 复杂功能拆分为多个 PR
- 使用 Draft PR 进行早期反馈

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Creating PR from main/master

```bash
# 错误
git checkout main
# 触发 skill

# 正确
git checkout -b feature/my-feature
# 开发...
# 触发 skill
```

### ❌ Mistake 2: Empty or Vague Commit Messages

```bash
# 错误
git commit -m "fix bug"
git commit -m "update code"

# 正确
git commit -m "fix: correct discount calculation in cart (closes #456)"
git commit -m "refactor: extract auth logic to separate module"
```

### ❌ Mistake 3: Creating PR with Uncommitted Changes

```bash
# 错误
echo "new code" >> file.js
# 触发 skill（工作区有未提交变更）

# 正确
echo "new code" >> file.js
git add file.js && git commit -m "feat: add new feature"
# 触发 skill
```

### ❌ Mistake 4: Not Testing Before PR

```bash
# 错误
git commit -am "feat: add feature"
# 触发 skill（未运行测试）

# 正确
git commit -am "feat: add feature"
npm test  # 或 pytest, go test, 等
# 确认测试通过后
# 触发 skill
```

---

## Integration with Development Workflow

### 典型开发流程

```
1. 需求分析
   ↓
2. 创建 feature 分支（git-workflow skill）
   ↓
3. 开发 + 提交（多个 commits）
   ↓
4. 整理 commits（git-workflow skill: fixup）
   ↓
5. 创建 PR（gh-pr-create skill） ← 此处
   ↓
6. Code review + CI checks
   ↓
7. 合并 PR（gh-workflow skill: tmg）
   ↓
8. 删除分支（git-workflow skill: td）
```

### CI/CD Integration

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  validate-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Validate PR Description
        run: |
          # 检查 PR 描述是否包含必要信息
          # gh-pr-create 生成的描述通常包含这些
          gh pr view ${{ github.event.number }} --json body -q .body | \
            grep -q "#### Summary" && \
            grep -q "#### Test plan"
```

---

## Summary

gh-pr-create skill 提供：
- ✅ 智能 PR 描述生成（基于 commits 和文件变更）
- ✅ 多种场景支持（feature/bugfix/refactor/docs/等）
- ✅ 安全检查和错误处理
- ✅ 与现有工作流集成（git-workflow, fork, monorepo）
- ✅ 灵活配置（draft/reviewer/labels/base branch）

通过这些场景，可以覆盖 95% 的日常 PR 创建需求。
