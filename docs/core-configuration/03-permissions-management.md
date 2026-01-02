# Permissions Management in Claude Code

## Table of Contents
1. [Introduction](#introduction)
2. [Permission Configuration Fundamentals](#permission-configuration-fundamentals)
3. [Configuring Allowed and Denied Tools](#configuring-allowed-and-denied-tools)
4. [File Access Patterns and Restrictions](#file-access-patterns-and-restrictions)
5. [Glob Patterns for Access Control](#glob-patterns-for-access-control)
6. [Trust Levels and Permission Prompts](#trust-levels-and-permission-prompts)
7. [Sandboxing for Security and Autonomy](#sandboxing-for-security-and-autonomy)
8. [Balancing Automation with Safety](#balancing-automation-with-safety)
9. [Team Permission Policies](#team-permission-policies)
10. [Best Practices for Secure Usage](#best-practices-for-secure-usage)

## Introduction

Permissions management is the cornerstone of secure AI-assisted development with Claude Code. The permission system acts as a safety harness that governs what the coding agent can do in your environment—from reading and editing files to running terminal commands and calling external tools. This comprehensive guide explores how to configure, manage, and optimize Claude Code's permission system to balance productivity with security.

## Permission Configuration Fundamentals

### Settings Hierarchy

Claude Code implements a **scope-based permission hierarchy** with four distinct levels, where higher scopes override lower ones:

1. **Enterprise** (highest priority) - System-level managed settings that cannot be overridden
2. **Project** - Team-shared settings in `.claude/settings.json`
3. **Local** - Personal settings in `.claude/settings.local.json`
4. **User** (lowest priority) - Global settings in `~/.claude/settings.json`

This hierarchical structure allows organizations to enforce security policies while still permitting individual developers to customize their workflows. If a permission is allowed in user settings but denied in project settings, the project setting takes precedence and the permission is blocked.

### Configuration Methods

There are multiple ways to configure permissions in Claude Code:

- **Interactive prompts**: Click "Always allow" when Claude requests permission
- **Chat commands**: Use `/permissions` command for a user-friendly UI
- **Config files**: Edit `.claude/settings.json` (project), `.claude/settings.local.json` (local), or `~/.claude/settings.json` (user)
- **CLI flags**: Use `--allowedTools` or `--disallowedTools` for session-specific configurations

## Configuring Allowed and Denied Tools

### Permission Rule Structure

The `permissions` object in settings.json supports three arrays that control tool access:

- **`allow`**: Explicitly permit tool use without prompting
- **`ask`**: Prompt user for confirmation before tool use
- **`deny`**: Block tool access completely (highest precedence)

### Basic Configuration Example

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Read(~/.zshrc)",
      "Edit",
      "Write"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(~/.ssh/**)",
      "WebFetch"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm publish:*)"
    ]
  }
}
```

### Available Tools in Claude Code

Claude Code provides 16 tools with varying permission requirements:

**Permission-Required Tools:**
- `Bash` - Execute shell commands
- `Edit` - Modify existing files
- `Write` - Create or overwrite files
- `NotebookEdit` - Edit Jupyter notebooks
- `Skill` - Execute specialized skills
- `SlashCommand` - Run custom slash commands
- `WebFetch` - Fetch web content
- `WebSearch` - Search the web
- `ExitPlanMode` - Exit planning-only mode

**No-Permission Tools (Always Available):**
- `AskUserQuestion` - Ask clarifying questions
- `BashOutput` - Read output from background shells
- `Glob` - Find files by pattern
- `Grep` - Search file contents
- `KillShell` - Stop background processes
- `Read` - Read file contents
- `Task` - Manage complex tasks
- `TodoWrite` - Track progress

### Granular Command Patterns

You can achieve fine-grained control using wildcard patterns:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm:*)",              // Allow all npm commands
      "Bash(git log:*)",          // Allow only git log, not git push
      "Bash(pytest tests/*)",     // Allow testing specific directory
      "Read(~/projects/**/*.md)"  // Allow reading markdown files
    ]
  }
}
```

The `:*` syntax creates a wildcard that matches anything after the command, enabling precise control over which operations are permitted.

### CLI Flags for Session-Specific Control

For temporary or exploratory sessions, use CLI flags:

```bash
# Allow specific tools for this session only
claude --allowedTools Edit,Write,Bash

# Deny specific tools for this session
claude --disallowedTools WebFetch,WebSearch

# Combine with specific commands
claude --allowedTools "Bash(npm run:*)" "Edit"
```

## File Access Patterns and Restrictions

### Write Access Boundaries

Claude Code implements strict write-access boundaries for security:

- **Write scope**: Claude Code can only write to the folder where it was started and its subfolders
- **Parent directory protection**: Cannot modify files in parent directories without explicit permission
- **Read flexibility**: Can read files outside the working directory (useful for accessing system libraries and dependencies)

This asymmetric approach creates a clear security boundary: broad read access for understanding context, but confined write operations to prevent unintended system modifications.

### Protecting Sensitive Files

The deny rules are your "nuclear shield" for blocking access to sensitive data:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(.env.local)",
      "Read(.env.production)",
      "Read(./secrets/**)",
      "Read(~/.ssh/**)",
      "Read(~/.aws/credentials)",
      "Read(~/.config/gcloud/**)",
      "Read(**/credentials.json)",
      "Read(**/*.key)",
      "Read(**/*.pem)"
    ]
  }
}
```

**Important**: The Read deny rules apply to the Read tool and other built-in tools like Grep, Glob, and LS. However, Bash commands can potentially bypass these restrictions, so it's crucial to also restrict shell commands that could access sensitive files.

### Working with Additional Directories

To grant access beyond the current project folder, use the `additionalDirectories` setting:

```json
{
  "additionalDirectories": [
    "~/shared-libs",
    "~/company-templates"
  ]
}
```

## Glob Patterns for Access Control

### Glob Pattern Syntax

Glob patterns provide powerful pattern-matching capabilities for file access control:

- `*` - Matches any characters except path separators
- `**` - Matches any characters including path separators (recursive)
- `?` - Matches a single character
- `[abc]` - Matches any character in the set
- `{a,b}` - Matches either pattern

### Practical Glob Examples

```json
{
  "permissions": {
    "allow": [
      "Read(src/**/*.js)",           // All JavaScript in src tree
      "Read(tests/**/*.test.ts)",    // All test files
      "Write(docs/*.md)",            // Markdown in docs (non-recursive)
      "Edit(src/components/**)"      // All files in components tree
    ],
    "deny": [
      "Read(**/.env*)",              // All .env files anywhere
      "Read(**/secrets/**)",         // All secrets directories
      "Read(**/*.key)",              // All key files
      "Write(config/production/**)"  // Production configs
    ]
  }
}
```

### Path-Specific Rules

Claude Code v2.0.64 introduced `.claude/rules/` for modular, path-specific project instructions. You can create rule files that apply only when Claude operates on files matching specific glob patterns:

```
.claude/
  rules/
    api/**/*.md        # Rules for API documentation
    frontend/**/*.md   # Rules for frontend code
    backend/**/*.md    # Rules for backend code
```

This provides more granular control than a single CLAUDE.md file, allowing different guidelines for different parts of your codebase.

### Known Limitations

There's a reported issue where `/**` glob patterns in `.claude/settings.local.json` may not always grant recursive access as expected. If you encounter permission prompts despite having wildcard permissions, you may need to grant permissions more explicitly or file an issue with the Claude Code team.

## Trust Levels and Permission Prompts

### Permission Modes

Claude Code offers four permission modes that represent different trust levels:

#### 1. Default Mode (Recommended)
- **Behavior**: Allows reads without prompting; asks before edits, writes, and command execution
- **Use case**: Standard development workflow with balanced security
- **Configuration**: Active by default, no configuration needed

```bash
claude  # Starts in default mode
```

#### 2. Plan Mode
- **Behavior**: Claude can analyze and read but not modify files or execute commands
- **Use case**: Exploratory analysis, code review, architecture planning
- **Configuration**: Start with `--mode plan` or use `/plan` command

```bash
claude --mode plan
```

#### 3. Accept Edits Mode
- **Behavior**: Automatically accepts file edit permissions for the session
- **Use case**: Rapid iteration on trusted codebases where you review changes via git diff
- **Configuration**: Use `--acceptEdits` flag

```bash
claude --acceptEdits
```

#### 4. Bypass Permissions Mode (Dangerous)
- **Behavior**: Skips all permission prompts—no safety checks
- **Use case**: Only in fully isolated environments (containers without internet)
- **Configuration**: Requires `--dangerously-skip-permissions` flag

```bash
# Only use in isolated, disposable environments!
docker run --rm -it my-isolated-env
claude --dangerously-skip-permissions
```

**Warning**: Never use bypass permissions mode with sensitive codebases, production environments, or systems with network access.

### Overriding Default Mode

You can configure which mode starts by default:

```json
{
  "defaultMode": "acceptEdits"
}
```

### Interactive Permission Management

The `/permissions` command provides a user-friendly UI to:
- View current permissions and their sources (user/project/enterprise)
- Explicitly allow or deny tools
- Navigate settings visually without manual JSON editing
- Understand which permissions are active and why

## Sandboxing for Security and Autonomy

### The Permission Paradox

Claude Code faces a fundamental tension: requiring permission for every action ensures safety but creates "approval fatigue" where users mechanically approve requests without careful review. Sandboxing solves this by creating pre-defined boundaries within which Claude can work autonomously.

### How Sandboxing Works

The sandboxing runtime uses OS-level primitives (Linux bubblewrap and macOS seatbelt) to enforce restrictions that cover not just Claude Code's direct interactions, but also any scripts, programs, or subprocesses spawned by commands.

### Two Security Boundaries

Effective sandboxing requires both boundaries working together:

#### 1. Filesystem Isolation
- **Purpose**: Restricts Claude to specific directories
- **Implementation**: Allows read and write access to the current working directory but blocks modification of files outside it
- **Protection**: Prevents access to sensitive system files like SSH keys and credentials

#### 2. Network Isolation
- **Purpose**: Controls which servers Claude can connect to
- **Implementation**: Only allows internet access through a unix domain socket connected to a proxy server running outside the sandbox
- **Protection**: Prevents data exfiltration and unauthorized external communications

### Impact on User Experience

In Anthropic's internal testing, **sandboxing safely reduces permission prompts by 84%**. This dramatic reduction eliminates approval fatigue while maintaining security through OS-level enforcement.

### Enabling Sandboxing

Activate sandboxing using the `/sandbox` command in Claude Code. You can configure specific file paths and network domains:

```bash
# Start Claude Code
claude

# Enable sandboxing
/sandbox
```

### Why Both Boundaries Matter

The article on sandboxing emphasizes that you need both mechanisms simultaneously:

- **Without network isolation**: A compromised agent could steal SSH keys and send them to external servers
- **Without filesystem isolation**: An agent could escape the sandbox by modifying system files or executables

## Balancing Automation with Safety

### The Allowlist-First Strategy

Think of permissions in Claude Code as your App Store approval system:

1. **Allowlist (permissions.allow)**: Only include commands that are 100% harmless and frequently needed
2. **Asklist (permissions.ask)**: Keep risky but necessary operations on ask for conscious approval
3. **Denylist (permissions.deny)**: Your "nuclear shield" for blocking dangerous operations

**Recommended Approach**: Use the allowlist as the first line of defense, and use denylists only on top of those. This creates a zero-trust environment where only explicitly approved operations proceed automatically.

### Progressive Trust Model

Start conservative and expand permissions based on experience:

#### Phase 1: Initial Setup (High Caution)
```json
{
  "permissions": {
    "allow": [
      "Read(src/**)",
      "Grep",
      "Glob"
    ],
    "deny": [
      "Read(**/.env*)",
      "Read(**/secrets/**)",
      "WebFetch",
      "Bash(curl:*)",
      "Bash(wget:*)"
    ]
  }
}
```

#### Phase 2: Trusted Workflows (Medium Caution)
```json
{
  "permissions": {
    "allow": [
      "Read(src/**)",
      "Edit",
      "Bash(npm run test:*)",
      "Bash(npm run lint:*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "ask": [
      "Write",
      "Bash(git commit:*)",
      "Bash(git push:*)"
    ]
  }
}
```

#### Phase 3: Mature Projects (Balanced)
```json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Bash(npm:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Read(**/.env*)",
      "WebFetch"
    ],
    "ask": [
      "Bash(git push:*)"
    ]
  }
}
```

### Command Blocklist

Claude Code includes a default **command blocklist** that blocks risky commands like `curl` and `wget` by default. These commands can fetch arbitrary content from the web and potentially exfiltrate data. Always maintain this protection unless you have specific, well-understood reasons to allow them.

### Approval Fatigue Mitigation

Beyond sandboxing, other strategies to reduce approval fatigue:

1. **Use allowlists for routine operations**: Identify safe, repetitive commands and add them to allow rules
2. **Leverage plan mode**: Use plan mode for exploration, then switch to default mode for implementation
3. **Review in batches**: Instead of approving individual edits, review collections via git diff
4. **Session-specific overrides**: Use CLI flags for temporary permission adjustments during specific tasks

## Team Permission Policies

### Version-Controlled Configuration

Store `.claude/settings.json` in source control to share permission policies across teams:

```json
{
  "permissions": {
    "allow": [
      "Read(src/**)",
      "Read(tests/**)",
      "Edit",
      "Bash(npm run test)",
      "Bash(npm run lint)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(secrets/**)",
      "Bash(npm publish:*)",
      "Bash(git push origin main)",
      "WebFetch"
    ],
    "ask": [
      "Write",
      "Bash(git commit:*)",
      "Bash(git push:*)"
    ]
  }
}
```

### Personal Overrides

Individual developers can create `.claude/settings.local.json` for personal preferences that aren't checked into source control:

```json
{
  "permissions": {
    "allow": [
      "Bash(git commit:*)"  // Personal preference to auto-allow commits
    ]
  }
}
```

### Enterprise Settings

Organizations can enforce global policies using enterprise settings that cannot be overridden:

- **`allowManagedHooksOnly`**: Block user/project hooks; load only managed and SDK hooks
- **`disableBypassPermissionsMode`**: Prevent the `--dangerously-skip-permissions` flag
- **`strictKnownMarketplaces`**: Allowlist plugin marketplace sources (exact matching required)

### Standardized Practices

Establish repository etiquette guidelines in `CLAUDE.md`:

```markdown
# Claude Code Guidelines

## Branch Naming
- Feature branches: `feature/description`
- Bug fixes: `fix/description`
- Use kebab-case for all branch names

## Merge Strategy
- Always use merge commits (no rebasing on shared branches)
- Require PR reviews before merging to main

## Testing Requirements
- Run `npm run test` before committing
- Run `npm run lint` and fix all issues
- Update tests for any code changes

## Permissions Policy
- Never commit .env files
- Always review git diffs before pushing
- Use plan mode for exploratory analysis
```

### MCP Server Safety

MCP (Model Context Protocol) servers are powerful but potentially dangerous if left unchecked. For team policies:

1. **Never use `enableAllProjectMcpServers: true`** - This is a security risk
2. **Explicitly enable only trusted servers** - Review what each server does before enabling
3. **Document approved MCP servers** - Maintain a list of organization-approved servers
4. **Audit server permissions** - Understand what data each server can access

```json
{
  "mcpServers": {
    "approved-database-tool": {
      "command": "npx",
      "args": ["-y", "@company/db-mcp-server"]
    }
  },
  "enableAllProjectMcpServers": false
}
```

## Best Practices for Secure Usage

### 1. Start Conservative, Then Expand

**Strategy**: Begin with restrictive permissions in read-only or planning mode for new or untrusted codebases. Gradually expand as you gain confidence.

```bash
# First exploration of a new codebase
claude --mode plan

# After understanding the codebase
claude --acceptEdits
```

### 2. Treat Claude Like an Untrusted but Powerful Intern

**Mindset**: Give Claude only the minimum permissions it actually needs. This follows the principle of least privilege from traditional security.

- Don't grant Write permissions if Edit is sufficient
- Don't allow all Bash commands if you only need specific npm scripts
- Don't enable WebFetch unless you have a specific use case

### 3. Deny Rules for Sensitive Files

**Critical Protection**: Always block access to credentials and secrets:

```json
{
  "permissions": {
    "deny": [
      "Read(.env*)",
      "Read(**/.env*)",
      "Read(secrets/**)",
      "Read(~/.ssh/**)",
      "Read(~/.aws/credentials)",
      "Read(~/.config/gcloud/**)",
      "Read(**/credentials.json)",
      "Read(**/*.key)",
      "Read(**/*.pem)",
      "Read(**/id_rsa*)"
    ]
  }
}
```

### 4. Network Command Restrictions

**Default deny for data exfiltration vectors**:

```json
{
  "permissions": {
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(nc:*)",
      "Bash(netcat:*)",
      "WebFetch"
    ]
  }
}
```

### 5. Use Sandboxing for Reduced Friction

**Enable sandboxing** to reduce permission prompts while maintaining security through OS-level boundaries. This is particularly valuable for:

- Long coding sessions
- Repetitive tasks
- Trusted codebases where you review changes via git

### 6. Review Changes Before Approval

**Leverage git for safety**:

```bash
# After Claude makes changes
git diff

# Review all changes before committing
git add -p

# Commit with descriptive message
git commit -m "Detailed description of changes"
```

### 7. Never Use Bypass Mode in Production

**Absolute rule**: Never use `--dangerously-skip-permissions` except in:
- Disposable containers
- Environments without network access
- Systems with no sensitive data
- Fully isolated development VMs

### 8. Regular Permission Audits

**Maintenance practice**: Periodically review your permission settings:

```bash
# Review current permissions
claude
/permissions

# Check what's in your settings files
cat ~/.claude/settings.json
cat .claude/settings.json
```

### 9. Planning Before Coding

**Performance optimization**: Ask Claude to research and plan first for complex problems. This prevents premature implementation and reduces risky decisions.

```bash
# Start in plan mode for complex tasks
claude --mode plan

# After planning is complete, switch modes
/exit-plan-mode
```

### 10. Iterative Verification

**Quality assurance**: Use multiple Claude instances in parallel:
- Have one Claude write code
- Have another Claude review it
- This separation often produces better results than single-instance workflows

### 11. Visual Feedback Loops

**Improved accuracy**: Provide screenshots, mockups, or test cases. Claude performs best when it has a clear target to iterate against.

### 12. Rotate API Keys

**Security hygiene**: If you use Claude Code with API keys or external services:
- Rotate keys regularly
- Use environment-specific keys (dev vs. prod)
- Never commit keys to version control
- Use secret management tools

### 13. Use Devcontainers for Additional Isolation

**Defense in depth**: Consider using VS Code devcontainers or Docker for an additional isolation layer:

```json
// .devcontainer/devcontainer.json
{
  "name": "Claude Code Environment",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:18",
  "customizations": {
    "vscode": {
      "extensions": ["claude.claude-code"]
    }
  },
  "remoteEnv": {
    "NODE_ENV": "development"
  }
}
```

### 14. Document Project-Specific Constraints

**Team alignment**: Create detailed CLAUDE.md files that document:
- Unexpected behaviors or warnings particular to the project
- Specific commands that should never be run
- Required testing procedures
- Code style preferences

### 15. First-Time Codebase Trust Verification

**Initial security check**: When opening a new codebase for the first time:

1. Review the `.claude/` directory for any suspicious configurations
2. Check for pre-configured MCP servers
3. Verify permission settings in project settings.json
4. Start in plan mode to explore before granting edit permissions

## Conclusion

Effective permissions management in Claude Code requires a thoughtful balance between security and productivity. By understanding the permission hierarchy, leveraging allowlists and denylists strategically, using sandboxing to reduce friction, and following security best practices, you can create a safe and efficient AI-assisted development workflow.

Remember the core principle: **Treat Claude Code like an untrusted but powerful intern**. Give it the minimum permissions necessary, audit its actions, and use security boundaries like sandboxing to contain potential issues. With proper configuration and mindful usage, Claude Code becomes a powerful productivity multiplier without compromising security.

## Sources and Further Reading

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Security Features](https://code.claude.com/docs/en/security)
- [Beyond Permission Prompts: Making Claude Code More Secure and Autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing)
- [Claude Code Best Practices for Agentic Coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [How to Use Allowed Tools in Claude Code](https://www.instructa.ai/blog/claude-code/how-to-use-allowed-tools-in-claude-code)
- [A Complete Guide to Claude Code Permissions](https://www.eesel.ai/blog/claude-code-permissions)
- [Claude Code Security Best Practices](https://www.backslash.security/blog/claude-code-security-best-practices)
- [Permission Model in Claude Code](https://skywork.ai/blog/permission-model-claude-code-vs-code-jetbrains-cli/)
- [Claude Code Permissions Course](https://stevekinney.com/courses/ai-development/claude-code-permissions)
- [Understanding Claude Code Permissions and Security Settings](https://www.petefreitag.com/blog/claude-code-permissions/)

---

*Document Version: 1.0 | Last Updated: January 2026*
