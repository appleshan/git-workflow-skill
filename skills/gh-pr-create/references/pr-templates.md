# PR Description Templates and Generation Strategies

智能 PR 描述生成策略和模板库，支持多种项目类型和变更场景。

## Template Structure

### 标准模板结构

```markdown
#### Summary
<1-3 bullet points describing WHAT and WHY>

#### Test plan
<Actionable testing checklist>

#### Additional notes (optional)
<Context, dependencies, breaking changes, etc.>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**设计原则**:
- Summary 聚焦 "why"（业务原因）而非 "what"（代码细节）
- Test Plan 必须可操作（checkbox 形式）
- 结构化且易于 scan（reviewer 快速理解）

## Summary Generation Strategy

### 1. Commit Message 解析

**Conventional Commits 支持**:

```python
# 伪代码示意 commit 分类逻辑
def classify_commits(commits):
    patterns = {
        'feat': r'^feat(\(.+\))?: ',
        'fix': r'^fix(\(.+\))?: ',
        'docs': r'^docs(\(.+\))?: ',
        'style': r'^style(\(.+\))?: ',
        'refactor': r'^refactor(\(.+\))?: ',
        'perf': r'^perf(\(.+\))?: ',
        'test': r'^test(\(.+\))?: ',
        'chore': r'^chore(\(.+\))?: ',
    }

    classified = defaultdict(list)
    for commit in commits:
        for type, pattern in patterns.items():
            if re.match(pattern, commit.message):
                classified[type].append(commit)
                break
        else:
            classified['other'].append(commit)

    return classified
```

**Scope 提取**:

```python
# 从 commit message 或文件路径提取 scope
def extract_scope(commit):
    # 方式1: 从 conventional commit 格式提取
    match = re.search(r'\((.+?)\):', commit.message)
    if match:
        return match.group(1)

    # 方式2: 从文件路径推断
    files = commit.changed_files
    if all(f.startswith('frontend/') for f in files):
        return 'frontend'
    if all(f.startswith('backend/api') for f in files):
        return 'backend-api'
    if all(f.endswith('.md') for f in files):
        return 'docs'

    # 方式3: 使用目录名
    common_dir = os.path.commonprefix([os.path.dirname(f) for f in files])
    return os.path.basename(common_dir) or 'core'
```

### 2. 变更类型识别

**基于文件扩展名**:

| 文件模式 | 变更类型 | Summary 前缀 |
|---------|---------|------------|
| `*.md`, `docs/**` | Documentation | [docs] |
| `*test*.py`, `*_test.go` | Testing | [test] |
| `frontend/**`, `*.jsx`, `*.vue` | Frontend | [frontend] |
| `backend/**`, `*.py` (非 test) | Backend | [backend] |
| `migrations/**`, `*.sql` | Database | [database] |
| `*.config.js`, `Dockerfile`, `.github/**` | Infrastructure | [infra] |

**基于变更规模**:

- 小型变更 (< 50 lines): "Minor update to..."
- 中型变更 (50-500 lines): "Enhance/Fix..."
- 大型变更 (> 500 lines): "Major refactor of..."

### 3. Summary Bullet Points 生成

**单 Commit 场景**:

```markdown
#### Summary
- [type] scope: <直接使用 commit message 的第一行>
```

**多 Commit 同类型场景**:

```markdown
#### Summary
- [feat] 用户认证模块: 实现 OAuth 登录、JWT token 管理和权限验证
- [test] 添加认证模块的单元测试和集成测试
```

**多 Commit 混合类型场景**:

```markdown
#### Summary
- [feat] 新增产品推荐功能（基于用户行为分析）
- [fix] 修复购物车价格计算错误
- [perf] 优化首页加载速度（减少 API 调用）
```

**合并逻辑**:

```python
def merge_similar_commits(commits):
    """合并相似的 commits 到单个 bullet point"""
    # 按 type 和 scope 分组
    groups = group_by(commits, key=lambda c: (c.type, c.scope))

    bullets = []
    for (type, scope), group_commits in groups.items():
        if len(group_commits) == 1:
            bullets.append(f"[{type}] {scope}: {group_commits[0].summary}")
        else:
            # 合并多个 commits
            reasons = [c.extract_reason() for c in group_commits]
            merged = f"[{type}] {scope}: {', '.join(reasons[:3])}"
            bullets.append(merged)

    return bullets[:3]  # 限制 3 个 bullets
```

## Test Plan Generation Strategy

### 1. 基于项目类型生成

#### Frontend 项目

```markdown
#### Test plan
✓ UI 功能验证
  - [ ] 新增组件渲染正常
  - [ ] 交互逻辑符合预期（点击、输入、滚动）
  - [ ] 样式在不同浏览器显示一致

✓ 响应式布局测试
  - [ ] 移动端显示正常（iOS Safari, Android Chrome）
  - [ ] 桌面端显示正常（Chrome, Firefox, Safari）
  - [ ] 不同屏幕尺寸适配正确

✓ 性能检查
  - [ ] Lighthouse 分数 ≥ 90（Performance）
  - [ ] 无 console errors/warnings
```

#### Backend 项目

```markdown
#### Test plan
✓ API 端点测试
  - [ ] 所有新增 endpoints 返回正确状态码
  - [ ] 请求/响应数据格式符合 schema
  - [ ] 错误处理符合预期（4xx/5xx）

✓ 单元测试
  - [ ] 新增代码测试覆盖率 ≥ 80%
  - [ ] 所有测试通过（pytest/go test/jest）

✓ 集成测试
  - [ ] 与数据库交互正常
  - [ ] 与外部 API 集成工作正常
```

#### Fullstack 项目

```markdown
#### Test plan
✓ 前端测试
  - [ ] UI 组件功能验证
  - [ ] 前后端数据流正确

✓ 后端测试
  - [ ] API 端点功能正常
  - [ ] 数据库操作正确

✓ 端到端测试
  - [ ] 完整用户流程测试（注册 → 登录 → 操作）
  - [ ] 跨域/认证/会话管理正常
```

### 2. 基于变更类型生成

#### Bug Fix

```markdown
#### Test plan
✓ 回归测试
  - [ ] 原始 bug 场景已修复（验证 issue #X）
  - [ ] 相关功能未受影响
  - [ ] 边界条件处理正确

✓ 单元测试
  - [ ] 添加测试用例覆盖 bug 场景
  - [ ] 所有现有测试仍然通过
```

#### New Feature

```markdown
#### Test plan
✓ 功能测试
  - [ ] 核心功能按需求工作
  - [ ] 边界条件处理正确
  - [ ] 错误处理符合预期

✓ 集成测试
  - [ ] 与现有功能集成无冲突
  - [ ] 依赖的服务/模块工作正常

✓ 性能测试
  - [ ] 响应时间符合 SLA
  - [ ] 资源使用（内存/CPU）在合理范围
```

#### Refactoring

```markdown
#### Test plan
✓ 行为一致性验证
  - [ ] 所有现有测试通过（无测试修改）
  - [ ] 输出结果与重构前一致

✓ 代码质量检查
  - [ ] Linter 无新增警告
  - [ ] 代码复杂度降低（cyclomatic complexity）
  - [ ] 重复代码减少（DRY 原则）
```

#### Database Migration

```markdown
#### Test plan
✓ 迁移测试
  - [ ] 正向迁移成功（migrate up）
  - [ ] 回滚迁移成功（migrate down）
  - [ ] 数据完整性保持（无数据丢失）

✓ 性能测试
  - [ ] 迁移时间可接受（< N 秒）
  - [ ] 查询性能未退化（对比前后 EXPLAIN）

✓ 兼容性测试
  - [ ] 现有代码兼容新 schema
  - [ ] 部署时零停机（如适用）
```

### 3. 智能增强

**检测测试文件变更**:

```python
def enhance_test_plan(changed_files):
    test_files = [f for f in changed_files if is_test_file(f)]

    if test_files:
        # 如果已有测试文件变更，优先级调整
        return [
            "✓ 运行新增测试用例（验证通过）",
            "✓ 回归测试（所有测试通过）",
        ]
```

**检测 breaking changes**:

```python
def detect_breaking_changes(git_diff):
    patterns = [
        r'remove.*function',
        r'rename.*class',
        r'change.*API.*endpoint',
        r'modify.*database.*schema',
    ]

    if any(re.search(p, git_diff, re.I) for p in patterns):
        return [
            "⚠️  Breaking Changes 验证",
            "  - [ ] 更新所有依赖此变更的代码",
            "  - [ ] 更新文档和 CHANGELOG",
            "  - [ ] 通知相关团队",
        ]
```

## Template Library

### Template 1: Feature Addition

**适用场景**: 新功能开发

```markdown
#### Summary
- [feat] {scope}: 实现 {功能描述}，支持 {核心能力1}、{核心能力2}
- [test] 添加 {scope} 的单元测试和集成测试
- [docs] 更新 {scope} 的 API 文档和使用指南

#### Test plan
✓ 功能验证
  - [ ] {核心功能1} 工作正常
  - [ ] {核心功能2} 工作正常
  - [ ] 边界条件处理正确

✓ 集成测试
  - [ ] 与现有 {模块A} 集成无冲突
  - [ ] 依赖的 {服务B} 调用正常

✓ 文档验证
  - [ ] README/API docs 更新准确
  - [ ] 示例代码可运行

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Template 2: Bug Fix

**适用场景**: 修复线上 bug

```markdown
#### Summary
- [fix] {scope}: 修复 {bug 描述}（关联 issue #{issue_number}）
- [test] 添加回归测试防止 bug 重现

#### Context
**Bug 原因**: {简短说明根本原因}
**影响范围**: {影响的用户/功能}

#### Test plan
✓ Bug 修复验证
  - [ ] 复现原始 bug 场景（已修复）
  - [ ] 相关边界条件测试通过

✓ 回归测试
  - [ ] {功能A} 未受影响
  - [ ] {功能B} 未受影响

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Template 3: Refactoring

**适用场景**: 代码重构

```markdown
#### Summary
- [refactor] {scope}: 重构 {模块/功能}，优化 {改进点}

#### Goals
- 提升代码可维护性（{具体改进}）
- 降低代码复杂度（{具体指标}）
- 无行为变更（功能保持一致）

#### Test plan
✓ 行为一致性
  - [ ] 所有现有测试通过（0 修改）
  - [ ] 输出结果与重构前一致

✓ 代码质量
  - [ ] Linter/静态分析无新增问题
  - [ ] 代码复杂度降低（cyclomatic < {N}）

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Template 4: Performance Optimization

**适用场景**: 性能优化

```markdown
#### Summary
- [perf] {scope}: 优化 {性能指标}，提升 {X}%

#### Optimizations
1. {优化点1}: {具体措施}
2. {优化点2}: {具体措施}

#### Metrics
- **Before**: {baseline 指标}
- **After**: {优化后指标}
- **Improvement**: {提升百分比}

#### Test plan
✓ 性能验证
  - [ ] Benchmark 测试显示 {X}% 提升
  - [ ] 真实场景测试符合预期

✓ 功能验证
  - [ ] 功能行为无变化
  - [ ] 所有测试通过

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Template 5: Documentation Update

**适用场景**: 纯文档变更

```markdown
#### Summary
- [docs] {scope}: 更新 {文档类型}，{改进描述}

#### Changes
- 新增: {新增内容}
- 修订: {修订内容}
- 删除: {删除内容}（如适用）

#### Test plan
✓ 文档质量
  - [ ] Markdown 格式正确（无 broken links）
  - [ ] 代码示例可运行
  - [ ] 截图/图表清晰

✓ 准确性验证
  - [ ] 技术细节准确
  - [ ] 与当前代码版本一致

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Template 6: Dependency Update

**适用场景**: 依赖库升级

```markdown
#### Summary
- [chore] 升级 {dependency} 从 {old_version} 到 {new_version}

#### Motivation
{升级原因: 安全修复/新功能/性能提升/等}

#### Breaking Changes
{列出 breaking changes，如无则写 "无"}

#### Test plan
✓ 兼容性验证
  - [ ] 所有测试通过
  - [ ] 无 deprecation warnings

✓ 功能验证
  - [ ] 核心功能正常
  - [ ] 依赖此库的功能无影响

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

## Advanced Generation Techniques

### 1. 多语言支持

**检测项目主语言**:

```bash
# 使用 GitHub Linguist 或文件扩展名统计
primary_language=$(gh repo view --json languages -q '.languages | to_entries | max_by(.value) | .key')

case $primary_language in
    Python) framework_hint="Django/FastAPI/Flask" ;;
    JavaScript|TypeScript) framework_hint="React/Vue/Node.js" ;;
    Go) framework_hint="Gin/Echo/std lib" ;;
    *) framework_hint="N/A" ;;
esac
```

**生成语言特定的测试建议**:

- Python: `pytest`, `coverage run`
- JavaScript: `npm test`, `jest`
- Go: `go test -race ./...`
- Ruby: `rspec`, `bundle exec rake test`

### 2. 上下文感知

**检测 CI/CD 配置**:

```python
ci_files = ['.github/workflows/', '.gitlab-ci.yml', 'Jenkinsfile']
if any(exists(f) for f in ci_files):
    test_plan.append("✓ CI/CD 管道通过（所有 checks green）")
```

**检测代码质量工具**:

```python
quality_tools = {
    '.eslintrc': 'ESLint',
    'pyproject.toml': 'Ruff/Black',
    '.pre-commit-config.yaml': 'pre-commit hooks',
}

for config, tool in quality_tools.items():
    if exists(config):
        test_plan.append(f"✓ {tool} 检查通过")
```

### 3. 智能优先级排序

**Summary bullets 排序逻辑**:

1. **Breaking changes** 优先（最重要）
2. **Features** 其次（用户可见）
3. **Fixes** 第三（问题修复）
4. **Performance** 第四（性能优化）
5. **Refactor/Docs/Chore** 最后（内部改进）

**Test plan items 排序逻辑**:

1. **核心功能测试** 优先
2. **回归测试** 其次
3. **性能/安全测试** 第三
4. **文档/格式检查** 最后

## Best Practices

### 1. Summary 编写原则

- ✅ **DO**: "实现用户登录功能（支持邮箱和手机号）"
- ❌ **DON'T**: "添加 login.js 文件和 auth 逻辑"

- ✅ **DO**: "修复购物车价格计算错误（折扣未正确应用）"
- ❌ **DON'T**: "修改 cart.py 第 42 行代码"

### 2. Test Plan 编写原则

- ✅ **DO**: "[ ] 使用 Postman 测试 /api/login 端点（正常和异常场景）"
- ❌ **DON'T**: "[ ] 测试登录"

- ✅ **DO**: "[ ] 验证折扣 ≥ 100 元时免运费（购物车总额测试）"
- ❌ **DON'T**: "[ ] 检查代码"

### 3. 长度控制

- Summary: 3-5 bullet points（单个 commit 可以 1 个）
- Test Plan: 5-10 items（复杂 PR 可以更多，但分组）
- 单个 bullet/item: ≤ 100 characters（易于扫描）

### 4. 模板选择决策树

```
Start
  ├─ 仅文档变更? → Template 5 (Documentation)
  ├─ 依赖升级? → Template 6 (Dependency Update)
  ├─ 性能优化? → Template 4 (Performance)
  ├─ 代码重构（无功能变更）? → Template 3 (Refactoring)
  ├─ Bug 修复? → Template 2 (Bug Fix)
  └─ 新功能/增强? → Template 1 (Feature Addition)
```

## Integration with Skill Logic

在 `SKILL.md` 中调用模板生成逻辑的位置：

```markdown
步骤 3: 智能分析与执行
    ├── 分析所有 commits
    ├── **调用 pr-templates.md 的生成策略** ← 此处
    ├── 生成 PR 描述
    └── 执行 gh pr create
```

**接口设计**:

```python
def generate_pr_description(
    commits: List[Commit],
    changed_files: List[str],
    base_branch: str,
    project_type: str
) -> PRDescription:
    """
    主入口：根据 commits 和文件变更生成 PR 描述

    返回:
        PRDescription(summary, test_plan, additional_notes)
    """
    # 1. 选择模板
    template = select_template(commits, changed_files)

    # 2. 生成 Summary
    summary = generate_summary(commits, changed_files)

    # 3. 生成 Test Plan
    test_plan = generate_test_plan(changed_files, project_type)

    # 4. 可选的额外信息
    additional = generate_additional_notes(commits, changed_files)

    return PRDescription(summary, test_plan, additional)
```

## Future Enhancements

1. **机器学习增强**: 分析历史 PR 学习项目特定的描述风格
2. **多语言描述**: 根据团队偏好生成中文/英文描述
3. **自动 reviewer 推荐**: 基于文件变更建议合适的 reviewer
4. **影响范围分析**: 自动识别 downstream dependencies
5. **风险评分**: 根据变更规模和类型评估 PR 风险等级
