# Git-Workflow Skill for Claude Code

智能 Git topic 工作流助手，基于生产级 aliases 提供上下文感知的分支管理、安全检查和 PR 准备。

## 特性

### 🎯 核心能力

- **意图识别**：自然语言 → Git 命令的智能映射
- **安全护栏**：三阶段检查（执行前/中/后）预防误操作
- **上下文感知**：根据当前分支状态推荐合适的操作
- **学习辅助**：提供工作流指导和问题诊断

### 📦 功能模块

| 模块 | 功能 | 文档 |
|-----|------|------|
| **核心工作流** | tnr/tn/tmg/td 分支生命周期管理 | [git-topic-workflow.md](~/.claude/skills/git-workflow/references/git-topic-workflow.md) |
| **安全机制** | 三阶段检查、安全边界、错误处理 | [git-safety-mechanisms.md](~/.claude/skills/git-workflow/references/git-safety-mechanisms.md) |
| **PR 准备** | 检查清单、质量保证、PR 描述模板 | [git-pr-preparation.md](~/.claude/skills/git-workflow/references/git-pr-preparation.md) |
| **高级操作** | fixup/amend/rebase/cherry-pick | [git-advanced-operations.md](~/.claude/skills/git-workflow/references/git-advanced-operations.md) |
| **故障排查** | 冲突解决、误操作恢复、紧急救援 | [git-troubleshooting.md](~/.claude/skills/git-workflow/references/git-troubleshooting.md) |

---

## 快速开始

### 前置要求

1. **Git Aliases 配置**（必需）：
   ```bash
   # 位置: ./git/aliases.gitconfig
   # 包含 tnr, tn, tmg, td, fixup, bdf, blg 等核心命令
   ```

2. **Claude Code**（必需）：
   - 版本：支持 Skills 功能的版本
   - 配置：`~/.claude/skills/` 目录已存在

3. **可选依赖**：
   - `fzf`：交互式选择（fixup、blf、pif）
   - `ripgrep`：仓库搜索（rg、rg-all）

---

### 安装

Skill 文件已部署到：
```
~/.claude/skills/git-workflow/
├── SKILL.md
└── references/
    ├── git-topic-workflow.md
    ├── git-safety-mechanisms.md
    ├── git-pr-preparation.md
    ├── git-advanced-operations.md
    └── git-troubleshooting.md
```

触发规则已添加到：
```
~/.claude/skills/skill-rules.json
```

**验证安装**：
```bash
# 检查文件
ls ~/.claude/skills/git-workflow/

# 验证触发规则
grep -A 20 "git-workflow" ~/.claude/skills/skill-rules.json
```

---

## 使用方法

### 基础命令映射

| 用户意图 | 推荐命令 | 说明 |
|---------|---------|-----|
| "开始新功能 X" | `git tnr feature/X` | 创建并推送到远程 |
| "本地实验" | `git tn experiment/X` | 仅本地分支 |
| "完成功能" | `git tmg` | Merge 并删除分支 |
| "删除分支" | `git td [branch]` | 智能同步删除 |
| "查看差异" | `git bdf` | 与 base 分支对比 |
| "查看日志" | `git blg` | Topic 分支 commits |
| "准备 PR" | 检查清单 + 流程 | PR 准备指导 |
| "修改历史" | `git fixup` | 交互式修改 commit |
| "解决冲突" | 冲突解决工作流 | edit-unmerged 流程 |

### 触发示例

**中文触发**：
```
你："开始新功能 user-authentication"
Skill 响应：
  ✓ 当前分支: main
  ✓ 工作区: 干净
  推荐命令: git tnr feature/user-authentication
  说明: 将创建并推送分支到远程
```

**英文触发**：
```
你："finish this feature"
Skill 响应：
  ✓ 当前分支: feature/user-auth
  ✓ 工作区: 干净
  ✓ 3 commits 待合并
  推荐命令: git tmg
  说明: 将 merge --no-ff 并自动删除分支
```

**混合触发**：
```
你："查看 branch diff"
Skill 响应：
  ✓ 当前分支: feature/user-auth
  推荐命令: git bdf
  说明: 显示与 main 的全部差异
```

---

## 典型工作流

### 场景 1：完整功能开发

```bash
# 1. 创建分支
你: "开始新功能 user-auth"
→ git tnr feature/user-auth

# 2. 开发中保存进度
你: "临时保存"
→ git save "WIP: implementing login"

# 3. 查看进度
你: "查看我改了什么"
→ git bdf  # 差异
→ git blg  # 日志

# 4. 修改历史
你: "修改之前的 commit"
→ git fixup  # fzf 选择

# 5. 准备 PR
你: "准备 PR"
→ 提供检查清单
→ git blg 确认 commits
→ git bdf --stat 统计修改

# 6. 合并
你: "完成功能"
→ git tmg  # merge 并删除分支
```

---

### 场景 2：冲突解决

```bash
# Merge 冲突
你: "merge 冲突怎么办"
Skill 提供：
  1. git edit-unmerged  # 编辑冲突
  2. 手动解决冲突
  3. git add-unmerged   # 标记解决
  4. git mgc            # 继续 merge
```

---

### 场景 3：误操作恢复

```bash
# 误删分支
你: "误删分支怎么办"
Skill 提供：
  1. git reflog | grep "branch-name"
  2. 找到分支最后的 commit
  3. git checkout -b recovered <hash>
```

---

## 架构设计

### 职责边界

```
用户自然语言
    ↓ (意图识别)
Git-Workflow Skill
    ↓ (状态检查 + 命令生成)
Production Git Aliases
    ↓ (执行 + 内置安全机制)
Git 操作
```

**Skill 负责**：
- ✅ 意图理解和命令映射
- ✅ 执行前状态检查
- ✅ 工作流指导
- ✅ 问题诊断

**Aliases 负责**：
- ✅ 实际 Git 操作
- ✅ 运行时安全检查（自动 stash、同步、保护）
- ✅ 错误处理

**不做的事**：
- ❌ 不重写 aliases 逻辑（YAGNI）
- ❌ 不修改 dotfiles 配置
- ❌ 不添加新的 shell 脚本

---

### 三阶段安全检查

```
执行前 (Pre-check)
├─ 工作区状态: git working-dir-dirty
├─ 当前分支: git current-branch
├─ 远程同步: git ahead-count / behind-count
└─ 分支存在: git remote-branch

执行中 (Runtime)
└─ Aliases 内置安全机制

执行后 (Post-check)
├─ 结果验证: git status / current-branch
└─ 预期确认: 分支切换/删除/merge commit
```

---

## 触发规则

### Keywords（关键词）

**中文**：
- topic分支、功能分支、合并分支、删除分支
- 修改提交、分支差异、准备PR、git工作流
- tnr、tmg

**英文**：
- topic branch、feature branch、git workflow
- merge branch、delete branch、fixup
- branch diff、branch log

### Intent Patterns（意图模式）

```regex
(start|create|new|开始|创建).*(feature|topic|branch|功能|分支)
(merge|finish|complete|合并|完成).*(branch|feature|topic|分支|功能)
(delete|remove|clean|删除|清理).*(branch|topic|分支)
(show|view|diff|log|查看|显示).*(branch|changes|差异|修改)
(prepare|ready|check|准备|检查).*(pr|pull request)
\bfixup\b|修改.*提交|amend
(conflict|resolve|冲突|解决)
git.*(workflow|工作流)
```

---

## 统计数据

| 指标 | 数值 |
|-----|------|
| 文档数量 | 6 个 |
| 总代码行数 | 3620 行 |
| 主文档 | 366 行 |
| 核心工作流 | 724 行 |
| 安全机制 | 227 行 |
| PR 准备 | 740 行 |
| 高级操作 | 766 行 |
| 故障排查 | 797 行 |

---

## 开发历程

### Phase 1: MVP（已完成）

**目标**：验证核心价值 - 意图识别 + 安全检查

**交付物**：
- ✅ SKILL.md
- ✅ git-topic-workflow.md
- ✅ git-safety-mechanisms.md
- ✅ skill-rules.json 触发规则

**验证标准**：
- ✅ 能识别 5 种用户意图
- ✅ 状态检查覆盖 4 个维度
- ✅ 生成正确且安全的命令

---

### Phase 2: 增强（已完成）

**目标**：添加 PR 准备、历史修改、恢复指导

**交付物**：
- ✅ git-pr-preparation.md
- ✅ git-advanced-operations.md
- ✅ git-troubleshooting.md

**验证标准**：
- ✅ PR 准备检查清单完整
- ✅ Fixup/amend 引导清晰
- ✅ 冲突解决工作流可操作
- ✅ 恢复方案覆盖常见误操作

---

### Phase 3: 优化（可选）

**目标**：根据实际使用反馈改进

**计划**：
- 调整触发规则（基于效果）
- 添加更多常见场景
- 完善错误提示
- 工作流可视化（如有价值）

---

## 贡献指南

### 文件结构

```
git-workflow-skill/
├── README.md                    # 本文件
├── docs/
│   └── testing.md              # 测试验证文档
└── examples/
    └── scenarios.md            # 使用场景演示
```

### 修改 Skill

1. **修改主文档**：
   ```bash
   vim ~/.claude/skills/git-workflow/SKILL.md
   ```

2. **修改参考文档**：
   ```bash
   vim ~/.claude/skills/git-workflow/references/<document>.md
   ```

3. **修改触发规则**：
   ```bash
   vim ~/.claude/skills/skill-rules.json
   # 修改 git-workflow 条目的 keywords 或 intentPatterns
   ```

4. **验证修改**：
   ```bash
   # JSON 格式检查
   python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

   # 测试触发
   # 在 Claude Code 中测试新的关键词或意图
   ```

---

## 故障排查

### Skill 未触发

**可能原因**：
1. 关键词不匹配
2. 意图模式不匹配
3. skill-rules.json 格式错误

**排查步骤**：
```bash
# 1. 检查 JSON 格式
python3 -m json.tool ~/.claude/skills/skill-rules.json

# 2. 查看触发规则
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# 3. 尝试精确关键词
# 输入: "tnr" 或 "git workflow"
```

---

### 命令不存在

**可能原因**：
Git aliases 未配置或路径不正确

**排查步骤**：
```bash
# 1. 检查 aliases 是否加载
git config --get-regexp alias.tnr
git config --get-regexp alias.tmg

# 2. 检查 aliases 文件路径
ls ./git/aliases.gitconfig

# 3. 确认 Git 配置引用
git config --get include.path
```

---

### 状态检查失败

**可能原因**：
辅助命令（working-dir-dirty、current-branch 等）不存在

**排查步骤**：
```bash
# 测试辅助命令
git working-dir-dirty
git current-branch
git base-branch

# 如果失败，检查 aliases 配置
git config --get-regexp alias | grep "working-dir-dirty"
```

---

## 许可证

MIT License

---

## 参考资料

### 相关文档

- Git Aliases：
  Local: `git/aliases.gitconfig`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- Git Aliases Reference Manual:
  Local: `git/Git-Aliases-Reference-Manual.md`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md

### 外部资源

- Feature Branch Workflow：https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- Git Flight Rules：https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!：https://ohshitgit.com/

---

## 联系方式

如有问题或建议，请：
1. 查阅本文档的"故障排查"章节
2. 参考 Skill 文档：`~/.claude/skills/git-workflow/SKILL.md`
3. 查看 Git Aliases Reference Manual：`git/Git-Aliases-Reference-Manual.md`

---

**祝开发愉快！** 🚀
