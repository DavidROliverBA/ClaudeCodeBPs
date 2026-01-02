# Claude Code Settings Hierarchy: A Comprehensive Guide

## Introduction

Claude Code implements a sophisticated, multi-tier settings system that enables fine-grained control over its behaviour across different scopes—from individual developers to entire organisations. Understanding this hierarchy is essential for maximising productivity, maintaining security, and ensuring consistent team collaboration.

This guide provides an in-depth exploration of Claude Code's settings architecture, configuration options, practical examples, and best practices for both individual developers and teams.

## Table of Contents

1. [Settings Hierarchy Overview](#settings-hierarchy-overview)
2. [User Settings](#user-settings)
3. [Project Settings](#project-settings)
4. [Local Settings](#local-settings)
5. [Enterprise/Managed Settings](#enterprisemanaged-settings)
6. [How Settings Merge and Override](#how-settings-merge-and-override)
7. [Common Settings Options](#common-settings-options)
8. [Practical Configuration Examples](#practical-configuration-examples)
9. [Best Practices](#best-practices)
10. [Additional Resources](#additional-resources)

---

## Settings Hierarchy Overview

Claude Code employs a **five-tier precedence system** for settings configuration. When identical settings exist at multiple levels, more specific configurations supersede broader ones. The hierarchy, from highest to lowest priority, is:

### 1. Enterprise/Managed Settings (Highest Priority)
- **Location**: System-level paths
  - macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
  - Linux/WSL: `/etc/claude-code/managed-settings.json`
  - Windows: `C:\Programme Files\ClaudeCode\managed-settings.json`
- **Purpose**: Organisation-wide policies enforced by IT/DevOps
- **Characteristics**: Cannot be overridden by any lower-level settings
- **Use Cases**: Security policies, compliance requirements, company coding standards

### 2. Command Line Arguments
- **Location**: Passed directly when invoking Claude Code
- **Purpose**: Temporary, session-specific overrides
- **Characteristics**: Override all file-based settings except managed settings

### 3. Local Project Settings
- **Location**: `.claude/settings.local.json` (in project directory)
- **Purpose**: Personal project-specific overrides
- **Characteristics**: Automatically added to `.gitignore`, never committed to version control
- **Use Cases**: Personal experimentation, machine-specific configurations

### 4. Shared Project Settings
- **Location**: `.claude/settings.json` (in project directory)
- **Purpose**: Team-shared project configuration
- **Characteristics**: Committed to version control, shared across team
- **Use Cases**: Project-specific permissions, team workflows, project standards

### 5. User Settings (Lowest Priority)
- **Location**: `~/.claude/settings.json`
- **Purpose**: Personal preferences across all projects
- **Characteristics**: Global to the user, applies to all projects unless overridden
- **Use Cases**: Personal coding preferences, frequently used permissions, personal tools

---

## User Settings

### Location and Purpose

User settings are stored in `~/.claude/settings.json` and define your personal preferences across **all projects**. These settings travel with you regardless of which project you're working on.

### When to Use User Settings

- **Personal coding preferences** that apply universally
- **Frequently used permissions** you want available everywhere
- **API keys and authentication helpers** specific to your account
- **Custom agents and tools** you've developed for personal use
- **Model preferences** for your typical workflow

### Example User Settings

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(npm run lint)",
      "Read(~/.zshrc)",
      "Read(~/.bashrc)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Read(.env*)",
      "Read(**/secrets/**)"
    ]
  },
  "env": {
    "DISABLE_TELEMETRY": "1",
    "ANTHROPIC_MODEL": "claude-sonnet-4-5-20250929"
  },
  "cleanupPeriodDays": 30,
  "attribution": {
    "commit": "Co-authored-by: Claude <[email protected]>",
    "pr": ""
  }
}
```

### Configuration Storage

In addition to `~/.claude/settings.json`, Claude Code also uses `~/.claude.json` for:
- User preferences (theme, notification settings, editor mode)
- OAuth session data
- MCP server configurations
- Per-project state and trust settings
- Various caches

---

## Project Settings

### Location and Purpose

Project settings reside in `.claude/settings.json` within your project directory. This file is **committed to version control** and shared with your entire team, ensuring consistent behaviour across all developers.

### When to Use Project Settings

- **Team-wide permission policies** for the project
- **Project-specific tool restrictions** (e.g., blocking certain bash commands)
- **Shared hooks** for code formatting, linting, or validation
- **Environment variables** that all team members need
- **Project-specific subagents** in `.claude/agents/`
- **MCP server configurations** in `.mcp.json`

### Example Project Settings

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(pytest:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm install:*)",
      "Bash(pip install:*)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Write(.env*)",
      "Bash(rm:*)",
      "Bash(curl:*)"
    ],
    "additionalDirectories": [
      "../shared-docs/",
      "../common-utils/"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'source .venv/bin/activate' >> \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  },
  "env": {
    "NODE_ENV": "development",
    "API_TIMEOUT": "30000"
  },
  "companyAnnouncements": [
    "Remember: All PRs require code review",
    "Use conventional commits format",
    "Run tests before pushing"
  ]
}
```

### Project-Scoped MCP Servers

MCP (Model Context Protocol) servers can be configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

---

## Local Settings

### Location and Purpose

Local settings are stored in `.claude/settings.local.json` within your project directory. This file is **automatically added to `.gitignore`** and represents personal overrides that are never shared with the team.

### When to Use Local Settings

- **Personal experimentation** with new permissions or tools
- **Machine-specific configurations** (e.g., different paths on different machines)
- **Temporary overrides** during development or debugging
- **Personal API keys** for local testing
- **Individual preferences** that differ from team defaults

### Example Local Settings

```json
{
  "permissions": {
    "allow": [
      "Bash(docker:*)",
      "Bash(make:*)",
      "Read(/Users/myname/Documents/notes/**)"
    ]
  },
  "env": {
    "DEBUG": "true",
    "LOG_LEVEL": "verbose"
  },
  "cleanupPeriodDays": 7,
  "sandbox": {
    "enabled": false
  }
}
```

### Important Notes on Local Settings

- **Automatic gitignore**: Claude Code automatically configures git to ignore `.claude/settings.local.json` when it's created
- **Merge behaviour**: Settings from this file are merged with project and user settings, with local settings taking precedence
- **Known issues**: There have been reported bugs where programmatic updates to this file may overwrite manual changes instead of merging them
- **Security**: For maximum security on sensitive projects, use a separate `settings.local.json` without sudo permissions

---

## Enterprise/Managed Settings

### Location and Purpose

Enterprise/managed settings provide organisation-level control over Claude Code configurations. These settings are distributed automatically through the Claude.ai admin console and cannot be overridden by individual users.

### When to Use Managed Settings

- **Security policies** that must be enforced across the organisation
- **Compliance requirements** (e.g., data handling, audit logging)
- **Company-wide tool restrictions**
- **Standardised workflows** across all teams
- **Hook restrictions** (`allowManagedHooksOnly: true`)
- **Plugin marketplace allowlisting** (`strictKnownMarketplaces`)

### Example Managed Settings

```json
{
  "permissions": {
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "WebFetch",
      "Read(**/credentials/**)",
      "Read(**/.env*)"
    ]
  },
  "allowManagedHooksOnly": true,
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "npm",
      "package": "@acme-corp/claude-tools"
    }
  ],
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/etc/claude-code/hooks/security-check.sh"
          }
        ]
      }
    ]
  },
  "companyAnnouncements": [
    "All code must comply with ACME security standards",
    "Contact [email protected] for Claude Code support"
  ]
}
```

### Enterprise Features

- **Automatic distribution**: Settings are fetched when users authenticate
- **Cannot be overridden**: Highest precedence in the hierarchy
- **Available to**: Claude for Enterprise customers
- **CLAUDE.md support**: System-wide CLAUDE.md files can be placed in enterprise locations

---

## How Settings Merge and Override

### Merging Behaviour

Claude Code settings **merge hierarchically** rather than replace entirely. This means:

1. **Lower-priority settings provide defaults**: User settings establish baseline behaviour
2. **Higher-priority settings augment or override**: Project settings can add to or override user settings
3. **Unspecified settings inherit**: If a setting isn't defined at a higher level, the lower level's value is preserved

### Practical Example

**User settings** (`~/.claude/settings.json`):
```json
{
  "permissions": {
    "allow": ["Bash(git status)", "Bash(npm run:*)"],
    "deny": ["Read(.env)"]
  },
  "env": {
    "DEBUG": "false"
  }
}
```

**Project settings** (`.claude/settings.json`):
```json
{
  "permissions": {
    "allow": ["Bash(pytest:*)"],
    "deny": ["Bash(npm run build)"]
  },
  "env": {
    "NODE_ENV": "production"
  }
}
```

**Resulting merged configuration**:
```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(npm run:*)",
      "Bash(pytest:*)"
    ],
    "deny": [
      "Read(.env)",
      "Bash(npm run build)"
    ]
  },
  "env": {
    "DEBUG": "false",
    "NODE_ENV": "production"
  }
}
```

### Override Priority Examples

**Example 1: Permission Conflict**
- **User settings**: `allow: ["Bash(curl:*)"]`
- **Project settings**: `deny: ["Bash(curl:*)"]`
- **Result**: Project setting takes precedence → `curl` is **denied**

**Example 2: Local Override**
- **Project settings**: `sandbox.enabled: true`
- **Local settings**: `sandbox.enabled: false`
- **Result**: Local setting takes precedence → sandbox is **disabled**

**Example 3: Managed Settings Override**
- **User settings**: `allow: ["WebFetch"]`
- **Managed settings**: `deny: ["WebFetch"]`
- **Result**: Managed setting takes precedence → WebFetch is **denied**

---

## Common Settings Options

### Permissions

Permissions control which tools and operations Claude Code can perform. They use pattern matching for granular control.

#### Permission Structure

```json
{
  "permissions": {
    "allow": ["Tool(pattern)"],
    "deny": ["Tool(pattern)"],
    "ask": ["Tool(pattern)"],
    "additionalDirectories": ["../path"],
    "defaultMode": "acceptEdits"
  }
}
```

#### Available Tools

- `Bash(command:*)` - Bash command execution (prefix matching)
- `Read(path/pattern)` - File reading operations
- `Write(path/pattern)` - File writing operations
- `Edit(path/pattern)` - File editing operations
- `WebFetch` - Web content fetching
- `WebSearch` - Web searching

#### Pattern Matching

- **Bash patterns use prefix matching**: `Bash(npm run:*)` matches any command starting with "npm run"
- **File paths support wildcards**: `Read(**/.env*)` matches all .env files recursively
- **Exact matches**: `Bash(git status)` matches only that exact command

### Environment Variables

Set environment variables that apply to all Claude Code sessions:

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-...",
    "ANTHROPIC_MODEL": "claude-sonnet-4-5-20250929",
    "NODE_ENV": "development",
    "DEBUG": "true",
    "DISABLE_TELEMETRY": "1",
    "BASH_DEFAULT_TIMEOUT_MS": "120000",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

### Hooks

Hooks execute custom commands at various lifecycle points:

#### Hook Types

- **PreToolUse**: Executes before a tool runs (can block execution)
- **PostToolUse**: Executes after a tool completes
- **SessionStart**: Executes when a session begins
- **Notification**: Executes when notifications are sent
- **Stop**: Executes when a session stops

#### Hook Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/security-check.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ]
  },
  "disableAllHooks": false,
  "allowManagedHooksOnly": false
}
```

#### Hook Exit Codes

- **0**: Allow/OK (continue normally)
- **2**: Block (PreToolUse only, prevents tool execution)
- **Other non-zero**: Non-blocking error shown to user

#### Hook Return Values (JSON)

Hooks can return structured JSON for sophisticated control:

```json
{
  "continue": true,
  "approve": true,
  "stopReason": "optional reason",
  "suppressOutput": false
}
```

### Sandbox Configuration

Advanced sandboxing isolates bash operations (macOS/Linux):

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker", "vagrant"],
    "network": {
      "allowUnixSockets": ["~/.ssh/agent-socket"],
      "allowLocalBinding": true
    }
  }
}
```

### Attribution

Customise git commit and PR attribution:

```json
{
  "attribution": {
    "commit": "Co-authored-by: Claude AI <[email protected]>",
    "pr": "Generated with AI assistance"
  }
}
```

Set to empty strings to suppress attribution:

```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

### Plugin Management

```json
{
  "enabledPlugins": {
    "github-tools": true,
    "database-helpers": true,
    "custom-linters": false
  },
  "extraKnownMarketplaces": [
    {
      "source": "github",
      "repo": "my-org/plugins"
    }
  ],
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "approved-org/plugins"
    }
  ]
}
```

### Other Common Options

```json
{
  "model": "claude-opus-4-5-20251101",
  "cleanupPeriodDays": 30,
  "includeCoAuthoredBy": true,
  "defaultMode": "acceptEdits",
  "apiKeyHelper": "/path/to/key-helper.sh",
  "otelHeadersHelper": "/path/to/otel-helper.sh",
  "statusLine": {
    "enabled": true,
    "format": "custom"
  }
}
```

---

## Practical Configuration Examples

### Example 1: Frontend Development Team

**Team project settings** (`.claude/settings.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git diff:*)",
      "Bash(git status)",
      "Bash(npx prettier:*)",
      "Bash(npx eslint:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm install:*)",
      "Bash(npm ci)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(./secrets/**)",
      "Bash(rm:*)",
      "Bash(curl:*)",
      "WebFetch"
    ],
    "additionalDirectories": ["../design-system/"]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\" && npx eslint --fix \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'source ~/.nvm/nvm.sh && nvm use' >> \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  },
  "env": {
    "NODE_ENV": "development",
    "CI": "false"
  },
  "companyAnnouncements": [
    "Use TypeScript for all new components",
    "Follow the design system guidelines",
    "Write tests for new features"
  ],
  "attribution": {
    "commit": "Co-authored-by: Claude AI <[email protected]>"
  }
}
```

### Example 2: Python Data Science Project

**Team project settings** (`.claude/settings.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(pytest:*)",
      "Bash(python:*)",
      "Bash(pip list)",
      "Bash(jupyter:*)",
      "Bash(git:*)",
      "Read(./data/**)",
      "Read(./notebooks/**)"
    ],
    "ask": [
      "Bash(pip install:*)",
      "Write(./data/**)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(./credentials/**)",
      "Bash(rm:*)",
      "Bash(curl:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "black \"$CLAUDE_FILE_PATHS\" && isort \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'source .venv/bin/activate' >> \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  },
  "env": {
    "PYTHONPATH": "./src",
    "JUPYTER_CONFIG_DIR": "./.jupyter"
  }
}
```

### Example 3: Individual Developer's User Settings

**User settings** (`~/.claude/settings.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git branch:*)",
      "Read(~/.gitconfig)",
      "Read(~/.zshrc)",
      "Read(~/.vimrc)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Read(.env*)",
      "Read(**/secrets/**)",
      "Read(**/.aws/**)"
    ]
  },
  "env": {
    "ANTHROPIC_MODEL": "claude-sonnet-4-5-20250929",
    "DISABLE_TELEMETRY": "1",
    "EDITOR": "vim",
    "BASH_DEFAULT_TIMEOUT_MS": "60000"
  },
  "cleanupPeriodDays": 14,
  "attribution": {
    "commit": "Co-authored-by: Claude <[email protected]>",
    "pr": ""
  },
  "includeCoAuthoredBy": true
}
```

### Example 4: Security-First Enterprise Configuration

**Managed settings** (`/etc/claude-code/managed-settings.json`):

```json
{
  "permissions": {
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(nc:*)",
      "Bash(sudo:*)",
      "Bash(rm -rf:*)",
      "WebFetch",
      "Read(**/credentials/**)",
      "Read(**/.env*)",
      "Read(**/.aws/**)",
      "Read(**/.ssh/**)",
      "Write(**/production/**)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(docker:*)"
    ]
  },
  "allowManagedHooksOnly": true,
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/etc/claude-code/hooks/security-audit.sh"
          }
        ]
      }
    ]
  },
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    }
  ],
  "env": {
    "DISABLE_TELEMETRY": "0",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_ENDPOINT": "https://telemetry.acme-corp.com"
  },
  "companyAnnouncements": [
    "All development must comply with SOC2 requirements",
    "Report security issues to [email protected]",
    "Code reviews required for all PRs"
  ]
}
```

### Example 5: Local Development Overrides

**Local settings** (`.claude/settings.local.json`):

```json
{
  "permissions": {
    "allow": [
      "Bash(docker:*)",
      "Bash(make:*)",
      "Bash(terraform:*)",
      "Read(/Users/alice/Documents/notes/**)",
      "Read(/Users/alice/scratch/**)"
    ]
  },
  "env": {
    "DEBUG": "true",
    "LOG_LEVEL": "verbose",
    "LOCAL_API_URL": "http://localhost:3000"
  },
  "sandbox": {
    "enabled": false
  },
  "cleanupPeriodDays": 7
}
```

---

## Best Practices

### Team vs. Individual Settings

#### Use Team Settings (`.claude/settings.json`) For:

✅ **Security policies** - Deny access to sensitive files and dangerous commands
✅ **Code quality standards** - Enforce formatting, linting, and testing
✅ **Project-specific workflows** - Common git commands, build scripts
✅ **Shared hooks** - Automatic formatting, type checking, pre-commit validation
✅ **Environment defaults** - Development environment variables
✅ **Documentation** - Company announcements, onboarding instructions
✅ **Access control** - Additional directories, permitted tools

#### Use Individual Settings (`~/.claude/settings.json`) For:

✅ **Personal preferences** - Your preferred model, cleanup period
✅ **Global tools** - Commands you use across all projects
✅ **API key helpers** - Personal credential management scripts
✅ **Attribution preferences** - Your co-authorship format
✅ **Personal security baselines** - Default denials for sensitive operations

#### Use Local Settings (`.claude/settings.local.json`) For:

✅ **Experimentation** - Testing new permissions before proposing to team
✅ **Machine-specific paths** - Local file system locations
✅ **Debug settings** - Verbose logging, extended timeouts
✅ **Temporary overrides** - Disabling sandbox for specific debugging
✅ **Personal productivity tools** - Docker, make, terraform on your machine

### General Best Practices

#### 1. Start Restrictive, Then Open Up

Begin with minimal permissions and gradually add more as needed:

```json
{
  "permissions": {
    "deny": [
      "Bash(*)",
      "WebFetch",
      "Write(*)"
    ],
    "allow": [
      "Read(*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ]
  }
}
```

#### 2. Use `ask` for Potentially Dangerous Operations

For operations that should require confirmation:

```json
{
  "permissions": {
    "ask": [
      "Bash(git push:*)",
      "Bash(npm install:*)",
      "Bash(docker:*)",
      "Write(.env*)"
    ]
  }
}
```

#### 3. Leverage Hooks for Consistency

Automate code quality with hooks:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ]
  }
}
```

#### 4. Document Your Settings

Add comments via company announcements:

```json
{
  "companyAnnouncements": [
    "This project uses Prettier for formatting",
    "Run 'npm run test' before committing",
    "Contact @devops for permission questions"
  ]
}
```

#### 5. Protect Sensitive Files

Always deny access to credentials and secrets:

```json
{
  "permissions": {
    "deny": [
      "Read(.env*)",
      "Read(**/.env*)",
      "Read(./secrets/**)",
      "Read(**/credentials/**)",
      "Read(**/.aws/**)",
      "Read(**/.ssh/**)",
      "Write(.env*)"
    ]
  }
}
```

#### 6. Use Additional Directories Wisely

Grant access to shared resources:

```json
{
  "permissions": {
    "additionalDirectories": [
      "../shared-components/",
      "../design-system/",
      "../documentation/"
    ]
  }
}
```

#### 7. Version Control Your Team Settings

**Do commit** (`.claude/settings.json`):
- Team permissions
- Shared hooks
- Project environment variables
- Company announcements

**Don't commit** (`.claude/settings.local.json`):
- Personal API keys
- Machine-specific paths
- Experimental configurations
- Debug settings

#### 8. Test Settings Changes

Before committing team settings:
1. Test in `.claude/settings.local.json` first
2. Verify hooks work correctly
3. Ensure permissions don't block legitimate workflows
4. Get team feedback

#### 9. Use SessionStart Hooks for Environment Setup

Automatically activate virtual environments or load tools:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'conda activate myenv' >> \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  }
}
```

#### 10. Monitor and Audit

For enterprise deployments:
- Use managed settings to enforce policies
- Enable telemetry for audit trails
- Set `allowManagedHooksOnly: true` for security
- Use PreToolUse hooks for logging and validation

### Common Pitfalls to Avoid

❌ **Don't commit `.claude/settings.local.json`** - It's for personal use only
❌ **Don't put API keys in committed settings** - Use environment variables or key helpers
❌ **Don't make team settings too restrictive** - Allow reasonable development workflows
❌ **Don't forget to test hooks** - Broken hooks can block all development
❌ **Don't override security policies locally** - Respect team and enterprise restrictions
❌ **Don't use overly broad permissions** - Be specific with patterns and paths
❌ **Don't ignore the merge hierarchy** - Understand how settings combine

---

## Additional Resources

### Official Documentation

- **Claude Code Settings Documentation**: [https://code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)
- **Claude Code Overview**: [https://docs.anthropic.com/en/docs/claude-code/overview](https://docs.anthropic.com/en/docs/claude-code/overview)
- **Claude Code on GitHub**: [https://github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)

### Community Guides and Examples

- **A developer's guide to settings.json in Claude Code**: [https://www.eesel.ai/blog/settings-json-claude-code](https://www.eesel.ai/blog/settings-json-claude-code)
- **Claude Code Configuration Guide - ClaudeLog**: [https://claudelog.com/configuration/](https://claudelog.com/configuration/)
- **Claude Code CLI Cheatsheet - Shipyard**: [https://shipyard.build/blog/claude-code-cheat-sheet/](https://shipyard.build/blog/claude-code-cheat-sheet/)
- **How I use Claude Code (+ best tips)**: [https://www.builder.io/blog/claude-code](https://www.builder.io/blog/claude-code)

### GitHub Example Repositories

- **feiskyer/claude-code-settings**: [https://github.com/feiskyer/claude-code-settings](https://github.com/feiskyer/claude-code-settings)
- **centminmod/my-claude-code-setup**: [https://github.com/centminmod/my-claude-code-setup](https://github.com/centminmod/my-claude-code-setup)
- **dwillitzer/claude-settings**: [https://github.com/dwillitzer/claude-settings](https://github.com/dwillitzer/claude-settings)
- **disler/claude-code-hooks-mastery**: [https://github.com/disler/claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- **zebbern/claude-code-guide**: [https://github.com/zebbern/claude-code-guide](https://github.com/zebbern/claude-code-guide)

### Specialised Topics

- **Claude Code Hook Examples**: [https://stevekinney.com/courses/ai-development/claude-code-hook-examples](https://stevekinney.com/courses/ai-development/claude-code-hook-examples)
- **Understanding Claude Code Permissions and Security**: [https://www.petefreitag.com/blog/claude-code-permissions/](https://www.petefreitag.com/blog/claude-code-permissions/)
- **Setting Global Instructions Guide**: [https://naqeebali-shamsi.medium.com/the-complete-guide-to-setting-global-instructions-for-claude-code-cli-cec8407c99a0](https://naqeebali-shamsi.medium.com/the-complete-guide-to-setting-global-instructions-for-claude-code-cli-cec8407c99a0)

---

## Conclusion

Claude Code's settings hierarchy provides a powerful and flexible system for configuring AI-assisted development workflows. By understanding the five-tier precedence system and leveraging the appropriate configuration level for each use case, teams can:

- **Maintain security** through enterprise-managed policies
- **Ensure consistency** with team-shared project settings
- **Preserve flexibility** via personal user and local settings
- **Automate quality** through hooks and permissions
- **Scale effectively** from individual developers to large organisations

The key to success is finding the right balance between security, productivity, and flexibility—using restrictive defaults with explicit allowances, leveraging hooks for automation, and respecting the hierarchy when configuring settings at different levels.

Whether you're an individual developer customising your workflow, a team lead establishing project standards, or an enterprise administrator enforcing organisational policies, Claude Code's settings system provides the tools you need to work efficiently and securely.

---

**Document Version**: 1.0
**Last Updated**: January 2, 2026
**Author**: Researched and compiled from official Anthropic documentation and community resources
