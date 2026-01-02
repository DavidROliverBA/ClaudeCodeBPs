# Complete Guide to Skills in Claude Code

## Table of Contents
1. [Introduction](#introduction)
2. [What Are Skills?](#what-are-skills)
3. [Skills vs Commands](#skills-vs-commands)
4. [Skill File Structure and Format](#skill-file-structure-and-format)
5. [Creating Custom Skills](#creating-custom-skills)
6. [Skill Discovery and Loading](#skill-discovery-and-loading)
7. [Progressive Disclosure: The Core Design Principle](#progressive-disclosure-the-core-design-principle)
8. [Best Practices for Skill Design](#best-practices-for-skill-design)
9. [Example Skills for Common Domains](#example-skills-for-common-domains)
10. [Installation and Platform Availability](#installation-and-platform-availability)
11. [Security Considerations](#security-considerations)

---

## Introduction

Agent Skills represent a paradigm shift in how AI assistants handle specialised tasks. Published as an open standard in December 2025, Skills are Anthropic's solution for equipping Claude with domain-specific expertise and repeatable workflows without overwhelming its context window. This guide provides a comprehensive overview of Skills in Claude Code, from basic concepts to advanced implementation strategies.

## What Are Skills?

**Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialised tasks.** They teach Claude how to complete specific tasks in a repeatable way, whether that's creating documents with your company's brand guidelines, analysing data using your organisation's specific workflows, or automating personal tasks.

Unlike traditional approaches that require all context upfront, Skills use **progressive disclosure**: Claude loads only the information it needs, when it needs it. Think of Skills as a well-organised manual with a table of contents, specific chapters, and a detailed appendix—Claude starts with the overview and dives deeper only when necessary.

### Core Characteristics

- **Auto-invoked**: Claude automatically applies Skills when your request matches the skill's purpose
- **Context-efficient**: Uses progressive disclosure to avoid overwhelming the context window
- **Multi-file**: Can bundle instructions, reference materials, scripts, and templates
- **Cross-platform**: Works in Claude Code, Claude.ai (paid plans), and the Claude API
- **Executable**: Can include Python or Bash scripts for deterministic operations

## Skills vs Commands

Understanding the distinction between Skills and slash commands is crucial for effective Claude Code usage:

### Skills
- **Invocation**: Automatically triggered by Claude based on context
- **Structure**: Directory-based with SKILL.md plus supporting files (scripts, references, templates)
- **Complexity**: Can handle rich, multi-step workflows with bundled resources
- **Discovery**: Auto-discovered from configured skill directories
- **Use Case**: Auto-applied capabilities that Claude recognises and uses when relevant

**Example**: A `test-driven-development` skill automatically activates when implementing features, enforcing RED-GREEN-REFACTOR methodology without explicit user invocation.

### Slash Commands
- **Invocation**: Explicitly typed by user (e.g., `/review-pr`) or auto-invoked based on description
- **Structure**: Single markdown file with prompt template
- **Complexity**: Best for simpler, repeatable workflows
- **Discovery**: Available through `/` autocomplete in terminal
- **Use Case**: Explicit, user-initiated shortcuts for repeated tasks

**Key Insight**: Use slash commands when you want an explicit terminal entry point. Use Skills when you want Claude to auto-apply a richer workflow. Use CLAUDE.md for short, always-true project conventions.

## Skill File Structure and Format

### Minimal Structure

At minimum, a Skill requires only a directory containing a `SKILL.md` file:

```
my-skill/
└── SKILL.md
```

### Standard Structure

The recommended structure uses three specialised directories:

```
my-skill/
├── SKILL.md           # Core prompt and instructions (required)
├── scripts/           # Executable Python/Bash scripts
├── references/        # Documentation loaded on demand
└── assets/            # Templates, fonts, and binary files
```

### The SKILL.md File

Every `SKILL.md` file must start with YAML frontmatter containing required metadata:

```yaml
---
name: my-skill-name
description: A clear, specific description of what this skill does and when Claude should use it (max 200 characters)
---

# My Skill Name

[Your detailed instructions go here]

## When to Use This Skill

- Specific scenario 1
- Specific scenario 2

## Instructions

1. Step one of the process
2. Step two of the process

## Examples

**Input:**
```
Example input here
```

**Output:**
```
Expected output here
```

## Reference Materials

For additional context, see:
- [Reference documentation](references/detailed-guide.md)
- [Templates](assets/template.html)
```

### Frontmatter Fields

**Required:**
- `name`: Unique lowercase identifier with hyphens (max 64 characters)
  - Example: `test-driven-development`, `changelog-generator`, `pdf-extractor`
- `description`: Complete explanation of functionality and use cases (max 200 characters)
  - This is crucial—Claude uses the description to determine when to load the skill

**Optional:**
- `dependencies`: Software packages required for script execution

### Directory Purposes

**`scripts/`**: Contains executable code that Claude runs via the Bash tool
- Python scripts for data processing
- Bash scripts for automation
- Validators and code generators
- Any deterministic operation

**`references/`**: Supplemental documentation loaded into context only when needed
- Detailed API documentation
- Style guides and standards
- Comprehensive examples
- Technical specifications

**`assets/`**: Templates and binary resources
- HTML/CSS templates
- Font files
- Configuration templates
- Sample data files

## Creating Custom Skills

### Step-by-Step Process

**1. Plan Your Skill**
- Identify the specific task or workflow to automate
- Define clear trigger scenarios (when should Claude use this?)
- List required resources (documentation, templates, scripts)

**2. Create the Directory Structure**
```bash
mkdir -p my-skill/{scripts,references,assets}
```

**3. Write SKILL.md**

Start with clear metadata:
```yaml
---
name: commit-message-generator
description: Generate structured commit messages following Conventional Commits standard with proper scope and breaking change notation
---
```

Add comprehensive instructions:
```markdown
# Commit Message Generator

Generate commit messages following the Conventional Commits specification.

## Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Types
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- refactor: Code refactoring
- test: Test additions/changes

## Process
1. Analyse git diff to understand changes
2. Determine appropriate type and scope
3. Write concise subject (max 50 chars)
4. Add detailed body explaining "why" (not "what")
5. Include breaking change footer if applicable

## Examples

**Feature:**
```
feat(auth): add OAuth2 integration

Implement Google and GitHub OAuth providers to give
users more login options and improve security.

BREAKING CHANGE: Session management now requires
AUTH_SECRET environment variable
```
```

**4. Add Reference Materials** (if needed)

Create `references/conventional-commits-spec.md` with detailed specification that Claude can read when needed.

**5. Add Executable Scripts** (if needed)

Create `scripts/validate-commit.py`:
```python
#!/usr/bin/env python3
import re
import sys

def validate_commit_message(message):
    pattern = r'^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,50}'
    return bool(re.match(pattern, message))

if __name__ == "__main__":
    message = sys.argv[1]
    if validate_commit_message(message):
        print("✓ Valid commit message")
        sys.exit(0)
    else:
        print("✗ Invalid commit message format")
        sys.exit(1)
```

**6. Test Your Skill**

Before deploying:
- Review SKILL.md for clarity and accuracy
- Verify description matches intended usage scenarios
- Confirm all referenced files exist in correct locations
- Test with sample prompts that should trigger the skill

After deploying:
- Try multiple trigger prompts
- Review Claude's reasoning to confirm skill loading
- Refine description if the skill isn't triggering appropriately
- Iterate based on real usage

### Example: Test-Driven Development Skill

Here's a complete example of a popular skill:

**Directory structure:**
```
test-driven-development/
├── SKILL.md
├── references/
│   └── tdd-principles.md
└── scripts/
    └── run-tests.sh
```

**SKILL.md:**
```yaml
---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code. Enforces RED-GREEN-REFACTOR cycle.
---

# Test-Driven Development

Apply strict TDD methodology to all code changes.

## Core Principle: RED-GREEN-REFACTOR

**NEVER write implementation code before tests.**

## Process

1. **RED**: Write a failing test first
   - Run test to verify it fails
   - Confirm failure message is clear and expected

2. **GREEN**: Write minimal code to pass
   - Implement only what's needed for this test
   - Run test to verify it passes
   - No refactoring yet

3. **REFACTOR**: Improve code quality
   - Clean up implementation
   - Ensure tests still pass
   - Commit changes

4. **COMMIT**: Save progress
   - Commit after each RED-GREEN-REFACTOR cycle
   - Use descriptive commit message

## Enforcement

If implementation code exists before tests:
- **DELETE** the implementation code
- Start over with tests first
- No exceptions

## When Not to Use

- Quick experiments (use `/experiment` command instead)
- Updating documentation only
- Configuration changes with no logic
```

## Skill Discovery and Loading

### How Claude Discovers Skills

Claude Code scans multiple locations to build the available skills list:

1. **User settings**: `~/.config/claude/skills/` or `~/.claude/skills/`
   - Global skills available across all projects
   - Personal workflow preferences

2. **Project settings**: `.claude/skills/`
   - Project-specific skills
   - Team-shared workflows via version control

3. **Plugin-provided**: Installed via Claude Code plugins
   - Marketplace skills
   - Third-party extensions

4. **Built-in skills**: Shipped with Claude Code
   - Standard capabilities
   - Core workflows

### Invocation Flow

When you make a request to Claude:

1. **Skill matching**: Claude examines all skill descriptions to determine relevance
2. **Tool invocation**: If a skill matches, Claude invokes the Skill tool with the skill name
3. **Context injection**: System responds with the skill's base path and full SKILL.md content (minus frontmatter)
4. **Execution**: Claude follows injected instructions, accessing additional files as needed
5. **Result delivery**: Claude completes the task using the skill's methodology

This on-demand loading means skills don't consume context until actually needed.

### Skill Metadata in Tool Definition

The Skill tool embeds an `<available_skills>` section listing all accessible skills:

```xml
<available_skills>
<skill>
<name>pdf</name>
<description>Extract and analyse text from PDF documents using embedded scripts</description>
<location>user</location>
</skill>
<skill>
<name>test-driven-development</name>
<description>Use when implementing any feature or bugfix, before writing implementation code</description>
<location>builtin</location>
</skill>
</available_skills>
```

Claude uses this metadata to make intelligent decisions about which skills to load.

## Progressive Disclosure: The Core Design Principle

Progressive disclosure is the fundamental principle that makes Skills scalable and efficient.

### The Three-Level Structure

**Level 1: Metadata** (always loaded)
- Skill name and description
- Pre-loaded in system prompt via tool definition
- Minimal token cost (~50 tokens per skill)

**Level 2: Core Instructions** (loaded when triggered)
- Full SKILL.md content
- Loaded only when skill is invoked
- Moderate token cost (~500-2000 tokens typical)

**Level 3: Extended Resources** (loaded as needed)
- Reference files, templates, detailed documentation
- Accessed only when Claude determines necessity
- Variable token cost depending on resource size

### Benefits

**Context efficiency**: Rather than loading all documentation upfront, Claude loads information progressively as needed.

**Scalability**: You can bundle comprehensive documentation without overwhelming the context window.

**Flexibility**: Skills can reference unlimited additional files without upfront cost.

### Implementation Tips

**Keep SKILL.md focused**: Under 500 lines (approximately 5,000 words) for optimal performance.

**Use references for detail**: Move extensive documentation, API specs, and examples to `references/` directory.

**One level deep**: Link directly from SKILL.md to reference files. Avoid deeply nested references (A→B→C) as Claude may partially read files.

**Clear reference markers**: Make it obvious when Claude should consult additional resources:

```markdown
## Advanced Configuration

For detailed information about configuration options, see [configuration reference](references/config-spec.md).

## API Integration

When working with the API, consult [API documentation](references/api-guide.md) for:
- Authentication methods
- Rate limiting details
- Error handling patterns
```

## Best Practices for Skill Design

### 1. Start with Evaluation

Before building a skill, test Claude on representative tasks to identify capability gaps. Ask:
- Where does Claude struggle without guidance?
- What workflows require repeated instructions?
- What domain knowledge would improve results?

### 2. Write Crystal-Clear Descriptions

The description field is your most important investment. Claude uses it to decide whether to load the skill.

**Good descriptions:**
- ✓ "Generate structured commit messages following Conventional Commits standard with proper scope and breaking change notation"
- ✓ "Extract text content from PDF files and analyse structure, tables, and formatting"
- ✓ "Test web applications using Playwright with screenshot capture and accessibility validation"

**Poor descriptions:**
- ✗ "Helps with commits" (too vague)
- ✗ "PDF skill" (doesn't explain when to use)
- ✗ "Testing" (ambiguous domain)

### 3. Include Concrete Examples

Show Claude what success looks like with input/output pairs:

```markdown
## Examples

### Example 1: Bug Fix Commit

**Git diff shows:**
```diff
- if (user.role === 'admin') {
+ if (user.role === 'admin' || user.role === 'moderator') {
```

**Generated message:**
```
fix(auth): allow moderators to access admin panel

Moderators need access to user management features
to perform their moderation duties effectively.

Fixes #234
```
```

### 4. Structure for Scale

When SKILL.md grows large:
- Split into multiple referenced documents
- Keep mutually exclusive contexts separate
- Use progressive disclosure aggressively

**Instead of one large file:**
```markdown
# Massive Skill (5000+ lines)

## Python Style Guide
[2000 lines of Python guidelines]

## JavaScript Style Guide
[2000 lines of JavaScript guidelines]

## SQL Style Guide
[1000 lines of SQL guidelines]
```

**Use multiple references:**
```markdown
# Code Review Skill

## Style Guidelines

Consult the appropriate language guide:
- [Python](references/python-style.md)
- [JavaScript](references/javascript-style.md)
- [SQL](references/sql-style.md)
```

### 5. Clarify Execution Intent

Make it explicit whether Claude should execute code or use it as reference:

```markdown
## Scripts

### Running Tests
Execute `scripts/run-tests.sh` to run the test suite:
```bash
./scripts/run-tests.sh --verbose
```

### Understanding the Test Framework
For reference only (DO NOT EXECUTE), the test framework structure is documented in `references/test-framework.md`.
```

### 6. Use Code Execution Wisely

Include executable scripts for:
- Sorting or filtering large datasets
- PDF field extraction
- File format conversions
- Validation logic
- Data transformation

**Avoid scripts for:**
- Operations Claude can perform via existing tools
- Tasks requiring complex decision-making
- One-off transformations

### 7. Iterate with Claude

When developing skills:
1. Use Claude to draft initial SKILL.md
2. Test the skill on real tasks
3. Have Claude self-reflect on mistakes
4. Capture successful approaches into the skill
5. Repeat until results are consistently good

### 8. Create Focused Skills

**DO**: Create separate skills for different workflows
- `code-review-backend`
- `code-review-frontend`
- `code-review-security`

**DON'T**: Create one mega-skill trying to cover everything
- `code-review-everything` (6000 lines covering all possible scenarios)

Focused skills are easier to maintain, trigger more reliably, and use context more efficiently.

### 9. Test Incrementally

Don't build a complex skill all at once:
1. Start with minimal SKILL.md
2. Test basic functionality
3. Add one feature at a time
4. Test after each addition
5. Verify skill still triggers correctly

### 10. Consider Your Audience

**Personal skills**: Can use shorthand and assume context
```markdown
description: Use my preferred commit format with emoji prefixes
```

**Team skills**: Need clear documentation and examples
```markdown
description: Generate commit messages following team convention (type/scope/subject) as documented in CONTRIBUTING.md
```

**Public skills**: Require comprehensive documentation and error handling
```markdown
description: Extract and analyse PDF documents with support for tables, images, and multi-column layouts. Handles corrupted PDFs gracefully.
```

## Example Skills for Common Domains

### Creative & Design

**`brand-guidelines`**
```yaml
---
name: brand-guidelines
description: Create documents following company brand guidelines including fonts, colours, tone, and visual identity
---
```
Use for: Marketing materials, presentations, documentation following brand standards

**`d3-visualisation`**
```yaml
---
name: d3-visualisation
description: Create interactive D3.js data visualisations with proper axis scaling, legends, and responsive design
---
```
Use for: Charts, graphs, interactive dashboards

### Development & Technical

**`test-driven-development`**
```yaml
---
name: test-driven-development
description: Implement features using TDD methodology - write failing tests first, then minimal code to pass, then refactor
---
```
Use for: Any feature or bugfix requiring verifiable tests

**`changelog-generator`**
```yaml
---
name: changelog-generator
description: Transform git commit history into user-facing release notes with categorised changes and version numbering
---
```
Use for: Release preparation, version documentation

**`webapp-testing`**
```yaml
---
name: webapp-testing
description: Test web applications using Playwright with screenshot capture, accessibility checks, and cross-browser validation
---
```
Use for: End-to-end testing, UI validation

**`api-documentation`**
```yaml
---
name: api-documentation
description: Generate OpenAPI/Swagger documentation from code with examples, error codes, and authentication details
---
```
Use for: API documentation, endpoint specifications

### Enterprise & Communication

**`meeting-notes`**
```yaml
---
name: meeting-notes
description: Transform meeting transcripts into structured notes with action items, decisions, and follow-ups
---
```
Use for: Meeting summaries, action tracking

**`email-templates`**
```yaml
---
name: email-templates
description: Draft professional emails following company tone and structure guidelines
---
```
Use for: Customer communication, internal announcements

### Data & Analysis

**`pdf-extractor`**
```yaml
---
name: pdf-extractor
description: Extract and analyse text, tables, and metadata from PDF documents with structure preservation
---
```
Use for: Document processing, data extraction

**`csv-analysis`**
```yaml
---
name: csv-analysis
description: Analyse CSV data with statistical summaries, visualisations, and anomaly detection
---
```
Use for: Data analysis, reporting

### Specialised Workflows

**`subagent-driven-development`**
```yaml
---
name: subagent-driven-development
description: Dispatch fresh subagent per task with two-stage review (spec compliance, then code quality)
---
```
Use for: Complex features requiring isolated context

**`security-review`**
```yaml
---
name: security-review
description: Review code for common vulnerabilities including SQL injection, XSS, authentication issues, and insecure dependencies
---
```
Use for: Security audits, code review

## Installation and Platform Availability

### Platform Support

Skills are available across multiple Claude platforms:

**Claude Code (CLI)**:
- Full skill support
- Local skill development and testing
- Command-line workflow integration

**Claude.ai (Web)**:
- Available on Pro, Max, Team, and Enterprise plans
- Feature preview requiring code execution enabled
- Upload custom skills via Settings → Capabilities

**Claude API**:
- Available in beta for all API users
- Requires code execution tool enabled
- Skills bundled with API requests

### Installation Methods

**1. User-level Skills (Global)**

Install in home directory for all projects:
```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/anthropics/skills.git
```

Or manually create:
```bash
mkdir -p ~/.claude/skills/my-skill
# Create SKILL.md and other files
```

**2. Project-level Skills (Team Sharing)**

Install in project directory for version control:
```bash
cd /path/to/your/project
mkdir -p .claude/skills
cd .claude/skills
# Add skill folders here
git add .claude/skills
git commit -m "Add team skills"
```

Team members get skills automatically when they clone the repo.

**3. Plugin Installation**

Install from marketplace:
```bash
claude plugins install skill-name
```

Skills from plugins are automatically discovered.

**4. Manual Upload (Claude.ai)**

For web interface:
1. Package skill as ZIP (skill folder as root, not nested)
2. Go to Settings → Capabilities → Skills
3. Click "Upload Custom Skill"
4. Select ZIP file
5. Enable the skill

### Verification

Confirm skill installation:
```bash
# In Claude Code, skills appear in tool definition
# Check by asking Claude: "What skills do you have available?"
```

Claude will list all discovered skills with their descriptions.

## Security Considerations

### Audit Before Installing

Skills execute with your permissions and can:
- Run arbitrary code (Python, Bash scripts)
- Access your filesystem
- Make network connections
- Modify your projects

**Before installing any skill:**

1. **Review SKILL.md**: Understand what the skill does
2. **Inspect scripts**: Read all code in `scripts/` directory
3. **Check dependencies**: Verify any required packages are trusted
4. **Review network access**: Look for URLs and external connections
5. **Test in isolation**: Try the skill in a test project first

### Trusted Sources Only

**Recommended sources:**
- ✓ Official Anthropic repository (`anthropics/skills`)
- ✓ Your organisation's internal repositories
- ✓ Well-known open source maintainers with verified repos
- ✓ Skills you've written yourself

**Approach with caution:**
- ⚠ Random GitHub repositories
- ⚠ Skills with obfuscated code
- ⚠ Skills requiring unusual dependencies
- ⚠ Skills making network requests to unknown endpoints

### Best Practices

**For personal use:**
- Store sensitive skills (with API keys, credentials) in user directory only
- Don't commit skills with secrets to version control
- Use environment variables for configuration

**For team use:**
- Code review skills before merging
- Document skill purpose and behaviour
- Regular security audits of skill scripts
- Version control for skill changes

**For public distribution:**
- Clear documentation of all capabilities
- No hardcoded credentials or secrets
- Explicit permission requests for network/filesystem access
- Versioned releases with changelogs

---

## Conclusion

Skills represent a powerful evolution in AI agent capabilities, enabling Claude to handle specialised tasks with expert-level consistency while maintaining efficient context usage. By following the progressive disclosure principle and best practices outlined in this guide, you can create robust, reusable skills that significantly enhance your Claude Code workflows.

Whether you're automating personal development workflows, codifying team standards, or building enterprise-grade tools, Skills provide the flexibility and structure needed for production-ready AI assistance.

### Additional Resources

- **Official Documentation**: [Claude Code Skills Docs](https://platform.claude.com/docs/en/docs/claude-code/skills)
- **Official Repository**: [anthropics/skills](https://github.com/anthropics/skills)
- **Engineering Blog**: [Equipping Agents for the Real World with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- **Anthropic Academy**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)
- **Community Collections**:
  - [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
  - [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
  - [I-Onlabs/claude-code-skills](https://github.com/I-Onlabs/claude-code-skills)

### Quick Start Template

Ready to create your first skill? Use this template:

```bash
# Create skill directory
mkdir -p my-first-skill/{scripts,references,assets}

# Create SKILL.md
cat > my-first-skill/SKILL.md << 'EOF'
---
name: my-first-skill
description: Brief description of what this skill does and when to use it
---

# My First Skill

## Purpose
Explain what this skill helps with

## When to Use
- Scenario 1
- Scenario 2

## Process
1. Step one
2. Step two
3. Step three

## Examples
[Add examples here]
EOF

# Install to Claude Code
mv my-first-skill ~/.claude/skills/

# Test it!
# Ask Claude something that should trigger your skill
```

Happy skill building!

---

**Document Version**: 1.0
**Last Updated**: January 2026
**Word Count**: ~4,800 words
