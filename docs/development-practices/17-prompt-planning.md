# Comprehensive Guide to Prompt Planning with Claude Code

## Introduction

Prompt planning is a critical strategy for maximizing the effectiveness of Claude Code, Anthropic's agentic coding assistant. By creating structured planning documents before implementation, you enable Claude to better understand project requirements, maintain context across sessions, and deliver more accurate results. This guide covers best practices, workflows, and templates for documentation-first development with Claude Code.

## 1. Using spec.md Files for Project Specifications

### What is spec.md?

A `spec.md` file is a project specification document that defines requirements, technical approach, and implementation details for a feature or project. It serves as the single source of truth that guides Claude's implementation work.

### Key Components of spec.md

A well-structured spec.md should include:

- **Problem Statement**: Current behavior, expected behavior, and impact
- **Requirements**: Functional and non-functional requirements
- **Technical Approach**: High-level solution architecture
- **Tech Stack**: Technologies, frameworks, and dependencies
- **Design Guidelines**: UI/UX considerations, coding standards
- **Milestones**: Up to 3-5 major checkpoints (avoid over-specification)
- **Success Criteria**: Measurable outcomes that define completion

### Spec.md Template

```markdown
# [Feature/Project Name]

## Problem Statement

**Current Behavior:**
[Describe what currently exists or doesn't work]

**Expected Behavior:**
[Describe the desired outcome]

**Impact:**
[Why this matters - business value, user benefit, technical debt reduction]

## Requirements

### Functional Requirements
- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

### Non-Functional Requirements
- [ ] Performance criteria
- [ ] Security considerations
- [ ] Accessibility standards

## Technical Approach

[High-level description of the solution architecture]

### Tech Stack
- **Framework**: [e.g., React, Next.js]
- **Backend**: [e.g., Node.js, Python]
- **Database**: [e.g., PostgreSQL, MongoDB]
- **Key Dependencies**:
  - [Dependency 1]: [When and why to use it]
  - [Dependency 2]: [When and why to use it]

### Design Guidelines
- Code style conventions
- Component structure patterns
- Testing requirements

## Implementation Milestones

### Milestone 1: [Name]
- Deliverables
- Success criteria

### Milestone 2: [Name]
- Deliverables
- Success criteria

### Milestone 3: [Name]
- Deliverables
- Success criteria

## Files to Modify/Create
- `path/to/file1.ts` - [Purpose]
- `path/to/file2.ts` - [Purpose]

## Testing Strategy
- Unit tests for [components/functions]
- Integration tests for [workflows]
- E2E tests for [user flows]

## Success Metrics
- [ ] All tests pass
- [ ] Code review approved
- [ ] Documentation updated
- [ ] Performance benchmarks met
```

### Best Practices for spec.md

1. **Be explicit and detailed**: Don't shortcut—describe exactly what you want and how you want it to work
2. **Save to project root**: Store `spec.md` in the root directory for easy access
3. **Use the best reasoning model**: Generate specs using Claude Opus 4.5 or Sonnet 4.5 with extended thinking
4. **Iterate on the spec**: Review and refine before implementation begins
5. **Keep it focused**: Include only universally applicable information; avoid bloat

## 2. Creating prompt_plan.md for Multi-Step Implementations

### What is prompt_plan.md?

A `prompt_plan.md` file breaks down complex implementations into sequential, manageable prompts. It serves as a roadmap for executing multi-phase development work systematically.

### Structure of prompt_plan.md

```markdown
# Implementation Plan: [Project Name]

## Overview
[Brief description of the overall implementation strategy]

## Phase Breakdown

### Phase 1: Research & Discovery
**Objective:** Understand current state and gather requirements

**Prompts:**
1. "Analyze the current codebase architecture focusing on [area]"
2. "Research best practices for [technology/pattern]"
3. "Document findings in RESEARCH.md"

**Expected Outputs:**
- RESEARCH.md with findings
- Architecture diagram or notes
- List of dependencies

**Approval Gate:** Review research before proceeding

---

### Phase 2: Design & Planning
**Objective:** Create technical design based on research

**Prompts:**
1. "Design the interface contracts for [component/module]"
2. "Create data models for [entities]"
3. "Document the design in DESIGN.md"

**Expected Outputs:**
- Interface definitions
- Data schemas
- Design document

**Approval Gate:** Review and approve design

---

### Phase 3: Test-Driven Development
**Objective:** Write tests before implementation

**Prompts:**
1. "Write unit tests for [component] based on these input/output pairs: [examples]"
2. "Create integration tests for [workflow]"
3. "Run tests and confirm they fail before implementation"

**Expected Outputs:**
- Test files with failing tests
- Test coverage report

**Approval Gate:** Verify tests fail as expected

---

### Phase 4: Core Implementation
**Objective:** Implement functionality to pass tests

**Prompts:**
1. "Implement [component] to satisfy the test requirements"
2. "Add error handling and edge cases"
3. "Run tests and verify they pass"

**Expected Outputs:**
- Implementation code
- All tests passing
- Code following style guidelines

**Approval Gate:** Code review and test verification

---

### Phase 5: Integration & Refinement
**Objective:** Integrate components and polish

**Prompts:**
1. "Integrate [components] into the main application"
2. "Add logging and monitoring"
3. "Update documentation and README"

**Expected Outputs:**
- Integrated, working feature
- Updated documentation
- Deployment-ready code

**Approval Gate:** Final review and acceptance

## Checkpoints

After each phase:
- [ ] Review outputs
- [ ] Commit progress to Git
- [ ] Update progress in PROGRESS.md
- [ ] Get approval before next phase

## Rollback Points

- Git commit after Phase 2 approval
- Git commit after Phase 4 tests pass
- Git commit after Phase 5 integration

## Notes

- Use Plan Mode (Shift+Tab) for read-only exploration
- Keep one task in_progress at a time
- Document blockers immediately
- Update this plan as discoveries emerge
```

### Multi-Prompt Chain Approach

The recommended pattern for complex implementations:

1. **Research best practices** → Save to RESEARCH.md
2. **Design strategy** → Await approval
3. **Write interfaces** → Review types
4. **Implement core logic** → TDD approach
5. **Add production hardening** → Generate tests

Each prompt builds on previous outputs, with no prompt starting until the previous is approved.

## 3. Breaking Complex Tasks into Phases

### Why Phase Breakdown Matters

Claude Code performs best when working incrementally on clearly defined tasks. Breaking complex work into phases:

- Prevents jumping straight to coding without planning
- Enables better state tracking across sessions
- Reduces hallucination drift on long tasks
- Creates natural checkpoints for review and rollback

### Phase Structure Best Practices

**1. Start Broad, Then Narrow**

```
Phase 1: Exploration (broad understanding)
Phase 2: Deep Dive (specific area focus)
Phase 3: Implementation (targeted changes)
Phase 4: Verification (testing and validation)
```

**2. Use Small, Testable Increments**

Each phase should produce verifiable output that can be tested independently.

**3. Define Clear Phase Boundaries**

Each phase should have:
- Clear objective
- Specific deliverables
- Success criteria
- Approval gate

### Example: E-Commerce Feature Phases

```markdown
## Phase 1: User Registration System
- Database models
- API endpoints
- Frontend forms
- Authentication integration
**Checkpoint:** User can register and login

## Phase 2: Product Catalog
- Product models and schema
- Admin CRUD interface
- Public product listing
- Search and filtering
**Checkpoint:** Products can be managed and viewed

## Phase 3: Shopping Cart
- Cart state management
- Add/remove/update items
- Persistent cart storage
- Cart UI components
**Checkpoint:** Users can manage cart

## Phase 4: Checkout Flow
- Payment integration
- Order processing
- Confirmation emails
- Order history
**Checkpoint:** Complete purchase flow works
```

## 4. Documentation-First Development

### The Documentation-First Philosophy

Documentation-first development means creating specification and planning documents before writing code. This approach:

- Forces clarity of requirements before implementation
- Provides Claude with better context for generating code
- Creates reviewable artifacts at each stage
- Maintains project history and decision rationale

### Key Documentation Files

**1. CLAUDE.md (Project Context)**

Your `CLAUDE.md` file becomes part of Claude's system prompt in every conversation. It should contain:

```markdown
# [Project Name]

## Overview
[Brief description of the project]

## Architecture
[High-level architecture patterns]

## Tech Stack
- Language: [Primary language]
- Framework: [Main framework]
- Database: [Database system]
- Key Libraries: [Important dependencies]

## Development Guidelines

### Code Style
- Use [style guide]
- Follow [naming conventions]
- Organize code as [structure pattern]

### Testing Requirements
- Unit tests required for [components]
- Integration tests for [workflows]
- Test command: `[test command]`

### Common Commands
```bash
# Development server
[dev command]

# Run tests
[test command]

# Build production
[build command]

# Linting
[lint command]
```

### Workflows

**Feature Development:**
1. Create spec.md with requirements
2. Use Plan Mode to create implementation plan
3. Implement with TDD approach
4. Run tests and build
5. Update documentation

**Bug Fixes:**
1. Document issue in bug report
2. Write failing test reproducing bug
3. Fix implementation
4. Verify test passes
5. Update changelog

## Project-Specific Notes
- [Important warnings or gotchas]
- [Environment-specific behaviors]
- [Team conventions]
```

**Best Practices for CLAUDE.md:**
- Keep it focused—only universally applicable information
- Refine it like any frequently used prompt
- Don't bloat it—LLMs perform better with relevant context
- Update it as project evolves

**2. constitution.md (Project Principles)**

For spec-driven development, create a `constitution.md` that establishes non-negotiable principles:

```markdown
# Project Constitution

## Core Principles

1. **Code Quality**
   - All code must have tests
   - Code coverage minimum: 80%
   - No commits without passing CI

2. **Documentation Standards**
   - Every feature requires spec.md
   - README updates for new functionality
   - API changes require changelog entry

3. **Security Requirements**
   - Input validation required
   - Authentication on sensitive endpoints
   - No secrets in code

4. **Performance Standards**
   - Page load under 2 seconds
   - API responses under 200ms
   - Database queries optimized

5. **Accessibility**
   - WCAG 2.1 AA compliance
   - Keyboard navigation support
   - Screen reader compatibility
```

**3. RESEARCH.md (Exploration Phase)**

Document research findings before design:

```markdown
# Research: [Feature/Problem]

## Date: [YYYY-MM-DD]

## Goal
[What we're trying to understand]

## Findings

### Approach 1: [Option Name]
**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

**Examples/Resources:**
- [Link to documentation]
- [Example implementation]

### Approach 2: [Option Name]
[Similar structure]

## Recommendation
[Recommended approach with rationale]

## Open Questions
- Question 1
- Question 2

## Next Steps
- Action 1
- Action 2
```

**4. DESIGN.md (Design Phase)**

Document technical design decisions:

```markdown
# Design: [Feature Name]

## Architecture

[Diagram or description of component architecture]

## Data Models

```typescript
interface User {
  id: string;
  email: string;
  createdAt: Date;
}
```

## API Contracts

### POST /api/users
**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "id": "user_123",
  "email": "user@example.com"
}
```

## Component Hierarchy

```
App
├── AuthProvider
├── Router
│   ├── HomePage
│   ├── LoginPage
│   └── DashboardPage
└── ErrorBoundary
```

## Dependencies

- library-name@version: [Purpose and usage]

## Implementation Notes

- [Important considerations]
- [Edge cases to handle]
```

### Documentation Workflow Integration

**Recommended Workflow:**
1. Start with broad exploration questions (documented in RESEARCH.md)
2. Create spec.md based on research
3. Design technical approach (DESIGN.md)
4. Create implementation plan (prompt_plan.md or plan.md)
5. Implement with documentation updates
6. Update README and CHANGELOG
7. Create GitHub issue or PR with plan reference

## 5. Iteration and Refinement Workflows

### Extended Thinking for Better Planning

Claude Code supports extended thinking modes that allocate more reasoning tokens for complex planning:

- `"think"` - Standard extended thinking
- `"think hard"` - Increased thinking budget
- `"think harder"` - Further increased budget
- `"ultrathink"` - Maximum thinking budget (up to 31,999 tokens)

**When to use:** Use extended thinking when planning complex features, architectural decisions, or evaluating multiple approaches.

### Plan Mode for Iterative Refinement

Access Plan Mode via `Shift+Tab` twice or `claude --permission-mode plan`.

**Plan Mode enables:**
- Read-only codebase analysis
- Safe exploration without changes
- Multi-file planning for complex changes
- Interactive clarifying questions (Opus 4.5)

**Workflow:**
1. Enter Plan Mode
2. Ask Claude to analyze and create a plan
3. Review the generated plan.md
4. Exit Plan Mode (Shift+Tab)
5. Claude reads plan.md and begins implementation

### Iterative Refinement Pattern

```markdown
## Iteration 1: Initial Implementation
- Implement basic functionality
- Get working but imperfect version
- **Checkpoint:** Commit to Git

## Iteration 2: Refinement
- Add error handling
- Improve edge case coverage
- **Checkpoint:** Commit to Git

## Iteration 3: Optimization
- Performance improvements
- Code cleanup and refactoring
- **Checkpoint:** Commit to Git

## Iteration 4: Polish
- Documentation updates
- Final testing
- **Checkpoint:** Final commit
```

### Feedback Loop Integration

**Visual Targets for UI Work:**
- Provide screenshots or design mocks
- Claude can iterate toward visual target
- Get visual feedback, refine, repeat

**Test-Driven Iteration:**
1. Write test with expected behavior
2. Run test (should fail)
3. Implement code
4. Run test (should pass)
5. Refactor if needed
6. Repeat for next feature

### Parallel Claude Instances

For complex workflows, run multiple Claude instances:
- **Instance 1:** Writing implementation code
- **Instance 2:** Code review and verification
- **Git worktrees:** Parallel features in isolation

Use `git worktree add` to create isolated environments for each instance.

## 6. Checkpoint and Progress Tracking

### Why Checkpoints Matter

Checkpoints prevent losing progress and enable recovery from hallucination drift. They create save points where development can resume with fresh context.

### Two Types of Checkpoints

**1. Code Checkpoints (Git)**

Use Git commits as code-level save points:

```bash
# After reviewing AI-generated code
git add .
git commit -m "feat: implement user authentication

- Add login/register endpoints
- Create User model with password hashing
- Add JWT token generation
- Tests: 15 passing"

# Creates a rollback point
```

**Best Practices:**
- Commit after each completed phase
- Write descriptive commit messages
- Review code before committing
- Tag major milestones

**2. Project Checkpoints (Documentation)**

Use documentation to track high-level progress and maintain direction across sessions.

### Progress Tracking Files

**PROGRESS.md Template:**

```markdown
# Project Progress: [Project Name]

## Current Phase: Phase 3 - Core Implementation

### Status Icons
- 🔄 In Progress
- ✅ Completed
- ⏸️ Pending
- ❌ Blocked

## Progress Overview

### Phase 1: Research & Discovery ✅
- [x] Analyzed current architecture
- [x] Researched authentication patterns
- [x] Documented findings in RESEARCH.md
- **Git Checkpoint:** commit abc123

### Phase 2: Design & Planning ✅
- [x] Created interface contracts
- [x] Designed data models
- [x] Documented in DESIGN.md
- **Git Checkpoint:** commit def456

### Phase 3: Core Implementation 🔄
- [x] Implemented User model
- [x] Created authentication endpoints
- [ ] Add password reset flow
- [ ] Implement email verification
- **Current Focus:** Password reset implementation
- **Blockers:** None

### Phase 4: Testing & Validation ⏸️
- [ ] Unit tests
- [ ] Integration tests
- [ ] E2E tests

### Phase 5: Documentation & Deployment ⏸️
- [ ] Update README
- [ ] Create deployment guide
- [ ] Update CHANGELOG

## Current Session

**Started:** 2026-01-02 10:30 AM
**Goal:** Complete password reset flow
**Progress:**
- Created reset token generation
- Added email template
- Next: Implement token validation

## Next Actions
1. Implement reset token validation
2. Create password update endpoint
3. Write tests for reset flow
4. Update API documentation

## Notes & Decisions
- Using JWT for reset tokens (expires in 1 hour)
- Email service: SendGrid
- Rate limiting: 3 attempts per hour per email

## Session Recovery Info
Last working commit: def456
Last successful test run: All 23 tests passing
Environment: Development
```

### State Tracking Best Practices

From Anthropic's official guidance:

**Use structured formats for structured data:**
```json
{
  "tasks": [
    {"id": 1, "status": "completed", "name": "Setup auth"},
    {"id": 2, "status": "in_progress", "name": "Password reset"},
    {"id": 3, "status": "pending", "name": "Email verification"}
  ]
}
```

**Use unstructured text for progress notes:**
```markdown
Made good progress on auth today. The JWT implementation works well
but we might need to reconsider the refresh token strategy...
```

**Use Git for state tracking:**
- Claude 4.5 models excel at using Git for session continuity
- Commits create natural recovery points
- Git log provides progress history

### TodoWrite for Task Management

For complex, multi-step tasks, Claude Code's TodoWrite tool automatically tracks:

- **pending** - Task not started
- **in_progress** - Currently working (only ONE at a time)
- **completed** - Finished successfully

**Important:** Only mark tasks completed when fully accomplished. If blocked or encountering errors, keep as in_progress and create a new task for the blocker.

### SESSION.md for Multi-Session Projects

```markdown
# Session Tracker

## Session 1: 2026-01-02
**Duration:** 2 hours
**Completed:**
- ✅ Project setup
- ✅ Database schema
- ✅ Basic CRUD endpoints

**Git Checkpoint:** commit 1a2b3c4
**Context for Next Session:** Ready to add authentication

---

## Session 2: 2026-01-03
**Duration:** 3 hours
**Completed:**
- ✅ User authentication
- ✅ JWT implementation
- ⏸️ Password reset (partially done)

**Git Checkpoint:** commit 5d6e7f8
**Blockers:** Need to configure email service
**Context for Next Session:** Complete password reset after email setup

---

## Session 3: 2026-01-04 🔄
**Current Focus:** Email service setup and password reset completion
**Progress:**
- Email service configured
- Reset token generation done
- Next: Token validation endpoint
```

## 7. Best Practices for Planning Documents

### Location and Organization

**Recommended Directory Structure:**

```
project-root/
├── .claude/
│   ├── commands/           # Custom slash commands
│   ├── specs/              # Feature specifications
│   │   └── 001-auth/
│   │       ├── spec.md
│   │       ├── plan.md
│   │       └── tasks.md
│   ├── templates/          # Document templates
│   │   ├── spec-template.md
│   │   ├── plan-template.md
│   │   └── phase-template.md
│   └── steering/           # Project governance
│       ├── product.md
│       ├── tech.md
│       └── structure.md
├── CLAUDE.md              # Project context (root)
├── spec.md                # Current spec (root)
├── prompt_plan.md         # Implementation plan (root)
├── PROGRESS.md            # Progress tracking (root)
├── RESEARCH.md            # Research findings (root)
├── DESIGN.md              # Technical design (root)
└── constitution.md        # Project principles (root)
```

### Document Naming Conventions

**Timestamp-based naming for specs:**
```
{YYMMDD-HHMMSS}-{kebab-case-description}.md

Examples:
260102-143022-user-authentication.md
260103-091530-password-reset-flow.md
```

**Standard names for active documents:**
- `spec.md` - Current active specification
- `prompt_plan.md` - Current implementation plan
- `PROGRESS.md` - Current progress tracker

### Writing Style Guidelines

**1. Be Specific and Explicit**

❌ Bad:
```markdown
Add user authentication
```

✅ Good:
```markdown
Implement JWT-based user authentication with:
- Email/password login endpoint
- Token refresh mechanism
- Password hashing using bcrypt (10 rounds)
- Token expiration: 24 hours
- Refresh token rotation for security
```

**2. Use Clear Success Criteria**

❌ Bad:
```markdown
Make the app faster
```

✅ Good:
```markdown
Optimize page load performance:
- Initial page load < 2 seconds
- Time to Interactive (TTI) < 3 seconds
- Lighthouse performance score > 90
- Reduce bundle size by 30%
```

**3. Provide Context and Constraints**

```markdown
## Constraints
- Must work with existing User model
- Cannot break backward compatibility with v1 API
- Must support PostgreSQL and MySQL
- Mobile-first responsive design required
```

**4. Include Examples**

```markdown
## Expected Behavior

When user submits login form:

**Input:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Output (Success):**
```json
{
  "token": "eyJhbGc...",
  "user": {
    "id": "user_123",
    "email": "user@example.com"
  }
}
```

**Output (Error):**
```json
{
  "error": "Invalid credentials",
  "code": "AUTH_FAILED"
}
```
```

### Document Maintenance

**1. Keep Documents Synchronized**

When implementation deviates from plan:
- Update spec.md with actual implementation
- Document decision rationale
- Update progress tracking

**2. Archive Completed Phases**

```
.claude/specs/
├── completed/
│   ├── 001-user-auth/
│   └── 002-product-catalog/
└── active/
    └── 003-shopping-cart/
```

**3. Version Control All Planning Docs**

Commit planning documents to Git:
```bash
git add spec.md prompt_plan.md PROGRESS.md
git commit -m "docs: add shopping cart implementation plan"
```

### Team Collaboration

**1. Use Templates for Consistency**

Create team-wide templates in `.claude/templates/` that all team members use.

**2. Code Review Planning Documents**

Planning documents should be reviewed like code:
- Are requirements clear?
- Is the approach sound?
- Are success criteria measurable?

**3. Share via Custom Slash Commands**

```markdown
<!-- .claude/commands/spec.md -->
Create a new feature specification using our team template:

1. Copy .claude/templates/spec-template.md to spec.md
2. Fill in all sections thoroughly
3. Include at least 3 concrete examples
4. Define measurable success criteria
5. List all files that will be modified
6. Create a corresponding prompt_plan.md

Remember: Be extremely clear about features, how they work, and the tech stack.
```

Usage: Type `/spec` in Claude Code

## 8. Example Planning Templates

### Template 1: Feature Development Plan

```markdown
# Feature Plan: [Feature Name]

**Created:** [Date]
**Owner:** [Team/Person]
**Status:** Planning | In Progress | Review | Complete

## Executive Summary

[2-3 sentence overview of what this feature is and why it matters]

## Requirements

### User Stories
- As a [user type], I want [goal] so that [benefit]
- As a [user type], I want [goal] so that [benefit]

### Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Design

### Architecture Changes
[Description of new components or changes to existing architecture]

### Data Model Changes
```typescript
// New or modified schemas
```

### API Changes
```
POST /api/endpoint
GET /api/endpoint/:id
```

### Dependencies
- New dependency 1: [Purpose]
- New dependency 2: [Purpose]

## Implementation Plan

### Phase 1: Foundation (Est: X hours)
**Tasks:**
1. Task 1
2. Task 2

**Deliverables:**
- Deliverable 1
- Deliverable 2

**Git Checkpoint:** After phase completion

### Phase 2: Core Implementation (Est: X hours)
[Similar structure]

### Phase 3: Integration & Testing (Est: X hours)
[Similar structure]

## Testing Strategy

### Unit Tests
- Test scenario 1
- Test scenario 2

### Integration Tests
- Test workflow 1
- Test workflow 2

### E2E Tests
- User journey 1
- User journey 2

## Rollout Plan

### Phase 1: Development
- [ ] Implementation complete
- [ ] Tests passing
- [ ] Code review approved

### Phase 2: Staging
- [ ] Deployed to staging
- [ ] QA testing complete
- [ ] Performance validated

### Phase 3: Production
- [ ] Feature flag enabled for 10% users
- [ ] Monitoring in place
- [ ] Full rollout

## Success Metrics

**Technical Metrics:**
- Performance: [Target]
- Error rate: [Target]
- Test coverage: [Target]

**Business Metrics:**
- User adoption: [Target]
- Conversion rate: [Target]
- User satisfaction: [Target]

## Risks & Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Risk 1 | High | Medium | Mitigation strategy |
| Risk 2 | Medium | Low | Mitigation strategy |

## Open Questions
- [ ] Question 1
- [ ] Question 2

## References
- [Link to design doc]
- [Link to research]
- [Link to related features]
```

### Template 2: Bug Fix Workflow

```markdown
# Bug Fix Plan: [Bug Description]

**Created:** [Date]
**Severity:** Critical | High | Medium | Low
**Status:** Investigation | Planning | In Progress | Testing | Complete

## Problem Description

### Current Behavior
[What's happening now]

### Expected Behavior
[What should happen]

### Impact
- Users affected: [Number/percentage]
- Business impact: [Description]
- Workaround available: Yes/No

## Root Cause Analysis

### Investigation Steps
1. Step 1: [Findings]
2. Step 2: [Findings]

### Root Cause
[Description of underlying issue]

### Why It Wasn't Caught
[Gap in testing, edge case, etc.]

## Solution Design

### Technical Approach
[How the fix will work]

### Files to Modify
- `path/to/file1.ts`: [Changes needed]
- `path/to/file2.ts`: [Changes needed]

### Alternative Approaches Considered
- Approach 1: [Why not chosen]
- Approach 2: [Why not chosen]

## Implementation Plan

### Step 1: Write Failing Test
```typescript
test('should handle edge case', () => {
  // Test that reproduces the bug
});
```

### Step 2: Implement Fix
[Code changes]

### Step 3: Verify Fix
- [ ] Test passes
- [ ] No regression in other tests
- [ ] Manual verification complete

### Step 4: Deploy
- [ ] Code review approved
- [ ] Deployed to staging
- [ ] Verified in production

## Prevention

### Test Coverage Added
- [ ] Unit test for edge case
- [ ] Integration test for workflow
- [ ] E2E test if applicable

### Process Improvements
- [ ] Update test checklist
- [ ] Add validation to prevent recurrence
- [ ] Document in runbook

## Verification

### Test Cases
- [ ] Original bug scenario
- [ ] Edge case 1
- [ ] Edge case 2
- [ ] Regression test for related features

## Rollback Plan

If fix causes issues:
1. Revert commit [hash]
2. Redeploy previous version
3. Re-enable workaround if applicable

## Post-Mortem

### Timeline
- Bug reported: [Date/Time]
- Investigation started: [Date/Time]
- Root cause identified: [Date/Time]
- Fix deployed: [Date/Time]

### Lessons Learned
- Learning 1
- Learning 2

### Action Items
- [ ] Action 1
- [ ] Action 2
```

### Template 3: Refactoring Plan

```markdown
# Refactoring Plan: [Component/Module Name]

**Created:** [Date]
**Motivation:** [Why refactor now]
**Risk Level:** Low | Medium | High

## Current State Analysis

### Problems with Current Implementation
1. Problem 1: [Description and impact]
2. Problem 2: [Description and impact]

### Technical Debt Metrics
- Cyclomatic complexity: [Number]
- Code duplication: [Percentage]
- Test coverage: [Percentage]
- Lines of code: [Number]

### Dependencies
[List of components/modules that depend on this code]

## Proposed Changes

### Architecture Improvements
[Description of new structure]

### Code Quality Goals
- Reduce complexity to: [Target]
- Increase test coverage to: [Target]
- Improve performance by: [Target]

### Before/After Examples

**Before:**
```typescript
// Current implementation
```

**After:**
```typescript
// Proposed implementation
```

## Migration Strategy

### Phase 1: Preparation
- [ ] Write comprehensive tests for current behavior
- [ ] Document all current functionality
- [ ] Identify all call sites

### Phase 2: Incremental Refactoring
- [ ] Refactor component A (maintain tests)
- [ ] Refactor component B (maintain tests)
- [ ] Verify no behavioral changes

### Phase 3: Cleanup
- [ ] Remove deprecated code
- [ ] Update documentation
- [ ] Performance benchmarking

## Testing Strategy

### Regression Prevention
- [ ] All existing tests still pass
- [ ] No change in test coverage
- [ ] Performance benchmarks maintained or improved

### New Tests
- [ ] Test for new edge cases
- [ ] Test for improved error handling

## Rollout Plan

### Validation Steps
1. Run full test suite
2. Performance benchmark comparison
3. Code review approval
4. Deploy to staging
5. Monitor for 24 hours
6. Production deployment

### Success Criteria
- [ ] All tests passing
- [ ] Performance maintained or improved
- [ ] Code complexity reduced
- [ ] No production incidents

## Rollback Plan

Git checkpoint before refactoring: [commit hash]

If issues arise:
```bash
git revert [commit-hash]
git push origin main
```

## Timeline

- Start Date: [Date]
- Target Completion: [Date]
- Review Date: [Date]
```

### Template 4: Sprint Planning Document

```markdown
# Sprint Plan: [Sprint Number/Name]

**Sprint Duration:** [Start Date] - [End Date]
**Team Capacity:** [Total person-days]

## Sprint Goal

[One sentence describing the sprint objective]

## Prioritized Backlog

### P0 - Must Have
1. **[Task 1]** - [Estimate] - [Owner]
   - Spec: spec.md#section
   - Acceptance criteria: [Link]

2. **[Task 2]** - [Estimate] - [Owner]

### P1 - Should Have
[Similar structure]

### P2 - Nice to Have
[Similar structure]

## Daily Implementation Plan

### Monday
- Morning: [Task focus]
- Afternoon: [Task focus]
- Checkpoint: [What should be done]

### Tuesday
[Similar structure for each day]

## Dependencies & Blockers

| Task | Dependency | Owner | Status |
|------|------------|-------|--------|
| Task 1 | API endpoint | Team B | In Progress |
| Task 2 | Design review | Design | Blocked |

## Definition of Done

- [ ] All acceptance criteria met
- [ ] Tests written and passing
- [ ] Code reviewed and approved
- [ ] Documentation updated
- [ ] Deployed to staging
- [ ] Product owner approval

## Sprint Review Preparation

### Demo Plan
1. Demo item 1
2. Demo item 2

### Metrics to Share
- Velocity: [Story points completed]
- Test coverage: [Percentage]
- Bug fix rate: [Number]

## Retrospective Topics

Things to discuss:
- What went well
- What could improve
- Action items from last retro
```

## Conclusion

Effective prompt planning transforms Claude Code from a reactive coding assistant into a strategic development partner. By investing time in structured planning documents—spec.md for requirements, prompt_plan.md for implementation strategy, and progress tracking files for continuity—you create a foundation for successful, maintainable AI-assisted development.

### Key Takeaways

1. **Plan before coding**: Use Plan Mode and extended thinking to create thorough plans
2. **Document everything**: Specs, designs, research, and progress should all be captured
3. **Break down complexity**: Phase-based implementation prevents overwhelm and drift
4. **Track progress rigorously**: Use Git checkpoints and progress documentation
5. **Iterate and refine**: Review, approve, and improve at each phase boundary
6. **Create team templates**: Standardize planning across your organization
7. **Maintain context**: Well-structured documents enable Claude to resume work across sessions

### Next Steps

1. Create your `.claude/` directory structure
2. Set up team templates from this guide
3. Write or refine your `CLAUDE.md` file
4. Try Plan Mode on your next feature
5. Establish a spec-driven workflow for your team

By following these practices, you'll maximize Claude Code's capabilities and build better software faster.

---

## Additional Resources

### Official Documentation
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Claude Code Documentation](https://code.claude.com/docs/en/overview)
- [Common Workflows Guide](https://code.claude.com/docs/en/common-workflows)

### Community Resources
- [ClaudeLog - Plan Mode Guide](https://claudelog.com/mechanics/plan-mode/)
- [Spec-Driven Development Guide](https://www.arsturn.com/blog/spec-driven-development-with-claude-code)
- [GitHub: claude-code-spec-workflow](https://github.com/Pimzino/claude-code-spec-workflow)
- [Checkpointing Code Projects with AI](https://hamy.xyz/blog/2025-07_ai-checkpointing)
- [CLAUDE.md Best Practices](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/)

### Tools and Extensions
- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Claude Code System Prompts Repository](https://github.com/Piebald-AI/claude-code-system-prompts)
- [Awesome Claude Code](https://github.com/hesreallyhim/awesome-claude-code)

---

*Last Updated: January 2, 2026*
