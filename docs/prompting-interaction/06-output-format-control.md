# Output Format Control in Claude Code

A comprehensive guide to controlling how Claude Code formats its responses, manages code style, and structures its output.

## Table of Contents
- [Overview](#overview)
- [Output Styles System](#output-styles-system)
- [CLAUDE.md Files for Persistent Format Rules](#claudemd-files-for-persistent-format-rules)
- [Specifying Code Style Preferences](#specifying-code-style-preferences)
- [Variable Naming Conventions](#variable-naming-conventions)
- [Documentation Format Requirements](#documentation-format-requirements)
- [Reducing Verbose Output](#reducing-verbose-output)
- [Controlling Comment Density](#controlling-comment-density)
- [Specifying File Formats and Structures](#specifying-file-formats-and-structures)
- [Prompt Techniques for Concise Responses](#prompt-techniques-for-concise-responses)
- [Advanced Configuration Patterns](#advanced-configuration-patterns)

## Overview

Claude Code provides multiple mechanisms for controlling output formatting, from simple command-line flags to sophisticated persistent configuration files. Understanding these tools allows you to shape Claude's responses to match your exact coding standards, communication preferences, and workflow requirements.

The key to effective output control lies in understanding Claude Code's hierarchical configuration system and choosing the right tool for each formatting requirement.

## Output Styles System

### What Are Output Styles?

Output styles are specialized personas that transform how Claude Code communicates and presents information while retaining all core capabilities like file manipulation, script execution, and TODO tracking. Think of them as different "modes" that control the tone, verbosity, and structure of Claude's responses.

### Built-in Output Styles

Claude Code includes three primary output styles:

1. **Default**: The standard system prompt designed for efficient software engineering task completion. Direct, professional, and focused on getting work done.

2. **Explanatory**: Provides educational "Insights" while completing tasks. This style helps you understand implementation choices, codebase patterns, and architectural decisions. Ideal for learning a new codebase or understanding complex systems.

3. **Learning**: A collaborative, learn-by-doing mode where Claude shares insights while coding and asks you to contribute small, strategic pieces of code yourself. Claude adds `TODO(human)` markers in your code for you to implement, making this perfect for skill development.

### Switching Output Styles

You can switch output styles using several methods:

**Interactive Menu:**
```bash
/output-style
```

**Direct Selection:**
```bash
/output-style explanatory
/output-style learning
```

**Via Configuration:**
Edit the `outputStyle` field in `.claude/settings.local.json` directly.

### Creating Custom Output Styles

Custom output styles are Markdown files with optional YAML frontmatter that get added to Claude's system prompt. This is one of the most powerful features for output control.

**Location:**
- User-level: `~/.claude/output-styles/`
- Project-level: `.claude/output-styles/`

**Example Custom Style (concise-technical.md):**
```markdown
---
name: Concise Technical
description: Minimal, technical responses without preamble
---

# Output Format Instructions

- Skip all preambles and pleasantries
- Use bullet points over paragraphs
- Provide only essential explanations
- Limit responses to 3-5 sentences unless complex analysis required
- Use technical terminology appropriate for senior developers
- Avoid restating user requests
- Get straight to implementation or solution
```

**Example Custom Style (detailed-documenter.md):**
```markdown
---
name: Detailed Documenter
description: Verbose documentation-focused output
---

# Documentation Focus

- Prioritize comprehensive inline comments
- Generate detailed docstrings for all functions
- Explain architectural decisions
- Include usage examples in documentation
- Document edge cases and error handling
- Create clear module-level documentation
```

Once created, these styles appear in the `/output-style` picker and persist across sessions.

## CLAUDE.md Files for Persistent Format Rules

### Understanding the Memory Hierarchy

Claude Code implements a sophisticated hierarchical memory system with four levels:

1. **Enterprise Policy** - System-wide (managed by IT/DevOps)
2. **Project Memory** - Team-shared via `./CLAUDE.md` or `./.claude/CLAUDE.md`
3. **Project Rules** - Modular instructions in `./.claude/rules/*.md`
4. **User Memory** - Personal preferences in `~/.claude/CLAUDE.md`

**Critical Principle:** CLAUDE.md content is treated as **authoritative system rules**, while user prompts are interpreted as flexible requests. This hierarchy ensures consistent behavior and higher instruction adherence throughout your sessions.

### File Locations and Discovery

Claude Code recursively searches for memory files starting from your current working directory up to (but not including) the root directory `/`. This means:

```
/project/
├── CLAUDE.md                    # Project-wide rules
├── src/
│   ├── CLAUDE.md               # Source-specific rules
│   └── auth/
│       └── CLAUDE.md           # Authentication module rules
└── CLAUDE.local.md             # Personal overrides (gitignored)
```

When working in `/project/src/auth/`, Claude loads all three CLAUDE.md files, with more specific rules taking precedence.

### CLAUDE.md Structure and Features

**Basic Structure:**
```markdown
# Project Name

## Code Style
- Use ES modules (import/export), not CommonJS (require)
- Destructure imports when possible
- Prefer const over let
- Max line length: 100 characters

## Documentation Standards
- JSDoc for all public functions
- Inline comments only for complex logic
- README.md in each feature directory

## Testing Approach
- Run individual tests for performance
- Avoid mocks unless absolutely necessary
- Use descriptive test names: it('should X when Y')
```

**Import Capability:**
```markdown
# Main Config

@.claude/rules/typescript.md
@.claude/rules/react.md
@docs/api-guidelines.md
```

Imports support recursive inclusion up to 5 levels deep and are not evaluated inside markdown code blocks.

### Modular Rules System

The `.claude/rules/` directory enables organized, topic-specific instructions. All `.md` files in this directory are automatically loaded with the same priority as `.claude/CLAUDE.md`.

**Path-Specific Rules:**
```markdown
---
paths: src/**/*.{ts,tsx}
---

# TypeScript/React Rules

- Use explicit return types for all functions
- Prefer interfaces over types for object shapes
- Component files must export single default component
- Use named exports for utilities and hooks
```

**Conditional Application:**
Rules with the `paths` field in frontmatter only apply when Claude works with files matching those glob patterns.

### Local Overrides

`CLAUDE.local.md` files are automatically added to `.gitignore`, making them ideal for:
- Personal code style preferences
- Machine-specific paths or configurations
- Experimental rules before proposing to team
- Private project notes

### Best Practices for CLAUDE.md

1. **Be Concise**: Claude's context window is precious. Keep instructions minimal and universally applicable.

2. **Use Clear Structure**: Markdown headings and bullet points work better than paragraphs.

3. **Be Specific, Not Vague**:
   - ❌ "Write good tests"
   - ✅ "Write unit tests covering happy path and error cases; avoid mocks"

4. **Explain the Why**: Claude generalizes better from motivated instructions.
   - ❌ "Don't use any"
   - ✅ "Avoid TypeScript 'any' type as it defeats type safety"

5. **Provide Examples**:
```markdown
## Error Handling

Wrap async operations in try-catch:

\`\`\`typescript
try {
  const data = await fetchData();
  return processData(data);
} catch (error) {
  logger.error('Failed to fetch data', error);
  throw new AppError('Data fetch failed', error);
}
\`\`\`
```

6. **Regular Review**: Update memory files as your project evolves. Use `/memory` command to open files for editing.

## Specifying Code Style Preferences

### Architecture and Module Patterns

Document your architectural decisions to ensure consistency:

```markdown
## Architecture Patterns

- Use Clean Architecture with domain/application/infrastructure layers
- Dependency injection via constructor parameters
- Repository pattern for data access
- Service layer for business logic

## Module Structure

Each feature module should contain:
\`\`\`
feature/
├── index.ts              # Public API
├── domain/               # Business entities
├── application/          # Use cases
├── infrastructure/       # External dependencies
└── __tests__/           # Tests
\`\`\`
```

### Language-Specific Conventions

**TypeScript Example:**
```markdown
## TypeScript Guidelines

- Strict mode enabled
- Explicit return types for exported functions
- Prefer readonly arrays: `readonly string[]` over `string[]`
- Use template literal types for string unions
- Enable exactOptionalPropertyTypes
```

**Python Example:**
```markdown
## Python Standards

- Black formatting (line length: 88)
- Type hints for all function signatures
- Docstrings: Google style
- Imports: stdlib, third-party, local (separated by blank lines)
- Use pathlib over os.path
```

### Import and Dependency Management

```markdown
## Import Conventions

### TypeScript/JavaScript
- Destructure imports: `import { func } from 'lib'`
- Group imports: external, internal, types
- Alphabetize within groups
- No default + named export mixing

### Python
- Absolute imports preferred
- One import per line
- Avoid wildcard imports (`from x import *`)
```

### Code Organization Rules

```markdown
## File Organization

- Max file length: 300 lines
- One class/component per file (except tightly coupled helpers)
- Test files co-located with source: `feature.ts` + `feature.test.ts`
- Index files only export, no logic
```

## Variable Naming Conventions

### Cross-Language Standards

Document naming conventions clearly with examples:

```markdown
## Naming Conventions

### TypeScript/JavaScript
- **Variables/Functions**: camelCase
  - `getUserData`, `isActive`, `hasPermission`
- **Classes/Interfaces**: PascalCase
  - `UserService`, `IUserRepository`, `ValidationError`
- **Constants**: UPPER_SNAKE_CASE
  - `MAX_RETRY_COUNT`, `API_BASE_URL`
- **Private fields**: prefix with underscore
  - `_internalState`, `_handleClick`
- **Boolean naming**: Use is/has/should prefix
  - `isLoading`, `hasError`, `shouldValidate`

### Python
- **Variables/Functions**: snake_case
  - `user_data`, `get_user_profile`
- **Classes**: PascalCase
  - `UserService`, `DataProcessor`
- **Constants**: UPPER_SNAKE_CASE
  - `MAX_CONNECTIONS`, `DEFAULT_TIMEOUT`
- **Private**: Single underscore prefix
  - `_internal_method`
- **Name mangling**: Double underscore
  - `__private_attribute`

### Database
- **Tables**: snake_case, plural
  - `user_profiles`, `order_items`
- **Columns**: snake_case, singular
  - `user_id`, `created_at`, `is_active`
- **Foreign keys**: singular_table_name_id
  - `user_id`, `product_id`
```

### Domain-Specific Conventions

```markdown
## Java Enterprise Conventions

- **Packages**: com.company.domain.subdomain
  - `com.acme.ecommerce.user.service`
- **Classes**: PascalCase + suffix pattern
  - Service: `UserService`, `OrderService`
  - Repository: `UserRepository`, `OrderRepository`
  - Controller: `UserController`, `OrderController`
  - DTO: `UserDto`, `CreateOrderRequest`
- **Methods**: camelCase, verb prefix
  - `getUserById`, `createOrder`, `validateInput`
- **Constants**: UPPER_SNAKE_CASE
  - `MAX_LOGIN_ATTEMPTS`, `SESSION_TIMEOUT_MINUTES`
```

### Anti-Patterns to Avoid

```markdown
## Naming Anti-Patterns

**Avoid:**
- ❌ Single letter variables (except loop counters i, j, k)
- ❌ Abbreviations: `usr`, `btn`, `ctx` (unless industry standard: HTTP, API, ID)
- ❌ Numeric suffixes: `data1`, `data2`, `user3`
- ❌ Generic names: `temp`, `data`, `info`, `item` (be specific)
- ❌ Hungarian notation: `strName`, `intCount`

**Prefer:**
- ✅ Descriptive names: `authenticatedUser`, `submitButton`, `applicationContext`
- ✅ Domain language: Use terms from business domain
- ✅ Intention-revealing: `customerTotalSpent` > `customerValue`
```

## Documentation Format Requirements

### Inline Code Documentation

Specify exact documentation standards for different contexts:

```markdown
## Documentation Standards

### Function Documentation (TypeScript)

All exported functions must have JSDoc:

\`\`\`typescript
/**
 * Retrieves user profile data by user ID
 *
 * @param userId - Unique identifier for the user
 * @param options - Optional fetch configuration
 * @returns Promise resolving to user profile or null if not found
 * @throws {AuthenticationError} If user is not authenticated
 * @throws {ValidationError} If userId is invalid
 *
 * @example
 * const profile = await getUserProfile('user_123');
 * if (profile) {
 *   console.log(profile.name);
 * }
 */
async function getUserProfile(
  userId: string,
  options?: FetchOptions
): Promise<UserProfile | null> {
  // implementation
}
\`\`\`

### Python Docstrings (Google Style)

\`\`\`python
def calculate_discount(price: float, discount_rate: float) -> float:
    """Calculate final price after applying discount.

    Args:
        price: Original price before discount
        discount_rate: Discount as decimal (0.1 for 10%)

    Returns:
        Final price after discount applied

    Raises:
        ValueError: If price is negative or discount_rate not in [0, 1]

    Examples:
        >>> calculate_discount(100.0, 0.2)
        80.0
    """
    pass
\`\`\`
```

### Module and Class Documentation

```markdown
## Module-Level Documentation

Every module must start with:

\`\`\`typescript
/**
 * @module user/authentication
 *
 * Handles user authentication flows including login, logout,
 * token refresh, and session management.
 *
 * Key components:
 * - AuthService: Main authentication service
 * - TokenManager: JWT token handling
 * - SessionStore: Session persistence
 *
 * @see {@link https://docs.example.com/auth} for authentication flow
 */
\`\`\`

### Class Documentation

\`\`\`typescript
/**
 * Manages user authentication and session lifecycle
 *
 * This service handles the complete authentication flow from initial
 * login through token refresh and logout. It integrates with the
 * TokenManager for JWT handling and SessionStore for persistence.
 *
 * @example
 * const auth = new AuthService(tokenManager, sessionStore);
 * const session = await auth.login(credentials);
 */
class AuthService {
  // implementation
}
\`\`\`
```

### Comment Guidelines

```markdown
## When to Comment

**DO comment:**
- Complex algorithms or business logic
- Non-obvious workarounds or bug fixes
- Performance optimizations
- Regex patterns
- Security-sensitive code

**DON'T comment:**
- Self-explanatory code
- What the code does (code should be self-documenting)
- Redundant restatements

**Examples:**

❌ Bad:
\`\`\`typescript
// Increment i
i++;

// Create new user
const user = new User();
\`\`\`

✅ Good:
\`\`\`typescript
// Retry connection with exponential backoff to handle transient network issues
const connection = await retryWithBackoff(() => connect(), MAX_RETRIES);

// WORKAROUND: Safari doesn't support lookbehind in regex (2024)
// Using split/filter instead of single regex
const tokens = input.split(/\s+/).filter(t => !t.startsWith('#'));
\`\`\`
```

### README and Documentation Files

```markdown
## Project Documentation Structure

### Root README.md
- Project overview and purpose
- Quick start guide
- Installation instructions
- Basic usage examples
- Link to detailed docs

### Module READMEs
Each feature directory should have README.md:
- Module purpose
- API documentation
- Usage examples
- Dependencies
- Testing instructions

### API Documentation
Generate from code comments using:
- TypeDoc for TypeScript
- Sphinx for Python
- JavaDoc for Java
```

## Reducing Verbose Output

### Output Style for Conciseness

Create a custom output style focused on brevity:

```markdown
---
name: Minimal
description: Ultra-concise technical responses
---

# Communication Protocol

- Maximum 3 sentences per response unless complex analysis required
- No preambles, acknowledgments, or sign-offs
- Bullet points over prose paragraphs
- Code snippets over verbal explanations
- Assume expert-level knowledge
- Skip explanations of basic concepts
- Direct answers only
```

### Prompt-Level Verbosity Control

Use explicit length constraints in your prompts:

**Specific Length:**
```
Explain this function in exactly 2 sentences.

Refactor this code. Response: code only, no explanation.

List the steps to fix this bug (bullet points, max 5 items).
```

**Prefilling Technique:**
If Claude includes unnecessary preambles, use prefilling or explicit requests:
```
Skip the preamble and get straight to the answer.

No acknowledgment needed, just show me the code.

Direct implementation only, hold all explanations.
```

### Command-Line Verbosity Flags

**Non-Interactive Mode:**
```bash
# Print mode: just results, no interaction
claude -p "analyze code quality" --output-format json

# Suppress color for piping
claude --no-color

# Quiet mode (if available in your version)
claude --quiet
```

**Debug vs Production:**
```bash
# Verbose for debugging
claude --verbose

# Standard for production (cleaner output)
claude
```

### CLAUDE.md Conciseness Rules

```markdown
## Response Format

- Provide implementation first, explanation second
- Use code blocks over prose descriptions
- Assume senior developer familiarity with tech stack
- Skip routine confirmations ("I'll help you with that...")
- No self-referential statements ("As an AI...")
- Maximum 100 words explanation unless requested
```

### Claude 4.5 Model Behavior

The newer Claude 4.5 models (Sonnet 4.5, Opus 4.5) default to more concise communication:
- More direct and grounded responses
- Fact-based progress reports over self-celebratory updates
- Less verbose, may skip detailed summaries unless prompted
- More natural, conversational tone

If you need more detail, explicitly request it:
```
Explain your reasoning in detail.
Include a comprehensive analysis.
Walk through each step thoroughly.
```

## Controlling Comment Density

### Defining Comment Density Levels

Create clear tiers of comment density in your CLAUDE.md:

```markdown
## Comment Density Standards

### Minimal (Production Code)
- Public API functions only
- Complex algorithms and business logic
- Non-obvious workarounds
- Security considerations
- NO comments for self-explanatory code

### Standard (Team Development)
- All exported functions and classes
- Complex logic blocks
- Important architectural decisions
- Type complexity explanations

### Verbose (Educational/Documentation)
- Every function including internal helpers
- Step-by-step algorithm explanations
- Rationale for implementation choices
- Examples and edge cases
- Learning-oriented commentary

**Default: Use Standard density unless specified**
```

### Context-Specific Comment Rules

Use path-specific rules for different comment requirements:

```markdown
---
paths: src/**/*.ts
---

# TypeScript Comment Standards

- JSDoc for all exported items
- Inline comments only for complex logic
- No comments restating code
- Document type complexity
```

```markdown
---
paths: examples/**/*
---

# Example Code Comment Standards

- Heavy inline comments explaining each step
- Assume reader is learning the concept
- Explain why, not just what
- Include common pitfalls and gotchas
```

### Comment Directive Pattern

Implement custom comment directives for special handling:

```markdown
## Comment Directives

### @implement
When you find `@implement` comments in files, use the instructions to implement the requested changes, then convert comment blocks to documentation blocks.

Example:
\`\`\`typescript
// @implement Add validation for email format
function validateUser(user: User) {
  // implementation here
}
\`\`\`

After implementation:
\`\`\`typescript
/**
 * Validates user data including email format
 * @param user - User object to validate
 * @throws {ValidationError} If email format is invalid
 */
function validateUser(user: User) {
  if (!isValidEmail(user.email)) {
    throw new ValidationError('Invalid email format');
  }
}
\`\`\`

### @explain
Mark complex sections requiring detailed inline comments

### @optimize
Highlight performance-critical code requiring optimization notes
```

### Requesting Specific Comment Density

In your prompts, be explicit:

```
Write this function with minimal comments (public API only).

Implement with verbose comments explaining each step for junior developers.

Add only essential comments for complex logic; skip obvious code.

Generate comprehensive documentation comments but minimal inline comments.
```

### Auto-Documentation Generation

```markdown
## Documentation Generation

When generating documentation:
1. Analyze existing comment density in the file
2. Match that density for consistency
3. If file has no comments, use Standard density
4. Focus on interface/public API documentation
5. Inline comments only where logic is non-trivial

Documentation should answer:
- What does this do? (brief)
- Why does it exist? (context)
- How do I use it? (examples)
- What can go wrong? (errors/edge cases)
```

## Specifying File Formats and Structures

### Project Structure Templates

Define your expected file organization:

```markdown
## Project Structure

### Feature Module Structure
\`\`\`
src/features/{feature-name}/
├── index.ts                 # Public API exports
├── {feature}.service.ts     # Business logic
├── {feature}.repository.ts  # Data access
├── {feature}.controller.ts  # API endpoints
├── {feature}.types.ts       # TypeScript types
├── {feature}.test.ts        # Unit tests
├── {feature}.e2e.test.ts    # E2E tests
└── README.md                # Feature documentation
\`\`\`

### When creating new features:
1. Create all structure files even if initially empty
2. Export from index.ts only
3. Keep related types in .types.ts
4. Co-locate tests with source
```

### File Templates

Provide templates for consistency:

```markdown
## TypeScript Service Template

\`\`\`typescript
/**
 * @module {module-name}
 */

import type { Dependencies } from './types';

/**
 * {Service description}
 */
export class {Name}Service {
  constructor(private readonly deps: Dependencies) {}

  /**
   * {Method description}
   */
  async methodName(): Promise<ReturnType> {
    // implementation
  }
}
\`\`\`

## Python Module Template

\`\`\`python
"""
{Module description}

This module provides...
"""

from typing import List, Optional
import logging

logger = logging.getLogger(__name__)

__all__ = ['public_function', 'PublicClass']


def public_function() -> None:
    """Public function description."""
    pass
\`\`\`
```

### Configuration File Formats

```markdown
## Configuration Standards

### TypeScript Config Files
- Use .ts for configs when possible (type safety)
- Export default object
- Include JSDoc for config options

\`\`\`typescript
/**
 * Application configuration
 */
export default {
  /** API base URL */
  apiUrl: process.env.API_URL || 'http://localhost:3000',

  /** Maximum retry attempts */
  maxRetries: 3,
} as const;
\`\`\`

### JSON Files
- 2-space indentation
- No trailing commas
- Sort keys alphabetically
- Include schema reference when available

### YAML Files
- 2-space indentation
- Use flow style for short arrays: `tags: [typescript, react]`
- Use block style for long arrays
- Comments for complex configurations
```

### Output Format Specification

Tell Claude exactly how to format responses:

```
Create a new React component following this structure:
1. File: ComponentName.tsx
2. Imports: React, types, hooks, utilities (in that order)
3. Interface definition above component
4. Component with TypeScript + JSDoc
5. Export statement at bottom

Do not include test file or storybook file unless requested.
```

### Response Templates

For recurring tasks, define expected response formats:

```markdown
## Code Review Response Format

When performing code reviews, respond with:

\`\`\`markdown
## Summary
[1-2 sentence overview]

## Critical Issues
- [Issue with severity HIGH]

## Improvements
- [Suggestion with severity MEDIUM]

## Nitpicks
- [Minor style issues, severity LOW]

## Positive Notes
- [Things done well]
\`\`\`
```

## Prompt Techniques for Concise Responses

### Specific Length Constraints

The most effective way to control response length is explicit specification:

```
Explain in exactly 2 sentences.

List the 3 main steps (bullet points only).

Code only, no explanation.

One-paragraph summary (max 50 words).
```

**Why it works:** Claude responds better to concrete constraints than vague requests like "be brief."

### Prefilling and Response Shaping

Guide Claude's response format by starting it for them:

```
Q: What does this function do?
A: It [complete this]

Bullet points:
-
-
-

The bug is caused by [complete]
```

### Eliminating Preambles

**Instead of getting:**
> "I'd be happy to help you with that! Let me analyze the code and provide you with a comprehensive solution..."

**Request:**
```
No preamble. Direct answer only.

Skip acknowledgment, just show the fix.

Code first, explanation after.
```

### Extended Thinking Modes

For complex problems, use thinking triggers:

```
think: Standard analysis
think hard: Deeper analysis
think harder: Comprehensive analysis
ultrathink: Maximum depth analysis
```

These allocate progressively more computational resources and produce more thorough responses. Use sparingly for genuinely complex problems.

### Matching Prompt Style to Desired Output

Claude mirrors your communication style:

**For technical, concise output:**
```
Bug: auth token null
Cause: ?
Fix: ?
```

**For detailed explanation:**
```
I'm trying to understand why the authentication token is coming back null.
Could you walk me through the possible causes and explain the best way to fix this?
```

**For code-only:**
```typescript
// Refactor this:
function oldFunction() { ... }

// New version:
```

### Iterative Refinement

Claude's outputs improve with iteration. Use multi-step prompts:

```
1. First, list the main issues (bullet points, max 5)
2. Then, show code fix for issue #1 only
3. After my confirmation, we'll tackle #2
```

This prevents overwhelming responses and gives you control over detail level.

### Format Anchors

Use explicit format specifications:

```
Response format:
Problem: [one sentence]
Solution: [code block]
Test: [test case]

Response format: YAML
key: value

Response format: JSON
{"step": "description"}
```

### Verification Requests

For complex tasks, request Claude verify its own work:

```
Implement the feature, then verify:
1. Does it handle edge case X?
2. Are types correctly defined?
3. Is it testable?

Report verification as checklist.
```

This encourages thoughtful implementation while keeping the response structured.

### Best Practices Summary

1. **Be specific over vague**: "2 sentences" beats "be brief"
2. **Constrain format explicitly**: Define exact structure wanted
3. **Match your style**: Terse prompts → terse responses
4. **Use prefilling**: Start Claude's response format
5. **Eliminate waste**: "No preamble, code only"
6. **Iterate deliberately**: Break complex tasks into steps
7. **Request verification**: Have Claude check its work
8. **Leverage system prompts**: Use CLAUDE.md for persistent style

## Advanced Configuration Patterns

### Combining Techniques

The most powerful approach combines multiple control mechanisms:

```markdown
# CLAUDE.md

## Response Style (GLOBAL RULE)
- Skip preambles and acknowledgments
- Code-first approach: implementation before explanation
- Maximum 3 sentences explanation unless complex
- Use bullet points over paragraphs

## Code Style
@.claude/rules/typescript.md
@.claude/rules/testing.md

## Documentation
@.claude/rules/documentation.md
```

Plus custom output style:
```markdown
# ~/.claude/output-styles/senior-dev.md
---
name: Senior Developer
description: Expert-level technical communication
---

Communicate as a senior developer to another senior developer:
- Assume deep technical knowledge
- Skip basic explanations
- Focus on architecture and design patterns
- Discuss trade-offs and alternatives
- Reference relevant documentation/RFCs
- Be direct and efficient
```

### Hooks for Deterministic Formatting

Create hooks that automatically format code after Claude modifies files:

```json
{
  "hooks": {
    "postWrite": "prettier --write {file} && eslint --fix {file}"
  }
}
```

**Advantages:**
- Formatting happens outside Claude's context
- Deterministic behavior (no AI variation)
- Enforces standards regardless of prompt quality
- Saves context window space

### Conditional Rules by File Type

```markdown
---
paths: "**/*.test.ts"
---

# Test File Standards

- Verbose comments explaining test scenarios
- Use descriptive test names: it('should X when Y')
- AAA pattern: Arrange, Act, Assert sections
- Include edge case documentation
```

```markdown
---
paths: "src/api/**/*.ts"
---

# API File Standards

- OpenAPI/Swagger JSDoc annotations
- Request/response examples in comments
- Error case documentation
- Authentication requirements noted
```

### Team vs Personal Preferences

**Team (./CLAUDE.md):**
```markdown
# Team Standards

- TypeScript strict mode
- Jest for testing
- ESLint config: airbnb-typescript
- PR requires passing tests
```

**Personal (./CLAUDE.local.md):**
```markdown
# Personal Preferences

- Prefer functional over class components
- Use absolute imports
- Generate TODO comments for follow-up
- Include performance considerations
```

### Dynamic Context Loading

Import different rules based on project phase:

```markdown
# CLAUDE.md

## Core Standards
@.claude/rules/typescript.md

## Phase-Specific
@.claude/rules/development.md
# @.claude/rules/production.md  # Uncomment when shipping
```

### Output Format Per Command Type

Create custom slash commands with format specifications:

```markdown
# .claude/commands/review.md

Review this code and respond in this exact format:

## Issues
- [Critical/Medium/Low]: Description

## Suggestions
- Description

## Approval Status
[Approved/Needs Changes]
```

Then use: `/review file.ts`

## Conclusion

Effective output format control in Claude Code requires understanding and leveraging multiple layers:

1. **Command-line flags** for session-level control
2. **Output styles** for communication mode
3. **CLAUDE.md files** for persistent project rules
4. **Modular rules** for context-specific standards
5. **Prompt techniques** for request-level precision
6. **Hooks** for deterministic formatting

Start with the most appropriate layer for your need:
- **One-off adjustments**: Use prompt techniques
- **Session preferences**: Use output styles
- **Project standards**: Use CLAUDE.md
- **File-type rules**: Use .claude/rules/ with path matchers
- **Code formatting**: Use hooks for deterministic results

The key to success is being explicit and specific. Claude Code responds far better to concrete constraints ("2 sentences") than vague requests ("be brief"). Document your standards clearly, provide examples, and iterate based on results.

With these techniques, you can shape Claude Code's output to match your exact requirements, whether you need terse technical responses, comprehensive educational content, or anything in between.

---

## Sources

- [Output styles - Claude Code Docs](https://code.claude.com/docs/en/output-styles)
- [Manage Claude's memory - Claude Code Docs](https://code.claude.com/docs/en/memory)
- [Claude Code: Best practices for agentic coding - Anthropic](https://www.anthropic.com/engineering/claude-code-best-practices)
- [What is CLAUDE.md in Claude Code - ClaudeLog](https://claudelog.com/faqs/what-is-claude-md/)
- [Using CLAUDE.MD files: Customizing Claude Code for your codebase](https://claude.com/blog/using-claude-md-files)
- [Prompting best practices - Claude Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-4-best-practices)
- [12 prompt engineering tips to boost Claude's output quality](https://www.vellum.ai/blog/prompt-engineering-tips-for-claude)
- [How I use Claude Code (+ my best tips)](https://www.builder.io/blog/claude-code)
- [Claude Code Output Styles: Explanatory, Learning, and Custom Options](https://williamcallahan.com/blog/claude-code-output-styles-learning-custom-options)
- [Modular Rules in Claude Code: Organizing Project Instructions](https://claude-blog.setec.rs/blog/claude-code-rules-directory)
- [Comment Directives for Claude Code](https://giuseppegurgone.com/comment-directives-claude-code)
- [What is --output-format in Claude Code - ClaudeLog](https://claudelog.com/faqs/what-is-output-format-in-claude-code/)
- [New Claude Models Default to Full Code Output](https://eval.16x.engineer/blog/claude-4-models-full-code-output)
