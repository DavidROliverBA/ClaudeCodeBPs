# CLAUDE.md Files: The Comprehensive Guide to Context Engineering in Claude Code

## Table of Contents
- [What is CLAUDE.md?](#what-is-claudemd)
- [Hierarchical Loading System](#hierarchical-loading-system)
- [What to Include in CLAUDE.md](#what-to-include-in-claudemd)
- [Best Practices for Writing Effective CLAUDE.md Files](#best-practices-for-writing-effective-claudemd-files)
- [Real-World Examples and Templates](#real-world-examples-and-templates)
- [How Claude Uses CLAUDE.md Context](#how-claude-uses-claudemd-context)
- [Maintaining and Updating CLAUDE.md Files](#maintaining-and-updating-claudemd-files)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)
- [Quick Start Template](#quick-start-template)

---

## What is CLAUDE.md?

CLAUDE.md is a special configuration file that Claude Code automatically loads into context at the start of every conversation. Think of it as Claude's "onboarding document" for your codebase—a persistent memory system that eliminates the need to repeatedly explain project-specific information.

**Key Characteristics:**

- **Automatic Loading**: Claude reads CLAUDE.md files without being prompted, making them part of the system prompt for every session
- **Persistent Context**: Unlike conversation history which resets, CLAUDE.md provides consistent context across all sessions
- **Team-Shareable**: When committed to git, these files share institutional knowledge with your entire development team
- **Hierarchical**: Multiple CLAUDE.md files can coexist at different directory levels, creating layered context

As Anthropic's engineering team notes, CLAUDE.md is "an ideal place for documenting repository etiquette, developer environment setup, and other project-specific details" that would otherwise require manual explanation in each session.

---

## Hierarchical Loading System

Claude Code implements a sophisticated four-tier memory hierarchy that loads context in a specific order, with higher-level context providing the foundation for more specific instructions.

### The Four Memory Tiers

1. **Enterprise Policy** (Highest Priority)
   - Organization-wide instructions at the system level
   - Typically managed by enterprise administrators
   - Applies across all projects and users

2. **User Memory** (`~/.claude/CLAUDE.md`)
   - Personal preferences and conventions
   - Located in your home directory: `~/.claude/CLAUDE.md`
   - Applies to ALL Claude Code sessions you run
   - Perfect for your personal coding style and commonly used tools

3. **Project Memory** (Project Root)
   - Team-shared instructions in `./CLAUDE.md` or `./.claude/CLAUDE.md`
   - Should be committed to version control
   - Provides project-specific context for all team members

4. **Local/Modular Rules** (Lowest Priority, Most Specific)
   - `./CLAUDE.local.md` - Project-specific but gitignored (personal overrides)
   - `./.claude/rules/*.md` - Modular rule files with optional path scoping
   - Directory-specific CLAUDE.md files in subdirectories

### How Recursive Loading Works

Claude Code recursively searches from your current working directory upward (but not including the filesystem root), loading any CLAUDE.md or CLAUDE.local.md files it encounters:

```
~/.claude/CLAUDE.md                 ← Loaded first (user level)
/project/CLAUDE.md                  ← Loaded second (project root)
/project/backend/CLAUDE.md          ← Loaded third (if cwd is in backend/)
/project/backend/api/CLAUDE.md      ← Loaded fourth (if cwd is in api/)
```

**Important**: Files higher in the hierarchy provide foundational context, while lower files add increasingly specific detail. This allows you to establish general principles at the project level and override or extend them in subdirectories.

### On-Demand Loading for Subtrees

Claude also discovers CLAUDE.md files in subdirectories beneath your current working directory, but loads them only when Claude actually reads files from those locations. This prevents context bloat while ensuring relevant information is available when needed.

### Import Capability

CLAUDE.md files support importing external files using the `@path/to/file` syntax:

```markdown
# Project Context

@docs/architecture.md
@docs/coding-standards.md

# Local Guidelines

- Use TypeScript strict mode
- All functions must have JSDoc comments
```

The import system supports up to 5 levels of recursive imports, enabling flexible knowledge organization across large projects.

---

## What to Include in CLAUDE.md

The most effective CLAUDE.md files follow the **WHAT, WHY, HOW** framework:

### WHAT: Project Structure and Technology

**Tech Stack:**
```markdown
# Technology Stack

- **Language**: TypeScript 5.3+
- **Runtime**: Node.js 20.x
- **Framework**: Next.js 14 (App Router)
- **Database**: PostgreSQL 15 + Prisma ORM
- **Testing**: Vitest + React Testing Library
- **Styling**: Tailwind CSS + shadcn/ui components
```

**Codebase Organization:**
```markdown
# Project Structure

- `app/` - Next.js App Router pages and layouts
- `components/` - Reusable React components
  - `ui/` - Base UI components (shadcn/ui)
  - `features/` - Feature-specific components
- `lib/` - Utility functions and shared logic
- `prisma/` - Database schema and migrations
- `tests/` - Test files (co-located with source files)
```

### WHY: Project Purpose and Context

```markdown
# Project Overview

This is a SaaS platform for managing customer support tickets. The system handles:
- Multi-tenant organization management
- Real-time ticket updates via WebSockets
- AI-powered ticket categorization and routing
- Integration with Slack, Discord, and email

**Key Design Decisions:**
- We use server components by default; client components are explicitly marked
- All database queries go through the Prisma client (no raw SQL)
- Authentication is handled via NextAuth.js with JWT tokens
```

### HOW: Development Workflow and Commands

**Essential Commands:**
```markdown
# Development Commands

- `pnpm dev` - Start development server (localhost:3000)
- `pnpm build` - Build for production
- `pnpm test` - Run all tests
- `pnpm test:watch` - Run tests in watch mode
- `pnpm lint` - Run ESLint
- `pnpm typecheck` - Run TypeScript compiler checks
- `pnpm db:push` - Push schema changes to database
- `pnpm db:studio` - Open Prisma Studio GUI

**Before committing:**
1. Run `pnpm typecheck` to verify TypeScript
2. Run `pnpm lint` to check code style
3. Run `pnpm test` to ensure all tests pass
```

**Git Workflow:**
```markdown
# Git Conventions

- **Branches**: `feature/description`, `fix/description`, `docs/description`
- **Commits**: Follow Conventional Commits (feat:, fix:, docs:, etc.)
- **PRs**: Require 1 approval, all checks must pass
- **Main branch**: Protected, requires PR for all changes
```

### Code Style and Conventions

```markdown
# Code Style Guidelines

**Imports:**
- Group imports: external libraries, then internal modules, then relative imports
- Use named exports (avoid default exports except for Next.js pages)
- Import types separately: `import type { User } from '@/types'`

**Naming Conventions:**
- Components: PascalCase (`UserProfile.tsx`)
- Functions/variables: camelCase (`getUserById`)
- Constants: UPPER_SNAKE_CASE (`MAX_RETRY_ATTEMPTS`)
- Files: kebab-case for utilities (`format-date.ts`)

**TypeScript:**
- Enable strict mode
- Avoid `any` - use `unknown` if type is truly unknown
- All function parameters and return types must be typed
- Prefer interfaces over types for object shapes

**React Patterns:**
- Use Server Components by default
- Mark Client Components with `'use client'` directive
- Prefer composition over prop drilling
- Extract repeated JSX into components (DRY principle)
```

### Testing Requirements

```markdown
# Testing Strategy

**Coverage Requirements:**
- Unit tests: All utility functions and hooks
- Integration tests: All API routes
- E2E tests: Critical user flows (signup, ticket creation, payment)
- Minimum coverage: 80% (enforced in CI)

**Test Naming:**
- Describe what the code should do, not implementation details
- Format: `it('should [expected behavior] when [condition]')`
- Group related tests with `describe()` blocks

**Example:**
```typescript
describe('formatDate()', () => {
  it('should format ISO date to MM/DD/YYYY', () => {
    expect(formatDate('2024-01-15')).toBe('01/15/2024');
  });

  it('should handle invalid dates gracefully', () => {
    expect(formatDate('invalid')).toBe('Invalid Date');
  });
});
```
```

### Environment and Configuration

```markdown
# Environment Setup

**Required Environment Variables:**
- `DATABASE_URL` - PostgreSQL connection string
- `NEXTAUTH_SECRET` - NextAuth.js secret (generate with `openssl rand -base64 32`)
- `NEXTAUTH_URL` - Application URL (http://localhost:3000 in dev)
- `OPENAI_API_KEY` - OpenAI API key for AI features

**First-Time Setup:**
1. Copy `.env.example` to `.env.local`
2. Fill in required environment variables
3. Run `pnpm install` to install dependencies
4. Run `pnpm db:push` to initialize database
5. Run `pnpm dev` to start development server
```

---

## Best Practices for Writing Effective CLAUDE.md Files

### 1. Keep It Concise (The Golden Rule)

Research indicates that frontier LLMs can follow approximately 150-200 instructions with reasonable consistency. Since Claude Code's system prompt already contains around 50 instructions, **your CLAUDE.md should ideally be fewer than 300 lines**.

**Why conciseness matters:**
- Every line consumes tokens from your context window
- Verbose files introduce noise that reduces instruction-following quality
- Your actual code needs most of the available context
- Focused documents outperform lengthy ones in practice

**Do:**
- Use short, declarative bullet points
- Trim redundancy (if a folder is named `components`, don't explain it contains components)
- Include only rules Claude needs to know to do the work

**Don't:**
- Write long narrative paragraphs
- Include commentary or nice-to-have information
- Duplicate information that's obvious from code structure

### 2. Universal Applicability Over Specificity

Include only universally applicable instructions that will be relevant across most sessions. Avoid task-specific guidance that won't apply broadly.

**Example - Good (Universal):**
```markdown
- All API routes must validate input with Zod schemas
- Database queries require error handling with try-catch
- User-facing errors must never expose internal details
```

**Example - Bad (Too Specific):**
```markdown
- When implementing the user dashboard, fetch data from /api/users/dashboard
- The dashboard should display total tickets, resolved tickets, and average resolution time
- Use the DashboardLayout component for consistent styling
```

The second example is too specific to a particular feature. This information belongs in a task-specific document or slash command, not in CLAUDE.md.

### 3. Never Send an LLM to Do a Linter's Job

One of the most important principles: **avoid style guidelines that deterministic tools can enforce**.

**Don't include in CLAUDE.md:**
- Indentation rules (use Prettier/ESLint)
- Import sorting (use eslint-plugin-import)
- Formatting preferences (use code formatters)
- Syntax preferences that linters catch

**Why?** LLMs are comparably expensive and incredibly slow for formatting tasks. Use pre-commit hooks, linters, and formatters to handle these automatically.

**Do include:**
- Architectural patterns linters can't enforce
- Domain-specific conventions
- Workflow and process guidelines
- Testing philosophy and requirements

### 4. Progressive Disclosure

Don't try to fit all project knowledge into CLAUDE.md. Instead, use it as a high-level guide that points to more detailed documentation.

**Structure:**
```
.claude/
├── CLAUDE.md                    # 150-200 lines: Overview and pointers
└── docs/
    ├── architecture.md          # Detailed architecture documentation
    ├── api-patterns.md          # API design patterns
    ├── database-schema.md       # Database design and migrations
    └── deployment.md            # Deployment procedures
```

**In CLAUDE.md, reference these files:**
```markdown
# Additional Documentation

For detailed information, see:
- Architecture: @.claude/docs/architecture.md
- API Patterns: @.claude/docs/api-patterns.md
- Database Schema: @.claude/docs/database-schema.md

Claude will reference these as needed, but they're not loaded by default.
```

This approach:
- Keeps CLAUDE.md focused and concise
- Prevents context bloat
- Allows detailed documentation where it's useful
- Uses imports only when relevant

### 5. Use Modular Organization

Break your CLAUDE.md into clear sections with markdown headers. This prevents "instruction bleeding" where guidelines from one area affect another.

**Good structure:**
```markdown
# Project Overview
[Brief description]

# Technology Stack
[Languages and frameworks]

# Code Conventions
## TypeScript
[TypeScript-specific rules]

## React
[React-specific patterns]

## Testing
[Testing guidelines]

# Development Workflow
[Git, PR, deployment processes]

# Common Commands
[Frequently used commands]
```

This modular approach helps Claude understand context boundaries and apply the right rules to the right situations.

### 6. Treat CLAUDE.md Like a Prompt

Your CLAUDE.md becomes part of Claude's system prompt, so apply the same rigor you would to any frequently used prompt:

**Iterative refinement:**
- Start simple with basic context
- Monitor Claude's responses for misunderstandings
- Add clarifications when Claude makes wrong assumptions
- Remove instructions that don't improve behavior

**Emphasize critical rules:**
```markdown
# Critical Rules

**IMPORTANT**: Never commit directly to the main branch
**YOU MUST**: Run `pnpm typecheck && pnpm test` before any commit
**NEVER**: Use `any` type in TypeScript - use `unknown` instead
```

Anthropic's team "occasionally runs CLAUDE.md files through the prompt improver and often tunes instructions (e.g., adding emphasis with 'IMPORTANT' or 'YOU MUST') to improve adherence."

### 7. Use the # Shortcut for Organic Growth

During Claude Code sessions, press the `#` key to give Claude an instruction that it will automatically incorporate into the relevant CLAUDE.md file. This enables organic documentation growth:

1. You discover a pattern while coding
2. Press `#` and describe it
3. Claude adds it to CLAUDE.md
4. Include the update in your next commit
5. Team members benefit from your discovery

This workflow creates a living document that grows smarter with each session.

### 8. Nouns in CLAUDE.md, Verbs in Slash Commands

A helpful mental model: use CLAUDE.md for "nouns" (what things are) and slash commands for "verbs" (how to do things).

**CLAUDE.md (Nouns):**
- Project overview and architecture
- Where things are located
- What technologies you use
- What standards you follow

**Slash Commands (Verbs):**
- How to run tests
- How to deploy to production
- How to generate a migration
- How to create a new component

Example slash command (`.claude/commands/new-component.md`):
```markdown
Create a new React component following our conventions:

1. Create file in appropriate directory (ui/ or features/)
2. Use TypeScript with proper types
3. Include JSDoc comment describing purpose
4. Export as named export
5. Create accompanying test file
6. Use existing UI components from shadcn/ui when possible

Component name: [user will specify]
```

---

## Real-World Examples and Templates

### Example 1: Next.js SaaS Application

```markdown
# Customer Support Platform

A multi-tenant SaaS platform for customer support ticket management.

## Tech Stack

- **Frontend**: Next.js 14 (App Router), React 18, TypeScript 5.3
- **Styling**: Tailwind CSS + shadcn/ui
- **Backend**: Next.js API Routes + tRPC
- **Database**: PostgreSQL 15 + Prisma ORM
- **Auth**: NextAuth.js v5
- **Real-time**: Pusher (WebSocket alternative)
- **Testing**: Vitest + Testing Library + Playwright

## Project Structure

- `app/` - Next.js App Router (pages, layouts, routes)
- `components/` - React components (ui/ and features/)
- `server/` - tRPC routers and procedures
- `lib/` - Shared utilities and helpers
- `prisma/` - Database schema and migrations
- `tests/` - Integration and E2E tests

## Code Conventions

**TypeScript:**
- Strict mode enabled
- No `any` types (use `unknown` or proper types)
- All functions must have return type annotations

**React:**
- Server Components by default
- Client Components marked with `'use client'`
- Use `cn()` utility for className merging
- Prefer composition over complex props

**Imports:**
- Use `@/` alias for root imports
- Group: external → internal → relative
- Type imports separate: `import type { ... }`

**File Naming:**
- Components: `PascalCase.tsx`
- Utilities: `kebab-case.ts`
- Test files: `*.test.ts` or `*.test.tsx`

## Development Workflow

**Common Commands:**
- `pnpm dev` - Development server (localhost:3000)
- `pnpm build` - Production build
- `pnpm test` - Run all tests
- `pnpm lint` - ESLint check
- `pnpm typecheck` - TypeScript check

**Before Committing:**
1. Run `pnpm typecheck` ✓
2. Run `pnpm lint` ✓
3. Run `pnpm test` ✓

**Git Conventions:**
- Branch: `feature/`, `fix/`, `docs/`
- Commits: Conventional Commits format
- PRs require 1 approval + passing checks

## Testing Requirements

- Unit tests for utilities and hooks
- Integration tests for API routes
- E2E tests for critical flows
- Minimum 80% coverage

## Environment Variables

Required in `.env.local`:
- `DATABASE_URL` - PostgreSQL connection
- `NEXTAUTH_SECRET` - Auth secret
- `NEXTAUTH_URL` - App URL
- `PUSHER_APP_ID`, `PUSHER_KEY`, `PUSHER_SECRET`

## Important Notes

- **NEVER** commit `.env.local` to git
- **ALWAYS** use Prisma for database queries (no raw SQL)
- **IMPORTANT**: Validate all user input with Zod schemas
- **SECURITY**: Never expose database errors to users
```

### Example 2: Python FastAPI Backend

```markdown
# Payment Processing API

RESTful API for processing payments, managing subscriptions, and handling webhooks.

## Tech Stack

- **Language**: Python 3.11+
- **Framework**: FastAPI
- **Database**: PostgreSQL + SQLAlchemy 2.0
- **Task Queue**: Celery + Redis
- **Testing**: pytest + httpx
- **Validation**: Pydantic v2
- **Migrations**: Alembic

## Project Structure

```
src/
├── api/           # API routes and endpoints
├── core/          # Core configuration and dependencies
├── models/        # SQLAlchemy models
├── schemas/       # Pydantic schemas
├── services/      # Business logic layer
├── tasks/         # Celery tasks
└── utils/         # Utility functions

tests/
├── unit/          # Unit tests
├── integration/   # Integration tests
└── conftest.py    # Pytest fixtures
```

## Code Conventions

**Python Style:**
- Follow PEP 8 (enforced by ruff)
- Type hints required on all functions
- Docstrings for public functions (Google style)
- Max line length: 100 characters

**Naming:**
- Functions/variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private attributes: `_leading_underscore`

**Import Order:**
1. Standard library
2. Third-party packages
3. Local application imports
4. Relative imports

**FastAPI Patterns:**
- Use dependency injection for database sessions
- All endpoints must have response models
- Use Pydantic models for validation
- HTTP exceptions for error handling

## Development Commands

- `poetry install` - Install dependencies
- `poetry run uvicorn src.main:app --reload` - Start dev server
- `poetry run pytest` - Run tests
- `poetry run pytest --cov` - Run tests with coverage
- `poetry run ruff check .` - Lint code
- `poetry run mypy .` - Type check

## Database

**Migrations:**
- `alembic revision --autogenerate -m "message"` - Create migration
- `alembic upgrade head` - Apply migrations
- `alembic downgrade -1` - Rollback one migration

**Important:**
- ALWAYS create migrations for schema changes
- NEVER edit migration files after they're merged
- Test migrations in both directions (up and down)

## Testing Strategy

**Required Tests:**
- Unit tests: All service layer functions
- Integration tests: All API endpoints
- Minimum coverage: 85%

**Test Structure:**
```python
def test_function_should_behavior_when_condition():
    # Arrange
    setup_data()

    # Act
    result = function_under_test()

    # Assert
    assert result == expected
```

## Environment Variables

Create `.env` file:
```
DATABASE_URL=postgresql://user:pass@localhost/dbname
REDIS_URL=redis://localhost:6379/0
SECRET_KEY=your-secret-key
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Critical Rules

**IMPORTANT**: All payment operations must be idempotent
**YOU MUST**: Verify webhook signatures before processing
**NEVER**: Log sensitive data (API keys, tokens, card numbers)
**SECURITY**: Use parameterized queries (no string concatenation)
```

### Example 3: Monorepo with Multiple Projects

```markdown
# E-Commerce Platform Monorepo

Full-stack e-commerce platform with web app, mobile app, and admin dashboard.

## Monorepo Structure

```
packages/
├── web/              # Next.js customer-facing web app
├── admin/            # React admin dashboard
├── mobile/           # React Native mobile app
├── api/              # Node.js API server
├── shared/           # Shared utilities and types
└── ui/               # Shared component library

apps/ and packages/ use pnpm workspaces
```

## Tech Stack (Shared)

- **Language**: TypeScript 5.3+
- **Package Manager**: pnpm (workspaces)
- **Build Tool**: Turbo
- **Testing**: Vitest
- **Linting**: ESLint + Prettier

## Project-Specific Tech

- **web/**: Next.js 14, Tailwind CSS
- **admin/**: Vite + React, Ant Design
- **mobile/**: React Native, Expo
- **api/**: Express.js, Prisma, PostgreSQL

## Development Commands

**Root Level:**
- `pnpm install` - Install all dependencies
- `pnpm dev` - Start all projects in dev mode
- `pnpm build` - Build all projects
- `pnpm test` - Run all tests
- `pnpm lint` - Lint all projects

**Package-Specific:**
- `pnpm --filter web dev` - Start web app only
- `pnpm --filter api test` - Test API only
- `pnpm --filter ui build` - Build UI library

## Code Conventions

**Shared Across All Packages:**
- TypeScript strict mode
- No `any` types
- Prettier for formatting (2-space indentation)
- ESLint for linting

**Import Internal Packages:**
```typescript
import { Button } from '@acme/ui';
import { formatCurrency } from '@acme/shared/utils';
import type { Product } from '@acme/shared/types';
```

## Git Workflow

- **Branches**: `feature/scope/description` (e.g., `feature/web/checkout-flow`)
- **Commits**: Conventional Commits with scope (e.g., `feat(api): add payment endpoint`)
- **Changesets**: Use `pnpm changeset` for versioning

## Testing Requirements

- Each package maintains 80%+ coverage
- Integration tests for API endpoints
- E2E tests for critical user flows in web/mobile

## Environment Setup

Each package has its own `.env.example`. Copy to `.env.local` and fill in:

**api/.env:**
- `DATABASE_URL`
- `JWT_SECRET`
- `STRIPE_SECRET_KEY`

**web/.env:**
- `NEXT_PUBLIC_API_URL`
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`

## Important Notes

- **ALWAYS** run commands from monorepo root (not package directories)
- **NEVER** import from one package to another using relative paths
- **USE** workspace protocol for inter-package dependencies
- **BUILD** shared packages before dependent packages
```

### Template Resources

For more real-world examples, explore these curated collections:

- **[awesome-claude-md](https://github.com/josix/awesome-claude-md)** - Curated CLAUDE.md files from top open-source projects
- **[claude-md-examples](https://github.com/ArthurClune/claude-md-examples)** - Sample files for different project types
- **[my-claude-code-setup](https://github.com/centminmod/my-claude-code-setup)** - Starter template configuration
- **[Claude-Flow Templates](https://github.com/ruvnet/claude-flow/wiki/CLAUDE-MD-Templates)** - Specialized templates for different use cases

---

## How Claude Uses CLAUDE.md Context

Understanding how Claude processes CLAUDE.md files helps you write more effective documentation.

### Context Loading Process

1. **Session Initialization**: When you start Claude Code, it reads all applicable CLAUDE.md files in hierarchical order
2. **System Prompt Integration**: The contents become part of Claude's system prompt for that session
3. **Persistent Throughout Session**: Once loaded, the context remains available for all subsequent interactions
4. **No Explicit Reference Needed**: Claude automatically considers CLAUDE.md content when responding

### Practical Implications

**Claude automatically considers CLAUDE.md when:**
- Generating code snippets
- Suggesting architectural approaches
- Choosing naming conventions
- Running tests or build commands
- Making git commits
- Answering questions about your codebase

**However**, research shows that while automatic loading works, explicitly reminding Claude can improve adherence:

```
"Review our CLAUDE.md conventions before implementing this feature"
"Check our testing guidelines in CLAUDE.md and verify this test follows them"
"Based on our CLAUDE.md standards, review this PR for compliance"
```

### Context Window Constraints

Claude has a finite context window (typically 200,000 tokens for Claude Opus 4.5). Your CLAUDE.md consumes part of this budget:

**Approximate token usage:**
- 100 lines of CLAUDE.md ≈ 500-800 tokens
- 200 lines of CLAUDE.md ≈ 1000-1600 tokens
- 300 lines of CLAUDE.md ≈ 1500-2400 tokens

This is why conciseness matters. A bloated 500-line CLAUDE.md could consume 3000-4000 tokens before you even start working, leaving less room for your actual code.

### Instruction Following Capacity

Research indicates frontier LLMs can consistently follow approximately 150-200 distinct instructions. Since Claude Code's base system prompt contains around 50 instructions, your CLAUDE.md should ideally add no more than 100-150 additional instructions.

**Counting instructions:**
- Each distinct rule or guideline = 1 instruction
- Lists of related items (e.g., file naming conventions) = 1-2 instructions
- Complex multi-step workflows = 3-5 instructions

### Progressive Context Loading

Claude Code also implements on-demand loading for subdirectory CLAUDE.md files:

1. Files in your current directory path are loaded at session start
2. Files in subdirectories are discovered but not loaded initially
3. When Claude reads a file from a subdirectory, its CLAUDE.md is loaded
4. This prevents context bloat while ensuring relevant information is available

**Example:**
```
/project/CLAUDE.md              ← Loaded at session start
/project/frontend/CLAUDE.md     ← Loaded when working with frontend code
/project/backend/api/CLAUDE.md  ← Loaded when working with API code
```

---

## Maintaining and Updating CLAUDE.md Files

CLAUDE.md is a living document that should evolve with your project. Here's how to keep it effective:

### 1. Establish a Review Cadence

**Monthly Reviews:**
- Check for outdated information
- Remove deprecated practices
- Add new conventions that emerged
- Verify accuracy of commands and file paths

**After Major Changes:**
- Update when migrating frameworks or tools
- Revise when team conventions change
- Refresh when onboarding new team members

### 2. Use the # Shortcut During Development

Press `#` during Claude Code sessions to add learnings in real-time:

**Workflow:**
1. Encounter a pattern worth documenting
2. Press `#`
3. Describe the rule or convention
4. Claude adds it to the appropriate CLAUDE.md
5. Review the addition
6. Include in your next commit

This creates a feedback loop where CLAUDE.md grows smarter with each coding session.

### 3. Monitor Claude's Behavior for Gaps

Pay attention to when Claude:
- Repeatedly asks for the same information → Add it to CLAUDE.md
- Makes wrong assumptions → Add clarification
- Ignores a convention → Emphasize with IMPORTANT or YOU MUST
- Suggests incorrect approaches → Document the correct pattern

### 4. Collaborative Refinement

Treat CLAUDE.md as a team responsibility:

**In Pull Requests:**
- Include CLAUDE.md updates when introducing new patterns
- Request CLAUDE.md changes if conventions were missed
- Discuss controversial guidelines before merging

**In Team Meetings:**
- Review effectiveness of current guidelines
- Propose new conventions based on recurring issues
- Deprecate rules that no longer apply

### 5. Use Version Control Effectively

**Commit Messages:**
```
docs(claude): add TypeScript strict mode requirement
docs(claude): update API testing guidelines
docs(claude): remove deprecated React class component patterns
```

**Review History:**
```bash
git log -p CLAUDE.md  # See all changes to CLAUDE.md
git blame CLAUDE.md   # See who added each line
```

This helps understand the evolution of your team's conventions.

### 6. A/B Test Instructions

When Claude isn't following a guideline:

**Try different phrasings:**
```markdown
# Version A
- Use ES modules syntax for imports

# Version B
**IMPORTANT**: Always use ES modules (import/export) syntax, never CommonJS (require)

# Version C
**YOU MUST** use ES modules syntax:
- ✓ Correct: `import { foo } from 'bar'`
- ✗ Wrong: `const { foo } = require('bar')`
```

Test which version Claude follows most consistently, then use that phrasing.

### 7. Measure Effectiveness

Track metrics to validate your CLAUDE.md's impact:

**Qualitative Measures:**
- Fewer repeated explanations needed
- Faster onboarding for new team members
- More consistent code style across Claude-generated code
- Reduced PR review comments about conventions

**Quantitative Measures:**
- Time to complete common tasks
- Number of failed checks in CI
- PR approval time
- Code review comments per PR

### 8. Periodic Compression

As CLAUDE.md grows, periodically compress it:

**Techniques:**
- Combine related bullets
- Remove redundant examples
- Move detailed docs to separate files with @ imports
- Convert verbose explanations to concise rules

**Before compression (verbose):**
```markdown
## Testing

When writing tests, you should always use descriptive test names. The test name should explain what the test does, what conditions it tests, and what the expected outcome is. This makes it easier for other developers to understand what each test is checking without reading the implementation.

For example, a good test name would be "should return 404 when user does not exist" rather than just "test user lookup". The first one clearly states the expected behavior, the condition being tested, and the expected result.
```

**After compression (concise):**
```markdown
## Testing

- Test names: `should [behavior] when [condition]`
- Example: `should return 404 when user does not exist`
```

### 9. Sunset Outdated Information

Don't let CLAUDE.md become a historical archive. Remove information that no longer applies:

**Examples of what to remove:**
- Deprecated frameworks or tools
- Old team members' personal preferences
- Temporary workarounds that have been fixed
- Experimental features that were abandoned

**Keep a changelog if needed:**
```markdown
# CLAUDE.md Changelog

## 2024-02-15
- Removed Webpack configuration (migrated to Vite)
- Updated testing framework from Jest to Vitest
- Added new API authentication patterns

## 2024-01-10
- Initial CLAUDE.md creation
```

### 10. Use Prompt Improvers

Anthropic's team "occasionally runs CLAUDE.md files through the prompt improver" to optimize instruction-following. You can do this too:

**Prompt to Claude:**
```
Please review this CLAUDE.md file and suggest improvements for clarity, conciseness, and instruction-following. Focus on:

1. Removing redundancy
2. Strengthening critical rules
3. Improving structure and organization
4. Ensuring consistent formatting

[Paste your CLAUDE.md content]
```

Then apply Claude's suggestions and test the results.

---

## Common Mistakes to Avoid

### 1. Over-Documentation

**Mistake**: Trying to document everything about your project in CLAUDE.md.

**Problem**: Creates context bloat, reduces instruction-following, consumes token budget.

**Solution**: Use progressive disclosure. Keep CLAUDE.md concise and point to detailed docs.

### 2. Using CLAUDE.md for Linting Rules

**Mistake**: Including formatting and style rules that linters can enforce.

```markdown
❌ Bad:
- Use 2-space indentation
- Add semicolons at end of statements
- Use single quotes for strings
- Sort imports alphabetically
```

**Problem**: LLMs are expensive and slow for deterministic tasks.

**Solution**: Configure ESLint, Prettier, or other linters. Let them handle formatting.

```markdown
✓ Good:
- Code style enforced by Prettier (see .prettierrc)
- Linting rules in .eslintrc.json
- Run `pnpm lint:fix` before committing
```

### 3. Auto-Generating with /init and Never Refining

**Mistake**: Running `/init` to generate CLAUDE.md and treating it as final.

**Problem**: Auto-generated files capture obvious patterns but miss nuances and team-specific practices.

**Solution**: Use `/init` as a starting point only. Manually refine based on your team's actual workflow.

### 4. Including Task-Specific Instructions

**Mistake**: Adding instructions for specific features or one-off tasks.

```markdown
❌ Bad:
- When implementing the user dashboard, fetch from /api/users/dashboard
- The dashboard should show total tickets, resolved tickets, and average time
- Use the DashboardCard component for each metric
```

**Problem**: These instructions only apply to one task and waste context budget.

**Solution**: Put task-specific instructions in slash commands or issue descriptions, not CLAUDE.md.

### 5. Forgetting to Update After Major Changes

**Mistake**: Migrating from Jest to Vitest but leaving Jest commands in CLAUDE.md.

**Problem**: Claude follows outdated instructions, causing errors and confusion.

**Solution**: Update CLAUDE.md as part of migration PRs. Make it a checklist item.

### 6. No Emphasis on Critical Rules

**Mistake**: Treating all instructions equally.

```markdown
❌ Weak:
- Don't commit to main branch
- Run tests before committing
- Use TypeScript strict mode
```

**Problem**: Claude may not distinguish between critical and nice-to-have rules.

**Solution**: Emphasize critical rules with IMPORTANT, YOU MUST, NEVER.

```markdown
✓ Strong:
**NEVER** commit directly to main branch
**YOU MUST** run `pnpm test` before committing
**IMPORTANT**: TypeScript strict mode is required
```

### 7. Verbose, Narrative Style

**Mistake**: Writing CLAUDE.md like documentation prose.

```markdown
❌ Verbose:
In our project, we have decided to use TypeScript because it provides type safety and helps us catch errors at compile time rather than runtime. When writing TypeScript code, please make sure to always include type annotations on function parameters and return types. This makes the code more maintainable and helps other developers understand what types are expected.
```

**Problem**: Consumes tokens, buries key information, harder for Claude to parse.

**Solution**: Use concise, bullet-point style.

```markdown
✓ Concise:
**TypeScript:**
- Provides type safety and compile-time error checking
- All function parameters and return types must be typed
- Improves maintainability and developer understanding
```

### 8. Not Sharing with the Team

**Mistake**: Keeping CLAUDE.md in CLAUDE.local.md (gitignored).

**Problem**: Team members don't benefit from documented conventions.

**Solution**: Commit CLAUDE.md to version control. Use CLAUDE.local.md only for personal overrides.

### 9. Duplicating Information Across Files

**Mistake**: Having the same conventions in multiple CLAUDE.md files in the hierarchy.

**Problem**: Inconsistency when one is updated but others aren't.

**Solution**: Put shared conventions in the highest-level CLAUDE.md. Use subdirectory files only for truly local context.

**Structure:**
```
/project/CLAUDE.md                # Shared TypeScript, testing, Git conventions
/project/frontend/CLAUDE.md       # Frontend-specific: React patterns, styling
/project/backend/CLAUDE.md        # Backend-specific: API patterns, database
```

### 10. No Feedback Loop

**Mistake**: Writing CLAUDE.md once and never measuring its effectiveness.

**Problem**: Can't tell if the file is helping or hurting.

**Solution**: Monitor Claude's behavior, collect team feedback, iterate based on results.

---

## Quick Start Template

Use this template to create your first CLAUDE.md:

```markdown
# [Project Name]

[One sentence describing what this project does]

## Tech Stack

- **Language**:
- **Framework**:
- **Database**:
- **Testing**:
- **Other Tools**:

## Project Structure

- `[directory]/` - [purpose]
- `[directory]/` - [purpose]
- `[directory]/` - [purpose]

## Code Conventions

**[Language] Style:**
- [Convention 1]
- [Convention 2]
- [Convention 3]

**File Naming:**
- [Pattern]: [Example]
- [Pattern]: [Example]

**Import Style:**
- [Guideline]

## Development Commands

- `[command]` - [description]
- `[command]` - [description]
- `[command]` - [description]

**Before Committing:**
1. Run `[command]` ✓
2. Run `[command]` ✓

## Git Workflow

- **Branches**: [pattern]
- **Commits**: [format]
- **PRs**: [requirements]

## Testing Requirements

- [Type of tests required]
- [Coverage requirement]
- [Test naming convention]

## Environment Setup

Required environment variables:
- `[VAR_NAME]` - [description]
- `[VAR_NAME]` - [description]

## Important Notes

**IMPORTANT**: [Critical rule]
**YOU MUST**: [Critical rule]
**NEVER**: [Critical rule]
```

### Usage

1. Copy the template above
2. Create `CLAUDE.md` in your project root
3. Fill in sections relevant to your project
4. Remove sections that don't apply
5. Keep it under 200 lines
6. Commit to version control
7. Iterate based on effectiveness

---

## Conclusion

CLAUDE.md files are the foundation of effective context engineering in Claude Code. When used properly, they:

- Eliminate repetitive explanations of project context
- Ensure consistent adherence to team conventions
- Accelerate onboarding for new team members
- Improve the quality of Claude-generated code
- Create a shared knowledge base that grows smarter over time

**Remember the key principles:**

1. **Conciseness**: Keep it under 300 lines, ideally under 200
2. **Universal Applicability**: Include only rules that apply broadly
3. **Progressive Disclosure**: Point to detailed docs rather than including everything
4. **Emphasis**: Use IMPORTANT, YOU MUST, NEVER for critical rules
5. **Living Document**: Update regularly based on team learnings
6. **Measured Iteration**: Test what works and refine accordingly

Start simple, measure effectiveness, and iterate. A well-crafted CLAUDE.md is one of the highest-leverage investments you can make in your development workflow.

---

## Additional Resources

### Official Documentation
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Anthropic's official guide
- [Claude Code Memory Documentation](https://code.claude.com/docs/en/memory) - Official memory system docs
- [Claude Code GitHub Repository](https://github.com/anthropics/claude-code) - Official repo

### Community Resources
- [awesome-claude-md](https://github.com/josix/awesome-claude-md) - Curated examples from top projects
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - Commands, hooks, and workflows
- [claude-md-examples](https://github.com/ArthurClune/claude-md-examples) - Sample files
- [my-claude-code-setup](https://github.com/centminmod/my-claude-code-setup) - Starter templates

### In-Depth Guides
- [Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) - HumanLayer's detailed guide
- [Maximising Claude Code](https://www.maxitect.blog/posts/maximising-claude-code-building-an-effective-claudemd) - Building effective files
- [Claude Code: The Complete Guide](https://www.siddharthbharath.com/claude-code-the-complete-guide/) - Comprehensive tutorial
- [Context Engineering for Claude Code](https://thomaslandgraf.substack.com/p/context-engineering-for-claude-code) - Advanced techniques

---

## Sources

This guide was compiled from extensive research of official documentation, community guides, and best practices articles:

- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Manage Claude's memory - Claude Code Docs](https://code.claude.com/docs/en/memory)
- [Using CLAUDE.MD files: Customizing Claude Code for your codebase](https://claude.com/blog/using-claude-md-files)
- [Writing a good CLAUDE.md | HumanLayer Blog](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [Maximising Claude Code: Building an Effective CLAUDE.md](https://www.maxitect.blog/posts/maximising-claude-code-building-an-effective-claudemd)
- [What is CLAUDE.md in Claude Code | ClaudeLog](https://claudelog.com/faqs/what-is-claude-md/)
- [GitHub - josix/awesome-claude-md](https://github.com/josix/awesome-claude-md)
- [GitHub - hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- [GitHub - ArthurClune/claude-md-examples](https://github.com/ArthurClune/claude-md-examples)
- [CLAUDE MD Templates - Claude-Flow Wiki](https://github.com/ruvnet/claude-flow/wiki/CLAUDE-MD-Templates)
- [Mastering the Vibe: Claude Code Best Practices That Actually Work](https://dinanjana.medium.com/mastering-the-vibe-claude-code-best-practices-that-actually-work-823371daf64c)
- [Cooking with Claude Code: The Complete Guide](https://www.siddharthbharath.com/claude-code-the-complete-guide/)
- [Context Engineering for Claude Code: Mastering Deep Technical Knowledge](https://thomaslandgraf.substack.com/p/context-engineering-for-claude-code)
- [What's a Claude.md File? 5 Best Practices](https://apidog.com/blog/claude-md/)
- [How I Use Every Claude Code Feature](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)

---

*Last Updated: January 2026*
