# Git Worktrees with Claude Code: A Comprehensive Guide

## Table of Contents
- [What are Git Worktrees?](#what-are-git-worktrees)
- [Benefits for Claude Code Workflows](#benefits-for-claude-code-workflows)
- [Running Parallel Claude Instances](#running-parallel-claude-instances)
- [Setting Up Worktrees for Claude Workflows](#setting-up-worktrees-for-claude-workflows)
- [Managing Multiple Feature Branches Simultaneously](#managing-multiple-feature-branches-simultaneously)
- [Worktree Directory Organization](#worktree-directory-organization)
- [Context Isolation Between Instances](#context-isolation-between-instances)
- [Best Practices for Worktree-Based Workflows](#best-practices-for-worktree-based-workflows)
- [Common Patterns and Use Cases](#common-patterns-and-use-cases)

---

## What are Git Worktrees?

Git worktree is a built-in Git feature that enables you to check out multiple branches from the same repository into separate directories simultaneously. Think of it as creating additional workspaces for your project, each with its own branch checked out, while sharing the same underlying Git history and repository data.

### Key Characteristics

- **Shared Repository**: All worktrees connect to the same `.git` repository, avoiding duplication of repository data
- **Independent Working Directories**: Each worktree has its own working directory with isolated files
- **Shared History**: All worktrees share the same Git history, reflog, and remote tracking information
- **Lightweight**: Unlike full repository clones, worktrees are merely linked directories requiring minimal storage overhead

When you run `git fetch` in one worktree or rename a branch in another, changes are immediately visible across all worktrees because they share the same underlying repository structure.

---

## Benefits for Claude Code Workflows

The combination of Git worktrees and Claude Code creates a powerful parallel development environment that addresses several key bottlenecks in AI-assisted coding:

### 1. **Eliminate Context Switching Overhead**

Traditional branch switching requires developers to stash or commit incomplete work, rebuild dependencies, and mentally re-establish context. Each context switch typically costs 10-15 minutes of setup time and breaks focus. With worktrees, you simply `cd` to a different directory where another branch is already checked out, complete with its own Claude Code session maintaining full conversation history.

### 2. **True Parallel Development**

Claude Code runs entirely in your terminal, making it easy to spin up multiple instances concurrently. You can have as many Claude Code sessions running as you want, each in its own worktree, tackling different parts of your project simultaneously. This addresses what many developers consider a major bottleneck: being limited to one task at a time.

### 3. **No Interference Between Sessions**

Each worktree has its own independent file state, making it perfect for parallel Claude Code sessions. Changes made in one worktree won't affect others, preventing Claude instances from overwriting each other's edits or manipulating another agent's context.

### 4. **Persistent Session Context**

Claude Code sessions are stored per project directory. The `/resume` picker shows sessions from the same Git repository, including worktrees. You can leave a task mid-conversation and return hours or even days later—Claude remembers exactly where you left off, what you were trying to accomplish, and what approaches you had already tried.

### 5. **Faster Development Velocity**

Real-world examples demonstrate significant productivity gains. Teams report tasks that would take 2 hours of manual work being completed in 10 minutes with Claude Code in isolated worktrees. The ability to run multiple Claude agents simultaneously on different features can dramatically accelerate development timelines.

---

## Running Parallel Claude Instances

### Basic Setup

Claude Code is designed to support multiple concurrent instances. Here's how to leverage this capability effectively:

**Single Directory Workaround (Not Recommended)**
While you can technically run multiple Claude Code instances in the same working directory, Anthropic doesn't recommend this approach. All changes would be committed to a single branch, or worst case, combined into one commit at the end of your sessions, creating merge chaos.

**Separate Worktrees (Recommended)**
Instead, use Git worktrees to create isolated environments:

```bash
# Terminal 1: Main feature development
cd ~/projects/myapp
claude

# Terminal 2: Bug fix in separate worktree
cd ~/projects/myapp-bugfix
claude

# Terminal 3: Code review in another worktree
cd ~/projects/myapp-review
claude
```

### Parallel Workflow Patterns

#### Pattern 1: Code Writer + Code Reviewer

Run two Claude instances where one writes code while another reviews or tests it. This mimics collaborative engineering dynamics:

```bash
# Worktree 1: Feature development
git worktree add ../myapp-feature-auth -b feature/auth
cd ../myapp-feature-auth
claude
# Prompt: "Implement OAuth2 authentication with JWT tokens"

# Worktree 2: Code review
git worktree add ../myapp-review feature/auth
cd ../myapp-review
claude
# Prompt: "Review the authentication implementation for security issues"
```

#### Pattern 2: Multiple Repository Checkouts

Maintain 3-4 Git checkouts in separate folders, running Claude in each with different tasks. Cycle through them to manage permission requests and review changes:

```bash
~/projects/
├── myapp/              # Main worktree - ongoing feature work
├── myapp-hotfix/       # Emergency production fix
├── myapp-refactor/     # Large refactoring task
└── myapp-docs/         # Documentation updates
```

#### Pattern 3: Cross-Instance Communication

For advanced workflows, you can have Claude instances communicate with each other using separate working scratchpads. This separation often yields better results than having a single Claude handle everything:

```bash
# Create a shared communication file
echo "## Agent Communication Log" > /tmp/claude-shared.md

# Agent 1: Backend API development
cd ~/projects/myapp-backend
claude
# Prompt: "Write API changes to /tmp/claude-shared.md for the frontend team"

# Agent 2: Frontend integration
cd ~/projects/myapp-frontend
claude
# Prompt: "Read API changes from /tmp/claude-shared.md and implement frontend integration"
```

---

## Setting Up Worktrees for Claude Workflows

### Creating Your First Worktree

The basic syntax for creating a worktree is straightforward:

```bash
# Create a new worktree with a new branch
git worktree add <path> -b <branch-name>

# Create a worktree for an existing branch
git worktree add <path> <existing-branch>

# Create from a specific commit
git worktree add <path> <commit-hash>
```

### Practical Examples

**Example 1: New Feature Branch**
```bash
# From your main repository
cd ~/projects/myapp
git worktree add ../myapp-feature-payments -b feature/stripe-integration

# Navigate and start Claude
cd ../myapp-feature-payments
npm install  # Initialize dependencies
claude
```

**Example 2: Hotfix from Main**
```bash
# Create worktree from main branch for emergency fix
git worktree add ../myapp-hotfix main

cd ../myapp-hotfix
# Make it a new branch
git checkout -b hotfix/critical-security-patch
claude
```

**Example 3: PR Review**
```bash
# Review someone's pull request in isolation
git fetch origin
git worktree add ../myapp-pr-review origin/pr/feature-name

cd ../myapp-pr-review
npm install
claude
# Prompt: "Review this code for potential issues and suggest improvements"
```

### Environment Initialization

**Critical Step**: Depending on your stack, you need to initialize your development environment in each new worktree. This is essential for Claude to run tests, start dev servers, and execute project-specific commands.

**JavaScript/TypeScript Projects**
```bash
cd ~/projects/myapp-feature-x
npm install
# or
yarn install
# or
pnpm install
```

**Python Projects**
```bash
cd ~/projects/myapp-feature-x
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

**Ruby Projects**
```bash
cd ~/projects/myapp-feature-x
bundle install
```

**Other Languages**
Follow your project's standard setup process. This might include:
- Installing system dependencies
- Compiling native extensions
- Setting up database connections
- Copying configuration files not in version control

---

## Managing Multiple Feature Branches Simultaneously

### The Worktree List Command

Keep track of all your active worktrees:

```bash
git worktree list

# Example output:
# /home/user/projects/myapp              abc123f [main]
# /home/user/projects/myapp-feature-auth  def456a [feature/auth]
# /home/user/projects/myapp-hotfix        789ghi1 [hotfix/login-bug]
# /home/user/projects/myapp-refactor      012jkl3 [refactor/api-layer]
```

This command shows:
- The path to each worktree
- The current commit hash
- The checked-out branch name

### Moving Worktrees

If you need to relocate a worktree:

```bash
git worktree move ../myapp-feature-auth ../new-location/myapp-feature-auth
```

If you manually moved a worktree outside of Git (not recommended), repair the connection:

```bash
git worktree repair
```

### Removing Completed Worktrees

Always clean up properly when you're done with a worktree:

```bash
# Safe removal (requires clean working directory)
git worktree remove ../myapp-feature-auth

# Force removal (even with uncommitted changes)
git worktree remove --force ../myapp-feature-auth

# Remove and delete the branch
git worktree remove ../myapp-feature-auth
git branch -d feature/auth
```

**Important**: Use `git worktree remove` rather than manually deleting directories. Manual deletion leaves stale administrative files that can confuse Git.

### Locking Worktrees

If you're using a worktree on a portable device or network share that isn't always mounted, lock it to prevent automatic pruning:

```bash
git worktree lock ../myapp-portable --reason "Network drive, not always mounted"

# Later, unlock it
git worktree unlock ../myapp-portable
```

### Pruning Stale Worktrees

Clean up administrative files from worktrees that no longer exist:

```bash
git worktree prune
```

---

## Worktree Directory Organization

Where you place worktrees matters significantly for workflow efficiency. Here are recommended organizational strategies:

### Strategy 1: Sibling Directories (Most Common)

Place all worktrees as siblings to your main repository:

```
~/projects/
├── myapp/                    # Main worktree
├── myapp-feature-auth/       # Feature worktree
├── myapp-feature-payments/   # Feature worktree
├── myapp-hotfix/             # Hotfix worktree
└── myapp-review/             # Review worktree
```

**Advantages**:
- Predictable paths
- Easy to find and navigate
- Works well with IDE workspaces
- Simple relative path calculations

### Strategy 2: Centralized Worktree Directory

Keep all worktrees in a dedicated directory:

```
~/projects/
├── myapp/                    # Main repository
└── worktrees/
    ├── myapp-feature-auth/
    ├── myapp-feature-payments/
    ├── myapp-hotfix/
    └── myapp-review/
```

**Advantages**:
- Clean separation of main repo and working branches
- Easier to identify which directories are worktrees
- Simplifies cleanup operations
- Used by many professional teams

### Strategy 3: Project-Specific Worktree Directories

For organizations with many projects:

```
~/projects/
├── worktrees/
│   ├── myapp/
│   │   ├── feature-auth/
│   │   ├── feature-payments/
│   │   └── hotfix-login/
│   └── other-project/
│       ├── feature-x/
│       └── feature-y/
└── repos/
    ├── myapp/
    └── other-project/
```

### Naming Conventions

Use consistent, descriptive names for worktree directories:

**Good Naming Patterns**:
- `{project}-{type}-{description}`: `myapp-feature-auth`, `myapp-hotfix-login`
- `{project}-{ticket-id}`: `myapp-JIRA-1234`, `myapp-GH-567`
- `{project}-{username}-{feature}`: `myapp-john-payments`, `myapp-sarah-refactor`

**Avoid**:
- Generic names like `temp`, `test`, `new`
- Special characters that complicate shell operations
- Names that don't identify the branch or purpose

### Automation with Helper Functions

Create a bash function for streamlined worktree management:

```bash
# Add to ~/.bashrc or ~/.zshrc
function w() {
    local project="${1}"
    local branch="${2}"
    local action="${3:-edit}"

    local worktree_dir="$HOME/projects/worktrees/${project}/${branch}"

    if [ ! -d "$worktree_dir" ]; then
        echo "Creating worktree for ${branch}..."
        git -C "$HOME/projects/${project}" worktree add "$worktree_dir" -b "${USER}/${branch}"
    fi

    cd "$worktree_dir"

    if [ "$action" = "claude" ]; then
        claude
    fi
}

# Usage:
# w myapp feature-auth          # Navigate to worktree
# w myapp feature-auth claude   # Navigate and start Claude
```

---

## Context Isolation Between Instances

One of the most powerful aspects of combining worktrees with Claude Code is the complete context isolation between sessions. Understanding how this works is essential for maximizing productivity.

### How Context Isolation Works

**File System Isolation**
Each worktree is a completely independent working directory. When Claude Code analyzes your codebase, it reads from its specific worktree's files. Changes in one worktree don't affect others until merged.

**Session Storage**
Claude Code stores conversation sessions per project directory. This means:
- Each worktree maintains its own `CLAUDE.md` context file
- Conversation history is specific to that worktree
- The `/resume` command shows only sessions from the current directory and its Git-related worktrees

**Process Isolation**
Each `claude` command launches a separate process with its own:
- Memory space
- File handles
- Terminal session
- Conversation context

### Benefits of Context Isolation

**1. Prevents Context Pollution**

When working on multiple features, keeping context separate prevents confusion:

```bash
# Worktree 1: Authentication feature
cd ~/projects/myapp-auth
claude
# Conversation focused on: OAuth, JWT, user sessions, security

# Worktree 2: Payment integration
cd ~/projects/myapp-payments
claude
# Conversation focused on: Stripe API, webhooks, transaction handling
```

Without isolation, mixing these contexts in a single Claude session could lead to:
- Incorrect suggestions applying authentication patterns to payment code
- Confused mental models about which feature is being discussed
- Difficulty resuming specific tasks

**2. Enables True Parallel Work**

With isolated contexts, Claude instances can work at full speed without:
- Waiting for other instances to finish
- Dealing with merge conflicts during development
- File locks or permission issues
- Overwriting each other's changes

**3. Maintains Focused Conversations**

Each Claude session maintains a clear, focused conversation about a specific task:

```bash
# Example: Focused bug fix conversation
cd ~/projects/myapp-bugfix
claude

# Conversation stays on track:
# "Let's debug the login timeout issue"
# "I found the problem in session.js line 47"
# "Let me write a test to verify the fix"
# "The test passes, bug is fixed"

# Meanwhile, in another worktree, a separate focused conversation:
cd ~/projects/myapp-feature
claude

# Different conversation, different context:
# "Implement the new dashboard widget"
# "Here's the React component structure"
# "Let's add TypeScript types"
# "Styling with Tailwind CSS"
```

### Managing Context File Conflicts

Each worktree can maintain its own context documentation:

**Per-Worktree CLAUDE.md**
```bash
# Main repository
~/projects/myapp/CLAUDE.md           # Project overview

# Feature worktrees
~/projects/myapp-auth/CLAUDE.md      # Auth-specific context
~/projects/myapp-payments/CLAUDE.md  # Payment-specific context
```

When you merge branches, you may encounter conflicts in `CLAUDE.md`. Handle these strategically:

```bash
# Option 1: Keep feature-specific context in the feature branch
git checkout feature/auth -- CLAUDE.md

# Option 2: Keep main branch context
git checkout main -- CLAUDE.md

# Option 3: Merge both with manual editing
# Edit CLAUDE.md to combine relevant information from both branches
```

### Preventing Cross-Contamination

**Best Practices**:

1. **One Task Per Worktree**: Don't mix multiple unrelated tasks in a single worktree
2. **Clear Naming**: Use descriptive worktree names that indicate the task
3. **Regular Cleanup**: Remove worktrees when tasks are complete
4. **Separate Terminal Windows**: Use different terminal windows/tabs for each worktree to avoid accidental commands in the wrong directory
5. **Limit Active Worktrees**: Stick to 3-4 active worktrees per project for manageable context switching

### When Context Sharing Is Needed

Sometimes you want Claude instances to communicate. Use explicit communication files:

```bash
# Create a shared status file
echo "# Shared Work Log" > ~/projects/myapp-shared-notes.md

# Agent 1 writes API contract
cd ~/projects/myapp-backend
claude
# Prompt: "Define the REST API contract and write it to ~/projects/myapp-shared-notes.md"

# Agent 2 reads and implements
cd ~/projects/myapp-frontend
claude
# Prompt: "Read the API contract from ~/projects/myapp-shared-notes.md and implement the frontend"
```

---

## Best Practices for Worktree-Based Workflows

### 1. Plan Before Creating Worktrees

Not every task needs a worktree. Consider worktree overhead:

**When to Use Worktrees**:
- Feature development requiring multiple commits
- Long-running refactoring tasks
- Code reviews that need testing
- Parallel work on independent features
- Maintaining multiple release branches

**When NOT to Use Worktrees**:
- Quick fixes Claude completes in under 10 minutes
- Single-file edits
- Documentation-only changes
- Tasks that don't require running code

**Rule of Thumb**: If setting up the worktree (creating directory + installing dependencies) takes longer than the task itself, just use branch switching.

### 2. Initialize Dependencies Immediately

Always run your project's setup commands right after creating a worktree:

```bash
git worktree add ../myapp-feature -b feature/new-thing
cd ../myapp-feature

# Immediately initialize
npm install
cp .env.example .env  # Copy config files
npm run db:migrate    # Update database schema
```

Create a script to automate this:

```bash
#!/bin/bash
# setup-worktree.sh

WORKTREE_PATH=$1
cd "$WORKTREE_PATH"

echo "Installing dependencies..."
npm install

echo "Copying environment files..."
cp ../.env .env 2>/dev/null || echo "No .env to copy"

echo "Running migrations..."
npm run db:migrate

echo "✓ Worktree ready for Claude Code"
```

### 3. Use Claude Plan Mode for Safety

Before running multiple Claude instances in parallel, use Plan Mode to review what each will do:

```bash
cd ~/projects/myapp-feature-auth
claude --plan
# Review the plan before allowing execution
```

This eliminates anxiety about unauthorized changes and enables true parallelization with confidence.

### 4. Organize Terminals Effectively

**Terminal Multiplexers**:
```bash
# tmux setup for multiple worktrees
tmux new-session -s myapp
tmux rename-window 'main'

tmux new-window -n 'auth'
tmux send-keys 'cd ~/projects/myapp-auth && claude' C-m

tmux new-window -n 'payments'
tmux send-keys 'cd ~/projects/myapp-payments && claude' C-m

tmux new-window -n 'review'
tmux send-keys 'cd ~/projects/myapp-review && claude' C-m
```

**iTerm2 Setup (macOS)**:
- Create separate terminal tabs for each worktree
- Enable notifications for when Claude needs attention
- Use tab coloring to distinguish worktrees
- Set up automatic titles based on current directory

### 5. Regular Cleanup

Remove worktrees when branches are merged:

```bash
# After merging feature/auth
git worktree remove ../myapp-auth
git branch -d feature/auth
git push origin --delete feature/auth

# Clean up any stale references
git worktree prune
git fetch --prune
```

Automate cleanup with a git alias:

```bash
git config alias.wclean '!f() { git worktree remove $1 && git branch -d $(basename $1); }; f'

# Usage:
git wclean ../myapp-auth
```

### 6. Avoid File System Conflicts

**Don't work on the same files across worktrees** unless intentionally testing conflicts. If both Claude agents modify the same file:
- They'll overwrite each other's edits
- Manual conflict resolution becomes necessary
- Context pollution occurs
- Work gets duplicated or lost

**Strategy**: Divide work by:
- Module/package boundaries
- Feature areas
- Layer (frontend/backend)
- File types (code/tests/docs)

### 7. Use Relative Paths Configuration

For teams or when moving repositories between machines:

```bash
git config worktree.useRelativePaths true
```

This makes worktree administrative files use relative paths, improving portability.

### 8. Leverage Voice Input for Complex Requirements

For features with numerous edge cases, use voice dictation to quickly capture requirements:

```bash
cd ~/projects/myapp-feature
claude

# Use voice input (if your terminal supports it)
# Speak: "I need a user authentication system with OAuth2, JWT tokens,
# password reset via email, two-factor authentication, session management,
# remember me functionality, and security logging for suspicious activity"
```

Five minutes of verbal brain-dumping can replace lengthy written specifications and provide Claude with rich context.

### 9. Monitor API Usage Across Instances

Running multiple Claude instances increases API usage. Track costs:

```bash
# Check recent API usage
claude --usage

# Set budget alerts
claude --budget-alert 100  # Alert at $100 usage
```

### 10. Document Your Worktree Strategy

Create a `WORKTREES.md` in your repository:

```markdown
# Worktree Strategy

## Naming Convention
- `myapp-feature-{name}` - New features
- `myapp-hotfix-{issue}` - Production fixes
- `myapp-review-{pr-number}` - PR reviews

## Standard Setup
```bash
git worktree add ../myapp-{type}-{name} -b {branch-name}
cd ../myapp-{type}-{name}
./scripts/setup-worktree.sh
```

## Active Worktrees
- Keep max 4 active worktrees per developer
- Clean up weekly
- Lock network-mounted worktrees
```

---

## Common Patterns and Use Cases

### Pattern 1: The Bug Fix + Feature Development Combo

**Scenario**: You're deep into feature development when a critical production bug is reported.

```bash
# Already working on feature
cd ~/projects/myapp-feature-dashboard
# Claude is mid-conversation about dashboard widgets

# Critical bug reported! Create hotfix worktree
git worktree add ../myapp-hotfix-login main
cd ../myapp-hotfix-login

# Create hotfix branch
git checkout -b hotfix/login-timeout
npm install

# Start new Claude session for debugging
claude
# Prompt: "Debug the login timeout issue in production. Users report
# timeouts after 30 seconds on the login page."

# Fix the bug, test, commit, deploy

# Return to feature work - conversation context preserved
cd ~/projects/myapp-feature-dashboard
claude --resume
# Continue exactly where you left off on dashboard
```

**Time Saved**: No context switching, no stashing, no interrupted flow.

### Pattern 2: The PR Review Workflow

**Scenario**: Review a teammate's pull request without disrupting your current work.

```bash
# Currently working on authentication feature
cd ~/projects/myapp-auth

# PR review request arrives
git fetch origin
git worktree add ../myapp-pr-review-1234 origin/pr/feature-payments

cd ../myapp-pr-review-1234
npm install

# Start Claude for review
claude
# Prompt: "Review this pull request for:
# - Code quality and best practices
# - Potential bugs or edge cases
# - Security vulnerabilities
# - Test coverage gaps
# - Performance concerns
#
# Provide specific feedback with file locations and suggestions."

# Claude analyzes the code and provides detailed review
# Leave comments, suggest changes

# Done with review - clean up
cd ~/projects/myapp-auth
git worktree remove ../myapp-pr-review-1234

# Back to authentication work
claude --resume
```

**Benefit**: Thorough code reviews without losing your place in current work.

### Pattern 3: The Parallel Feature Factory

**Scenario**: Sprint planning assigns you three independent features that can be developed simultaneously.

```bash
# Feature 1: User profile page
git worktree add ../myapp-profile -b feature/user-profile
cd ../myapp-profile
npm install
claude &
# Background process - Prompt: "Create user profile page with bio, avatar, settings"

# Feature 2: Email notifications
git worktree add ../myapp-notifications -b feature/email-notifications
cd ../myapp-notifications
npm install
claude &
# Background process - Prompt: "Implement email notification system with templates"

# Feature 3: Search functionality
git worktree add ../myapp-search -b feature/search
cd ../myapp-search
npm install
claude
# Interactive - Prompt: "Build search feature with filters and autocomplete"

# Rotate through terminals, approve plans, monitor progress
# All three features develop in parallel
```

**Result**: Features that would take 3 days sequentially complete in 1 day with proper oversight.

### Pattern 4: The Refactoring + Test Writing Duo

**Scenario**: Large refactoring needs comprehensive test coverage before and after changes.

```bash
# Worktree 1: Write tests for current implementation
git worktree add ../myapp-tests main
cd ../myapp-tests
git checkout -b tests/auth-coverage
npm install

claude
# Prompt: "Write comprehensive tests for the authentication module
# to establish baseline coverage before refactoring"

# Worktree 2: Perform refactoring
git worktree add ../myapp-refactor main
cd ../myapp-refactor
git checkout -b refactor/auth-module
npm install

claude
# Prompt: "Refactor the authentication module to use dependency
# injection and improve testability"

# Workflow:
# 1. Tests written in worktree 1
# 2. Merge tests to main
# 3. Refactoring in worktree 2 with existing tests passing
# 4. Verify all tests still pass after refactor
# 5. Merge refactoring
```

**Advantage**: Tests are ready before refactoring starts, ensuring safety.

### Pattern 5: The Documentation Sprint

**Scenario**: Codebase needs documentation while development continues.

```bash
# Main development continues
cd ~/projects/myapp
claude
# Ongoing feature work

# Separate worktree for documentation
git worktree add ../myapp-docs main
cd ../myapp-docs
git checkout -b docs/api-documentation

claude
# Prompt: "Generate comprehensive API documentation from the codebase:
# - Endpoint descriptions
# - Request/response examples
# - Authentication requirements
# - Error codes and handling
# Update the docs/ directory with markdown files"

# Documentation generates in parallel with development
```

### Pattern 6: The Experiment Sandbox

**Scenario**: Try a radical new approach without risking your main branch.

```bash
# Create experimental worktree
git worktree add ../myapp-experiment --detach
cd ../myapp-experiment
git checkout -b experiment/new-architecture
npm install

claude
# Prompt: "Experiment with moving from REST to GraphQL. Show what
# the architecture would look like, convert one module as a proof of concept"

# If experiment succeeds, continue development
# If experiment fails, simply remove worktree with no impact on main work

git worktree remove ../myapp-experiment
```

**Freedom**: Experiment boldly without fear of corrupting your main codebase.

### Pattern 7: The Multi-Version Support

**Scenario**: Maintain multiple versions of your application simultaneously.

```bash
# Version 1.x maintenance
git worktree add ../myapp-v1 release/1.x
cd ../myapp-v1
npm install
claude
# Backport critical fixes to v1

# Version 2.x current stable
git worktree add ../myapp-v2 release/2.x
cd ../myapp-v2
npm install
claude
# Maintain current release

# Version 3.x next development
cd ~/projects/myapp  # main branch
claude
# Develop next major version
```

### Pattern 8: The Tooling Improvement Loop

**Scenario**: Improve development tools while using them.

```bash
# Use the tool in main worktree
cd ~/projects/myapp
npm run build  # Takes 2 minutes - too slow!

# Improve the tool in separate worktree
git worktree add ../myapp-tooling -b tooling/faster-build
cd ../myapp-tooling

claude
# Prompt: "The build takes 2 minutes. Analyze the build configuration
# and suggest optimizations. Implement parallel processing if possible."

# Claude implements improvements
# Test in this worktree
npm run build  # Now 1 minute!

# Merge improvements
git checkout main
git merge tooling/faster-build

# All worktrees benefit from the improvement
```

**Real Example**: A team reduced API generation time by 18% by having Claude analyze and parallelize their build process, at a cost of just $8 in API usage.

---

## Troubleshooting Common Issues

### Issue 1: "Cannot create worktree - branch already checked out"

**Error**:
```
fatal: 'feature/auth' is already checked out at '/home/user/projects/myapp-auth'
```

**Solution**: A branch can only be checked out in one worktree at a time. Either:
- Remove the existing worktree: `git worktree remove ../myapp-auth`
- Create a new branch from the existing one: `git worktree add ../myapp-auth-v2 -b feature/auth-v2 feature/auth`
- Use a different branch

### Issue 2: Stale Administrative Files

**Symptom**: Worktrees that no longer exist still show in `git worktree list`

**Solution**:
```bash
git worktree prune
```

### Issue 3: Dependencies Out of Sync

**Symptom**: Code works in one worktree but fails in another

**Solution**: Reinstall dependencies in each worktree after pulling changes
```bash
cd ~/projects/myapp-feature
git pull origin main
npm install  # Critical - don't skip this
```

### Issue 4: Port Already in Use

**Symptom**: Development server won't start because port is occupied by another worktree

**Solution**: Configure each worktree to use different ports
```bash
# Worktree 1
cd ~/projects/myapp-feature-1
PORT=3000 npm run dev

# Worktree 2
cd ~/projects/myapp-feature-2
PORT=3001 npm run dev

# Worktree 3
cd ~/projects/myapp-feature-3
PORT=3002 npm run dev
```

### Issue 5: Database Conflicts

**Symptom**: Multiple worktrees trying to use the same development database

**Solution**: Use different database instances per worktree
```bash
# Option 1: Different database names
# .env in worktree 1
DATABASE_URL=postgresql://localhost/myapp_feature1

# .env in worktree 2
DATABASE_URL=postgresql://localhost/myapp_feature2

# Option 2: Docker containers with different ports
cd ~/projects/myapp-feature-1
docker run -p 5432:5432 postgres

cd ~/projects/myapp-feature-2
docker run -p 5433:5432 postgres
```

---

## Performance Considerations

### Storage Space

Worktrees share the Git repository but require separate:
- Working directory files
- Node_modules (for JS projects)
- Virtual environments (for Python)
- Build artifacts

**Estimate**: For a typical JavaScript project:
- Main repository: 500 MB (includes node_modules)
- Each additional worktree: 400-500 MB (mostly node_modules)
- Shared .git directory: 50 MB (one copy for all worktrees)

**Optimization**:
```bash
# Use hard links for node_modules (experimental)
npm install --prefer-offline --cache ~/.npm-cache

# Share build caches
export TURBO_CACHE_DIR=~/.turbo-cache  # For turborepo
export NX_CACHE_DIRECTORY=~/.nx-cache  # For nx
```

### API Costs

Running multiple Claude instances increases API usage proportionally.

**Typical Costs** (as of 2026):
- Single feature implementation: $0.50 - $5.00
- Code review: $0.10 - $1.00
- Large refactoring: $5.00 - $20.00

**Cost Management**:
- Use Plan Mode to preview before execution
- Set clear, focused prompts to avoid unnecessary exploration
- Review changes regularly rather than letting Claude run unattended
- Monitor usage: `claude --usage`

### Memory Usage

Each Claude Code instance consumes system memory.

**Typical Usage**:
- Per Claude instance: 200-500 MB
- Per Node.js dev server: 200-400 MB
- Per IDE window: 500-1000 MB

**Recommendation**: For smooth operation with 4 parallel worktrees, ensure at least 16 GB RAM.

---

## Conclusion

Git worktrees combined with Claude Code represent a powerful paradigm shift in AI-assisted development. By enabling true parallel workflows with complete context isolation, this approach addresses fundamental bottlenecks in software development:

**Key Takeaways**:

1. **Worktrees eliminate context switching**: No more stashing, committing incomplete work, or mental overhead when moving between tasks

2. **Claude Code parallelization**: Run multiple AI agents simultaneously on independent features, reviews, and fixes

3. **Context isolation prevents interference**: Each Claude session maintains focused conversation about its specific task

4. **Real productivity gains**: Teams report 2-10x speed improvements on appropriate tasks

5. **Strategic use matters**: Not every task needs a worktree—balance setup overhead against task complexity

**Getting Started**:

```bash
# Your first parallel Claude workflow
git worktree add ../myproject-feature -b feature/new-thing
cd ../myproject-feature
npm install
claude
# Let Claude know: "This is a separate worktree for feature development"
```

**The Future**: As AI coding assistants become more powerful, the ability to orchestrate multiple agents working in parallel on different aspects of your codebase will become increasingly valuable. Mastering Git worktrees with Claude Code today positions you at the forefront of this evolution in software development.

---

## Additional Resources

### Official Documentation
- [Anthropic Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Claude Code Common Workflows](https://docs.claude.com/en/docs/claude-code/common-workflows)
- [Git Worktree Official Documentation](https://git-scm.com/docs/git-worktree)

### Community Guides
- [Mastering Git Worktrees with Claude Code for Parallel Development Workflow](https://medium.com/@dtunai/mastering-git-worktrees-with-claude-code-for-parallel-development-workflow-41dc91e645fe)
- [Git worktree + Claude Code: My Secret to 10x Developer Productivity](https://dev.to/kevinz103/git-worktree-claude-code-my-secret-to-10x-developer-productivity-520b)
- [How we're shipping faster with Claude Code and Git Worktrees](https://incident.io/blog/shipping-faster-with-claude-code-and-git-worktrees)
- [Git Worktree Tutorial: Work on Multiple Branches Without Switching](https://www.datacamp.com/tutorial/git-worktree-tutorial)
- [Working on two git branches at once with git worktree](https://andrewlock.net/working-on-two-git-branches-at-once-with-git-worktree/)

### Tools and Scripts
- [ccswitch](https://www.ksred.com/building-ccswitch-managing-multiple-claude-code-sessions-without-the-chaos/) - Tool for managing multiple Claude Code sessions
- [Git Tower - Git Worktree Guide](https://www.git-tower.com/learn/git/faq/git-worktree)

---

*Last updated: January 2026*
