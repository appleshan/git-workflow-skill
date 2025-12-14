# Git-Workflow Skill 测试验证

本文档提供完整的测试用例，用于验证 skill 的触发规则和功能正确性。

---

## 测试环境

### 前置条件

- ✅ Claude Code 已安装
- ✅ Skill 文件已部署到 `~/.claude/skills/git-workflow/`
- ✅ skill-rules.json 已添加 git-workflow 条目
- ✅ Git aliases 已配置

### 验证安装

```bash
# 1. 检查 Skill 文件
ls ~/.claude/skills/git-workflow/
# 期望输出: SKILL.md references/

ls ~/.claude/skills/git-workflow/references/
# 期望输出: 5 个 .md 文件

# 2. 检查触发规则
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json
# 期望输出: git-workflow 配置块

# 3. 验证 JSON 格式
python3 -m json.tool ~/.claude/skills/skill-workflow/skill-rules.json > /dev/null
echo $?
# 期望输出: 0 (成功)

# 4. 检查 Git aliases
git config --get-regexp alias | grep -E "tnr|tmg|td"
# 期望输出: tnr, tn, tmg, td 等命令
```

---

## 测试用例

### 1. 触发测试

#### 测试 1.1：中文关键词触发

**输入**：
```
开始新功能 user-authentication
```

**期望**：
- ✅ git-workflow skill 被触发
- ✅ 输出包含 "tnr" 或 "topic-new-remote"
- ✅ 提供命令说明

**验证点**：
- [ ] Skill 名称显示为 "git-workflow"
- [ ] 提供了具体的 git 命令
- [ ] 说明了命令的作用

---

#### 测试 1.2：英文关键词触发

**输入**：
```
start new feature authentication
```

**期望**：
- ✅ Skill 触发
- ✅ 推荐 `git tnr` 或 `git tn`
- ✅ 解释两者区别

---

#### 测试 1.3：混合触发

**输入**：
```
查看 branch diff
```

**期望**：
- ✅ Skill 触发
- ✅ 推荐 `git bdf` 或 `git diff`
- ✅ 根据当前状态给出建议

---

#### 测试 1.4：直接命令触发

**输入**：
```
tnr
```

**期望**：
- ✅ Skill 触发
- ✅ 提供 tnr 命令详解
- ✅ 列出使用场景

---

#### 测试 1.5：意图模式触发

**输入**：
```
完成这个功能
```

**期望**：
- ✅ Skill 触发
- ✅ 推荐 `git tmg`
- ✅ 执行前检查清单

---

### 2. 功能测试

#### 测试 2.1：创建分支指导

**场景**：用户想创建新分支

**输入**：
```
开始新功能 user-profile
```

**期望输出结构**：
```
✓ 当前状态检查
  - 当前分支: main
  - 工作区: 干净/脏

✓ 推荐命令
  - git tnr feature/user-profile (推送到远程)
  - git tn feature/user-profile (仅本地)

✓ 说明
  - tnr 适用于团队协作
  - tn 适用于实验性开发
```

**验证点**：
- [ ] 检查了当前状态
- [ ] 提供了两种选择（tnr vs tn）
- [ ] 说明了适用场景

---

#### 测试 2.2：合并分支指导

**场景**：功能开发完成，准备合并

**输入**：
```
完成功能
```

**期望输出结构**：
```
✓ 安全检查
  - 当前分支: feature/user-profile (不能是 main)
  - 工作区: 必须干净
  - 未推送 commits: 3 个

✓ 推荐命令
  - git tmg

✓ 执行流程
  1. 切换到 base 分支
  2. Merge --no-ff
  3. 自动删除 topic 分支
```

**验证点**：
- [ ] 检查了安全条件
- [ ] 说明了执行流程
- [ ] 提醒了 --no-ff 的意义

---

#### 测试 2.3：PR 准备指导

**输入**：
```
准备 PR
```

**期望输出结构**：
```
✓ 代码审查清单
  - git blg (查看 commits)
  - git bdf (查看差异)

✓ 质量检查
  - 测试通过？
  - Lint 通过？
  - 文档更新？

✓ 同步检查
  - 与 base 分支同步
  - 推送到远程
```

**验证点**：
- [ ] 提供了完整检查清单
- [ ] 列出了具体命令
- [ ] 说明了检查项

---

#### 测试 2.4：冲突解决指导

**输入**：
```
解决 merge 冲突
```

**期望输出结构**：
```
✓ 冲突解决工作流
  1. git edit-unmerged (编辑冲突文件)
  2. 手动解决冲突
  3. git add-unmerged (标记已解决)
  4. git mgc (继续 merge)

✓ 查看冲突原因
  - git merge-log (两条分支的提交)
  - git conflict-log (只看冲突的提交)
```

**验证点**：
- [ ] 提供了分步工作流
- [ ] 列出了辅助命令
- [ ] 说明了每步的作用

---

#### 测试 2.5：误操作恢复指导

**输入**：
```
误删分支怎么办
```

**期望输出结构**：
```
✓ 恢复方法
  1. git reflog | grep "branch-name"
  2. 找到分支最后的 commit hash
  3. git checkout -b recovered <hash>

✓ 远程分支恢复
  - 如果本地还在: git push -u origin branch-name
  - 如果本地也删除: 联系管理员
```

**验证点**：
- [ ] 提供了具体恢复步骤
- [ ] 区分了本地和远程
- [ ] 给出了备选方案

---

### 3. 状态感知测试

#### 测试 3.1：工作区状态感知

**场景 1：工作区干净**

**输入**：
```
查看差异
```

**期望**：
- 如果在 topic 分支 → 推荐 `git bdf`
- 如果在 main 分支 → 提示创建新分支

---

**场景 2：工作区脏**

**输入**：
```
查看差异
```

**期望**：
- 推荐 `git diff` 或 `git dc`
- 提示有未提交修改

---

#### 测试 3.2：分支状态感知

**场景 1：在 main 分支**

**输入**：
```
完成功能
```

**期望**：
- ❌ 拒绝操作
- 提示：不能在 main 上执行 tmg
- 建议切换到 topic 分支

---

**场景 2：在 topic 分支**

**输入**：
```
完成功能
```

**期望**：
- ✅ 允许操作
- 提供安全检查清单
- 推荐 `git tmg`

---

### 4. 边界测试

#### 测试 4.1：不相关输入

**输入**：
```
今天天气怎么样
```

**期望**：
- ❌ git-workflow skill 不应触发
- 其他 skill 或默认响应

---

#### 测试 4.2：模糊输入

**输入**：
```
git 怎么用
```

**期望**：
- git-workflow skill 可能触发（包含 "git"）
- 或 claude-code-guide skill 触发（更合适）

---

#### 测试 4.3：部分匹配

**输入**：
```
branch
```

**期望**：
- git-workflow skill 可能触发（关键词匹配）
- 提供分支相关的通用指导

---

## 性能测试

### 响应时间

**测试方法**：
1. 输入触发词
2. 记录从输入到 skill 响应的时间

**期望**：
- < 2 秒：首次触发
- < 1 秒：后续触发

---

### 准确性

**测试方法**：
输入 20 个测试用例，统计触发准确率

**期望**：
- 正确触发率 ≥ 90%
- 误触发率 ≤ 5%

---

## 回归测试清单

### 修改触发规则后

- [ ] 测试 1.1 - 1.5 (触发测试)
- [ ] 测试 4.1 - 4.3 (边界测试)

### 修改 SKILL.md 后

- [ ] 测试 2.1 - 2.5 (功能测试)
- [ ] 测试 3.1 - 3.2 (状态感知测试)

### 修改参考文档后

- [ ] 相关功能测试
- [ ] 交叉引用验证

---

## 已知问题

### Issue 1：触发规则过宽

**描述**：包含 "git" 的输入可能误触发

**影响**：用户询问一般 Git 问题时可能触发 git-workflow

**缓解**：
- intentPatterns 中避免单独的 "\\bgit\\b"
- 要求组合词：git.*workflow

---

### Issue 2：中英混合触发

**描述**：部分中英混合输入可能不触发

**示例**："start 新功能"

**缓解**：
- 增加混合模式的 intentPatterns
- 或提示用户使用纯中文或纯英文

---

## 测试报告模板

```markdown
## 测试报告

**测试日期**：2025-XX-XX
**测试人员**：XXX
**Skill 版本**：1.0.0

### 测试结果

| 测试用例 | 状态 | 备注 |
|---------|------|------|
| 1.1 中文关键词触发 | ✅ Pass | - |
| 1.2 英文关键词触发 | ✅ Pass | - |
| 1.3 混合触发 | ✅ Pass | - |
| 1.4 直接命令触发 | ✅ Pass | - |
| 1.5 意图模式触发 | ✅ Pass | - |
| 2.1 创建分支指导 | ✅ Pass | - |
| 2.2 合并分支指导 | ✅ Pass | - |
| 2.3 PR 准备指导 | ✅ Pass | - |
| 2.4 冲突解决指导 | ✅ Pass | - |
| 2.5 误操作恢复指导 | ✅ Pass | - |
| 3.1 工作区状态感知 | ✅ Pass | - |
| 3.2 分支状态感知 | ✅ Pass | - |
| 4.1 不相关输入 | ✅ Pass | - |
| 4.2 模糊输入 | ⚠ Warning | 见 Issue 1 |
| 4.3 部分匹配 | ✅ Pass | - |

### 统计

- 总用例数：15
- 通过：14
- 警告：1
- 失败：0
- 通过率：93.3%

### 问题

1. Issue 1：触发规则过宽（优先级：低）

### 建议

1. 调整 intentPatterns，增加组合词要求
2. 添加更多测试用例覆盖边界情况
```

---

## 自动化测试（未来）

### 测试脚本框架

```bash
#!/bin/bash
# test-git-workflow-skill.sh

test_trigger() {
    local input="$1"
    local expected_skill="$2"

    # TODO: 调用 Claude Code API 测试触发
    # 验证返回的 skill 名称
}

test_trigger "开始新功能 X" "git-workflow"
test_trigger "今天天气怎么样" "NOT:git-workflow"
```

---

## 参考资料

- Claude Code Skills 文档
- skill-rules.json 格式规范
- Skill 测试最佳实践
