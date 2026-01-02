# CI/CD Integration with Claude Code: A Comprehensive Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Print Mode for Scripting](#print-mode-for-scripting)
3. [Non-Interactive Mode for Pipelines](#non-interactive-mode-for-pipelines)
4. [Environment Variable Configuration](#environment-variable-configuration)
5. [GitHub Actions Integration](#github-actions-integration)
6. [Other CI/CD Platform Integration](#other-cicd-platform-integration)
7. [Authentication in CI](#authentication-in-ci)
8. [Best Practices for Automated Claude Usage](#best-practices-for-automated-claude-usage)
9. [Error Handling and Logging](#error-handling-and-logging)
10. [Conclusion](#conclusion)

---

## Introduction

Claude Code enables powerful CI/CD automation through its Agent SDK, allowing developers to integrate AI-powered code analysis, review, and implementation directly into development workflows. The headless/programmatic mode (activated with the `-p` flag) transforms Claude Code from an interactive tool into a scriptable automation engine suitable for continuous integration pipelines, pre-commit hooks, and custom development workflows.

This guide covers everything you need to know about integrating Claude Code into your CI/CD pipelines, from basic scripting to enterprise-grade automation across multiple platforms.

---

## Print Mode for Scripting

### Overview

The `-p` (or `--print`) flag enables non-interactive execution of Claude Code, formerly known as "headless mode." This allows Claude to run programmatically from command-line scripts and automation tools without any interactive UI.

### Basic Syntax

```bash
claude -p "Your prompt here" [options]
```

### Key Use Cases

Print mode is ideal for:
- Automation scripts and CI/CD pipelines
- SRE bots and automated code reviews
- Pre-commit hooks and build scripts
- Agent integrations and custom workflows

### Essential Flags and Options

| Flag | Purpose |
|------|---------|
| `-p` / `--print` | Enables non-interactive execution |
| `--continue` | Resumes most recent conversation |
| `--resume [ID]` | Continues specific session by ID |
| `--allowedTools` | Pre-approves tools without prompting |
| `--output-format` | Controls response formatting (text, json, stream-json) |
| `--append-system-prompt` | Adds custom instructions |
| `--json-schema` | Defines structured output schema |
| `--max-turns` | Limits conversation iterations |
| `--model` | Specifies which Claude model to use |

### Practical Examples

#### Basic Code Review

```bash
claude -p "Review the changes in this PR for security issues" \
  --allowedTools "Bash,Read" \
  --output-format json
```

#### Automated Test Fixing

```bash
claude -p "Run tests and fix any failures" \
  --allowedTools "Bash,Read,Edit" \
  --permission-mode acceptEdits \
  --max-turns 10
```

#### Multi-Turn Conversations

```bash
# Initial analysis
claude -p "Analyse performance issues in the codebase"

# Follow-up with continuation
claude -p "Now focus specifically on database queries" --continue
```

#### Structured JSON Output

```bash
result=$(claude -p "Extract all function signatures from src/" --output-format json)
code=$(echo "$result" | jq -r '.result')
cost=$(echo "$result" | jq -r '.cost_usd')
echo "Analysis complete. Cost: $$cost"
```

### Output Formats

**Text (default)**: Plain text responses suitable for logging and display.

```bash
claude -p "Summarise recent changes"
```

**JSON**: Structured output with session metadata, perfect for programmatic parsing.

```bash
claude -p "List security vulnerabilities" --output-format json
```

**Stream JSON**: Newline-delimited JSON for real-time streaming and progress tracking.

```bash
claude -p "Refactor authentication module" --output-format stream-json
```

### Important Limitations

- Slash commands (like `/commit`, `/review`) are only available in interactive mode
- In `-p` mode, describe the task you want to accomplish instead of using slash commands
- Headless mode does not persist state between sessions unless you use `--continue` or `--resume`

---

## Non-Interactive Mode for Pipelines

### Understanding Non-Interactive Execution

Non-interactive mode is critical for CI/CD pipelines where no human is present to approve actions. This requires careful configuration to ensure Claude has the right permissions without compromising security.

### Permission Modes

Claude Code offers several permission modes for automation:

| Mode | Description | Use Case |
|------|-------------|----------|
| `default` | Allows reads, asks before modifications | Not suitable for CI |
| `plan` | Analyse only, no modifications | Safe for analysis jobs |
| `acceptEdits` | Bypasses prompts for file edits | Code review and fixes |
| `bypassPermissions` | No permission prompts (dangerous) | Sandboxed environments only |

### Configuring Non-Interactive Runs

#### Using Command-Line Flags

```bash
claude -p "Fix TypeScript errors in src/" \
  --allowedTools "Bash,Read,Edit" \
  --permission-mode acceptEdits \
  --max-turns 5
```

#### Using Settings Files

Create `.claude/settings.json` in your repository:

```json
{
  "defaultMode": "acceptEdits",
  "allowedTools": [
    {
      "tool": "Bash",
      "allow": [
        "npm test",
        "npm run build",
        "git diff*",
        "git status*"
      ]
    },
    {
      "tool": "Read",
      "allow": ["**/*.ts", "**/*.js", "**/*.json"]
    },
    {
      "tool": "Edit",
      "allow": ["src/**/*", "tests/**/*"]
    }
  ],
  "deniedTools": [
    {
      "tool": "Bash",
      "deny": ["rm -rf*", "sudo*", "curl*"]
    }
  ]
}
```

### Tool Auto-Approval

The `--allowedTools` flag pre-approves specific tools, eliminating approval prompts:

```bash
# Allow specific tools
claude -p "Generate unit tests" --allowedTools "Read,Edit,Bash"

# Allow with granular permissions
claude -p "Review changes" --allowedTools "Bash(git diff:*),Bash(git status:*)"
```

### Session Management

For complex workflows requiring multiple steps, use session IDs:

```bash
# Initial run - capture session ID
result=$(claude -p "Start implementing feature X" --output-format json)
session_id=$(echo "$result" | jq -r '.session_id')

# Continue with specific session
claude -p "Add error handling" --resume "$session_id"
claude -p "Write tests" --resume "$session_id"
```

---

## Environment Variable Configuration

### Authentication Variables

Claude Code prioritises environment variable API keys over authenticated subscriptions, making them ideal for CI/CD environments.

#### Primary Authentication

```bash
# Anthropic API (recommended for CI)
export ANTHROPIC_API_KEY="your-api-key-here"

# Alternative: Add to shell profile
echo 'export ANTHROPIC_API_KEY="your-api-key"' >> ~/.bashrc
```

#### Cloud Provider Authentication

**AWS Bedrock:**

```bash
export CLAUDE_CODE_USE_BEDROCK="1"
export AWS_REGION="us-east-1"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"

# Optional: Custom endpoint
export ANTHROPIC_BEDROCK_BASE_URL="https://bedrock-runtime.us-east-1.amazonaws.com"
```

**Google Vertex AI:**

```bash
export CLAUDE_CODE_USE_VERTEX="1"
export CLOUD_ML_REGION="us-central1"
export GCP_PROJECT_ID="your-project-id"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```

### Configuration Variables

#### Performance and Timeout Settings

```bash
# Disable non-essential network traffic (recommended for CI)
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=true

# Equivalent to setting individual disable flags:
# DISABLE_AUTOUPDATER, DISABLE_BUG_COMMAND,
# DISABLE_ERROR_REPORTING, DISABLE_TELEMETRY

# Configure timeouts
export BASH_DEFAULT_TIMEOUT_MS=30000  # 30 seconds
export MCP_TIMEOUT=60000              # 60 seconds
export CLAUDE_CODE_API_KEY_HELPER_TTL_MS=300000  # 5 minutes
```

#### Model Selection

```bash
# Specify Claude model
export CLAUDE_CODE_MODEL="claude-sonnet-4-5-20250929"

# Or use command-line flag
claude -p "Your prompt" --model "claude-opus-4-5-20251101"
```

### Verifying Configuration

Check which authentication method is active:

```bash
claude /status
```

This displays:
- Active API key source (environment variable vs. authenticated session)
- Current model and provider
- Active settings and permissions

### Security Best Practices

**Never commit API keys to version control:**

```bash
# .gitignore
.env
.env.local
.claude/secrets.json
```

**Use CI platform secrets management:**

```bash
# GitHub Actions - reference secrets
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# GitLab CI - use masked variables
# Set in Settings → CI/CD → Variables
# Mark as "Masked" and "Protected"
```

---

## GitHub Actions Integration

### Overview

Claude Code GitHub Actions enables AI-powered automation in your development workflow through the official `anthropics/claude-code-action`. By mentioning `@claude` in pull requests or issues, Claude can analyse code, create PRs, implement features, and fix bugs.

### Quick Setup

#### Method 1: Using Claude Code CLI

```bash
# In Claude Code terminal
/install-github-app
```

This guides you through:
1. Installing the Claude GitHub app (requires admin access)
2. Adding `ANTHROPIC_API_KEY` to repository secrets
3. Creating the workflow file

#### Method 2: Manual Setup

**Step 1**: Install the Claude GitHub app at https://github.com/apps/claude

Required permissions:
- Contents (read/write)
- Issues (read/write)
- Pull requests (read/write)

**Step 2**: Add secrets to your repository

Navigate to `Settings → Secrets and variables → Actions`:

```
Name: ANTHROPIC_API_KEY
Value: sk-ant-api03-...
```

**Step 3**: Create workflow file

`.github/workflows/claude.yml`:

```yaml
name: Claude Code Assistant

on:
  pull_request:
    types: [opened, synchronise]
  issue_comment:
    types: [created]
  issues:
    types: [opened, assigned]

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Optional: Add custom configuration
          claude_args: "--max-turns 10 --model claude-sonnet-4-5-20250929"
```

### Common Workflow Patterns

#### Automatic PR Review

```yaml
name: Automated Code Review

on:
  pull_request:
    types: [opened, synchronise]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review this pull request for:
            - Security vulnerabilities
            - Performance issues
            - Code style violations
            - Missing error handling
            - Incomplete test coverage

            Provide specific, actionable feedback.
          claude_args: "--max-turns 5"
```

#### Path-Specific Reviews

```yaml
name: Security Review for Critical Paths

on:
  pull_request:
    paths:
      - 'src/auth/**'
      - 'src/payment/**'
      - 'src/api/admin/**'

jobs:
  security-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Perform a security-focused review with emphasis on:
            - Authentication and authorisation
            - Input validation and sanitization
            - SQL injection and XSS vulnerabilities
            - Sensitive data exposure
            - OWASP Top 10 compliance
```

#### Issue-to-PR Automation

```yaml
name: Implement Features from Issues

on:
  issues:
    types: [labelled]

jobs:
  implement:
    if: contains(github.event.issue.labels.*.name, 'claude-implement')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Implement the feature described in this issue.
            Follow the project's coding standards defined in CLAUDE.md.
            Create a new branch and open a pull request with:
            - Complete implementation
            - Unit tests
            - Updated documentation
```

#### Documentation Sync

```yaml
name: Daily Documentation Update

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
  workflow_dispatch:     # Allow manual triggers

jobs:
  update-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for accurate diff

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review all commits from the last 24 hours.
            Update documentation to reflect:
            - New features and APIs
            - Changed behaviour
            - Deprecated functionality

            Create or update a PR titled "docs: Daily documentation sync"
```

#### External Contributor Reviews

```yaml
name: Review External Contributions

on:
  pull_request_target:  # Use with caution - runs in base context
    types: [opened]

jobs:
  review-external:
    if: github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review this first-time contribution with a welcoming tone.
            Check for:
            - Contribution guidelines compliance
            - Code quality and style
            - Adequate testing
            - Security concerns

            Provide constructive, encouraging feedback.
```

### Configuration Options (v1.0+)

The GA version (v1.0) introduced simplified configuration:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    # Required
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

    # Optional: Direct prompt (auto-detects mode)
    prompt: "Your instructions here"

    # Optional: Pass flags to Claude Code CLI
    claude_args: |
      --max-turns 5
      --model claude-sonnet-4-5-20250929
      --system-prompt "Follow our coding standards in CLAUDE.md"
      --output-format json

    # Optional: Cloud provider configuration
    # (AWS Bedrock example)
    aws_region: us-east-1
    aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Using CLAUDE.md for Project Context

Create `CLAUDE.md` in your repository root to define project-specific guidelines:

```markdown
# Project Guidelines for Claude

## Coding Standards

- Use TypeScript with strict mode enabled
- Follow ESLint rules defined in .eslintrc.json
- Write tests for all new features (minimum 80% coverage)
- Use conventional commits format

## Review Criteria

When reviewing code, prioritise:
1. Security vulnerabilities
2. Performance implications
3. Test coverage
4. Documentation completeness

## Architecture Patterns

- Use dependency injection for services
- Implement repository pattern for data access
- Follow SOLID principles
- Prefer composition over inheritance

## Specific Rules

- Never commit sensitive data or API keys
- All API endpoints must have rate limiting
- Database queries must use parameterized statements
- Error messages must not expose internal details
```

### Interactive Usage

Users can interact with Claude in PR comments:

```
@claude implement the user authentication feature described in this issue

@claude how should I handle error cases in the payment processing flow?

@claude fix the TypeError in the UserDashboard component

@claude review the performance implications of these database changes
```

---

## Other CI/CD Platform Integration

### GitLab CI/CD

Claude Code integrates with GitLab through the official GitLab-maintained integration (currently in beta).

#### Quick Setup

**Step 1**: Add API key as CI/CD variable

Navigate to `Settings → CI/CD → Variables`:

```
Key: ANTHROPIC_API_KEY
Value: sk-ant-api03-...
Flags: [x] Masked  [x] Protected
```

**Step 2**: Create `.gitlab-ci.yml`

```yaml
claude-review:
  image: node:20-alpine
  stage: test
  before_script:
    - npm install -g @anthropic-ai/claude-code
  script:
    - |
      claude -p "Review this merge request for code quality issues" \
        --allowedTools "Bash,Read" \
        --output-format json > review.json
    - cat review.json
  artifacts:
    reports:
      codequality: review.json
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

#### Provider Options

**Claude API (SaaS)**

```yaml
variables:
  ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY

claude-job:
  script:
    - claude -p "Your prompt" --output-format json
```

**AWS Bedrock (OIDC)**

```yaml
variables:
  AWS_ROLE_TO_ASSUME: "arn:aws:iam::123456789012:role/GitLabClaude"
  AWS_REGION: "us-east-1"

claude-job:
  id_tokens:
    AWS_ID_TOKEN:
      aud: https://gitlab.com
  before_script:
    - export AWS_WEB_IDENTITY_TOKEN_FILE=$AWS_ID_TOKEN
    - export CLAUDE_CODE_USE_BEDROCK="1"
  script:
    - claude -p "Your prompt"
```

**Google Vertex AI (Workload Identity Federation)**

```yaml
variables:
  GCP_WORKLOAD_IDENTITY_PROVIDER: "projects/123/locations/global/workloadIdentityPools/gitlab/providers/gitlab"
  GCP_SERVICE_ACCOUNT: "claude-ci@project.iam.gserviceaccount.com"
  CLOUD_ML_REGION: "us-central1"

claude-job:
  id_tokens:
    GCP_ID_TOKEN:
      aud: https://gitlab.com
  before_script:
    - export GOOGLE_APPLICATION_CREDENTIALS=$GCP_ID_TOKEN
    - export CLAUDE_CODE_USE_VERTEX="1"
  script:
    - claude -p "Your prompt"
```

#### Event-Driven Workflows

```yaml
# Trigger on @claude mentions
claude-on-demand:
  rules:
    - if: '$CI_MERGE_REQUEST_DESCRIPTION =~ /@claude/'
      when: manual
    - if: '$CI_COMMIT_MESSAGE =~ /@claude/'
      when: manual
  script:
    - |
      claude -p "Analyse the changes in this MR" \
        --allowedTools "Bash,Read" \
        --permission-mode plan
```

### Jenkins

```groovy
pipeline {
    agent any

    environment {
        ANTHROPIC_API_KEY = credentials('anthropic-api-key')
        CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC = 'true'
    }

    stages {
        stage('Code Review') {
            steps {
                script {
                    def result = sh(
                        script: '''
                            claude -p "Review the changes in this commit" \
                              --allowedTools "Bash,Read" \
                              --output-format json
                        ''',
                        returnStdout: true
                    ).trim()

                    def review = readJSON text: result
                    echo "Review complete: ${review.result}"
                }
            }
        }

        stage('Fix Issues') {
            when {
                expression {
                    return currentBuild.result == null || currentBuild.result == 'UNSTABLE'
                }
            }
            steps {
                sh '''
                    claude -p "Fix failing tests and linting errors" \
                      --allowedTools "Bash,Read,Edit" \
                      --permission-mode acceptEdits \
                      --max-turns 10
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/claude-*.log', allowEmptyArchive: true
        }
    }
}
```

### CircleCI

```yaml
version: 2.1

executors:
  claude-executor:
    docker:
      - image: cimg/node:20.0
    environment:
      CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC: "true"

jobs:
  code-review:
    executor: claude-executor
    steps:
      - checkout

      - run:
          name: Install Claude Code
          command: npm install -g @anthropic-ai/claude-code

      - run:
          name: Run Review
          command: |
            claude -p "Review this pull request" \
              --allowedTools "Bash,Read" \
              --output-format json | tee review.json

      - store_artifacts:
          path: review.json

workflows:
  version: 2
  review:
    jobs:
      - code-review:
          context: anthropic-credentials
          filters:
            branches:
              ignore: main
```

### Azure Pipelines

```yaml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: anthropic-credentials  # Variable group with ANTHROPIC_API_KEY

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node.js'

  - script: |
      npm install -g @anthropic-ai/claude-code
    displayName: 'Install Claude Code'

  - script: |
      claude -p "Review this build for issues" \
        --allowedTools "Bash,Read" \
        --output-format json > $(Build.ArtifactStagingDirectory)/review.json
    displayName: 'Run Claude Review'
    env:
      ANTHROPIC_API_KEY: $(ANTHROPIC_API_KEY)
      CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC: true

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: '$(Build.ArtifactStagingDirectory)'
      artifactName: 'claude-review'
```

### Bitbucket Pipelines

```yaml
pipelines:
  pull-requests:
    '**':
      - step:
          name: Claude Review
          image: node:20-alpine
          caches:
            - node
          script:
            - npm install -g @anthropic-ai/claude-code
            - |
              claude -p "Review this pull request" \
                --allowedTools "Bash,Read" \
                --output-format json | tee review.json
          artifacts:
            - review.json
          services:
            - docker
```

---

## Authentication in CI

### API Key Management

#### Best Practices

1. **Never hardcode API keys** - Always use secret management
2. **Use dedicated CI API keys** - Separate from development keys for tracking and rotation
3. **Implement key rotation** - Regularly rotate API keys
4. **Monitor usage** - Track API consumption per pipeline

#### Setting Up API Keys

**GitHub Actions:**

```yaml
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

**GitLab CI:**

```yaml
variables:
  ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY  # Set in CI/CD settings
```

**Jenkins:**

```groovy
environment {
    ANTHROPIC_API_KEY = credentials('anthropic-api-key')
}
```

### Cloud Provider Authentication

#### AWS Bedrock with OIDC (Recommended)

Eliminates static credentials by using OpenID Connect:

**GitHub Actions:**

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  claude-job:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsClaude
          aws-region: us-east-1

      - name: Run Claude with Bedrock
        env:
          CLAUDE_CODE_USE_BEDROCK: "1"
          AWS_REGION: us-east-1
        run: |
          claude -p "Review code" --output-format json
```

**GitLab CI:**

```yaml
claude-bedrock:
  id_tokens:
    AWS_ID_TOKEN:
      aud: https://gitlab.com
  variables:
    AWS_ROLE_TO_ASSUME: "arn:aws:iam::123456789012:role/GitLabClaude"
    AWS_REGION: "us-east-1"
    CLAUDE_CODE_USE_BEDROCK: "1"
  before_script:
    - export AWS_WEB_IDENTITY_TOKEN_FILE=$AWS_ID_TOKEN
  script:
    - claude -p "Your prompt"
```

#### Google Vertex AI with Workload Identity Federation

**GitLab CI:**

```yaml
claude-vertex:
  id_tokens:
    GCP_ID_TOKEN:
      aud: https://gitlab.com
  variables:
    GCP_WORKLOAD_IDENTITY_PROVIDER: "projects/123/locations/global/workloadIdentityPools/gitlab/providers/gitlab"
    GCP_SERVICE_ACCOUNT: "claude-ci@project.iam.gserviceaccount.com"
    CLOUD_ML_REGION: "us-central1"
    CLAUDE_CODE_USE_VERTEX: "1"
  before_script:
    - export GOOGLE_APPLICATION_CREDENTIALS=$GCP_ID_TOKEN
  script:
    - claude -p "Your prompt"
```

**GitHub Actions:**

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/123/locations/global/workloadIdentityPools/github/providers/github'
    service_account: 'claude-ci@project.iam.gserviceaccount.com'

- name: Run Claude with Vertex AI
  env:
    CLAUDE_CODE_USE_VERTEX: "1"
    CLOUD_ML_REGION: us-central1
  run: |
    claude -p "Review code" --output-format json
```

### Security Considerations

#### Principle of Least Privilege

```json
// .claude/settings.json for CI
{
  "defaultMode": "plan",
  "allowedTools": [
    {
      "tool": "Bash",
      "allow": [
        "npm test",
        "npm run build",
        "npm run lint",
        "git diff*",
        "git status*",
        "git log*"
      ],
      "deny": [
        "rm *",
        "sudo *",
        "curl *",
        "wget *",
        "ssh *",
        "scp *"
      ]
    },
    {
      "tool": "Read",
      "allow": ["**/*"],
      "deny": [".env*", "**/secrets/**", "**/*.key", "**/*.pem"]
    }
  ],
  "deniedTools": [
    "Write",  // Prevent file modifications in review-only jobs
    "WebFetch",
    "WebSearch"
  ]
}
```

#### Sandboxing and Isolation

**Docker-based Isolation:**

```yaml
# GitHub Actions
jobs:
  claude-review:
    runs-on: ubuntu-latest
    container:
      image: node:20-alpine
      options: --user 1001:1001  # Non-root user
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g @anthropic-ai/claude-code
      - run: |
          claude -p "Review code" \
            --allowedTools "Read" \
            --permission-mode plan
```

**Read-Only Filesystem:**

```yaml
container:
  image: node:20-alpine
  options: --read-only --tmpfs /tmp
```

---

## Best Practices for Automated Claude Usage

### 1. Define Clear Prompts

**Bad:**
```bash
claude -p "check the code"
```

**Good:**
```bash
claude -p "Review the changes in this pull request for:
1. Security vulnerabilities (SQL injection, XSS, CSRF)
2. Performance issues (N+1 queries, inefficient algorithms)
3. Code style violations per our ESLint rules
4. Missing error handling
5. Inadequate test coverage

Provide specific line numbers and actionable recommendations."
```

### 2. Limit Conversation Turns

Prevent runaway costs and execution time:

```bash
claude -p "Fix failing tests" \
  --max-turns 5 \
  --allowedTools "Bash,Read,Edit"
```

### 3. Use Project Context Files

Create `.claude/commands/` directory for reusable workflows:

**`.claude/commands/security-review.md`:**
```markdown
Review this code for security issues:

1. Authentication and authorisation vulnerabilities
2. Input validation and sanitization
3. SQL injection and XSS risks
4. Sensitive data exposure
5. Cryptographic weaknesses
6. OWASP Top 10 compliance

For each issue found:
- Specify the file and line number
- Explain the vulnerability
- Provide a secure code example
- Rate severity (Critical/High/Medium/Low)
```

Use in CI:

```bash
claude -p "$(cat .claude/commands/security-review.md)" \
  --allowedTools "Read" \
  --output-format json
```

### 4. Implement Quality Gates

```yaml
# GitHub Actions
- name: Run Claude Review
  id: review
  run: |
    result=$(claude -p "Review this PR" --output-format json)
    echo "$result" > review.json

    # Extract severity counts
    critical=$(echo "$result" | jq '[.issues[] | select(.severity=="critical")] | length')
    high=$(echo "$result" | jq '[.issues[] | select(.severity=="high")] | length')

    echo "critical=$critical" >> $GITHUB_OUTPUT
    echo "high=$high" >> $GITHUB_OUTPUT

- name: Check Quality Gate
  if: steps.review.outputs.critical > 0 || steps.review.outputs.high > 3
  run: |
    echo "Quality gate failed: ${{ steps.review.outputs.critical }} critical, ${{ steps.review.outputs.high }} high severity issues"
    exit 1
```

### 5. Optimise for Cost and Performance

**Target Metrics:**
- Review job latency: ≤ 3-5 minutes
- Token usage: Monitor and set budgets
- Success rate: ≥ 95%

**Strategies:**

```yaml
# Use faster models for simple tasks
- name: Quick Lint Check
  run: |
    claude -p "Check for style violations" \
      --model claude-sonnet-4-5-20250929 \
      --max-turns 1

# Use more capable models for complex analysis
- name: Architecture Review
  run: |
    claude -p "Review system architecture" \
      --model claude-opus-4-5-20251101 \
      --max-turns 10
```

### 6. Implement Caching Strategies

```yaml
# Cache Claude Code installation
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-claude-${{ hashFiles('**/package-lock.json') }}
```

### 7. Use Custom System Prompts

```bash
claude -p "Review this PR" \
  --append-system-prompt "You are reviewing code for a financial services application. Security and compliance are paramount. Follow PCI-DSS and SOC 2 requirements."
```

### 8. Validate Outputs

```bash
# Run Claude review
result=$(claude -p "Generate unit tests" --output-format json)

# Validate tests actually pass
if echo "$result" | jq -e '.result' > /dev/null; then
    npm test
    if [ $? -eq 0 ]; then
        echo "Generated tests pass"
        git add tests/
    else
        echo "Generated tests fail - discarding"
        git restore tests/
        exit 1
    fi
fi
```

### 9. Implement Retry Logic

```bash
#!/bin/bash
max_retries=3
retry_count=0

while [ $retry_count -lt $max_retries ]; do
    result=$(claude -p "Fix tests" --output-format json 2>&1)
    exit_code=$?

    if [ $exit_code -eq 0 ]; then
        echo "Success!"
        echo "$result"
        exit 0
    fi

    retry_count=$((retry_count + 1))
    echo "Attempt $retry_count failed, retrying..."
    sleep 5
done

echo "Failed after $max_retries attempts"
exit 1
```

### 10. Monitor and Alert

```yaml
- name: Claude Review with Monitoring
  run: |
    start_time=$(date +%s)

    result=$(claude -p "Review code" --output-format json)
    exit_code=$?

    end_time=$(date +%s)
    duration=$((end_time - start_time))

    # Extract token usage
    tokens=$(echo "$result" | jq -r '.usage.total_tokens // 0')
    cost=$(echo "$result" | jq -r '.cost_usd // 0')

    # Log metrics
    echo "Duration: ${duration}s"
    echo "Tokens: $tokens"
    echo "Cost: \$$cost"

    # Alert if thresholds exceeded
    if [ $duration -gt 300 ]; then
        echo "::warning::Claude review took longer than 5 minutes"
    fi

    if (( $(echo "$cost > 1.0" | bc -l) )); then
        echo "::warning::Claude review cost exceeded \$1.00"
    fi
```

---

## Error Handling and Logging

### Enable Verbose Logging

For debugging Claude invocations:

```bash
claude -p "Your prompt" --verbose
```

**Note:** Disable verbose mode in production for cleaner output.

### Structured Error Handling

```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Function to handle errors
handle_error() {
    local exit_code=$1
    local line_number=$2
    echo "Error on line $line_number (exit code: $exit_code)" >&2

    # Log to file
    echo "[$(date -u +"%Y-%m-%dT%H:%M:%SZ")] Error on line $line_number" >> claude-errors.log

    # Send notification (example with Slack)
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"Claude CI job failed: Line $line_number (exit $exit_code)\"}" \
        "$SLACK_WEBHOOK_URL"

    exit $exit_code
}

trap 'handle_error $? $LINENO' ERR

# Run Claude with error capture
result=$(claude -p "Review code" --output-format json 2>&1) || {
    echo "Claude execution failed"
    echo "$result" | tee -a claude-errors.log
    exit 1
}

# Validate JSON output
if ! echo "$result" | jq empty 2>/dev/null; then
    echo "Invalid JSON output from Claude" >&2
    echo "$result" >> claude-errors.log
    exit 1
fi

echo "Success: $result"
```

### Logging Best Practices

#### Structured Logging

```bash
#!/bin/bash

# Logging function
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

    echo "{\"timestamp\":\"$timestamp\",\"level\":\"$level\",\"message\":\"$message\"}" \
        | tee -a claude-ci.log
}

log "INFO" "Starting Claude code review"

result=$(claude -p "Review PR" --output-format json 2>&1) || {
    log "ERROR" "Claude execution failed: $result"
    exit 1
}

log "INFO" "Review complete"

# Extract and log metrics
tokens=$(echo "$result" | jq -r '.usage.total_tokens // 0')
cost=$(echo "$result" | jq -r '.cost_usd // 0')

log "METRICS" "Tokens used: $tokens, Cost: \$$cost"
```

#### GitHub Actions Annotations

```yaml
- name: Claude Review
  run: |
    result=$(claude -p "Review code" --output-format json)

    # Parse issues and create annotations
    echo "$result" | jq -r '.issues[] |
      "::warning file=\(.file),line=\(.line)::\(.message)"'

    # Error for critical issues
    critical=$(echo "$result" | jq '[.issues[] | select(.severity=="critical")] | length')
    if [ "$critical" -gt 0 ]; then
      echo "::error::Found $critical critical issues"
      exit 1
    fi
```

### Handling API Rate Limits

```bash
#!/bin/bash

call_claude_with_backoff() {
    local prompt=$1
    local max_attempts=5
    local attempt=1
    local wait_time=1

    while [ $attempt -le $max_attempts ]; do
        log "INFO" "Attempt $attempt of $max_attempts"

        result=$(claude -p "$prompt" --output-format json 2>&1)
        exit_code=$?

        # Check for rate limit error
        if echo "$result" | grep -q "rate_limit_error"; then
            log "WARN" "Rate limited, waiting ${wait_time}s before retry"
            sleep $wait_time
            wait_time=$((wait_time * 2))  # Exponential backoff
            attempt=$((attempt + 1))
        elif [ $exit_code -eq 0 ]; then
            echo "$result"
            return 0
        else
            log "ERROR" "Unexpected error: $result"
            return $exit_code
        fi
    done

    log "ERROR" "Max retries exceeded"
    return 1
}

# Use the function
result=$(call_claude_with_backoff "Review this code")
```

### Timeout Handling

```bash
# Set timeout for Claude execution
timeout 300s claude -p "Review code" --output-format json > result.json || {
    exit_code=$?
    if [ $exit_code -eq 124 ]; then
        echo "::error::Claude review timed out after 5 minutes"
        # Fall back to simpler review
        timeout 60s claude -p "Quick syntax check only" --output-format json
    else
        echo "::error::Claude review failed with code $exit_code"
        exit $exit_code
    fi
}
```

### Continuous Monitoring

```yaml
# GitHub Actions - Send metrics to monitoring system
- name: Upload Metrics
  if: always()
  run: |
    # Extract metrics from Claude output
    tokens=$(jq -r '.usage.total_tokens // 0' result.json)
    cost=$(jq -r '.cost_usd // 0' result.json)
    duration=${{ steps.review.outputs.duration }}

    # Send to monitoring (example: Datadog)
    curl -X POST "https://api.datadoghq.com/api/v1/series" \
      -H "Content-Type: application/json" \
      -H "DD-API-KEY: ${{ secrets.DD_API_KEY }}" \
      -d @- <<EOF
    {
      "series": [
        {
          "metric": "claude.ci.tokens",
          "points": [[$(date +%s), $tokens]],
          "type": "count",
          "tags": ["repo:${{ github.repository }}", "branch:${{ github.ref_name }}"]
        },
        {
          "metric": "claude.ci.cost",
          "points": [[$(date +%s), $cost]],
          "type": "gauge",
          "tags": ["repo:${{ github.repository }}", "branch:${{ github.ref_name }}"]
        },
        {
          "metric": "claude.ci.duration",
          "points": [[$(date +%s), $duration]],
          "type": "gauge",
          "tags": ["repo:${{ github.repository }}", "branch:${{ github.ref_name }}"]
        }
      ]
    }
    EOF
```

### Handling Hook Errors

Use pre/post hooks for validation:

```json
// .claude/hooks/stop.js
// Runs after Claude finishes responding

const { readFileSync } = require('fs');
const { execSync } = require('child_process');

async function stopHook() {
  console.log('Running post-execution validation...');

  try {
    // Read edit logs to find modified files
    const editLog = JSON.parse(readFileSync('.claude/edit-log.json', 'utf8'));
    const modifiedFiles = editLog.files || [];

    // Run tests on modified files
    for (const file of modifiedFiles) {
      if (file.endsWith('.ts') || file.endsWith('.js')) {
        console.log(`Validating ${file}...`);

        try {
          // Run TypeScript check
          execSync(`npx tsc --noEmit ${file}`, { stdio: 'pipe' });
          console.log(`✓ ${file} passed type check`);
        } catch (error) {
          console.error(`✗ ${file} has type errors:`);
          console.error(error.stdout.toString());
          process.exit(1);
        }
      }
    }

    console.log('All validations passed!');
  } catch (error) {
    console.error('Post-execution validation failed:', error);
    process.exit(1);
  }
}

module.exports = stopHook;
```

---

## Conclusion

Integrating Claude Code into CI/CD pipelines transforms AI from a development tool into an automated quality assurance and implementation engine. By following the practices outlined in this guide, teams can:

- **Automate code reviews** with AI-powered analysis that catches issues traditional linters miss
- **Implement features** directly from issue descriptions with automated PR creation
- **Fix failing tests** and build errors without manual intervention
- **Maintain documentation** that stays synchronised with code changes
- **Enforce security standards** through automated vulnerability scanning
- **Reduce review burden** on senior developers while maintaining code quality

### Key Takeaways

1. **Use Print Mode (`-p`)** for all CI/CD integrations to enable non-interactive execution
2. **Configure permissions carefully** - use the principle of least privilege
3. **Leverage official integrations** (GitHub Actions, GitLab CI) for the smoothest experience
4. **Implement proper authentication** - prefer OIDC/WIF over static credentials
5. **Define clear prompts** and use `CLAUDE.md` for project-specific guidelines
6. **Monitor costs and performance** - set limits and track metrics
7. **Handle errors gracefully** with retries, timeouts, and structured logging
8. **Validate AI outputs** - always run tests and checks after AI modifications

### Getting Started Checklist

- [ ] Install Claude Code CLI or use official GitHub Action
- [ ] Set up API authentication (API key or cloud provider)
- [ ] Create `.claude/settings.json` with appropriate permissions for CI
- [ ] Write `CLAUDE.md` with project coding standards
- [ ] Implement basic workflow (e.g., PR review on push)
- [ ] Add error handling and logging
- [ ] Set up cost monitoring and alerting
- [ ] Test in non-production environment first
- [ ] Document the integration for your team
- [ ] Gradually expand automation based on results

### Additional Resources

- **Official Documentation**: https://code.claude.com/docs
- **GitHub Action Repository**: https://github.com/anthropics/claude-code-action
- **Community Examples**: https://github.com/hesreallyhim/awesome-claude-code
- **Best Practices Guide**: https://www.anthropic.com/engineering/claude-code-best-practices

By treating Claude Code as a powerful but controlled automation tool—with proper sandboxing, permission management, and monitoring—teams can safely harness AI capabilities to improve development velocity and code quality.

---

## Sources

- [Claude Code GitLab CI/CD Documentation](https://code.claude.com/docs/en/gitlab-ci-cd)
- [Streamlined CI/CD Pipelines Using Claude Code & GitHub Actions](https://medium.com/@itsmybestview/streamlined-ci-cd-pipelines-using-claude-code-github-actions-74be17e51499)
- [How to Integrate Claude Code with CI/CD: Full 2025 DevOps Guide](https://skywork.ai/blog/how-to-integrate-claude-code-ci-cd-guide-2025/)
- [GitHub - anthropics/claude-code-action](https://github.com/anthropics/claude-code-action)
- [Claude Code GitHub Actions Documentation](https://code.claude.com/docs/en/github-actions)
- [Claude Code Action Official - GitHub Marketplace](https://github.com/marketplace/actions/claude-code-action-official)
- [Run Claude Code Programmatically - Headless Mode Documentation](https://code.claude.com/docs/en/headless)
- [Shipyard Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Claude Code: Best Practices for Agentic Coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Managing API Key Environment Variables in Claude Code](https://support.claude.com/en/articles/12304248-managing-api-key-environment-variables-in-claude-code)
- [Claude Code Environment Variables: A Complete Reference Guide](https://medium.com/@dan.avila7/claude-code-environment-variables-a-complete-reference-guide-41229ef18120)
- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Understanding Claude Code Permissions and Security Settings](https://www.petefreitag.com/blog/claude-code-permissions/)
- [How to Set Claude Code Permission Mode](https://claudelog.com/faqs/how-to-set-claude-code-permission-mode/)
- [ClaudeLog - Documentation and Best Practices](https://claudelog.com/)
