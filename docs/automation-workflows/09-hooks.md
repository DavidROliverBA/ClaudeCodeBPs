# Claude Code Hooks: Comprehensive Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Hook Types and Triggers](#hook-types-and-triggers)
3. [Configuration](#configuration)
4. [Pre-Tool-Use and Post-Tool-Use Hooks](#pre-tool-use-and-post-tool-use-hooks)
5. [Common Use Cases](#common-use-cases)
6. [Hook Execution Flow and Timing](#hook-execution-flow-and-timing)
7. [Error Handling](#error-handling)
8. [Best Practices](#best-practices)
9. [Performance Considerations](#performance-considerations)
10. [Advanced Patterns](#advanced-patterns)

---

## Introduction

Claude Code hooks are user-defined shell commands that execute automatically at specific points in the agent lifecycle. They provide deterministic control over Claude's behavior, transforming suggestions into app-level guarantees. As stated in the official documentation: hooks "turn suggestions into app-level code that executes every time it is expected to run."

Hooks execute with your current environment's credentials and can access, modify, or delete any files your user account can reach. This power makes them invaluable for automation while requiring careful security consideration.

## Hook Types and Triggers

Claude Code supports **eight distinct hook events** that trigger at different lifecycle stages:

### 1. **PreToolUse**
Runs after Claude creates tool parameters but before processing the tool call. Ideal for:
- Validating inputs before execution
- Blocking dangerous operations
- Modifying tool parameters (v2.0.10+)
- Security enforcement

### 2. **PostToolUse**
Executes immediately after a successful tool completes. Perfect for:
- Automatic code formatting
- Quick validation checks
- Linting and style enforcement
- Generating derived artifacts

### 3. **UserPromptSubmit**
Fires when users submit prompts, before Claude processes them. Enables:
- Adding contextual information
- Prompt validation
- Blocking certain request types
- Injecting project-specific context

### 4. **PermissionRequest** (v2.0.45+)
Triggers when permission dialogs appear. Allows:
- Automated approval/denial logic
- Custom permission workflows
- Audit logging of permission requests

### 5. **Stop / SubagentStop**
Activates when agents finish responding. Useful for:
- End-of-turn quality gates
- AI-powered feedback generation
- Session summarization
- Post-completion notifications

### 6. **Notification**
Runs when Claude Code sends notifications. Supports:
- Desktop alerts
- Custom notification routing
- Integration with external systems

### 7. **SessionStart / SessionEnd**
Manages session lifecycle. Common uses:
- Loading development context
- Setting up environment variables
- Cleanup operations
- Session logging

### 8. **PreCompact**
Executes before compacting operations. Used for:
- Transcript backups
- State preservation
- Archive creation

### 9. **PostMcpToolUse**
Runs after MCP (Model Context Protocol) tools complete but before PostToolUse hooks, allowing modification of MCP tool outputs before further processing.

## Configuration

### Settings File Structure

Hooks are configured in settings files located at:
- **User-level**: `~/.claude/settings.json`
- **Project-level**: `.claude/settings.json`
- **Local**: `.claude/settings.local.json` (gitignored)

### Basic Configuration Syntax

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "path/to/script.sh",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### Matcher Patterns

Matchers are **case-sensitive** and support multiple patterns:

- **Exact match**: `"Write"` matches only the Write tool
- **Pipe syntax**: `"Edit|Write"` matches either tool
- **Regex patterns**: `"Notebook.*"` matches NotebookEdit, NotebookRun, etc.
- **Wildcard**: `"*"` or empty string matches all tools
- **MCP tools**: Use pattern `"mcp__server__tool"` (e.g., `"mcp__.*"` for all MCP tools)

### Hook Types

**Command Hooks** (`type: "command"`):
```json
{
  "type": "command",
  "command": "bash /path/to/script.sh",
  "timeout": 60
}
```

**Prompt Hooks** (`type: "prompt"`) - Stop/SubagentStop only:
```json
{
  "type": "prompt",
  "prompt": "Analyze the agent's response for quality issues"
}
```

### Configuration UI

Instead of manually editing JSON, use the `/hooks` interactive command:
1. Select hook event type
2. Choose tool matcher
3. Register command
4. Save to desired settings level

This approach minimizes syntax errors and provides a safer configuration experience.

## Pre-Tool-Use and Post-Tool-Use Hooks

### PreToolUse Hooks

PreToolUse hooks intercept tool calls before execution, providing strict gating capabilities.

**Input Schema:**
```json
{
  "session_id": "string",
  "transcript_path": "string",
  "cwd": "/absolute/path",
  "permission_mode": "string",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  }
}
```

**Example: Block Dangerous Commands**
```bash
#!/bin/bash
# Block rm -rf, sudo rm, chmod 777

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

if echo "$COMMAND" | grep -qE '(rm\s+-rf|sudo\s+rm|chmod\s+777)'; then
    echo '{"decision": "deny", "reason": "Dangerous command blocked"}' >&2
    exit 2
fi

exit 0
```

**Example: Modify Tool Input (v2.0.10+)**
```bash
#!/bin/bash
# Auto-correct file paths

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Normalize path
CORRECTED_PATH=$(realpath -m "$FILE_PATH")

# Output modified JSON
echo "$INPUT" | jq --arg path "$CORRECTED_PATH" '.tool_input.file_path = $path'
exit 0
```

### PostToolUse Hooks

PostToolUse hooks process results after successful tool completion, ideal for automatic quality enforcement.

**Input Schema:**
```json
{
  "session_id": "string",
  "transcript_path": "string",
  "cwd": "/absolute/path",
  "hook_event_name": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {...},
  "tool_output": "..."
}
```

**Example: Auto-Format TypeScript/JavaScript**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'INPUT=$(cat) && FILE=$(echo \"$INPUT\" | jq -r \".tool_input.file_path\") && [[ \"$FILE\" =~ \\.(ts|tsx|js|jsx)$ ]] && prettier --write \"$FILE\" || exit 0'"
          }
        ]
      }
    ]
  }
}
```

**Example: Run Linter on Python Files**
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Only process Python files
if [[ "$FILE_PATH" =~ \.py$ ]]; then
    # Run ruff linter with auto-fix
    ruff check --fix "$FILE_PATH"
    # Run black formatter
    black "$FILE_PATH"
fi

exit 0
```

## Common Use Cases

### 1. Automatic Code Formatting

**Prettier (JavaScript/TypeScript)**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/format-prettier.sh"
          }
        ]
      }
    ]
  }
}
```

**format-prettier.sh**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

if [[ "$FILE" =~ \.(js|jsx|ts|tsx)$ ]]; then
    if command -v prettier &> /dev/null; then
        prettier --write "$FILE" 2>/dev/null
    fi
fi
exit 0
```

**gofmt (Go)**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

if [[ "$FILE" =~ \.go$ ]]; then
    gofmt -w "$FILE"
fi
exit 0
```

### 2. Linting and Type Checking

**ESLint with Auto-Fix**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

if [[ "$FILE" =~ \.(js|jsx|ts|tsx)$ ]]; then
    if command -v eslint &> /dev/null; then
        eslint --fix "$FILE" 2>/dev/null || true
    fi
fi
exit 0
```

**TypeScript Compilation Check**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

if [[ "$FILE" =~ \.(ts|tsx)$ ]]; then
    tsc --noEmit "$FILE" 2>&1 | head -20 >&2
fi
exit 0
```

### 3. Security Enforcement

**Block Sensitive File Modifications**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

SENSITIVE_PATTERNS=(
    "\.env"
    "\.pem$"
    "\.key$"
    "\.git/"
    "credentials\.json"
    "secrets\."
)

for pattern in "${SENSITIVE_PATTERNS[@]}"; do
    if echo "$FILE_PATH" | grep -qE "$pattern"; then
        echo "Blocked modification to sensitive file: $FILE_PATH" >&2
        exit 2
    fi
done

exit 0
```

### 4. Command Logging and Audit

**Log All Bash Commands**:
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')
TIMESTAMP=$(date -Iseconds)
LOG_FILE="$HOME/.claude/logs/bash_commands.jsonl"

mkdir -p "$(dirname "$LOG_FILE")"
echo "$INPUT" | jq --arg ts "$TIMESTAMP" '. + {logged_at: $ts}' >> "$LOG_FILE"

exit 0
```

### 5. Context Injection (UserPromptSubmit)

**Add Git Status to Prompts**:
```bash
#!/bin/bash
INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

if git rev-parse --git-dir > /dev/null 2>&1; then
    GIT_STATUS=$(git status --short 2>/dev/null)
    BRANCH=$(git branch --show-current 2>/dev/null)

    CONTEXT="Current branch: $BRANCH\nModified files:\n$GIT_STATUS"

    echo "$INPUT" | jq --arg ctx "$CONTEXT" \
        '.hookSpecificOutput.UserPromptSubmit = {additionalContext: $ctx}'
fi

exit 0
```

### 6. Quality Gates (Stop Hook)

**Validate End-of-Turn State**:
```bash
#!/bin/bash
INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

# Run tests
if npm test > /dev/null 2>&1; then
    echo "✓ All tests passing"
else
    echo '{"decision": "block", "reason": "Tests failing - fix before continuing"}' >&2
    exit 2
fi

exit 0
```

## Hook Execution Flow and Timing

### Execution Order

For a typical tool invocation, hooks execute in this sequence:

```
1. User/Claude initiates tool call
2. PreToolUse hooks execute (parallel if multiple)
   └─ Can block, modify input, or allow
3. Tool executes (if not blocked)
4. PostMcpToolUse hooks execute (MCP tools only)
5. PostToolUse hooks execute (parallel if multiple)
6. Tool output returned to Claude
```

### Parallel Execution

- **All matching hooks run simultaneously**
- Identical commands are automatically deduplicated
- Timeouts apply per command independently
- One slow hook doesn't delay others

### Timeout Behavior

```json
{
  "type": "command",
  "command": "long-running-script.sh",
  "timeout": 120  // 120 seconds, default is 60
}
```

- Timeout applies to individual hooks
- Exceeded timeouts terminate that hook only
- Other hooks continue normally
- Timeout shown as error in verbose mode (Ctrl+O)

### Hook Snapshots

Hooks are **snapshot at startup**. Changes to hook configurations during runtime require:
1. Reviewing via menu
2. Restarting Claude Code session
3. Accepting updated hooks

This prevents malicious external modifications from auto-executing.

## Error Handling

### Exit Codes

| Exit Code | Behavior | Output Handling |
|-----------|----------|-----------------|
| **0** | Success | stdout shown in verbose mode; JSON parsed for control |
| **2** | Blocking error | stderr shown to user/Claude; execution prevented |
| **Other** | Non-blocking error | stderr in verbose mode; continues execution |

### Exit Code Examples

**Success (0)**:
```bash
#!/bin/bash
prettier --write "$FILE"
exit 0  # Continue regardless of prettier result
```

**Blocking Error (2)**:
```bash
#!/bin/bash
if [[ "$FILE" == *".env"* ]]; then
    echo "Cannot modify .env files" >&2
    exit 2  # Block the operation
fi
```

**Non-blocking Error (other)**:
```bash
#!/bin/bash
eslint "$FILE" || exit 1  # Show error but don't block
```

### JSON Output Control

Hooks returning exit code 0 can output structured JSON for sophisticated control:

```json
{
  "continue": false,
  "decision": "deny",
  "reason": "File validation failed",
  "systemMessage": "Warning: deprecated API detected",
  "suppressOutput": true,
  "hookSpecificOutput": {
    "UserPromptSubmit": {
      "additionalContext": "Context to inject..."
    }
  }
}
```

**Control Fields**:
- `continue`: boolean - Allow (true) or prevent (false) execution
- `decision`: string - "approve", "block", "deny", "allow", "ask"
- `reason`: string - User-facing explanation
- `systemMessage`: string - Warning message shown to user
- `suppressOutput`: boolean - Hide stdout from transcript

**Decision Values by Hook Type**:
- **PreToolUse**: `allow`, `deny`, `ask`
- **PostToolUse/Stop**: `block` or undefined
- **UserPromptSubmit**: `block` or undefined
- **PermissionRequest**: `approve`, `deny`, `ask`

**Priority Hierarchy**:
```
continue: false > decision: block/deny > Exit code 2 > Other exit codes
```

### Error Logging Best Practices

**Don't log to stdout** - Claude sees that as output:
```bash
# WRONG
echo "Processing $FILE..."
prettier "$FILE"

# RIGHT
echo "Processing $FILE..." >&2  # stderr
prettier "$FILE"
```

**Use dedicated log files**:
```bash
LOG_FILE="$HOME/.claude/logs/hooks.log"
echo "$(date -Iseconds) - Processing $FILE" >> "$LOG_FILE"
```

### Graceful Degradation

**Check tool availability**:
```bash
if ! command -v prettier &> /dev/null; then
    echo "prettier not installed, skipping formatting" >&2
    exit 0  # Don't block - gracefully skip
fi
```

**Handle missing files**:
```bash
if [[ ! -f "$FILE" ]]; then
    echo "File not found: $FILE" >&2
    exit 0  # Continue - file may be deleted intentionally
fi
```

## Best Practices

### Security

#### 1. Input Validation
**Never trust input data blindly**. Always validate:

```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Check for path traversal
if [[ "$FILE_PATH" == *".."* ]]; then
    echo "Path traversal detected" >&2
    exit 2
fi

# Verify file is in project directory
PROJECT_DIR=$(echo "$INPUT" | jq -r '.cwd')
if [[ ! "$FILE_PATH" == "$PROJECT_DIR"* ]]; then
    echo "File outside project directory" >&2
    exit 2
fi
```

#### 2. Quote All Variables
**Always use `"$VAR"` not `$VAR`** to prevent issues with spaces, special characters, and Unicode:

```bash
# WRONG - breaks with spaces
cat $FILE

# RIGHT
cat "$FILE"

# WRONG - command injection risk
eval $COMMAND

# RIGHT
"$COMMAND"
```

#### 3. Use Absolute Paths
```bash
# WRONG - relative path
command="./scripts/format.sh"

# RIGHT - absolute path
command="${CLAUDE_PROJECT_DIR}/.claude/hooks/format.sh"
```

#### 4. Block Sensitive Files
```bash
SENSITIVE_PATTERNS=(
    '\.env'
    '\.pem$'
    '\.key$'
    '\.git/'
    'credentials\.json'
    'secrets\.'
    'config/database\.yml'
)

for pattern in "${SENSITIVE_PATTERNS[@]}"; do
    if echo "$FILE_PATH" | grep -qE "$pattern"; then
        echo "Blocked: sensitive file" >&2
        exit 2
    fi
done
```

### Reliability

#### 1. Handle Errors Gracefully
```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

trap 'echo "Hook failed at line $LINENO" >&2; exit 1' ERR

# Your hook logic here
```

#### 2. Validate Dependencies
```bash
REQUIRED_TOOLS=("jq" "prettier" "git")

for tool in "${REQUIRED_TOOLS[@]}"; do
    if ! command -v "$tool" &> /dev/null; then
        echo "Required tool missing: $tool" >&2
        exit 0  # Don't block - gracefully skip
    fi
done
```

#### 3. Test Edge Cases

Test hooks with:
- **Filenames with spaces**: `"my file.txt"`
- **Unicode characters**: `"файл.txt"`, `"文件.txt"`
- **Deep paths**: `"/very/deep/nested/path/file.txt"`
- **Missing files**: Non-existent paths
- **Empty input**: `echo '{}' | your-hook.sh`
- **Malformed JSON**: Invalid input data

### Documentation

#### 1. Clear Descriptions
```json
{
  "type": "command",
  "command": "format-typescript.sh",
  // Add comment describing what it does
  // "Runs prettier on edited TypeScript/JavaScript files"
}
```

#### 2. Document Dependencies
Create a README in `.claude/hooks/`:

```markdown
# Project Hooks

## Dependencies
- prettier (npm install -g prettier)
- eslint (npm install -g eslint)
- jq (brew install jq / apt install jq)

## Installation
npm install
./setup-hooks.sh
```

### Configuration Management

#### 1. Use Settings Levels Appropriately

- **User settings** (`~/.claude/settings.json`): Personal preferences, global tools
- **Project settings** (`.claude/settings.json`): Team-shared, committed to git
- **Local settings** (`.claude/settings.local.json`): Machine-specific, gitignored

#### 2. Version Control
```gitignore
# .gitignore
.claude/settings.local.json
.claude/logs/
.claude/transcripts/
```

Commit project hooks:
```bash
git add .claude/settings.json
git add .claude/hooks/
git commit -m "Add project-level hooks for code quality"
```

## Performance Considerations

### 1. Matcher Specificity

**Use specific matchers** to reduce invocations:

```json
// SLOW - runs for every tool
{
  "matcher": "*",
  "hooks": [...]
}

// FAST - runs only for Write
{
  "matcher": "Write",
  "hooks": [...]
}
```

### 2. Execution Speed

**Keep hooks fast** - they block operations:

```bash
# SLOW - formats entire project
prettier --write "**/*.ts"

# FAST - formats only changed file
prettier --write "$FILE"
```

### 3. Background Execution

For slow operations, run in background:

```bash
#!/bin/bash
# Run expensive check in background, don't block
(npm run type-check > /tmp/typecheck.log 2>&1 &)
exit 0  # Continue immediately
```

### 4. Caching

**Cache expensive validations**:

```bash
#!/bin/bash
CACHE_FILE="/tmp/eslint-cache-$(echo $FILE | md5sum | cut -d' ' -f1)"
FILE_HASH=$(md5sum "$FILE" | cut -d' ' -f1)

if [[ -f "$CACHE_FILE" ]] && [[ "$(cat $CACHE_FILE)" == "$FILE_HASH" ]]; then
    exit 0  # Already validated
fi

eslint "$FILE"
echo "$FILE_HASH" > "$CACHE_FILE"
```

### 5. Parallel Validation

**Run independent checks concurrently**:

```bash
#!/bin/bash
# Run multiple validators in parallel
(prettier --check "$FILE" &)
(eslint "$FILE" &)
(tsc --noEmit "$FILE" &)

wait  # Wait for all to complete
```

### 6. Performance Monitoring

**Track execution time**:

```bash
#!/bin/bash
START=$(date +%s%N)

# Your hook logic here
prettier --write "$FILE"

END=$(date +%s%N)
DURATION=$(( (END - START) / 1000000 ))  # Convert to ms

if [[ $DURATION -gt 5000 ]]; then
    echo "Warning: Hook took ${DURATION}ms" >&2
fi
```

### 7. Smart Dispatching

**Single entry point with intelligent routing**:

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')
EXT="${FILE##*.}"

case "$EXT" in
    ts|tsx|js|jsx)
        prettier --write "$FILE"
        eslint --fix "$FILE"
        ;;
    py)
        black "$FILE"
        ruff check --fix "$FILE"
        ;;
    go)
        gofmt -w "$FILE"
        ;;
    *)
        exit 0  # Skip unsupported types quickly
        ;;
esac
```

## Advanced Patterns

### 1. Input Modification (PreToolUse v2.0.10+)

Transform tool inputs before execution:

```bash
#!/bin/bash
# Auto-correct common path mistakes
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Normalize path
NORMALIZED=$(realpath -m "$FILE_PATH")

# Replace tilde with home
EXPANDED="${NORMALIZED/#\~/$HOME}"

# Output modified JSON
echo "$INPUT" | jq --arg path "$EXPANDED" '.tool_input.file_path = $path'
```

### 2. Multi-Step Validation Pipeline

```bash
#!/bin/bash
set -euo pipefail
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Step 1: Format
prettier --write "$FILE" || exit 1

# Step 2: Lint
eslint --fix "$FILE" || exit 1

# Step 3: Type check
tsc --noEmit "$FILE" || exit 1

# Step 4: Security scan
if grep -qE '(eval|exec|innerHTML)' "$FILE"; then
    echo "Security concern detected" >&2
    exit 2
fi

exit 0
```

### 3. Conditional Hook Execution

```bash
#!/bin/bash
INPUT=$(cat)

# Only run in CI environment
if [[ -z "${CI:-}" ]]; then
    exit 0
fi

# Only run on main branch
BRANCH=$(git branch --show-current)
if [[ "$BRANCH" != "main" ]]; then
    exit 0
fi

# Run expensive validation
npm run test:full
```

### 4. Context-Aware Hooks (UserPromptSubmit)

```bash
#!/bin/bash
INPUT=$(cat)
PROMPT=$(echo "$INPUT" | jq -r '.prompt')

# Detect prompt type and inject relevant context
if echo "$PROMPT" | grep -qi "test"; then
    # Add test-related context
    CONTEXT=$(cat <<EOF
Recent test failures:
$(npm test 2>&1 | tail -20)

Coverage: $(npm run coverage 2>&1 | grep 'Coverage')
EOF
)
elif echo "$PROMPT" | grep -qi "bug"; then
    # Add debugging context
    CONTEXT=$(git log --oneline -10)
else
    exit 0
fi

echo "$INPUT" | jq --arg ctx "$CONTEXT" \
    '.hookSpecificOutput.UserPromptSubmit = {additionalContext: $ctx}'
```

### 5. AI-Powered Hooks (Stop Hook)

```bash
#!/bin/bash
INPUT=$(cat)
TRANSCRIPT=$(echo "$INPUT" | jq -r '.transcript_path')

# Generate completion summary using Claude
SUMMARY=$(cat "$TRANSCRIPT" | claude analyze --prompt "Summarize what was accomplished")

# Convert to audio feedback
echo "$SUMMARY" | text-to-speech --voice samantha

echo '{"continue": true}'
```

### 6. State Preservation (SessionStart/End)

**SessionStart**:
```bash
#!/bin/bash
INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

# Load project context
git status --short > /tmp/claude-session-git-status.txt
git log -10 --oneline > /tmp/claude-session-git-log.txt

# Set environment variables for session
ENV_FILE=$(echo "$INPUT" | jq -r '.CLAUDE_ENV_FILE // empty')
if [[ -n "$ENV_FILE" ]]; then
    echo "export PROJECT_CONTEXT_LOADED=true" >> "$ENV_FILE"
fi

exit 0
```

**SessionEnd**:
```bash
#!/bin/bash
INPUT=$(cat)

# Archive session transcript
TRANSCRIPT=$(echo "$INPUT" | jq -r '.transcript_path')
ARCHIVE_DIR="$HOME/.claude/archives/$(date +%Y-%m)"
mkdir -p "$ARCHIVE_DIR"
cp "$TRANSCRIPT" "$ARCHIVE_DIR/session-$(date +%s).jsonl"

# Cleanup temp files
rm -f /tmp/claude-session-*

exit 0
```

### 7. UV Single-File Python Scripts

Using Astral's UV for isolated Python hooks:

```python
#!/usr/bin/env -S uv run --quiet --script
# /// script
# dependencies = [
#   "pydantic>=2.0",
#   "requests>=2.31",
# ]
# ///

import sys
import json
from pydantic import BaseModel

class ToolInput(BaseModel):
    session_id: str
    tool_name: str
    tool_input: dict

# Read stdin
data = json.load(sys.stdin)
validated = ToolInput(**data)

# Your logic here
if validated.tool_input.get("file_path", "").endswith(".env"):
    print(json.dumps({
        "decision": "deny",
        "reason": "Cannot modify .env files"
    }), file=sys.stderr)
    sys.exit(2)

sys.exit(0)
```

## Environment Variables

Claude Code provides several environment variables to hooks:

| Variable | Description | Available In |
|----------|-------------|--------------|
| `CLAUDE_PROJECT_DIR` | Project root absolute path | All hooks |
| `CLAUDE_ENV_FILE` | File to persist environment variables | SessionStart only |
| `CLAUDE_CODE_REMOTE` | "true" for web, empty for CLI | All hooks |
| `CLAUDE_FILE_PATHS` | Space-separated paths (PostToolUse) | PostToolUse for file tools |

**Usage Example**:
```bash
#!/bin/bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR}"
CONFIG_FILE="$PROJECT_ROOT/.prettierrc"

if [[ -f "$CONFIG_FILE" ]]; then
    prettier --config "$CONFIG_FILE" --write "$FILE"
else
    prettier --write "$FILE"
fi
```

## Real-World Examples

### Complete TypeScript/JavaScript Project Setup

**.claude/settings.json**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/format-and-lint.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/security-check.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/run-tests.sh",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

**.claude/hooks/format-and-lint.sh**:
```bash
#!/bin/bash
set -euo pipefail

INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

# Only process JS/TS files
if [[ ! "$FILE" =~ \.(js|jsx|ts|tsx)$ ]]; then
    exit 0
fi

# Check dependencies
if ! command -v prettier &> /dev/null || ! command -v eslint &> /dev/null; then
    echo "prettier or eslint not found" >&2
    exit 0
fi

# Format
prettier --write "$FILE" 2>/dev/null

# Lint with auto-fix
eslint --fix "$FILE" 2>/dev/null || true

exit 0
```

**.claude/hooks/security-check.sh**:
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path')
CONTENT=$(echo "$INPUT" | jq -r '.tool_input.content // ""')

# Block sensitive file modifications
SENSITIVE='\.env|\.pem$|\.key$|credentials\.json'
if echo "$FILE_PATH" | grep -qE "$SENSITIVE"; then
    echo "Cannot modify sensitive file: $FILE_PATH" >&2
    exit 2
fi

# Scan content for security issues
DANGEROUS_PATTERNS=(
    'eval\('
    'exec\('
    'innerHTML\s*='
    'document\.write'
    '(password|secret|key)\s*=\s*["\'][^"\']+["\']'
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
    if echo "$CONTENT" | grep -qE "$pattern"; then
        echo "Security concern detected: $pattern" >&2
        # Don't block, but warn
        exit 1
    fi
done

exit 0
```

**.claude/hooks/run-tests.sh**:
```bash
#!/bin/bash
set -euo pipefail

INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd')

cd "$CWD" || exit 0

# Run tests
if npm test > /tmp/test-output.txt 2>&1; then
    echo "✓ All tests passing"
    exit 0
else
    echo "✗ Tests failed:" >&2
    tail -30 /tmp/test-output.txt >&2

    # Non-blocking - warn but allow
    exit 1
fi
```

## Troubleshooting

### Common Issues

**1. Hook not executing**:
- Check matcher pattern (case-sensitive)
- Verify tool name in verbose mode (Ctrl+O)
- Ensure hook script is executable: `chmod +x hook.sh`
- Review configuration with `/hooks` command

**2. Permission denied**:
```bash
chmod +x .claude/hooks/*.sh
```

**3. JSON parsing errors**:
```bash
# Test JSON output
echo '{"tool_input": {"file_path": "test.js"}}' | your-hook.sh
echo $?  # Check exit code
```

**4. Hook timeout**:
- Increase timeout in configuration
- Move slow operations to background
- Optimize hook execution speed

**5. Path issues**:
```bash
# Always use absolute paths
command="${CLAUDE_PROJECT_DIR}/.claude/hooks/script.sh"

# Quote paths with spaces
cat "$FILE_PATH"
```

## Additional Resources

### Official Documentation
- [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Hooks Getting Started Guide](https://code.claude.com/docs/en/hooks-guide)
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)

### Community Examples
- [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery) - Comprehensive examples of all 8 hook types
- [claude-code-quality-pipeline](https://github.com/jseldess/claude-code-quality-pipeline) - Production-ready quality automation
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - Curated hooks and workflows

### Tools and Utilities
- [cclint](https://github.com/carlrannaberg/cclint) - Linter for Claude Code project files
- [jq](https://stedolan.github.io/jq/) - JSON processor for bash hooks
- [UV](https://github.com/astral-sh/uv) - Python package manager for single-file scripts

---

## Conclusion

Claude Code hooks transform the agent from a helpful assistant into a reliable, deterministic development tool. By encoding requirements as hooks rather than relying on prompts, you ensure consistent behavior across all sessions.

Key takeaways:

1. **Security first**: Always validate inputs, quote variables, and block sensitive operations
2. **Keep hooks fast**: Target specific files, use caching, run expensive operations in background
3. **Handle errors gracefully**: Don't block on non-critical failures
4. **Test thoroughly**: Verify hooks with edge cases before production
5. **Document dependencies**: Make setup clear for team members
6. **Use the right hook type**: PreToolUse for prevention, PostToolUse for fixes, Stop for validation

Hooks are powerful but require responsibility. As the documentation states: "You are solely responsible for the commands you configure." Review hook code carefully, test in safe environments, and start with non-blocking implementations before adding strict gates.

With well-designed hooks, Claude Code becomes a trusted development partner that consistently enforces your team's standards while maintaining the flexibility to adapt to changing requirements.

---

**Document Version**: 1.0
**Last Updated**: 2026-01-02
**Word Count**: ~5,800 words
