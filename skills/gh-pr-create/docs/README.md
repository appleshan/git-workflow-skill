# gh-pr-create Skill for Claude Code

Intelligent GitHub Pull Request creation assistant, built on GitHub CLI for automated PR workflow.

## Overview

gh-pr-create is a Claude Code skill that simplifies and automates the GitHub Pull Request creation process. It intelligently analyzes commits and file changes to automatically generate high-quality PR descriptions.

### Core Features

- 🤖 **Smart PR Description Generation**: Auto-generates Summary and Test Plan based on commits and file changes
- 🔍 **Base Branch Detection**: Automatically identifies target branch (supports main/master/custom)
- ✅ **Safety Checks**: Pre-execution validation of gh authentication, branch status, and working directory
- 🎯 **Context Awareness**: Adjusts PR descriptions based on project type (frontend/backend/fullstack)
- 🔄 **Workflow Integration**: Seamlessly works with git-workflow skill
- 📝 **Multiple Templates**: Supports feature/bugfix/refactor/docs/perf and more

---

## Quick Start

### Prerequisites

**Required: GitHub CLI**

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install GitHub.cli
```

**Authenticate GitHub CLI**:

```bash
gh auth login
```

### Installation

**Deployment Location**:

```
~/.claude/skills/gh-pr-create/
├── SKILL.md
└── references/
    ├── pr-templates.md
    ├── gh-integration.md
    └── base-branch-detection.md
```

**Deploy from Repository**:

```bash
# Sync all files (excluding development docs)
rsync -av --exclude 'docs/' --exclude 'examples/' \
  skills/gh-pr-create/ ~/.claude/skills/gh-pr-create/

# Verify deployment
ls ~/.claude/skills/gh-pr-create/
```

**Verify Installation**:

```bash
# Check gh CLI authentication
gh auth status

# Check skill files
ls ~/.claude/skills/gh-pr-create/SKILL.md
ls ~/.claude/skills/gh-pr-create/references/

# Verify trigger rules
grep -A 20 "gh-pr-create" ~/.claude/skills/skill-rules.json
```

---

## Usage

### Basic Workflow

```bash
# 1. Develop feature and commit
git checkout -b feature/user-auth
echo "auth code" >> src/auth.js
git add . && git commit -m "feat: implement user authentication"

# 2. Trigger skill in Claude Code
# Input: "创建 PR" (Chinese)
# Or: "create pr" (English)

# 3. Skill automatically:
#   - Checks gh authentication and branch status
#   - Collects commits and file changes
#   - Generates PR description (Summary + Test Plan)
#   - Pushes branch to remote (if needed)
#   - Creates PR and returns URL
```

### Trigger Examples

#### Chinese Trigger

```
You: "创建 PR"
Skill Response:
  ✓ gh CLI authenticated
  ✓ Branch: feature/user-auth (3 commits)
  ✓ Remote: up to date

  Creating PR with auto-generated description...
  → PR created: https://github.com/user/repo/pull/123
```

#### English Trigger

```
You: "create pr"
Skill Response:
  ✓ gh CLI authenticated
  ✓ Branch: feature/api-refactor (5 commits)
  ⚠ Branch not pushed yet - will push first

  Pushing and creating PR...
  → PR created: https://github.com/user/repo/pull/124
```

### Advanced Usage

```bash
# Create draft PR
"create draft pr"

# Specify base branch
"create pr with base develop"

# Specify reviewers
"create pr, reviewers @alice @bob"

# Add labels
"create pr with labels bug urgent"
```

---

## Workflow

### Three-Stage Process

```
┌─────────────────────────────────────────────────────────────┐
│ Stage 1: Pre-flight Checks                                  │
├─────────────────────────────────────────────────────────────┤
│ ✓ Check gh CLI authentication status                        │
│ ✓ Check current branch (avoid from main/master)             │
│ ✓ Check working directory cleanliness                       │
│ ✓ Identify base branch                                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Stage 2: Information Gathering - Parallel Execution         │
├─────────────────────────────────────────────────────────────┤
│ ⚡ git status            # Working directory state          │
│ ⚡ git diff              # staged + unstaged changes         │
│ ⚡ git log base..HEAD    # commit history                    │
│ ⚡ git diff base...HEAD  # complete diff                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Stage 3: Analysis & Execution                               │
├─────────────────────────────────────────────────────────────┤
│ 🤖 Analyze all commits (extract type/scope/reason)          │
│ 📝 Generate PR description (Summary + Test Plan)            │
│ 🚀 Push branch to remote (if needed)                        │
│ ✅ Execute gh pr create                                      │
│ 🔗 Return PR URL                                             │
└─────────────────────────────────────────────────────────────┘
```

### PR Description Generation

#### Summary Section

```markdown
#### Summary
- [type] scope: change description (focus on WHY, not WHAT)

Examples:
- [feat] user auth: implement OAuth 2.0 login and JWT token management
- [fix] shopping cart: fix discount calculation error (closes #456)
- [perf] database: add query cache, 65% improvement
```

**Features**:
- Auto-detect Conventional Commits prefixes (feat/fix/docs/etc)
- Extract scope (from commit message or file paths)
- Merge similar commits (avoid verbosity)
- Limit to 3 bullet points (keep concise)

#### Test Plan Section

```markdown
#### Test plan
✓ Functional Testing
  - [ ] Core features work as required
  - [ ] Edge cases handled correctly

✓ Regression Testing
  - [ ] Existing features unaffected
  - [ ] All tests passing
```

**Smart Adjustments**:
- **Frontend projects**: Add UI testing, responsive layout checks
- **Backend projects**: Add API testing, unit test coverage
- **Bug fixes**: Emphasize regression testing and bug verification
- **Refactoring**: Emphasize behavioral consistency

---

## Integration with git-workflow

### Collaborative Workflow

```
User Development Flow:
1. git-workflow: Create topic branch (tnr/tn)
2. [Development and commits...]
3. git-workflow: Organize commits (fixup/amend)
4. gh-pr-create: Create PR ← This skill
5. [Code review + CI checks]
6. git-workflow: Merge and cleanup (tmg/td)
```

### Responsibility Division

| Skill | Scope |
|-------|---------|
| **git-workflow** | Local Git Operations<br>- Branch management (create/merge/delete)<br>- Commit organization (fixup/amend)<br>- Conflict resolution<br>- Local workspace management |
| **gh-pr-create** | GitHub Remote Operations<br>- PR creation and description generation<br>- Base branch detection<br>- gh CLI integration<br>- Fork workflow support |

---

## Typical Workflows

### Scenario 1: Standard Feature Development

```bash
git checkout -b feature/user-profile
# Development...
git commit -m "feat(profile): add user profile component"
# Trigger: "create pr"
```

### Scenario 2: Bug Fix (with Issue Reference)

```bash
git checkout -b hotfix/cart-calculation
git commit -m "fix: correct discount calculation (closes #456)"
# Trigger: "create pr"
```

### Scenario 3: Large-Scale Refactoring

```bash
git checkout -b refactor/auth-module
# Multiple refactor commits...
# Trigger: "create pr"
# Auto-generates behavioral consistency checklist
```

### Scenario 4: Fork Contribution

```bash
git remote add upstream https://github.com/original/repo.git
git checkout -b feature/dark-mode
# Trigger: "create pr to upstream"
# Auto-detects fork scenario and creates to upstream
```

More scenarios: [scenarios.md](../examples/scenarios.md)

---

## Configuration

### Base Branch Configuration

**Method 1: Git Config** (recommended for long-term branches):

```bash
# Configure base for current branch
git config branch.$(git branch --show-current).base develop

# Configure for specific branch
git config branch.feature-x.base release/v1.2
```

**Method 2: User Explicit Specification**:

```
Input: "create pr with base develop"
```

**Method 3: Auto-Detection** (default):

Priority: User specified > Git config > GitHub default branch > Common conventions (main/master/develop)

### Trigger Rules

Trigger rules are defined in `~/.claude/skills/skill-rules.json`.

**Example Keywords**:
- Chinese: "创建 PR", "开 PR", "提交 PR"
- English: "create pr", "open pr", "submit pr"

---

## Documentation

### Core Documentation

- **[SKILL.md](../SKILL.md)**: Main skill definition and core workflow
- **[testing.md](testing.md)**: Complete test verification checklist
- **[scenarios.md](../examples/scenarios.md)**: 10+ real-world usage scenarios

### Reference Documentation

- **[pr-templates.md](../references/pr-templates.md)**: PR description templates and generation strategies
- **[gh-integration.md](../references/gh-integration.md)**: GitHub CLI integration details
- **[base-branch-detection.md](../references/base-branch-detection.md)**: Base branch detection algorithm

---

## Statistics

| Metric | Value |
|-----|------|
| Document Count | 4 |
| Total Lines of Code | 2409 |
| Main Document | 468 lines |
| PR Templates | 619 lines |
| GitHub Integration | 695 lines |
| Base Branch Detection | 627 lines |

---

## Troubleshooting

### Issue 1: gh: command not found

**Cause**: gh CLI not installed

**Solution**:

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh
```

---

### Issue 2: gh: To get started with GitHub CLI, please run: gh auth login

**Cause**: Not authenticated

**Solution**:

```bash
gh auth login
```

---

### Issue 3: GraphQL: A pull request already exists for...

**Cause**: Current branch already has a PR

**Solution**:

```bash
# View existing PR
gh pr view

# Update description if needed
gh pr edit --body "$(cat new-description.md)"
```

---

### Issue 4: Base Branch Detection Error

**Cause**: Auto-detection may be inaccurate

**Solution**:

```bash
# Method 1: Configure git config
git config branch.$(git branch --show-current).base correct-base

# Method 2: Explicit specification
# Input: "create pr with base correct-base"
```

More troubleshooting: [gh-integration.md#troubleshooting](../references/gh-integration.md#troubleshooting)

---

## Best Practices

### 1. Commit Message Conventions

Use Conventional Commits format:

```
type(scope): description

type: feat, fix, docs, style, refactor, perf, test, chore
scope: module or feature changed
description: brief description (≤50 chars)
```

### 2. Commit Atomicity

Each commit solves a single problem:

```bash
# ✅ Good
git commit -m "feat: add user login"
git commit -m "feat: add JWT token management"

# ❌ Bad
git commit -m "add login and token stuff"
```

### 3. PR Size Control

Single PR recommendations:
- ≤ 20 commits
- ≤ 500 lines changed
- Focus on single feature or bug

Split large changes into multiple PRs:

```bash
# PR 1: Infrastructure
# PR 2: Core functionality
# PR 3: UI integration
```

### 4. Timely PR Creation

Create PR immediately after feature branch completion:
- Avoid backlog
- Early feedback
- Reduce merge conflicts

### 5. Use Draft PRs

Use Draft PRs for incomplete features:

```
Input: "create draft pr"
```

Mark as ready when complete:

```bash
gh pr ready <PR-number>
```

---

## Related Skills

- **[git-workflow](../../git-workflow/docs/README.md)**: Topic-based branch management with safety checks
- Works seamlessly with gh-pr-create for complete feature development cycle

---

## References

### Official Documentation

- [GitHub CLI Manual](https://cli.github.com/manual/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Pull Request Best Practices](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)

### Related Resources

- Git Workflow Skill: [../../git-workflow/docs/README.md](../../git-workflow/docs/README.md)
- Testing Guide: [testing.md](testing.md)
- Example Scenarios: [scenarios.md](../examples/scenarios.md)

---

## License

MIT License

---

## Contact

If you have questions or suggestions:
1. Check the "Troubleshooting" section
2. Consult the skill documentation: `~/.claude/skills/gh-pr-create/SKILL.md`
3. Review GitHub CLI documentation: https://cli.github.com/manual/

---

**Version**: 1.0.0
**Last Updated**: 2025-12-24
**Compatibility**: gh CLI v2.0+, git 2.20+

**Happy Coding!** 🚀
