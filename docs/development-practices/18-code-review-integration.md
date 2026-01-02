# Code Review Integration with Claude Code

## Overview

Claude Code is an agentic AI coding assistant that provides sophisticated code review capabilities far beyond traditional linting tools. With its large context window and advanced reasoning abilities, Claude can understand entire codebases, identify security vulnerabilities, detect logic errors, and ensure adherence to project-specific coding standards. This guide covers comprehensive integration strategies for automating code reviews in your development workflow.

## Table of Contents

1. [Configuring claude-code-review.yml for PR Reviews](#configuring-claude-code-reviewyml-for-pr-reviews)
2. [Customising Review Focus Areas](#customising-review-focus-areas)
3. [Automated Review in CI/CD Pipelines](#automated-review-in-cicd-pipelines)
4. [Review Comment Formatting](#review-comment-formatting)
5. [Severity Levels and Categorisation](#severity-levels-and-categorisation)
6. [Integration with GitHub and GitLab](#integration-with-github-and-gitlab)
7. [Custom Review Rules and Checks](#custom-review-rules-and-checks)
8. [Best Practices for AI-Assisted Code Review](#best-practices-for-ai-assisted-code-review)

---

## Configuring claude-code-review.yml for PR Reviews

### Quick Setup

The fastest way to set up Claude Code for automated PR reviews is through the terminal:

```bash
# Run this command in Claude Code terminal
/install-github-app
```

This command guides you through:
- Installing the GitHub App with appropriate permissions
- Configuring required secrets (ANTHROPIC_API_KEY)
- Creating the initial workflow configuration

### Basic Configuration File

When you set up the integration, Claude creates a `claude-code-review.yml` file in your `.github/workflows/` directory. Here's a recommended starting configuration:

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronise]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Please review this pull request and look for bugs and security issues.
            Only report on bugs and potential vulnerabilities you find.
            Be concise.
          claude_args: "--max-turns 5"
```

### Customising the Prompt

The default configuration can be verbose, commenting on minor issues. To focus on what matters most, customise the `prompt` field:

```yaml
# Example: Security-Focused Review
prompt: |
  Please review this pull request focusing on:
  1. Security vulnerabilities (SQL injection, XSS, authentication flaws)
  2. Critical bugs that could cause runtime errors
  3. Data validation issues

  Do NOT report:
  - Code style issues
  - Minor refactoring suggestions
  - Issues that linters will catch

  Be concise and provide specific line numbers.
```

```yaml
# Example: Architecture Review
prompt: |
  Review this PR for architectural consistency:
  - Does it follow our established patterns?
  - Are there any breaking changes to public APIs?
  - Is error handling consistent with our standards?
  - Are new dependencies necessary and appropriate?
```

### Conditional Triggers

You can configure when reviews run based on specific conditions:

```yaml
# Only review PRs that modify specific paths
on:
  pull_request:
    types: [opened, synchronise]
    paths:
      - 'src/**'
      - 'lib/**'
      - '!**/*.md'
      - '!**/*.txt'
```

```yaml
# Trigger on @claude mentions in comments
on:
  issue_comment:
    types: [created]

jobs:
  review:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    # ... rest of configuration
```

---

## Customising Review Focus Areas

### The CLAUDE.md File

The single most important configuration for customising Claude's behaviour is the `CLAUDE.md` file at your repository root. This file serves as Claude's "constitution" for understanding your project.

**Location Hierarchy:**
1. `~/.claude/CLAUDE.md` (home directory - applies to all projects)
2. `<project-root>/CLAUDE.md` (project-specific)
3. `<subdirectory>/CLAUDE.md` (directory-specific for monorepos)

### What to Include in CLAUDE.md

```markdown
# Project: MyApp

## Architecture
This is a microservices application using:
- Node.js with Express for API services
- PostgreSQL for persistent data
- Redis for caching and sessions
- React for the frontend

## Code Review Priorities
When reviewing code, focus on:
1. **Security**: SQL injection, XSS, authentication bypasses
2. **Data integrity**: Proper validation, transaction handling
3. **API consistency**: RESTful patterns, error responses
4. **Performance**: N+1 queries, unnecessary database calls

## Coding Standards
- Use async/await, not callbacks or .then()
- All database queries must use parameterized statements
- API endpoints must have rate limiting
- All user input must be validated with Joi schemas
- Functions should be under 50 lines; extract helpers if longer

## Testing Requirements
- Unit tests required for all business logic
- Integration tests for all API endpoints
- Minimum 80% code coverage
- Tests must pass before any PR is merged

## Common Pitfalls to Check
- Ensure all database connections are properly closed
- Check for proper error handling in async functions
- Verify CORS settings don't expose sensitive endpoints
- Confirm environment variables are used (not hardcoded values)
```

### Agent-Specific Customisation

The code review plugin uses four parallel agents that can be customised in `.claude/commands/code-review.md`:

```markdown
# Code Review Command

Review this pull request using the following agents:

## Agent 1: CLAUDE.md Compliance
Check if changes follow all guidelines in CLAUDE.md.
Confidence threshold: 80

## Agent 2: CLAUDE.md Compliance (Redundant)
Double-check compliance with CLAUDE.md guidelines.
Confidence threshold: 80

## Agent 3: Bug Detection
Look for obvious bugs in changed code only:
- Logic errors
- Null pointer exceptions
- Off-by-one errors
- Type mismatches
- Unhandled error cases
Confidence threshold: 85

## Agent 4: Context Analysis
Analyse git history and surrounding code:
- Does this change break existing functionality?
- Are there similar patterns elsewhere that should be updated?
- Does commit history reveal intent we should preserve?
Confidence threshold: 75

Filter out any issues with a score less than 80.
```

---

## Automated Review in CI/CD Pipelines

### GitHub Actions Integration

Claude Code integrates seamlessly with GitHub Actions to provide automated reviews on every pull request:

```yaml
name: Automated Claude Review
on:
  pull_request:
    types: [opened, synchronise, reopened]

jobs:
  security-review:
    name: Security Analysis
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Run a security-focused review using the /security-review command.
            Focus on HIGH-CONFIDENCE vulnerabilities only.
          claude_args: "--max-turns 3"

  code-quality-review:
    name: Code Quality Analysis
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review code quality and adherence to CLAUDE.md guidelines.
            Check for bugs, logic errors, and architectural consistency.
          claude_args: "--max-turns 5"
```

### GitLab CI/CD Integration

For GitLab users, Claude Code provides beta support through the GitLab CI/CD integration:

```yaml
# .gitlab-ci.yml
stages:
  - review

claude-mr-review:
  stage: review
  image: node:24-alpine
  variables:
    ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - npm install -g @anthropic-ai/claude-code
    - |
      claude --model=claude-opus-4.5 \
        --max-turns=5 \
        --tools="Bash(*) Read(*) Edit(*) Write(*) mcp__gitlab" \
        --prompt="Review this MR for bugs and security issues. Follow CLAUDE.md guidelines. Be concise."
```

### Multi-Provider Authentication

Claude Code supports multiple cloud providers for enterprise environments:

#### AWS Bedrock (OIDC)

```yaml
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - uses: anthropics/claude-code-action@v1
        with:
          use_bedrock: true
          prompt: "Review this PR for security issues"
```

#### Google Vertex AI (Workload Identity Federation)

```yaml
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    steps:
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

      - uses: anthropics/claude-code-action@v1
        with:
          use_vertex: true
          prompt: "Review this PR comprehensively"
```

### Pipeline Best Practices

1. **Parallel Reviews**: Run security and quality reviews in parallel jobs to save time
2. **Conditional Execution**: Use path filters to avoid reviewing documentation-only changes
3. **Cost Control**: Set `--max-turns` limits to prevent runaway token usage
4. **Fail-Safe**: Use `continue-on-error: true` so reviews don't block merges
5. **Caching**: Cache dependencies to speed up pipeline execution

---

## Review Comment Formatting

### Structured Output Format

Claude Code formats review comments with consistent structure for easy parsing and action:

```markdown
## Security Issue: SQL Injection Vulnerability

**File**: `src/api/users.js`
**Line**: 42
**Severity**: HIGH
**Category**: sql_injection
**Confidence**: 95/100

### Description
The user input from `req.query.username` is directly interpolated into the SQL query without parameterization, creating a SQL injection vulnerability.

### Exploit Scenario
An attacker could submit a malicious username like:
`admin' OR '1'='1' --`

This would bypass authentication and return all user records.

### Fix Recommendation
Use parameterized queries:
```javascript
const query = 'SELECT * FROM users WHERE username = $1';
const result = await db.query(query, [req.query.username]);
```

### References
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- CWE-89: SQL Injection
```

### Terminal Output vs. PR Comments

Claude Code provides two output modes:

**Terminal Mode** (default):
```bash
/code-review
```
- Outputs to terminal only
- Useful for local development
- No GitHub API calls

**Comment Mode**:
```bash
/code-review --comment
```
- Posts findings as PR comments
- Requires GitHub authentication
- Appears inline on specific lines
- Enables team discussion

### Customising Comment Format

You can customise the output format by editing `.claude/commands/code-review.md`:

```markdown
For each issue found, output in this format:

### Issue: [Brief Title]
- **File**: `path/to/file.js` (line X)
- **Type**: [bug|security|quality|performance]
- **Confidence**: X/100

**What's wrong**: [1-2 sentence description]

**How to fix**: [Specific actionable recommendation]

**Code example**:
```language
[corrected code snippet]
```
```

---

## Severity Levels and Categorisation

### Confidence-Based Scoring System

Claude Code uses a 0-100 confidence score to filter false positives:

| Score Range | Interpretation | Action |
|-------------|---------------|--------|
| 0-25 | Not confident, likely false positive | Auto-filtered |
| 26-50 | Somewhat confident, might be real | Review manually |
| 51-75 | Moderately confident, real but minor | Include in report |
| 76-90 | Highly confident, real and important | Priority review |
| 91-100 | Absolutely certain, definitely real | Critical fix required |

**Default Threshold**: 80 (only issues scoring 80+ are reported)

**Customising Threshold**:
```markdown
# In .claude/commands/code-review.md
Filter out any issues with a score less than 85.
```

### Security Severity Levels

For security reviews, Claude categorises vulnerabilities by severity:

| Severity | Description | Examples |
|----------|-------------|----------|
| **CRITICAL** | Immediate exploitation risk, data breach potential | SQL injection, RCE, authentication bypass |
| **HIGH** | Significant security risk, may lead to compromise | XSS, insecure deserialization, broken access control |
| **MEDIUM** | Moderate risk, requires specific conditions | Weak cryptography, insecure random generation |
| **LOW** | Minor security concern, limited impact | Information disclosure, verbose error messages |
| **INFO** | Security-relevant observation, not a vulnerability | Deprecated algorithms, missing security headers |

### Vulnerability Categories

Claude's `/security-review` command checks for these categories:

**Injection Attacks:**
- SQL injection
- Command injection
- LDAP injection
- XPath injection
- NoSQL injection
- XXE (XML External Entity)

**Authentication & Authorisation:**
- Broken authentication
- Privilege escalation
- Insecure direct object references
- Authorisation bypass logic
- Session management flaws

**Data Exposure:**
- Hardcoded secrets and credentials
- Sensitive data logging
- Information disclosure
- PII handling violations
- Insecure data storage

**Cryptographic Issues:**
- Weak cryptographic algorithms
- Improper key management
- Insecure random number generation
- Inadequate encryption

**Additional Patterns:**
- Cross-site scripting (XSS)
- CSRF vulnerabilities
- Path traversal
- Insecure data handling
- Dependency vulnerabilities

### What Gets Filtered Out

Claude intentionally excludes certain issue types to reduce noise:

- Pre-existing issues unrelated to the PR
- Code that appears buggy but functions correctly
- Pedantic nitpicks and style preferences
- Issues that automated linters will catch
- General quality concerns (unless specified in CLAUDE.md)
- Code with explicit lint ignore comments
- Theoretical race conditions without practical impact
- Low-severity web vulnerabilities (tabnabbing, XS-Leaks, etc.)

---

## Integration with GitHub and GitLab

### GitHub Integration Setup

#### Step 1: Install the GitHub App

1. Navigate to https://github.com/apps/claude
2. Click "Install" and select your repository
3. Grant these permissions:
   - **Contents**: Read & Write (or Read only if not committing)
   - **Issues**: Read & Write
   - **Pull Requests**: Read & Write

#### Step 2: Configure Secrets

Add your API key as a repository secret:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `ANTHROPIC_API_KEY`
4. Value: Your Anthropic API key

#### Step 3: Create Workflow File

Create `.github/workflows/claude.yml`:

```yaml
name: Claude Code Assistant
on:
  pull_request:
    types: [opened, synchronise]
  issue_comment:
    types: [created]

jobs:
  claude:
    # Only run on PR comments or PR events
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))

    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: write

    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            ${{ github.event_name == 'pull_request' &&
                'Review this PR for bugs and security issues. Be concise.' ||
                github.event.comment.body }}
          claude_args: "--max-turns 5"
```

### GitLab Integration Setup

#### Step 1: Add CI/CD Variables

1. Navigate to **Settings** → **CI/CD** → **Variables**
2. Add `ANTHROPIC_API_KEY` as a masked variable

#### Step 2: Configure .gitlab-ci.yml

```yaml
stages:
  - review

variables:
  CLAUDE_MODEL: "claude-opus-4.5"

.claude_base:
  image: node:24-alpine
  before_script:
    - npm install -g @anthropic-ai/claude-code
  variables:
    ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY

claude_mr_review:
  extends: .claude_base
  stage: review
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  script:
    - |
      claude --model=$CLAUDE_MODEL \
        --max-turns=5 \
        --tools="Bash(*) Read(*) Edit(*) Write(*) mcp__gitlab" \
        --prompt="Review this MR comprehensively. Follow CLAUDE.md. Report bugs and security issues."
  allow_failure: true

claude_security_review:
  extends: .claude_base
  stage: review
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      changes:
        - "**/*.js"
        - "**/*.ts"
        - "**/*.py"
        - "**/*.java"
  script:
    - |
      claude --model=$CLAUDE_MODEL \
        --max-turns=3 \
        --prompt="/security-review"
  allow_failure: true
```

### Interactive Features

Both platforms support interactive modes where team members can invoke Claude:

**GitHub**: Comment `@claude [instruction]` on any PR
```
@claude review the authentication logic for security issues
@claude explain why this test is failing
@claude refactor this function to be more readable
```

**GitLab**: Use MCP (Model Context Protocol) integration
```yaml
# Claude can create MRs, comment on issues, and implement fixes
script:
  - claude --prompt="Fix issue #123 and create an MR"
```

### Permissions and Security

**Least Privilege Principle:**

```yaml
# Minimum permissions for read-only reviews
permissions:
  contents: read
  pull-requests: write

# Add write access only if Claude should commit fixes
permissions:
  contents: write
  pull-requests: write
```

**Security Best Practices:**
- Always store API keys as encrypted secrets
- Use organisation-level secrets for team-wide access
- Rotate API keys periodically
- Enable branch protection rules
- Require human approval before merging Claude's suggestions
- Use CODEOWNERS to protect sensitive files
- Never skip hooks with `--no-verify`

---

## Custom Review Rules and Checks

### Creating Custom Slash Commands

Custom review commands live in `.claude/commands/` and become available as `/project:command-name`:

#### Example: Frontend-Specific Review

Create `.claude/commands/frontend-review.md`:

```markdown
# Frontend Code Review

Review this pull request with a focus on frontend best practices:

## Accessibility
- Are all interactive elements keyboard accessible?
- Do images have alt text?
- Is proper semantic HTML used?
- Are ARIA labels used correctly?
- Is colour contrast sufficient (WCAG AA)?

## Performance
- Are images optimised and lazy-loaded?
- Are there unnecessary re-renders in React components?
- Is code-splitting implemented appropriately?
- Are there memory leaks in useEffect hooks?

## User Experience
- Is loading state properly indicated?
- Are error messages user-friendly?
- Is form validation immediate and helpful?
- Are animations smooth (60fps)?

## Security
- Is user input sanitized before rendering?
- Are XSS vulnerabilities present?
- Is sensitive data exposed in client-side code?

Confidence threshold: 75
Output concise, actionable feedback with specific line numbers.
```

Usage: `/project:frontend-review`

#### Example: Database Migration Review

Create `.claude/commands/migration-review.md`:

```markdown
# Database Migration Review

Analyse this database migration for safety and correctness:

## Safety Checks
- Are destructive operations (DROP, DELETE) necessary and justified?
- Is there a rollback plan?
- Will this migration lock tables for an unacceptable duration?
- Are indexes created CONCURRENTLY (PostgreSQL)?
- Is there a risk of data loss?

## Correctness
- Are foreign key constraints properly defined?
- Are indexes created on appropriate columns?
- Are data types appropriate and efficient?
- Are default values sensible?
- Is NULL handling correct?

## Performance Impact
- Will this migration cause downtime?
- Are there better strategies (e.g., multi-phase migration)?
- Will indexes improve query performance?
- Are there N+1 query risks?

Confidence threshold: 85
Flag any HIGH or CRITICAL issues immediately.
```

Usage: `/project:migration-review`

### Hook-Based Enforcement

Hooks enforce deterministic rules that complement CLAUDE.md suggestions.

#### PreToolUse Hook for Test Enforcement

Create `.claude/hooks/PreToolUse.md`:

```markdown
# Pre-Tool-Use Hook

Before any Bash command that includes "git commit":

1. Run tests: `npm test`
2. Check that all tests pass
3. If tests fail:
   - DO NOT proceed with commit
   - Analyse failures
   - Fix issues
   - Re-run tests
4. Only allow commit if tests are green

This creates a "test-and-fix" loop ensuring broken code never gets committed.
```

#### PreToolUse Hook for Security Scanning

```markdown
# Security Scan Hook

Before any git push command:

1. Run security audit: `npm audit --audit-level=high`
2. Check for high or critical vulnerabilities
3. If vulnerabilities found:
   - Report them
   - Suggest fixes (npm audit fix)
   - Block push until resolved
4. Scan for hardcoded secrets: `git diff --cached | grep -E "(api_key|password|secret)"`
5. Warn if potential secrets detected

Only proceed with push if all checks pass.
```

### Multi-Agent Review System

The built-in code review plugin uses four parallel agents for comprehensive analysis:

```markdown
# Advanced Multi-Agent Review Configuration

## Agent 1: CLAUDE.md Compliance Checker
Task: Verify all changes follow guidelines in CLAUDE.md
Focus: Coding standards, architecture patterns, testing requirements
Confidence threshold: 80

## Agent 2: CLAUDE.md Compliance Checker (Redundant)
Task: Double-check CLAUDE.md compliance for critical issues
Focus: Same as Agent 1, provides redundancy
Confidence threshold: 80

## Agent 3: Bug Detector
Task: Find obvious bugs in changed code only
Focus:
- Logic errors (off-by-one, null checks, etc.)
- Type mismatches
- Unhandled errors
- Resource leaks
Exclusions: Pre-existing bugs, style issues
Confidence threshold: 85

## Agent 4: Historical Context Analyser
Task: Analyse git history and surrounding code
Focus:
- Breaking changes to existing functionality
- Inconsistencies with established patterns
- Related code that should be updated
- Commit messages revealing intent
Confidence threshold: 75

## Aggregation Strategy
- Combine findings from all agents
- De-duplicate similar issues
- Filter by confidence threshold
- Prioritise by severity
- Output sorted by file and line number
```

---

## Best Practices for AI-Assisted Code Review

### 1. Keep CLAUDE.md Focused and Concise

**Why**: Frontier LLMs can follow ~150-200 instructions consistently. Overloading CLAUDE.md dilutes effectiveness.

**How**:
- Include only universally applicable instructions
- Focus on project-specific quirks and critical standards
- Avoid restating general programming principles
- Keep it under 500 lines
- Use clear, imperative language

**Example of Good vs. Bad**:

❌ **Too Verbose**:
```markdown
Functions should be well-designed and follow good software engineering
principles. They should do one thing and do it well. They should be
easy to understand and maintain. Functions should not be too long...
```

✅ **Concise**:
```markdown
- Functions must be under 50 lines; extract helpers if longer
- One function = one responsibility
- Prefer pure functions; isolate side effects
```

### 2. Use Claude as a First Pass, Not Final Authority

**Approach**:
- Let Claude catch obvious bugs and security issues
- Have humans review Claude's findings for context
- Require human approval for all merges
- Use Claude to augment, not replace, human judgement

**Real-world results**: Graphite found that Claude met their standards for code review after testing against 500 pull requests, including synthetic and real-world examples with known bugs that even experienced engineers struggled to spot.

### 3. Customise Reviews for Your Workflow

**Strategy**:
- Create project-specific review commands in `.claude/commands/`
- Use different review prompts for different PR types
- Adjust confidence thresholds based on false positive rates
- Share custom commands via version control

**Example Workflow**:
```bash
# Different reviews for different contexts
/project:frontend-review    # UI changes
/project:api-review         # Backend endpoints
/project:migration-review   # Database changes
/security-review            # Security-critical code
```

### 4. Integrate with Existing Quality Gates

**Don't Replace**:
- Linters (ESLint, Pylint, etc.)
- Type checkers (TypeScript, mypy)
- Test suites
- Security scanners (Snyk, Dependabot)

**Do Complement**:
- Claude finds issues linters miss (logic errors, security flaws)
- Claude understands context and intent
- Claude suggests architectural improvements
- Claude explains complex code

### 5. Control Costs and Performance

**Strategies**:
- Set `--max-turns` to limit iterations (3-5 is usually sufficient)
- Use path filters to avoid reviewing non-code changes
- Run expensive reviews (security) only on sensitive paths
- Cache dependencies in CI/CD
- Use `allow_failure: true` to avoid blocking merges

**Cost Comparison**:
```yaml
# Expensive: Reviews everything on every commit
on: [push]

# Optimal: Reviews only PR changes
on:
  pull_request:
    types: [opened, synchronise]
    paths:
      - 'src/**'
      - '!**/*.md'
```

### 6. Establish Clear Review Ownership

**Guidelines**:
- Claude provides suggestions; developers make decisions
- Document who is responsible for acting on findings
- Create a workflow for triaging Claude's comments
- Track false positive rates and adjust thresholds
- Regularly audit Claude's review quality

### 7. Use Severity Levels to Prioritise

**Workflow**:
1. **Critical/High**: Must fix before merge
2. **Medium**: Should fix, but can be addressed in follow-up
3. **Low/Info**: Nice to have, optional improvements

**Example Merge Policy**:
```markdown
# PR Merge Policy

Required before merge:
- ✅ All CRITICAL and HIGH security issues resolved
- ✅ All HIGH-confidence bugs fixed
- ✅ Test suite passing

Can be deferred:
- MEDIUM security issues → Create follow-up issue
- LOW-confidence suggestions → Developer discretion
- INFO-level observations → Optional
```

### 8. Continuously Improve Review Quality

**Feedback Loop**:
1. Track false positives and false negatives
2. Update CLAUDE.md with new patterns
3. Adjust confidence thresholds based on accuracy
4. Share learnings across teams
5. Refine custom commands over time

**Metrics to Track**:
- False positive rate (Claude flags non-issues)
- False negative rate (Claude misses real bugs)
- Time saved vs. manual review
- Bugs caught before production
- Developer satisfaction

### 9. Leverage Community Resources

**Resources**:
- [Anthropic's Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - Curated commands and workflows
- [Claude Code Solutions Guide](https://github.com/anthropics/claude-code-action) - Complete working examples
- Community discussions and shared CLAUDE.md files

### 10. Security and Privacy Considerations

**Best Practices**:
- Never commit API keys; use secret management
- Review Claude's suggestions before merging (security audit)
- Use organisation secrets for team-wide access
- Rotate API keys periodically
- For sensitive code, use on-premise models (Bedrock/Vertex)
- Ensure compliance with data handling policies
- Log and monitor Claude's actions

**Data Privacy**:
- Code is sent to Anthropic API (or chosen cloud provider)
- Consider using AWS Bedrock or Google Vertex for regulated industries
- Review Anthropic's data usage policies
- Implement network controls if required

---

## Conclusion

Claude Code transforms code review from a time-consuming bottleneck into an efficient, automated process that catches bugs, security vulnerabilities, and architectural issues that humans often miss. By following the configuration strategies, customisation techniques, and best practices outlined in this guide, teams can:

- **Reduce review time** by 40x (as demonstrated by Graphite)
- **Catch critical security issues** before they reach production
- **Maintain consistent code quality** across all pull requests
- **Free up senior developers** to focus on high-value architectural decisions
- **Accelerate development velocity** without sacrificing quality

The key to success is thoughtful configuration: customise CLAUDE.md for your project, create focused review commands for different scenarios, integrate seamlessly with CI/CD pipelines, and use Claude as a powerful augmentation to human expertise rather than a replacement.

Start with the quick setup (`/install-github-app`), experiment with custom prompts, and iteratively refine your configuration based on what works for your team. The investment in proper setup pays dividends in every pull request.

---

## Sources and Further Reading

### Official Documentation
- [Claude Code GitHub Repository](https://github.com/anthropics/claude-code)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Claude Code Action Repository](https://github.com/anthropics/claude-code-action)
- [Code Review Plugin Documentation](https://github.com/anthropics/claude-code/blob/main/plugins/code-review/README.md)
- [Claude Code Security Review](https://github.com/anthropics/claude-code-security-review)
- [Automated Security Reviews Documentation](https://support.claude.com/en/articles/11932705-automated-security-reviews-in-claude-code)

### Integration Guides
- [How to Use Claude Code for PRs and Code Reviews](https://skywork.ai/blog/how-to-use-claude-code-for-prs-code-reviews-guide/)
- [Integrating Claude Code with GitHub Actions](https://stevekinney.com/courses/ai-development/integrating-with-github-actions)
- [How to Integrate Claude Code with CI/CD: Full 2025 DevOps Guide](https://skywork.ai/blog/how-to-integrate-claude-code-ci-cd-guide-2025/)
- [GitLab CI/CD Integration](https://code.claude.com/docs/en/gitlab-ci-cd)
- [Streamlined CI/CD Pipelines Using Claude Code & GitHub Actions](https://medium.com/@itsmybestview/streamlined-ci-cd-pipelines-using-claude-code-github-actions-74be17e51499)

### Best Practices and Tutorials
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Using CLAUDE.MD files: Customising Claude Code](https://claude.com/blog/using-claude-md-files)
- [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [How I use Claude Code (+ my best tips)](https://www.builder.io/blog/claude-code)
- [awesome-claude-code Repository](https://github.com/hesreallyhim/awesome-claude-code)
- [Shipyard Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)

### Case Studies and Real-World Usage
- [Graphite uses Claude to speed up code review by 40x](https://www.claude.com/customers/graphite)
- [Automate security reviews with Claude Code](https://www.anthropic.com/news/automate-security-reviews-with-claude-code)
- [Streamlining GitHub PR Reviews with Custom Claude Code Slash Commands](https://nakamasato.medium.com/resolve-github-pr-reviews-consistently-and-rapidly-with-custom-claude-code-slash-command-3cdb25e1c2cf)
- [How to Automate Code Reviews and Testing with Claude](https://ragaboutit.com/how-to-automate-code-reviews-and-testing-with-claude-in-your-development-pipeline/)

### Community Resources
- [awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
- [ClaudeLog - Docs, Guides, Tutorials](https://claudelog.com/)
- [Combining Claude Code with GitHub Actions and Pull Requests to Scale AI Coding](https://www.aiengineering.report/p/combining-claude-code-with-github)

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Created for: Claude Code Best Practices Repository*
