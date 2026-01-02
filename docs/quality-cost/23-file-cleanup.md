# File Cleanup Best Practices in Claude Code

## Table of Contents
1. [Introduction](#introduction)
2. [Instructing Claude to Remove Temporary Files](#instructing-claude-to-remove-temporary-files)
3. [Cleanup Prompts and Patterns](#cleanup-prompts-and-patterns)
4. [Identifying Files to Clean Up](#identifying-files-to-clean-up)
5. [Automated Cleanup with Hooks](#automated-cleanup-with-hooks)
6. [Preventing File Accumulation](#preventing-file-accumulation)
7. [Post-Task Cleanup Checklists](#post-task-cleanup-checklists)
8. [Git-Ignored Temporary Files](#git-ignored-temporary-files)
9. [Best Practices for Workspace Hygiene](#best-practices-for-workspace-hygiene)
10. [Known Issues and Solutions](#known-issues-and-solutions)

---

## Introduction

Maintaining a clean workspace is essential when working with Claude Code. As an agentic coding tool, Claude Code can create numerous temporary files, cache data, and intermediate artifacts during development sessions. Without proper hygiene practices, these files accumulate and clutter your workspace, consume disk space, and potentially introduce confusion in your development workflow.

This comprehensive guide covers best practices for file cleanup in Claude Code, from manual cleanup prompts to automated solutions using hooks, ensuring your development environment remains organized and efficient.

---

## Instructing Claude to Remove Temporary Files

### Direct Cleanup Instructions

The most straightforward approach is to explicitly instruct Claude to clean up after itself. When starting a task that may generate temporary files, include cleanup requirements in your prompt:

**Effective Cleanup Prompt:**
```
If you create any temporary new files, scripts, or helper files for iteration,
clean up these files by removing them at the end of the task.
```

This proactive instruction ensures Claude treats cleanup as part of task completion rather than an afterthought.

### Task-Specific Cleanup

For specific tasks, be explicit about what needs cleanup:

```
Create a build script to compile the project. After verifying it works,
remove any temporary build artifacts, intermediate files, and test outputs.
Leave only the final compiled binary.
```

### The Junior Developer Mindset

Think of Claude Code as "a junior developer who's incredibly fast but needs good direction." Junior developers often need reminders about cleanup tasks. Don't assume Claude will automatically clean up—make it an explicit requirement.

---

## Cleanup Prompts and Patterns

### Pattern 1: Cleanup as Final Step

Structure your requests to include cleanup as a final verification step:

```
1. Generate test data files
2. Run the test suite
3. Verify all tests pass
4. Remove all generated test data files
5. Confirm the workspace is clean
```

### Pattern 2: Conditional Cleanup

For exploratory work where you might want to keep some outputs:

```
Create temporary analysis scripts to investigate the performance issue.
After we've identified the root cause, ask me which files to keep
before cleaning up the temporary scripts.
```

### Pattern 3: Cleanup with Verification

Always verify cleanup was successful:

```
After completing the migration, remove all backup files with .bak extension.
Then run 'git status' to confirm no untracked temporary files remain.
```

### Pattern 4: Scope-Limited Cleanup

Be specific about what should and shouldn't be removed:

```
Clean up all .tmp files in the /output directory, but preserve any
files in /output/results that were generated today.
```

---

## Identifying Files to Clean Up

### Claude Code System Files

Claude Code creates various temporary and cache files in specific locations:

**macOS/Linux:**
```bash
~/Library/Application Support/claude-code  # User data
~/Library/Caches/claude-code              # Cache files
~/Library/Logs/claude-code                # Log files
/tmp/claude-*-cwd                         # Working directory tracking
~/.config/claude-code                     # Linux config
~/.cache/claude-code                      # Linux cache
```

**Windows:**
```
%APPDATA%\.claude-code           # User data
%LOCALAPPDATA%\.claude-code-cache  # Cache
%TEMP%\claude-code-*             # Temporary files
```

### Project-Level Temporary Files

Common temporary files created during Claude Code sessions:

- **Configuration backups**: `.claude.json.backup`
- **Session state files**: `.claude.json`
- **Build artifacts**: `dist/`, `build/`, `target/`
- **Test outputs**: `coverage/`, `.pytest_cache/`, `test-results/`
- **Temporary scripts**: `temp_*.py`, `test_*.sh`, `debug_*.js`
- **Log files**: `*.log`, `debug.txt`, `output.txt`

### Identifying AI-Generated Files

Look for patterns that indicate Claude-generated temporary files:

```bash
# Find recently created files (last hour)
find . -type f -mmin -60 -not -path "./.git/*"

# Find files with common temporary naming patterns
find . -type f \( -name "temp_*" -o -name "tmp_*" -o -name "debug_*" -o -name "test_*" \)

# Check for uncommitted files
git status --porcelain
```

---

## Automated Cleanup with Hooks

### SessionEnd Hooks

Claude Code's most powerful cleanup mechanism is the SessionEnd hook, which executes when a session terminates. This enables automatic cleanup without manual intervention.

**Configuration Location:**
Add to `~/.claude/settings.json` or `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/cleanup-script.sh",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### SessionEnd Hook Input

Your cleanup script receives JSON via stdin with session information:

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../session.jsonl",
  "cwd": "/Users/username/project",
  "permission_mode": "default",
  "hook_event_name": "SessionEnd",
  "reason": "clear"
}
```

The `reason` field can be:
- `clear` - Session cleared with /clear command
- `logout` - User logged out
- `prompt_input_exit` - User exited during prompt input
- `other` - Other exit reasons

### Example Cleanup Script

Create a shell script at `/path/to/cleanup-script.sh`:

```bash
#!/bin/bash

# Read JSON input from Claude Code
INPUT=$(cat)

# Extract working directory
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

# Remove common temporary files
find . -type f \( -name "temp_*" -o -name "tmp_*" -o -name "*.tmp" \) -delete

# Remove empty directories
find . -type d -empty -delete

# Clean up Claude-specific temp files older than 1 day
find /tmp -name "claude-*-cwd" -type f -mtime +1 -delete 2>/dev/null

# Log cleanup
echo "$(date): Cleaned up workspace at $CWD" >> ~/.claude/cleanup.log
```

Make it executable:
```bash
chmod +x /path/to/cleanup-script.sh
```

### Advanced Cleanup Script with Git Integration

```bash
#!/bin/bash

INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

# Check if this is a git repository
if [ -d .git ]; then
  # Remove all untracked files that match temporary patterns
  git ls-files --others --exclude-standard | \
    grep -E '(temp_|tmp_|debug_|\.tmp$|\.log$)' | \
    xargs -r rm -f

  # Clean up git-ignored build artifacts
  git clean -fdX build/ dist/ target/ 2>/dev/null
fi

# Remove Claude Code session backups older than 7 days
find . -name ".claude.json.backup" -mtime +7 -delete
```

### Hook Execution Characteristics

- **Timeout**: Default 60 seconds, configurable per command
- **Parallelization**: All matching hooks run in parallel
- **Deduplication**: Identical commands are automatically deduplicated
- **Non-blocking**: SessionEnd hooks cannot block session termination
- **Security**: Hooks execute with your user account permissions—review carefully

### Hook Security Warning

⚠️ **IMPORTANT**: Claude Code hooks execute arbitrary shell commands on your system automatically. You are solely responsible for the commands you configure. Hooks can modify, delete, or access any files your user account can access.

Always review hook configurations in the `/hooks` menu before they take effect.

---

## Preventing File Accumulation

### Use /clear Frequently

The `/clear` command is your first line of defense against context bloat:

> "Pro tip: use /clear often. Every time you start something new, clear the chat. You don't need all that history eating your tokens."

Frequent clearing prevents:
- Accumulated conversation history
- Cached file contents
- Old command outputs
- Unnecessary context that degrades performance

### Session Cleanup Settings

Configure automatic session cleanup in `~/.claude/settings.json`:

```json
{
  "cleanupPeriodDays": 7
}
```

This automatically removes inactive sessions after the specified number of days.

### Git Worktrees for Parallel Work

For multi-workspace development, use git worktrees instead of creating multiple file copies:

```bash
# Create worktree
git worktree add ../project-feature-a feature-a

# Work in the worktree
cd ../project-feature-a

# Clean up when done
git worktree remove ../project-feature-a
```

This prevents accumulation of duplicate project directories.

### Known Issue: /tmp/claude-*-cwd Accumulation

Claude Code creates temporary working directory tracking files at `/tmp/claude-{random-4-hex}-cwd` for each Bash command execution. These files accumulate without cleanup.

**Impact**: Heavy users may accumulate 174+ files per day (~22 bytes each).

**Temporary Workaround**: Add to your SessionEnd hook or crontab:

```bash
# Clean up old tracking files (older than 1 day)
find /tmp -name "claude-*-cwd" -type f -mtime +1 -delete
```

**System cleanup**: Most Linux systems use `systemd-tmpfiles-clean` which removes `/tmp` files daily, providing automatic cleanup. macOS users may need manual cleanup.

---

## Post-Task Cleanup Checklists

### Essential Cleanup Checklist

After completing a development task, verify:

- [ ] **Temporary scripts removed**: No `temp_*.py`, `debug_*.sh`, or similar files
- [ ] **Build artifacts cleaned**: `dist/`, `build/`, `target/` directories removed or gitignored
- [ ] **Test outputs cleared**: Coverage reports, test databases, fixture files removed
- [ ] **Log files handled**: Deleted or moved to appropriate log directory
- [ ] **Git status clean**: Run `git status` to verify no unwanted untracked files
- [ ] **Configuration backups removed**: Old `.backup` or `.bak` files deleted
- [ ] **Documentation updated**: If kept files, ensure they're documented in README
- [ ] **Dependencies verified**: No unnecessary packages added to requirements/package.json

### Development Phase-Specific Checklists

**After Debugging:**
- [ ] Remove debug print statements
- [ ] Delete debug output files
- [ ] Remove temporary logging configurations
- [ ] Clean up test data files

**After Refactoring:**
- [ ] Remove old backup files
- [ ] Delete commented-out code blocks
- [ ] Clean up intermediate refactoring steps
- [ ] Verify no duplicate files exist

**After Testing:**
- [ ] Remove test fixtures from production directories
- [ ] Clean up test database files
- [ ] Delete temporary test scripts
- [ ] Clear cached test results

**Before Committing:**
- [ ] Review `git status` for unintended files
- [ ] Check `git diff` for debug code
- [ ] Verify `.gitignore` is up to date
- [ ] Confirm no secrets or credentials in files

### Systematic Cleanup Workflow

Follow this structured approach for thorough cleanup:

1. **Review**: Run `git status` and `find . -mmin -60 -type f` to see recent files
2. **Categorize**: Sort files into keep/delete/review
3. **Verify**: Ensure kept files are necessary and properly documented
4. **Execute**: Delete temporary files in batches
5. **Confirm**: Re-run `git status` to verify workspace cleanliness
6. **Document**: Note any kept files in commit message or documentation

---

## Git-Ignored Temporary Files

### Strategic .gitignore Configuration

Configure `.gitignore` at three levels:

**1. Project `.gitignore` (committed to repository):**
```gitignore
# Build outputs
dist/
build/
target/
*.pyc
__pycache__/

# Test outputs
coverage/
.pytest_cache/
test-results/

# Logs
*.log
logs/

# Temporary files (project-specific patterns)
temp_*
tmp_*
debug_*
scratch/
```

**2. Personal `.git/info/exclude` (local, not shared):**
```gitignore
# Personal workflow files
.claude.json
.claude.json.backup
TODO.md
NOTES.md
```

**3. Global `~/.gitignore` (all repositories):**
```gitignore
# OS files
.DS_Store
Thumbs.db
desktop.ini

# Editor temp files
*~
*.swp
*.swo
.vscode/
.idea/

# Claude Code specific
.claude.local.md
```

Configure global gitignore:
```bash
git config --global core.excludesFile ~/.gitignore
```

### Best Practice: Global vs. Project Ignores

**Use global gitignore for:**
- Operating system files (`.DS_Store`, `Thumbs.db`)
- Editor temporary files (`.swp`, `~`, `*.swo`)
- Personal development tools
- IDE configurations specific to you

**Use project gitignore for:**
- Build outputs (`dist/`, `build/`)
- Dependencies (`node_modules/`, `venv/`)
- Generated files specific to the project
- Common temporary file patterns everyone should ignore

**Why this matters:**
> "It's unwise to try to ignore editor temp files with rules in a repository's .gitignore file. It's far better for developers to define their own rules to exclude their own editors temp files."

Different developers use different tools—global ignores prevent conflicts.

### CLAUDE.md vs CLAUDE.local.md

- **`CLAUDE.md`**: Committed to git, shared with team, contains project conventions
- **`CLAUDE.local.md`**: Gitignored (add to `.gitignore`), contains personal preferences

Example `.gitignore` entry:
```gitignore
CLAUDE.local.md
.claude/local/
```

### Temporarily Ignoring Files

For files you want to modify locally but not commit:

```bash
# Tell Git to ignore changes to a tracked file
git update-index --skip-worktree config.json

# Resume tracking changes
git update-index --no-skip-worktree config.json
```

**Note**: Avoid `--assume-unchanged` as it can be reverted by `git pull`.

---

## Best Practices for Workspace Hygiene

### 1. The Junior Developer Principle

Treat Claude Code as a junior developer:
- **Be explicit**: Don't assume cleanup will happen automatically
- **Provide checklists**: Include cleanup in your task breakdowns
- **Review work**: Check what files were created before ending sessions
- **Give feedback**: If Claude forgets cleanup, remind it explicitly

### 2. Plan Mode for Cleanup Verification

Use Plan Mode to review proposed changes before execution:
- Verify cleanup steps are included in the plan
- Check that only intended files will be modified/deleted
- Approve cleanup operations before they execute

### 3. Break Down Complex Tasks

Large tasks generate more temporary files:
```
Instead of: "Refactor the entire authentication system"
Try:
  1. "Refactor the login module, clean up temp files"
  2. "Refactor the session module, clean up temp files"
  3. "Refactor the permissions module, clean up temp files"
```

This prevents overwhelming accumulation and makes cleanup more manageable.

### 4. Isolated Development Environments

For high-risk cleanup operations or major refactoring:

**Docker containers:**
```bash
docker run -it -v $(pwd):/workspace node:18 bash
# Work in isolated environment
# Exit and remove container cleans everything
```

**Python virtual environments:**
```bash
python -m venv .venv
source .venv/bin/activate
# Work with isolated dependencies
deactivate
rm -rf .venv  # Complete cleanup
```

### 5. Regular Cache Cleanup

Schedule monthly cleanup of Claude Code cache:

```bash
# Check cache size
du -sh ~/Library/Caches/claude-code

# Clean cache (macOS/Linux)
rm -rf ~/Library/Caches/claude-code/*
rm -rf ~/.cache/claude-code/*

# Clean logs
rm -rf ~/Library/Logs/claude-code/*
```

**Important**: Completely exit Claude Code before cleaning cache.

### 6. Test-Driven Development for Cleanup

Write tests that verify cleanup:

```python
def test_cleanup_removes_temp_files():
    # Run process that creates temp files
    process()

    # Verify cleanup
    temp_files = glob.glob("temp_*.txt")
    assert len(temp_files) == 0, f"Found {len(temp_files)} uncleaned temp files"
```

### 7. Pre-Commit Hooks for Cleanup Verification

Add to `.git/hooks/pre-commit`:

```bash
#!/bin/bash

# Check for common temporary file patterns
TEMP_FILES=$(git ls-files --others --exclude-standard | grep -E '(temp_|tmp_|debug_)')

if [ -n "$TEMP_FILES" ]; then
  echo "ERROR: Temporary files detected:"
  echo "$TEMP_FILES"
  echo ""
  echo "Please remove temporary files before committing."
  exit 1
fi
```

### 8. Documentation-Driven Cleanup

Maintain a `CLEANUP.md` in your project:

```markdown
# Project Cleanup Guide

## Temporary File Patterns
- `temp_*.py` - Debug scripts
- `output/*.json` - Test outputs
- `scratch/` - Experimental code

## Cleanup Commands
- `npm run clean` - Remove build artifacts
- `make clean` - Remove compiled files
- `./scripts/cleanup.sh` - Full cleanup

## Safe to Delete
- Anything in `tmp/`
- Files matching `debug_*`
- `.pytest_cache/`

## DO NOT Delete
- `config/` - Configuration files
- `data/fixtures/` - Test fixtures
```

---

## Known Issues and Solutions

### Issue 1: /tmp/claude-*-cwd Memory Leak

**Problem**: Every Bash command creates a tracking file that's never deleted. Heavy users accumulate 500+ files daily.

**Solution**: Implement automated cleanup via cron or SessionEnd hook:

```bash
# Cron job (add to crontab)
0 * * * * find /tmp -name "claude-*-cwd" -type f -mtime +0 -delete

# Or in SessionEnd hook
find /tmp -name "claude-*-cwd" -type f -delete 2>/dev/null
```

**Status**: Reported issue, simple fix pending in future release.

### Issue 2: Session Files in Project Directories

**Problem**: `.claude.json` and `.claude.json.backup` files clutter project directories.

**Solution**: Add to project `.gitignore`:

```gitignore
.claude.json
.claude.json.backup
```

### Issue 3: Build Artifact Accumulation

**Problem**: Multiple build runs create large `dist/` or `build/` directories.

**Solution**: Add cleanup to build scripts:

```json
// package.json
{
  "scripts": {
    "prebuild": "rm -rf dist/",
    "build": "webpack --config webpack.config.js"
  }
}
```

### Issue 4: Test Output Accumulation

**Problem**: Test runs create coverage reports, screenshots, videos that accumulate.

**Solution**: Configure test frameworks to clean up:

```javascript
// jest.config.js
module.exports = {
  coverageDirectory: 'coverage',
  clearMocks: true,
  resetMocks: true,
  restoreMocks: true
};
```

And gitignore test outputs:
```gitignore
coverage/
.nyc_output/
test-results/
screenshots/
videos/
```

---

## Conclusion

Effective file cleanup in Claude Code requires a multi-layered approach:

1. **Explicit instructions** to Claude about cleanup requirements
2. **Automated hooks** for session-end cleanup
3. **Strategic gitignore** configuration at multiple levels
4. **Regular maintenance** of cache and temporary files
5. **Systematic checklists** for post-task verification
6. **Workspace hygiene habits** integrated into daily workflow

By implementing these practices, you maintain a clean, efficient development environment that prevents file accumulation, reduces confusion, and keeps your projects organized.

Remember: Claude Code is a powerful tool, but like any junior developer, it needs clear guidance about cleanup expectations. Make cleanup an explicit part of your workflow, not an afterthought.

---

## Additional Resources

- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Official Anthropic engineering blog
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code/overview) - Official documentation
- [Hooks Reference](https://code.claude.com/docs/en/hooks) - Detailed hook configuration guide
- [Git Ignore Documentation](https://git-scm.com/docs/gitignore) - Official Git ignore patterns
- [Atlassian Git Ignore Tutorial](https://www.atlassian.com/git/tutorials/saving-changes/gitignore) - Comprehensive gitignore guide

---

**Document Version**: 1.0
**Last Updated**: 2026-01-02
**Word Count**: ~3,500 words
