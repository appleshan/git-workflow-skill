# Git-Workflow Skill for Claude Code

Intelligent Git topic workflow assistant based on production-grade aliases, providing context-aware branch management, safety checks, and PR preparation.

## Features

### 🎯 Core Capabilities

- **Intent Recognition**: Intelligent mapping from natural language → Git commands
- **Safety Guardrails**: Three-stage checks (pre/during/post-execution) to prevent misoperations
- **Context Awareness**: Recommends appropriate actions based on current branch state
- **Learning Assistance**: Provides workflow guidance and problem diagnosis

### 📦 Feature Modules

| Module | Functionality | Documentation |
|-----|------|------|
| **Core Workflow** | tnr/tn/tmg/td branch lifecycle management | [git-topic-workflow.md](skills/git-workflow/references/git-topic-workflow.md) |
| **Safety Mechanisms** | Three-stage checks, safety boundaries, error handling | [git-safety-mechanisms.md](skills/git-workflow/references/git-safety-mechanisms.md) |
| **PR Preparation** | Checklists, quality assurance, PR description templates | [git-pr-preparation.md](skills/git-workflow/references/git-pr-preparation.md) |
| **Advanced Operations** | fixup/amend/rebase/cherry-pick | [git-advanced-operations.md](skills/git-workflow/references/git-advanced-operations.md) |
| **Troubleshooting** | Conflict resolution, misoperation recovery, emergency rescue | [git-troubleshooting.md](skills/git-workflow/references/git-troubleshooting.md) |

---

## Quick Start

### Prerequisites

1. **Git Aliases Configuration** (Required):
   ```bash
   # Location: ./git/aliases.gitconfig
   # Contains core commands: tnr, tn, tmg, td, fixup, bdf, blg, etc.
   ```

2. **Claude Code** (Required):
   - Version: Supports Skills functionality
   - Configuration: `~/.claude/skills/` directory exists

3. **Optional Dependencies**:
   - `fzf`: Interactive selection (fixup, blf, pif)
   - `ripgrep`: Repository search (rg, rg-all)

---

### Installation

Skill files are deployed to:
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

Trigger rules added to:
```
~/.claude/skills/skill-rules.json
```

**Verify Installation**:
```bash
# Check files
ls ~/.claude/skills/git-workflow/

# Verify trigger rules
grep -A 20 "git-workflow" ~/.claude/skills/skill-rules.json
```

---

## Usage

### Basic Command Mapping

| User Intent | Recommended Command | Description |
|---------|---------|-----|
| "Start new feature X" | `git tnr feature/X` | Create and push to remote |
| "Local experiment" | `git tn experiment/X` | Local branch only |
| "Finish feature" | `git tmg` | Merge and delete branch |
| "Delete branch" | `git td [branch]` | Smart synchronized deletion |
| "View diff" | `git bdf` | Compare with base branch |
| "View log" | `git blg` | Topic branch commits |
| "Prepare PR" | Checklist + workflow | PR preparation guidance |
| "Modify history" | `git fixup` | Interactive commit modification |
| "Resolve conflicts" | Conflict resolution workflow | edit-unmerged workflow |

### Trigger Examples

**Chinese Trigger**:
```
You: "开始新功能 user-authentication"
Skill Response:
  ✓ Current branch: main
  ✓ Working directory: clean
  Recommended command: git tnr feature/user-authentication
  Description: Will create and push branch to remote
```

**English Trigger**:
```
You: "finish this feature"
Skill Response:
  ✓ Current branch: feature/user-auth
  ✓ Working directory: clean
  ✓ 3 commits pending merge
  Recommended command: git tmg
  Description: Will merge --no-ff and auto-delete branch
```

**Mixed Trigger**:
```
You: "查看 branch diff"
Skill Response:
  ✓ Current branch: feature/user-auth
  Recommended command: git bdf
  Description: Shows all differences with main
```

---

## Typical Workflows

### Scenario 1: Complete Feature Development

```bash
# 1. Create branch
You: "开始新功能 user-auth"
→ git tnr feature/user-auth

# 2. Save progress during development
You: "临时保存"
→ git save "WIP: implementing login"

# 3. Check progress
You: "查看我改了什么"
→ git bdf  # Diff
→ git blg  # Log

# 4. Modify history
You: "修改之前的 commit"
→ git fixup  # fzf selection

# 5. Prepare PR
You: "准备 PR"
→ Provides checklist
→ git blg confirm commits
→ git bdf --stat statistics

# 6. Merge
You: "完成功能"
→ git tmg  # merge and delete branch
```

---

### Scenario 2: Conflict Resolution

```bash
# Merge conflict
You: "merge 冲突怎么办"
Skill provides:
  1. git edit-unmerged  # Edit conflicts
  2. Manually resolve conflicts
  3. git add-unmerged   # Mark as resolved
  4. git mgc            # Continue merge
```

---

### Scenario 3: Misoperation Recovery

```bash
# Accidentally deleted branch
You: "误删分支怎么办"
Skill provides:
  1. git reflog | grep "branch-name"
  2. Find the last commit of the branch
  3. git checkout -b recovered <hash>
```

---

## Architecture Design

### Responsibility Boundaries

```
User Natural Language
    ↓ (Intent Recognition)
Git-Workflow Skill
    ↓ (State Check + Command Generation)
Production Git Aliases
    ↓ (Execution + Built-in Safety Mechanisms)
Git Operations
```

**Skill Responsibilities**:
- ✅ Intent understanding and command mapping
- ✅ Pre-execution state checks
- ✅ Workflow guidance
- ✅ Problem diagnosis

**Aliases Responsibilities**:
- ✅ Actual Git operations
- ✅ Runtime safety checks (auto stash, sync, protection)
- ✅ Error handling

**What We Don't Do**:
- ❌ Don't rewrite aliases logic (YAGNI)
- ❌ Don't modify dotfiles configuration
- ❌ Don't add new shell scripts

---

### Three-Stage Safety Checks

```
Pre-execution (Pre-check)
├─ Working directory state: git working-dir-dirty
├─ Current branch: git current-branch
├─ Remote sync: git ahead-count / behind-count
└─ Branch existence: git remote-branch

During execution (Runtime)
└─ Aliases built-in safety mechanisms

Post-execution (Post-check)
├─ Result verification: git status / current-branch
└─ Expectation confirmation: branch switch/delete/merge commit
```

---

## Trigger Rules

### Keywords

**Chinese**:
- topic分支、功能分支、合并分支、删除分支
- 修改提交、分支差异、准备PR、git工作流
- tnr、tmg

**English**:
- topic branch, feature branch, git workflow
- merge branch, delete branch, fixup
- branch diff, branch log

### Intent Patterns

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

## Statistics

| Metric | Value |
|-----|------|
| Document Count | 6 |
| Total Lines of Code | 3620 |
| Main Document | 366 lines |
| Core Workflow | 724 lines |
| Safety Mechanisms | 227 lines |
| PR Preparation | 740 lines |
| Advanced Operations | 766 lines |
| Troubleshooting | 797 lines |

---

## Development History

### Phase 1: MVP (Completed)

**Goal**: Validate core value - intent recognition + safety checks

**Deliverables**:
- ✅ SKILL.md
- ✅ git-topic-workflow.md
- ✅ git-safety-mechanisms.md
- ✅ skill-rules.json trigger rules

**Validation Criteria**:
- ✅ Can recognize 5 types of user intents
- ✅ State checks cover 4 dimensions
- ✅ Generate correct and safe commands

---

### Phase 2: Enhancement (Completed)

**Goal**: Add PR preparation, history modification, recovery guidance

**Deliverables**:
- ✅ git-pr-preparation.md
- ✅ git-advanced-operations.md
- ✅ git-troubleshooting.md

**Validation Criteria**:
- ✅ PR preparation checklist complete
- ✅ Fixup/amend guidance clear
- ✅ Conflict resolution workflow actionable
- ✅ Recovery solutions cover common misoperations

---

### Phase 3: Optimization (Optional)

**Goal**: Improve based on actual usage feedback

**Plan**:
- Adjust trigger rules (based on effectiveness)
- Add more common scenarios
- Improve error messages
- Workflow visualization (if valuable)

---

## Contributing Guidelines

### File Structure

```
git-workflow-skill/
├── README.md                    # This file
├── docs/
│   └── testing.md              # Test verification documentation
└── examples/
    └── scenarios.md            # Usage scenario demonstrations
```

### Modifying the Skill

1. **Modify Main Document**:
   ```bash
   vim ~/.claude/skills/git-workflow/SKILL.md
   ```

2. **Modify Reference Documents**:
   ```bash
   vim ~/.claude/skills/git-workflow/references/<document>.md
   ```

3. **Modify Trigger Rules**:
   ```bash
   vim ~/.claude/skills/skill-rules.json
   # Modify keywords or intentPatterns in git-workflow entry
   ```

4. **Verify Modifications**:
   ```bash
   # JSON format check
   python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

   # Test trigger
   # Test new keywords or intents in Claude Code
   ```

---

## Troubleshooting

### Skill Not Triggering

**Possible Causes**:
1. Keywords don't match
2. Intent patterns don't match
3. skill-rules.json format error

**Troubleshooting Steps**:
```bash
# 1. Check JSON format
python3 -m json.tool ~/.claude/skills/skill-rules.json

# 2. View trigger rules
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# 3. Try exact keywords
# Input: "tnr" or "git workflow"
```

---

### Command Not Found

**Possible Causes**:
Git aliases not configured or incorrect path

**Troubleshooting Steps**:
```bash
# 1. Check if aliases are loaded
git config --get-regexp alias.tnr
git config --get-regexp alias.tmg

# 2. Check aliases file path
ls ./git/aliases.gitconfig

# 3. Confirm Git config reference
git config --get include.path
```

---

### State Check Failure

**Possible Causes**:
Helper commands (working-dir-dirty, current-branch, etc.) don't exist

**Troubleshooting Steps**:
```bash
# Test helper commands
git working-dir-dirty
git current-branch
git base-branch

# If failed, check aliases configuration
git config --get-regexp alias | grep "working-dir-dirty"
```

---

## License

MIT License

---

## References

### Related Documentation

- Git Aliases:
  Local: `git/aliases.gitconfig`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- Git Aliases Reference Manual:
  Local: `git/Git-Aliases-Reference-Manual.md`
  Github: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md

### External Resources

- Feature Branch Workflow: https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- Git Flight Rules: https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!: https://ohshitgit.com/

---

## Contact

If you have questions or suggestions:
1. Refer to the "Troubleshooting" section of this document
2. Consult the Skill documentation: `~/.claude/skills/git-workflow/SKILL.md`
3. Check Git Aliases Reference Manual: `git/Git-Aliases-Reference-Manual.md`

---

**Happy Coding!** 🚀
