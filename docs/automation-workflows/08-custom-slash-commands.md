# Custom Slash Commands in Claude Code: Complete Guide

## Table of Contents
- [Introduction](#introduction)
- [Command Directory Structure](#command-directory-structure)
- [Creating Your First Command](#creating-your-first-command)
- [Command File Format](#command-file-format)
- [Dynamic Arguments](#dynamic-arguments)
- [Practical Examples](#practical-examples)
- [Project vs User-Level Commands](#project-vs-user-level-commands)
- [Command Naming Conventions](#command-naming-conventions)
- [Advanced Command Patterns](#advanced-command-patterns)
- [Best Practices](#best-practices)
- [Real-World Use Cases](#real-world-use-cases)

## Introduction

Custom slash commands transform repetitive prompts into reusable shortcuts, enabling you to codify frequently-used workflows as simple Markdown files. Instead of typing the same instructions repeatedly, you invoke commands using `/command-name` syntax, passing optional arguments for dynamic execution.

Slash commands provide a powerful abstraction layer between your intent and Claude's execution, enabling consistent, repeatable workflows across your team. They're particularly valuable for complex multi-step processes, integration with CLI tools, and domain-specific development patterns.

## Command Directory Structure

Claude Code recognises custom slash commands from two distinct locations, each serving different purposes:

### Project Commands: `.claude/commands/`

Project commands live in your repository at `.claude/commands/` and are:
- **Shared with your team** when committed to version control
- **Project-specific** to the current repository
- **Marked "(project)"** when listed in `/help`
- **Available to all team members** who clone the repository

Example structure:
```
my-project/
└── .claude/
    └── commands/
        ├── fix-issue.md
        ├── review-pr.md
        └── workflows/
            ├── feature-development.md
            └── security-scan.md
```

### Personal Commands: `~/.claude/commands/`

User-level commands reside in your home directory at `~/.claude/commands/` and are:
- **Available across all projects** in all sessions
- **Personal to you** and not shared via version control
- **Marked "(user)"** when listed in `/help`
- **Perfect for personal productivity workflows**

Example structure:
```
~/.claude/
└── commands/
    ├── analyse-architecture.md
    ├── optimise-code.md
    └── personal/
        ├── daily-standup.md
        └── review-notes.md
```

## Creating Your First Command

Creating a custom slash command is straightforward. The filename (without `.md` extension) becomes the command name.

### Simple Example

Create `.claude/commands/optimise.md`:
```markdown
Analyse this code for performance issues and suggest optimisations. Focus on:
- Time complexity
- Memory usage
- Common bottlenecks
- Best practices for the detected language
```

**Usage:** Type `/optimise` in Claude Code, and Claude will execute this prompt.

### Command Creation Steps

1. **Create the directory** (if it doesn't exist):
   ```bash
   mkdir -p .claude/commands
   ```

2. **Create your command file**:
   ```bash
   touch .claude/commands/my-command.md
   ```

3. **Write your prompt** in the Markdown file

4. **Test it** by typing `/my-command` in Claude Code

5. **Check availability** using `/help` to see your command listed

## Command File Format

Custom slash commands are Markdown files that can include optional YAML frontmatter for metadata and configuration.

### Basic Format

```markdown
Your prompt instructions go here. Be specific and detailed.
```

### Format with Frontmatter

```markdown
---
description: Brief description shown in /help
argument-hint: [issue-number] [priority]
allowed-tools: Bash(git:*), Read, Edit
model: claude-sonnet-4-5-20250929
---

Your detailed command instructions here.
```

### Frontmatter Options

**Essential Fields:**

- **`description`**: Brief explanation of what the command does (shown in `/help` and required for SlashCommand tool)
- **`argument-hint`**: Shows expected arguments during autocomplete (e.g., `[file] [mode]`)
- **`allowed-tools`**: Restricts which tools Claude can use during execution
- **`model`**: Specifies which Claude model to use for this command
- **`disable-model-invocation`**: Set to `true` to prevent the SlashCommand tool from executing it programmatically

**Example with Full Frontmatter:**

```markdown
---
description: Create a git commit with provided message
argument-hint: [commit-message]
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
model: claude-3-5-haiku-20241022
---

Create a git commit with the following message: $ARGUMENTS

Steps:
1. Run git status to check current changes
2. Add relevant files to staging
3. Create commit with the provided message
4. Show the commit summary
```

## Dynamic Arguments

One of the most powerful features of custom slash commands is the ability to accept dynamic input through argument placeholders.

### Using `$ARGUMENTS` (All Arguments)

`$ARGUMENTS` captures everything passed to the command after the command name.

**Command File:** `.claude/commands/fix-issue.md`
```markdown
---
description: Fix a GitHub issue by number
argument-hint: [issue-number] [additional-notes]
---

Fix issue #$ARGUMENTS following our coding standards:
1. Use the gh CLI to fetch issue details
2. Analyse the problem and propose a solution
3. Implement the fix
4. Write appropriate tests
5. Update documentation if needed
```

**Usage:**
```
/fix-issue 123 high-priority security-related
```

In this example, `$ARGUMENTS` becomes `"123 high-priority security-related"`.

### Using Positional Parameters ($1, $2, $3...)

For more structured commands, use positional parameters similar to shell scripts.

**Command File:** `.claude/commands/review-pr.md`
```markdown
---
description: Review a pull request with specific focus
argument-hint: [pr-number] [focus-area] [reviewer-name]
---

Review pull request #$1 with focus on $2 and assign to $3.

Review steps:
1. Fetch PR #$1 using gh CLI
2. Focus particularly on $2 aspects
3. Check code quality, tests, and documentation
4. Provide constructive feedback
5. Tag $3 for follow-up review
```

**Usage:**
```
/review-pr 456 security alice
```

Here, `$1` = "456", `$2` = "security", `$3` = "alice".

### Combining Positional and All Arguments

You can mix approaches for flexible patterns:

```markdown
---
description: Create feature branch with optional flags
argument-hint: [feature-name] [additional-flags]
---

Create a feature branch named "$1" with the following specifications: ${@:2}

1. Create branch: feature/$1
2. Apply additional settings: ${@:2}
3. Set up tracking and push to remote
```

## Practical Examples

### Example 1: GitHub Issue Fixer

**File:** `.claude/commands/fix-github-issue.md`
```markdown
---
description: Automatically fix a GitHub issue end-to-end
argument-hint: [issue-number]
allowed-tools: Bash(gh:*), Read, Edit, Grep, Glob
---

Fix GitHub issue #$ARGUMENTS by following these steps:

1. **Fetch issue details** using: `gh issue view $ARGUMENTS`
2. **Analyse the problem** based on the issue description
3. **Search the codebase** for relevant files using Grep and Glob
4. **Implement the solution** with proper error handling
5. **Write or update tests** to cover the fix
6. **Verify the fix** by running the test suite
7. **Update the issue** with a summary of changes
8. **Offer to create a PR** linking to the issue

Always follow our coding standards and ensure backward compatibility.
```

**Usage:** `/fix-github-issue 789`

### Example 2: Code Review Assistant

**File:** `.claude/commands/code-review.md`
```markdown
---
description: Perform comprehensive code review
allowed-tools: Bash(git:*), Read, Grep
model: claude-sonnet-4-5-20250929
---

Perform a thorough code review focusing on:

## Code Quality
- Readability and maintainability
- Adherence to project conventions
- DRY principle and code duplication

## Architecture
- Design patterns and structure
- Separation of concerns
- Modularity and reusability

## Testing
- Test coverage adequacy
- Edge cases handled
- Test quality and clarity

## Security
- Input validation
- Authentication/authorisation
- Common vulnerabilities (OWASP Top 10)

## Performance
- Algorithmic efficiency
- Resource usage
- Scalability considerations

## Documentation
- Code comments where needed
- API documentation
- README updates

Provide specific, actionable feedback with examples.
```

**Usage:** `/code-review`

### Example 3: Test Generator

**File:** `.claude/commands/generate-tests.md`
```markdown
---
description: Generate comprehensive tests for a file
argument-hint: [file-path]
allowed-tools: Read, Write, Bash
---

Generate comprehensive test coverage for: $ARGUMENTS

Steps:
1. Read the file at $ARGUMENTS
2. Detect the testing framework (Jest, pytest, Go testing, etc.)
3. Analyse all functions, methods, and classes
4. Generate tests covering:
   - Happy path scenarios
   - Edge cases
   - Error handling
   - Boundary conditions
   - Integration points
5. Follow project testing conventions
6. Save tests in appropriate location
7. Run tests to verify they work
8. Show coverage report

Ensure tests are clear, maintainable, and follow best practices.
```

**Usage:** `/generate-tests src/utils/validator.js`

### Example 4: Database Migration Creator

**File:** `.claude/commands/create-migration.md`
```markdown
---
description: Create a database migration
argument-hint: [migration-name] [description]
allowed-tools: Write, Bash, Read
---

Create a database migration named "$1" for: ${@:2}

1. Detect the migration framework (Alembic, Flyway, Rails, etc.)
2. Generate migration file with timestamp
3. Create both upgrade and downgrade paths
4. Include:
   - Clear migration description
   - Schema changes
   - Data migrations if needed
   - Rollback strategy
   - Index creation
5. Add comments explaining complex changes
6. Verify migration syntax
7. Show how to apply and rollback

Follow project migration conventions and ensure idempotency.
```

**Usage:** `/create-migration add_user_roles "Add roles table and user_roles junction table"`

### Example 5: API Endpoint Scaffolder

**File:** `.claude/commands/scaffold-api.md`
```markdown
---
description: Scaffold a complete API endpoint
argument-hint: [resource-name] [methods]
allowed-tools: Write, Read, Edit, Bash
---

Scaffold API endpoint for resource: $1 with methods: ${@:2}

Generate the following:

## Route Definition
- Define routes for specified HTTP methods
- Apply appropriate middleware
- Add request validation

## Controller/Handler
- Implement handler functions
- Input validation and sanitization
- Error handling with proper status codes
- Response formatting

## Data Model
- Define schema/model
- Add validation rules
- Include timestamps and metadata

## Tests
- Unit tests for handlers
- Integration tests for endpoints
- Test success and error cases

## Documentation
- OpenAPI/Swagger spec
- Example requests/responses
- Authentication requirements

Follow REST best practices and project conventions.
```

**Usage:** `/scaffold-api products "GET POST PUT DELETE"`

## Project vs User-Level Commands

Understanding when to use project-level versus user-level commands is crucial for team collaboration and personal productivity.

### Project-Level Commands (`.claude/commands/`)

**Use when:**
- Commands are specific to the project's domain or architecture
- The entire team should use the same workflow
- Commands integrate with project-specific tools or processes
- You want to standardise team practices

**Examples:**
- `/fix-github-issue` - Team's standard issue resolution workflow
- `/deploy-staging` - Project-specific deployment process
- `/run-e2e-tests` - Project's end-to-end testing suite
- `/create-component` - Framework-specific component generation

**Best Practices:**
- Commit to version control
- Document in your project's README
- Review command updates in PRs
- Keep commands updated as project evolves
- Use clear, project-specific naming

### User-Level Commands (`~/.claude/commands/`)

**Use when:**
- Commands reflect your personal coding style or preferences
- Workflows are applicable across multiple projects
- You're experimenting with new command patterns
- Commands contain personal preferences or shortcuts

**Examples:**
- `/analyse-architecture` - Your personal architecture review checklist
- `/daily-standup` - Your standup report template
- `/code-golf` - Personal code optimisation experiments
- `/explain-like-im-five` - Personal learning preference

**Best Practices:**
- Organise by category in subdirectories
- Create a personal README documenting your commands
- Back up your `~/.claude/commands/` directory
- Share useful patterns with team (promote to project-level)
- Refine based on usage patterns

### Hierarchy and Precedence

When commands exist in both locations with the same name, **project commands take precedence** over user commands. This allows projects to override general commands with project-specific implementations.

## Command Naming Conventions

Effective naming conventions make commands discoverable, memorable, and maintainable. The Claude Code ecosystem has evolved several patterns.

### Basic Naming Patterns

**Kebab-case is standard:**
```
fix-issue.md          ✓ Recommended
create-component.md   ✓ Recommended
fixIssue.md          ✗ Avoid camelCase
fix_issue.md         ✗ Avoid snake_case
```

**Verb-first naming:**
```
analyse-code.md       ✓ Action-oriented
generate-tests.md     ✓ Clear purpose
review-pr.md          ✓ Explicit action
```

**Resource-focused naming:**
```
code-analyser.md      ~ Acceptable but less clear
test-generator.md     ~ Less action-oriented
```

### Namespace Organisation

Organise related commands in subdirectories for logical grouping. The directory name provides context without affecting the command invocation.

**Directory Structure:**
```
.claude/commands/
├── workflows/
│   ├── feature-development.md    (/workflows:feature-development)
│   ├── security-scan.md          (/workflows:security-scan)
│   └── deployment.md             (/workflows:deployment)
├── tools/
│   ├── format-code.md            (/tools:format-code)
│   ├── check-deps.md             (/tools:check-deps)
│   └── analyse-bundle.md         (/tools:analyse-bundle)
└── github/
    ├── create-issue.md           (/github:create-issue)
    ├── review-pr.md              (/github:review-pr)
    └── sync-linear.md            (/github:sync-linear)
```

**Common Namespace Categories:**

- **`workflows/`** - Multi-step orchestration commands
- **`tools/`** - Single-purpose utility commands
- **`dev/`** - Development-focused commands
- **`test/`** - Testing-related commands
- **`security/`** - Security scanning and hardening
- **`deploy/`** - Deployment and release commands
- **`docs/`** - Documentation generation
- **`setup/`** - Environment and configuration
- **`team/`** - Collaboration and communication

### Domain-Specific Conventions

**For GitHub/GitLab workflows:**
```
fix-issue.md
review-pr.md
create-release.md
sync-fork.md
```

**For testing:**
```
generate-tests.md
run-coverage.md
test-e2e.md
snapshot-update.md
```

**For deployment:**
```
deploy-staging.md
deploy-production.md
rollback-release.md
health-check.md
```

**For code quality:**
```
lint-fix.md
format-all.md
analyse-complexity.md
check-security.md
```

## Advanced Command Patterns

### Pattern 1: Bash Command Integration

Execute bash commands before Claude processes the prompt by using the `!` prefix. The output becomes part of the context.

**File:** `.claude/commands/explain-git-status.md`
```markdown
---
description: Explain current git changes with context
allowed-tools: Bash(git:*)
---

Current repository status:
!`git status`

Staged changes:
!`git diff --cached`

Unstaged changes:
!`git diff`

Please explain:
1. What changes are staged and why they might be grouped
2. What's modified but not staged
3. Suggestions for logical commit organisation
4. Any potential issues or conflicts
```

The `!` prefix executes the command, and the output is embedded before Claude processes the prompt.

### Pattern 2: File References with @

Reference specific files directly in your commands using the `@` prefix.

**File:** `.claude/commands/compare-implementations.md`
```markdown
---
description: Compare two file implementations
argument-hint: [file1] [file2]
---

Compare the implementation approaches between @$1 and @$2.

Analyse:
1. Architectural differences
2. Performance implications
3. Code complexity and maintainability
4. Best practices adherence
5. Recommendation for which approach to standardise on

Provide specific examples from both files.
```

**Usage:** `/compare-implementations src/old-api.js src/new-api.js`

### Pattern 3: Multi-Agent Workflows

Create commands that delegate to multiple specialised "agents" for complex tasks.

**File:** `.claude/commands/workflows/full-feature.md`
```markdown
---
description: Complete feature development workflow
argument-hint: [feature-description]
---

Implement the following feature end-to-end: $ARGUMENTS

Delegate to specialised agents in sequence:

## Phase 1: Architecture (Architect Agent)
- Design system architecture
- Define interfaces and contracts
- Identify dependencies
- Plan database schema if needed

## Phase 2: Implementation (Development Agent)
- Implement core functionality
- Follow established patterns
- Add error handling
- Include logging

## Phase 3: Testing (QA Agent)
- Generate comprehensive tests
- Test edge cases
- Verify error handling
- Check integration points

## Phase 4: Security (Security Agent)
- Review for vulnerabilities
- Check input validation
- Verify authentication/authorisation
- Scan dependencies

## Phase 5: Documentation (Documentation Agent)
- Write API documentation
- Update README
- Add code comments
- Create usage examples

## Phase 6: Code Review (Review Agent)
- Final quality check
- Performance review
- Best practices verification
- Suggest improvements

Coordinate between agents, maintaining context throughout.
```

### Pattern 4: Conditional Execution

Create commands that adapt based on project structure or environment.

**File:** `.claude/commands/run-tests.md`
```markdown
---
description: Auto-detect and run project tests
allowed-tools: Bash, Read, Glob
---

Run the test suite for this project:

1. **Detect test framework:**
   - Check for package.json (Jest, Mocha, Vitest)
   - Check for pytest.ini or setup.py (pytest)
   - Check for go.mod (Go testing)
   - Check for Cargo.toml (Rust tests)
   - Check for pom.xml or build.gradle (JUnit)

2. **Run appropriate command:**
   - npm test / yarn test
   - pytest
   - go test ./...
   - cargo test
   - mvn test / gradle test

3. **Analyse results:**
   - Show pass/fail summary
   - Highlight any failing tests
   - Show coverage if available
   - Suggest fixes for failures

4. **If tests fail:**
   - Identify root cause
   - Propose fixes
   - Offer to implement fixes
   - Re-run after fixes
```

### Pattern 5: Iterative Workflows

Commands that loop until a condition is met.

**File:** `.claude/commands/fix-until-green.md`
```markdown
---
description: Fix issues until all tests pass
allowed-tools: Bash, Read, Edit, Grep
---

Iteratively fix issues until all tests pass:

## Loop Process:
1. Run test suite
2. If all pass → DONE, summarise changes
3. If failures exist:
   a. Analyse the first failure
   b. Identify root cause
   c. Implement fix
   d. Return to step 1

## Rules:
- Fix one issue at a time
- Verify each fix doesn't break other tests
- Maximum 10 iterations (prevent infinite loops)
- If stuck, ask for human guidance
- Keep a log of all changes

## Success Criteria:
- All tests passing
- No new test failures introduced
- Code quality maintained
```

### Pattern 6: Context-Aware Commands

Commands that gather context before executing.

**File:** `.claude/commands/smart-refactor.md`
```markdown
---
description: Context-aware code refactoring
argument-hint: [target-file]
allowed-tools: Read, Edit, Grep, Bash
---

Refactor $ARGUMENTS with full context awareness:

## Context Gathering:
1. Read target file: @$ARGUMENTS
2. Find all files importing/using target: `grep -r "import.*$ARGUMENTS"`
3. Read test files for target
4. Check git history: `git log --follow $ARGUMENTS`
5. Identify related files in same directory

## Analysis:
- Current code smells and issues
- Usage patterns from importers
- Test coverage gaps
- Historical change frequency
- Coupling and dependencies

## Refactoring:
- Apply appropriate patterns
- Maintain backward compatibility OR provide migration guide
- Update all dependent files
- Update or create tests
- Update documentation

## Verification:
- Run affected tests
- Check type errors
- Verify no broken imports
- Performance comparison if relevant
```

### Pattern 7: Report Generation

Commands that produce structured output or documentation.

**File:** `.claude/commands/security-audit-report.md`
```markdown
---
description: Generate comprehensive security audit report
allowed-tools: Bash, Read, Grep, Write
---

Generate a security audit report for this project:

## Scan Categories:

### 1. Dependency Vulnerabilities
- Run: `npm audit` or `pip-audit` or equivalent
- List all high/critical vulnerabilities
- Provide remediation steps

### 2. Code Security
- SQL injection risks
- XSS vulnerabilities
- CSRF protection
- Input validation
- Authentication/authorisation issues

### 3. Configuration Security
- Environment variable handling
- Secrets in code (API keys, passwords)
- CORS configuration
- Security headers

### 4. Infrastructure
- Docker security
- CI/CD security
- Cloud configuration

## Report Format:

```markdown
# Security Audit Report
**Date:** [current-date]
**Project:** [project-name]

## Executive Summary
- Total issues found: X
- Critical: X
- High: X
- Medium: X
- Low: X

## Critical Issues
[Detailed list with remediation]

## Recommendations
[Prioritised action items]

## Compliance
[Relevant standards: OWASP, PCI-DSS, etc.]
```

Save report to: `docs/security-audit-[date].md`
```

## Best Practices

### 1. Command Design Principles

**Be Explicit and Detailed:**
```markdown
❌ Fix the bug
✓ Analyse the issue, identify root cause, implement fix, add tests, verify solution
```

**Include Verification Steps:**
```markdown
✓ After implementation:
  1. Run test suite
  2. Check for type errors
  3. Verify no breaking changes
  4. Update documentation
```

**Specify Output Format:**
```markdown
✓ Provide output as:
  - Summary of changes
  - Files modified
  - Tests added/updated
  - Next steps for deployment
```

### 2. Tool Restrictions

Use `allowed-tools` to prevent unintended side effects:

```markdown
---
allowed-tools: Read, Grep, Glob  # Read-only command
---
```

```markdown
---
allowed-tools: Bash(git:*), Bash(npm:test)  # Limited bash access
---
```

### 3. Model Selection

Choose appropriate models based on task complexity:

```markdown
---
model: claude-3-5-haiku-20241022  # For fast, simple tasks
---
```

```markdown
---
model: claude-sonnet-4-5-20250929  # For complex reasoning
---
```

### 4. Documentation Within Commands

**Add context comments:**
```markdown
---
description: Deploy to staging environment
---

# Staging Deployment Process
<!-- This command follows our deployment checklist from docs/deployment.md -->

Deploy to staging with these steps:
1. Run pre-deployment checks
2. Build production bundle
3. Run smoke tests locally
4. Deploy to staging
5. Run E2E tests against staging
6. Notify team in Slack

<!-- Prerequisites: AWS credentials configured, Docker running -->
```

### 5. Error Handling

**Build in failure modes:**
```markdown
If deployment fails:
1. Check logs in CloudWatch
2. Verify AWS credentials
3. Check for infrastructure issues
4. Rollback if necessary
5. Alert on-call engineer
```

### 6. Argument Validation

**Guide users on proper usage:**
```markdown
---
argument-hint: [pr-number (required)] [focus-area (optional)]
---

Review PR #$1 ${2:+with focus on $2}

Note: PR number is required. Usage: /review-pr 123 [security|performance|tests]
```

### 7. Idempotency

**Design commands to be safely re-runnable:**
```markdown
Create database migration:
1. Check if migration already exists
2. If exists, ask before overwriting
3. Generate with unique timestamp
4. Verify no conflicts with existing migrations
```

### 8. Team Collaboration

**For project commands:**
- Document in project README
- Use consistent naming across team
- Review command changes in PRs
- Version control command updates
- Include command usage in onboarding

**Template for README section:**
```markdown
## Custom Slash Commands

Our project includes custom slash commands to streamline common workflows:

### Development
- `/fix-issue [number]` - Fix a GitHub issue end-to-end
- `/generate-tests [file]` - Generate comprehensive tests for a file
- `/code-review` - Perform thorough code review

### Deployment
- `/deploy-staging` - Deploy to staging environment
- `/deploy-production` - Production deployment workflow
- `/rollback [version]` - Rollback to previous version

See `.claude/commands/` for full list and implementation details.
```

### 9. Performance Considerations

**Avoid expensive operations without warning:**
```markdown
❌ Analyse all files in the repository
✓ Analyse all TypeScript files in src/ (approximately 50 files, ~30 seconds)
```

**Provide progress indicators:**
```markdown
This will:
1. Scan ~200 files [~1 minute]
2. Run security analysis [~2 minutes]
3. Generate report [~30 seconds]

Total estimated time: 3-4 minutes
```

### 10. Maintenance

**Regular review cycles:**
- Quarterly review of all project commands
- Remove unused commands
- Update commands when project evolves
- Consolidate similar commands
- Improve based on usage patterns

## Real-World Use Cases

### Use Case 1: Hugo Static Site Management

From the community, developers manage Hugo blogs with commands for:

**Post Creation** (`.claude/commands/posts/new.md`):
```markdown
---
description: Create new blog post with proper frontmatter
argument-hint: [post-title]
---

Create a new Hugo blog post: "$ARGUMENTS"

1. Generate kebab-case filename from title
2. Create file in content/posts/
3. Add frontmatter:
   - title
   - date (today's date)
   - draft: true
   - tags: []
   - categories: []
4. Add initial content structure
5. Open file for editing
```

**Link Validation** (`.claude/commands/site/check-links.md`):
```markdown
---
description: Validate all internal and external links
allowed-tools: Bash, Read, Grep
---

Check all links in Hugo content:

1. Find all markdown files
2. Extract all links
3. For internal links:
   - Verify file exists
   - Check anchors are valid
4. For external links:
   - Option to ping (can be slow)
5. Report broken links with file locations
```

### Use Case 2: GitHub PR Review Automation

**Comprehensive PR Review** (`.claude/commands/github/review-pr.md`):
```markdown
---
description: Automated PR review with GitHub integration
argument-hint: [pr-number]
allowed-tools: Bash(gh:*), Read, Grep
---

Review GitHub PR #$ARGUMENTS:

1. **Fetch PR details:** `gh pr view $ARGUMENTS`
2. **Checkout branch:** `gh pr checkout $ARGUMENTS`
3. **Analyse changes:**
   - Get file list: `gh pr diff $ARGUMENTS --name-only`
   - Review each changed file
4. **Check for:**
   - Code quality issues
   - Security vulnerabilities
   - Missing tests
   - Documentation needs
   - Breaking changes
5. **Run checks:**
   - Linting
   - Tests
   - Type checking
6. **Generate review:**
   - Positive feedback
   - Issues found with line numbers
   - Suggestions for improvement
7. **Post review:** Offer to comment on PR with findings
```

### Use Case 3: Multi-Service Microservices Development

**Service Scaffolding** (`.claude/commands/workflows/new-service.md`):
```markdown
---
description: Scaffold a new microservice with all boilerplate
argument-hint: [service-name]
---

Create new microservice: $ARGUMENTS

## Structure
services/$ARGUMENTS/
├── src/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── README.md
└── package.json (or equivalent)

## Generate:
1. **Service skeleton** following our architecture
2. **API endpoints** with OpenAPI spec
3. **Database models** and migrations
4. **Docker configuration** with health checks
5. **CI/CD pipeline** (.github/workflows/)
6. **Tests** (unit and integration)
7. **Documentation** with usage examples
8. **Service registration** in API gateway

## Integration:
- Add to docker-compose.yml
- Update API gateway routes
- Add to monitoring
- Configure logging
```

### Use Case 4: Database Migration Management

**Safe Migration Workflow** (`.claude/commands/db/create-migration.md`):
```markdown
---
description: Create database migration with safety checks
argument-hint: [migration-description]
allowed-tools: Bash, Write, Read
---

Create migration: "$ARGUMENTS"

## Pre-checks:
1. Verify database connection
2. Check for pending migrations
3. Backup current schema

## Migration Generation:
1. Create migration file with timestamp
2. Include both upgrade and downgrade
3. Add these sections:
   ```sql
   -- Upgrade
   BEGIN;
   [changes here]
   COMMIT;

   -- Downgrade
   BEGIN;
   [rollback here]
   COMMIT;
   ```

## Safety Features:
- Transaction wrapping
- Idempotency checks
- Foreign key handling
- Index creation (CONCURRENT if PostgreSQL)

## Documentation:
- Comment explaining changes
- Impact assessment
- Rollback strategy
- Data migration notes if applicable

## Verification:
1. Dry-run on development database
2. Check for locking issues
3. Estimate execution time
4. List affected tables/indexes
```

### Use Case 5: Feature Flag Management

**Feature Flag Creation** (`.claude/commands/feature/new-flag.md`):
```markdown
---
description: Create feature flag with full integration
argument-hint: [flag-name] [description]
---

Create feature flag: $1

Description: ${@:2}

## Tasks:
1. **Add to feature flag config:**
   - Generate unique flag key
   - Set default state (off)
   - Add description
   - Set targeting rules

2. **Code integration:**
   - Add flag check in relevant code
   - Implement both code paths
   - Add logging for flag evaluation

3. **Testing:**
   - Test with flag on
   - Test with flag off
   - Test flag toggle

4. **Documentation:**
   - Update feature flag registry
   - Add to deployment docs
   - Note cleanup date (30-90 days)

5. **Monitoring:**
   - Add metrics for flag evaluation
   - Set up alerts if needed
```

## Conclusion

Custom slash commands transform Claude Code from a conversational AI assistant into a personalized development automation platform. By codifying your workflows as reusable commands, you create a shared language between your team and Claude, ensuring consistent, high-quality execution of common tasks.

**Key Takeaways:**

1. **Start simple** - Begin with frequently-typed prompts
2. **Iterate based on usage** - Refine commands as you learn what works
3. **Share with your team** - Project commands create consistency
4. **Organise thoughtfully** - Use namespaces and clear naming
5. **Document well** - Include argument hints and descriptions
6. **Build safely** - Use tool restrictions and verification steps
7. **Think reusably** - Design commands for repeated use
8. **Leverage arguments** - Make commands flexible with `$ARGUMENTS`

The most successful slash command implementations combine:
- **Clear purpose** - Each command does one thing well
- **Proper scope** - Project vs. user-level appropriately chosen
- **Good documentation** - Both in frontmatter and command body
- **Error handling** - Graceful failures and recovery steps
- **Team alignment** - Shared understanding of workflows

As you build your command library, you'll develop a powerful toolkit that captures your team's best practices, streamlines onboarding, and accelerates development velocity.

---

## Sources

- [Slash commands - Claude Code Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Slash Commands in the SDK - Claude Docs](https://platform.claude.com/docs/en/agent-sdk/slash-commands)
- [GitHub - wshobson/commands: A collection of production-ready slash commands](https://github.com/wshobson/commands)
- [GitHub - qdhenry/Claude-Command-Suite](https://github.com/qdhenry/Claude-Command-Suite)
- [GitHub - hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- [Claude Code Tips & Tricks: Custom Slash Commands](https://cloudartisan.com/posts/2025-04-14-claude-code-tips-slash-commands/)
- [How to Create Custom Slash Commands in Claude Code - BioErrorLog](https://en.bioerrorlog.work/entry/claude-code-custom-slash-command)
- [Claude Code Developer Cheatsheet](https://awesomeclaude.ai/code-cheatsheet)
- [How I use Claude Code (+ my best tips)](https://www.builder.io/blog/claude-code)
