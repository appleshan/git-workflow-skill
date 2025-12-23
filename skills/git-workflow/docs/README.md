# Git-Workflow Skill for Claude Code

Intelligent Git topic workflow assistant built on production-grade aliases, providing context-aware branch management, safety checks, and PR preparation guidance.

## Overview

Git-Workflow Skill wraps production Git aliases to provide three layers of value:

1. **Intent Recognition**: Natural language → Git command mapping
2. **Safety Guardrails**: Pre-execution state checks to prevent misoperations
3. **Learning Assistance**: Workflow guidance and problem diagnosis

**Position**: Does not rewrite aliases logic; focuses on intelligent wrapping and context awareness.

**Relationship with Aliases**:
```
User Natural Language
    ↓ (Intent Recognition)
Git-Workflow Skill
    ↓ (State Check + Command Generation)
Production Git Aliases
    ↓ (Execution + Built-in Safety Mechanisms)
Git Operations
```

---

## Features

### 📦 Feature Modules

| Module | Functionality | Documentation |
|-----|------|------|
| **Core Workflow** | tnr/tn/tmg/td branch lifecycle management | [git-topic-workflow.md](../references/git-topic-workflow.md) |
| **Safety Mechanisms** | Three-stage checks, safety boundaries, error handling | [git-safety-mechanisms.md](../references/git-safety-mechanisms.md) |
| **PR Preparation** | Checklists, quality assurance, PR description templates | [git-pr-preparation.md](../references/git-pr-preparation.md) |
| **Advanced Operations** | fixup/amend/rebase/cherry-pick | [git-advanced-operations.md](../references/git-advanced-operations.md) |
| **Troubleshooting** | Conflict resolution, misoperation recovery, emergency rescue | [git-troubleshooting.md](../references/git-troubleshooting.md) |

---

## Prerequisites

### Required: Git Aliases Configuration

```bash
# Location: ../../git/aliases.gitconfig
# Contains core commands: tnr, tn, tmg, td, fixup, bdf, blg, etc.

# Verify aliases are loaded
git config --get-regexp alias.tnr
git config --get-regexp alias.tmg
```

**Reference**:
- Local: `../../git/aliases.gitconfig`
- GitHub: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- Manual: `../../git/Git-Aliases-Reference-Manual.md`

### Optional Dependencies

- **fzf**: Interactive selection (fixup, blf, pif)
- **ripgrep**: Repository search (rg, rg-all)

---

## Installation

### Deployment Location

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

### Deploy from Repository

```bash
# Sync all files (excluding development docs)
rsync -av --exclude 'docs/' --exclude 'examples/' \
  skills/git-workflow/ ~/.claude/skills/git-workflow/

# Verify deployment
ls ~/.claude/skills/git-workflow/
```

### Verify Installation

```bash
# Check skill files
ls ~/.claude/skills/git-workflow/SKILL.md
ls ~/.claude/skills/git-workflow/references/

# Verify trigger rules
grep -A 30 "git-workflow" ~/.claude/skills/skill-rules.json

# Test Git aliases
git tnr --help  # Should show usage
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

#### Chinese Trigger

```
You: "开始新功能 user-authentication"
Skill Response:
  ✓ Current branch: main
  ✓ Working directory: clean
  Recommended command: git tnr feature/user-authentication
  Description: Will create and push branch to remote
```

#### English Trigger

```
You: "finish this feature"
Skill Response:
  ✓ Current branch: feature/user-auth
  ✓ Working directory: clean
  ✓ 3 commits pending merge
  Recommended command: git tmg
  Description: Will merge --no-ff and auto-delete branch
```

#### Mixed Trigger

```
You: "查看 branch diff"
Skill Response:
  ✓ Current branch: feature/user-auth
  Recommended command: git bdf
  Description: Shows all differences with main
```

---

## Typical Workflows

### Scenario 1: Conflict Resolution

```bash
# Merge conflict
You: "merge 冲突怎么办"
Skill provides:
  1. git edit-unmerged  # Edit conflicts
  2. Manually resolve conflicts
  3. git add-unmerged   # Mark as resolved
  4. git mgc            # Continue merge
```

### Scenario 2: Misoperation Recovery

```bash
# Accidentally deleted branch
You: "误删分支怎么办"
Skill provides:
  1. git reflog | grep "branch-name"
  2. Find the last commit of the branch
  3. git checkout -b recovered <hash>
```

### Scenario 3: Feature Development (Git-Workflow Only)

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

# 5. Final merge (after PR merged on GitHub)
You: "完成功能"
→ git tmg  # merge and delete branch
```

**Note**: For PR creation, see the [gh-pr-create skill](../../gh-pr-create/docs/README.md).

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

**What We Don't Do** (YAGNI):
- ❌ Don't rewrite aliases logic
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

**State Check Commands** (from aliases):
- `git working-dir-dirty` → Check if working directory has uncommitted changes
- `git current-branch` → Get current branch name
- `git base-branch` → Get base branch (usually main/master)
- `git ahead-count` → Number of commits ahead of remote
- `git behind-count` → Number of commits behind remote
- `git remote-branch` → Check if branch exists on remote

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

**Configuration**:
Trigger rules are defined in `~/.claude/skills/skill-rules.json`

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

## Development Guidelines

### Modifying the Skill

**1. Modify Main Document**:
```bash
vim SKILL.md
rsync -av --exclude 'docs/' --exclude 'examples/' . ~/.claude/skills/git-workflow/
```

**2. Modify Reference Documents**:
```bash
vim references/<document>.md
rsync -av --exclude 'docs/' --exclude 'examples/' . ~/.claude/skills/git-workflow/
```

**3. Modify Trigger Rules**:
```bash
vim ~/.claude/skills/skill-rules.json
# Modify "git-workflow" entry: keywords or intentPatterns

# Verify JSON format
python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

# Test triggers in Claude Code
```

### Testing

Refer to [testing.md](testing.md) for comprehensive test cases:

**Core Test Categories**:
1. **Trigger Tests**: Chinese/English/mixed keyword accuracy
2. **Functionality Tests**: Branch creation, merge, PR prep, conflict resolution
3. **State Awareness Tests**: Working directory and branch state recognition
4. **Boundary Tests**: Unrelated input, fuzzy input, partial matches

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
ls ../../git/aliases.gitconfig

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

## References

### Related Documentation

- **Git Aliases**:
  - Local: `../../git/aliases.gitconfig`
  - GitHub: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/aliases.gitconfig
- **Git Aliases Reference Manual**:
  - Local: `../../git/Git-Aliases-Reference-Manual.md`
  - GitHub: https://github.com/appleshan/dotfiles/blob/stow/git/.config/git/conf/Git-Aliases-Reference-Manual.md
- **Testing Guide**: [testing.md](testing.md)
- **Example Scenarios**: [scenarios.md](../examples/scenarios.md)

### External Resources

- Feature Branch Workflow: https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow
- Git Flight Rules: https://github.com/k88hudson/git-flight-rules
- Oh Shit, Git!: https://ohshitgit.com/

---

## Related Skills

- **gh-pr-create**: GitHub Pull Request creation skill - [README](../../gh-pr-create/docs/README.md)
- Works seamlessly with git-workflow for complete feature development cycle

---

## License

MIT License

---

## Contact

If you have questions or suggestions:
1. Refer to the "Troubleshooting" section of this document
2. Consult the Skill documentation: `~/.claude/skills/git-workflow/SKILL.md`
3. Check Git Aliases Reference Manual: `../../git/Git-Aliases-Reference-Manual.md`

---

**Happy Coding!** 🚀
