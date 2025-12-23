# Git-Workflow Skills for Claude Code

Two intelligent Git workflow assistants that streamline your development process:

- **git-workflow**: Topic-based branch management with safety checks and workflow guidance
- **gh-pr-create**: Automated GitHub Pull Request creation with smart change analysis

## Features

### 🎯 Core Capabilities

- **Intent Recognition**: Intelligent mapping from natural language → Git commands
- **Safety Guardrails**: Three-stage checks (pre/during/post-execution) to prevent misoperations
- **Context Awareness**: Recommends appropriate actions based on current branch state
- **Learning Assistance**: Provides workflow guidance and problem diagnosis

### 📦 Skills Overview

#### [git-workflow](skills/git-workflow/docs/README.md)

Topic-based branch management with safety checks and workflow guidance.

**Key Features**: tnr/tn/tmg/td lifecycle | Three-stage safety checks | PR preparation | Advanced operations (fixup/amend) | Conflict resolution

**[→ Full Documentation](skills/git-workflow/docs/README.md)**

#### [gh-pr-create](skills/gh-pr-create/docs/README.md)

Automated GitHub Pull Request creation with smart change analysis.

**Key Features**: Auto-generate PR descriptions | Smart base branch detection | gh CLI integration | Structured templates

**[→ Full Documentation](skills/gh-pr-create/docs/README.md)**

---

## Quick Start

### Prerequisites

1. **Git Aliases Configuration** (Required for git-workflow):
   ```bash
   # Location: ./git/aliases.gitconfig
   # Contains core commands: tnr, tn, tmg, td, fixup, bdf, blg, etc.
   ```

2. **GitHub CLI** (Required for gh-pr-create):
   ```bash
   # Install gh CLI
   # macOS: brew install gh
   # Linux: see https://github.com/cli/cli#installation

   # Authenticate
   gh auth login
   ```

3. **Claude Code** (Required):
   - Version: Supports Skills functionality
   - Configuration: `~/.claude/skills/` directory exists

4. **Optional Dependencies**:
   - `fzf`: Interactive selection (fixup, blf, pif)
   - `ripgrep`: Repository search (rg, rg-all)

---

### Installation

Skills are deployed to:
```
~/.claude/skills/git-workflow/
├── SKILL.md
└── references/
    ├── git-topic-workflow.md
    ├── git-safety-mechanisms.md
    ├── git-pr-preparation.md
    ├── git-advanced-operations.md
    └── git-troubleshooting.md

~/.claude/skills/gh-pr-create/
├── SKILL.md
└── references/
    ├── pr-templates.md
    ├── gh-integration.md
    └── base-branch-detection.md
```

Trigger rules added to:
```
~/.claude/skills/skill-rules.json
```

**Verify Installation**:
```bash
# Check git-workflow files
ls ~/.claude/skills/git-workflow/

# Check gh-pr-create files
ls ~/.claude/skills/gh-pr-create/

# Verify trigger rules
grep -A 20 "git-workflow" ~/.claude/skills/skill-rules.json
grep -A 20 "gh-pr-create" ~/.claude/skills/skill-rules.json

# Verify gh CLI authentication (for gh-pr-create)
gh auth status
```

---

## Usage

### Quick Examples

**git-workflow**: Natural language commands for branch management
```
"开始新功能 user-auth" → Creates and pushes feature branch
"完成功能" → Merges and cleans up branch
"查看 branch diff" → Shows changes vs base branch
```

**gh-pr-create**: Automated PR creation
```
"创建 PR" → Analyzes commits, generates description, creates PR
"create pull request" → Same with auto-push if needed
```

**[→ git-workflow Full Usage Guide](skills/git-workflow/docs/README.md)**<br>
**[→ gh-pr-create Full Usage Guide](skills/gh-pr-create/docs/README.md)**

---

## Typical Workflow

### Complete Feature Development (End-to-End)

Combining both skills for a full development cycle:

```bash
# 1. Create branch (git-workflow)
You: "开始新功能 user-auth"
→ git tnr feature/user-auth

# 2. Save progress during development (git-workflow)
You: "临时保存"
→ git save "WIP: implementing login"

# 3. Check progress (git-workflow)
You: "查看我改了什么"
→ git bdf  # Diff
→ git blg  # Log

# 4. Modify history (git-workflow)
You: "修改之前的 commit"
→ git fixup  # fzf selection

# 5. Create PR (gh-pr-create)
You: "创建 PR"
→ Analyzes all commits and file changes
→ Generates structured PR description (Summary + Test Plan)
→ Pushes branch if needed
→ Creates PR: https://github.com/user/repo/pull/123

# 6. After PR merged on GitHub, cleanup (git-workflow)
You: "完成功能"
→ git tmg  # merge and delete branch
```

**More Scenarios**:
- **[git-workflow scenarios](skills/git-workflow/docs/README.md#typical-workflows)**: Conflict resolution, misoperation recovery
- **[gh-pr-create scenarios](skills/gh-pr-create/docs/README.md#typical-workflows)**: PR templates, base branch detection

---

## Architecture

### Design Principles

**Separation of Concerns**:
- **Skills Layer**: Intent recognition, safety checks, guidance
- **Aliases Layer**: Git operations, runtime safety, error handling

**YAGNI Approach**:
- Don't rewrite aliases logic
- Focus on intelligent wrapping and context awareness

**[→ Full Architecture Documentation](skills/git-workflow/docs/README.md#architecture-design)**

---

## Trigger Rules

Trigger rules are defined in `~/.claude/skills/skill-rules.json`.

### git-workflow

**Example Keywords**: "开始新功能", "完成功能", "git workflow", "branch diff", "fixup"

**[→ Complete Trigger Rules](skills/git-workflow/docs/README.md#trigger-rules)**

### gh-pr-create

**Example Keywords**: "创建 PR", "create pr", "open pull request"

**[→ Complete Trigger Rules](skills/gh-pr-create/docs/README.md#trigger-rules)**

---

## Project Statistics

| Metric | Value |
|-----|------|
| Total Skills | 2 |
| Total Documents | 10 |
| Total Lines | 6029 |

**Breakdown**:
- **[git-workflow](skills/git-workflow/docs/README.md#statistics)**: 6 documents, 3620 lines
- **[gh-pr-create](skills/gh-pr-create/docs/README.md#statistics)**: 4 documents, 2409 lines

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
├── README.md                    # This file (English)
├── README_zh-CN.md             # Chinese version
├── skills/
│   ├── git-workflow/           # Git workflow skill
│   │   ├── SKILL.md
│   │   ├── references/
│   │   ├── docs/
│   │   │   ├── README.md       # Development documentation (English)
│   │   │   ├── README_zh-CN.md # Development documentation (Chinese)
│   │   │   └── testing.md
│   │   └── examples/scenarios.md
│   └── gh-pr-create/           # GitHub PR creation skill
│       ├── SKILL.md
│       ├── references/
│       ├── docs/
│       │   ├── README.md       # Development documentation (English)
│       │   ├── README_zh-CN.md # Development documentation (Chinese)
│       │   └── testing.md
│       └── examples/scenarios.md
└── git/                         # Git aliases configuration
    ├── aliases.gitconfig
    └── Git-Aliases-Reference-Manual.md
```

### Modifying the Skills

**git-workflow Skill**:

1. **Modify Main Document**:
   ```bash
   vim skills/git-workflow/SKILL.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/
   ```

2. **Modify Reference Documents**:
   ```bash
   vim skills/git-workflow/references/<document>.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/git-workflow/ ~/.claude/skills/git-workflow/
   ```

**gh-pr-create Skill**:

1. **Modify Main Document**:
   ```bash
   vim skills/gh-pr-create/SKILL.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/
   ```

2. **Modify Reference Documents**:
   ```bash
   vim skills/gh-pr-create/references/<document>.md
   rsync -av --exclude 'docs/' --exclude 'examples/' skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/
   ```

3. **Verify gh CLI**:
   ```bash
   gh auth status
   ```

**Trigger Rules**:

1. **Modify Trigger Rules**:
   ```bash
   vim ~/.claude/skills/skill-rules.json
   # Modify keywords or intentPatterns for git-workflow or gh-pr-create
   ```

2. **Verify Modifications**:
   ```bash
   # JSON format check
   python3 -m json.tool ~/.claude/skills/skill-rules.json > /dev/null

   # Test triggers in Claude Code
   ```

---

## Troubleshooting

### Common Issues

**Skills Not Triggering**:
```bash
# Verify trigger rules
grep -E "git-workflow|gh-pr-create" ~/.claude/skills/skill-rules.json

# Verify skill files exist
ls ~/.claude/skills/git-workflow/
ls ~/.claude/skills/gh-pr-create/
```

**Git Aliases Not Found**:
```bash
# Check if aliases are loaded
git config --get-regexp alias.tnr

# Verify aliases file
ls ./git/aliases.gitconfig
```

**gh CLI Issues**:
```bash
# Authenticate with GitHub
gh auth login

# Verify authentication
gh auth status
```

**Detailed Troubleshooting**:
- **[git-workflow troubleshooting](skills/git-workflow/docs/README.md#troubleshooting)**: Trigger issues, command errors, state checks
- **[gh-pr-create troubleshooting](skills/gh-pr-create/docs/README.md#troubleshooting)**: Authentication, base branch detection, PR creation

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
