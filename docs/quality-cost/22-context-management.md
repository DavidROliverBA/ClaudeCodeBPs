# Context Management in Claude Code: A Comprehensive Guide

## Table of Contents
- [Introduction](#introduction)
- [Understanding Context Compaction](#understanding-context-compaction)
- [Context Window Limits and Management](#context-window-limits-and-management)
- [When Compaction Occurs](#when-compaction-occurs)
- [Saving State Before Context Refresh](#saving-state-before-context-refresh)
- [Persistent Autonomous Operation](#persistent-autonomous-operation)
- [Memory and Continuity Patterns](#memory-and-continuity-patterns)
- [Strategies for Long-Running Tasks](#strategies-for-long-running-tasks)
- [Best Practices for Context Efficiency](#best-practices-for-context-efficiency)
- [Practical Workflows](#practical-workflows)

---

## Introduction

Context management is one of the most critical skills for effective use of Claude Code. The context window—the information Claude uses to understand your project and generate responses—has finite limits. How you manage this limited resource directly impacts code quality, session continuity, and overall productivity during extended development sessions.

This guide provides comprehensive strategies for optimizing context usage, maintaining state across sessions, and enabling autonomous operation without hitting context limits or experiencing performance degradation.

---

## Understanding Context Compaction

### What is Compaction?

Compaction is Claude Code's intelligent context window management system that automatically summarizes conversations when approaching memory limits. Rather than failing when the context window fills, Claude creates a condensed summary of the conversation history and starts fresh with that summary preloaded.

### How Compaction Works

When compaction triggers, Claude Code:
1. Analyzes the entire conversation history
2. Creates a summary containing essential information
3. Clears the original conversation messages
4. Loads the summary as the new starting context
5. Continues operation with reduced token usage

This process allows extended sessions to continue without interruption, though some detail and nuance from earlier conversations may be lost in the summarization.

### Types of Compaction

**Auto-Compaction**: Triggered automatically when the context window reaches approximately 95% capacity (or 25% remaining). The system decides what to summarize based on internal heuristics.

**Manual Compaction**: Initiated by the `/compact` command, giving you control over when summarization occurs. You can provide specific instructions about what to preserve or how to structure the summary.

### Context Editing Strategies

Modern Claude Code implements advanced context editing with two key strategies:

**Tool Use Clearing** (`clear_tool_uses_20250919`): Automatically clears the oldest tool results in chronological order when conversation context exceeds configured thresholds, replacing them with placeholder text. This preserves the conversation flow while reducing token consumption.

**Thinking Block Clearing** (`clear_thinking_20251015`): Manages thinking blocks in conversations when extended thinking is enabled, automatically clearing older thinking blocks from previous turns while retaining the conclusions and outputs.

---

## Context Window Limits and Management

### Technical Specifications

Claude Code's context window varies by model tier, but typical limits include:
- **Claude Sonnet**: 200,000 tokens
- **Claude Opus**: 200,000 tokens
- **Claude Haiku**: 200,000 tokens

Each token represents roughly 4 characters of text, meaning a 200k token window can hold approximately 800,000 characters or about 150,000 words.

### Working Memory Considerations

A critical insight from recent research: **the context window is not just storage—it's working memory**. Language models need functional space beyond just storing information. They use context space to:
- Develop and evaluate response alternatives
- Construct complex outputs
- Plan multi-step operations
- Maintain reasoning chains

**Performance degradation occurs significantly as you approach context limits.** When 95% of context is consumed, only 10,000 tokens remain for these computational processes—insufficient for high-quality responses on complex tasks.

### The 75% Rule

Recent observations indicate Claude Code triggers compaction around **75% utilization** rather than the historical 90%+. With a 200k token window, this means:
- Compaction triggers at ~150k tokens
- Leaves 50k tokens (25%) for reasoning and working memory
- Prevents mid-operation interruptions and context corruption

This "infrastructure thinking" accepts apparent inefficiency to maintain consistent output quality throughout sessions.

### What Consumes Context

Your context window fills with:
1. **Conversation history**: All messages between you and Claude
2. **File contents**: Every file Claude reads or edits
3. **Tool outputs**: Results from bash commands, searches, and other operations
4. **CLAUDE.md**: Project configuration automatically loaded each session
5. **System prompts**: Hidden instructions that configure Claude's behavior
6. **Thinking blocks**: Extended reasoning when using "think" commands

---

## When Compaction Occurs

### Automatic Triggers

Auto-compaction activates when:
- Context utilization reaches approximately **75-95%** (varies by configuration)
- Token count approaches the model's maximum window size
- The system detects insufficient working memory for quality responses

### Warning Signs

You'll know compaction is imminent when:
- Claude explicitly states "Context left until auto-compact: X%"
- Response quality begins degrading (repetitive suggestions, missing context)
- Claude starts "forgetting" earlier conversation details
- Tool outputs become more verbose or less focused

### Compaction Timing Issues

**The Mid-Task Problem**: Auto-compaction doesn't understand task boundaries. It may trigger during:
- Complex debugging sessions
- Multi-file refactoring operations
- Long test execution and analysis
- Critical decision-making moments

This can cause Claude to lose essential task state, leading to the dreaded "infinite compaction loop" where:
1. Claude tries to compact to fit context window
2. The compaction loses critical task state
3. Claude re-reads files to "recover" context
4. The cycle repeats indefinitely

### Best Practice: Proactive Manual Compaction

**Don't wait for auto-compact.** When you've finished a feature, fixed a bug, or reached a logical deployment point, run `/compact` yourself. This ensures summarization happens when context is clean and coherent, not mid-operation.

---

## Saving State Before Context Refresh

### The Persistence Principle

As stated in Anthropic's best practices: "As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching."

### State Preservation Strategies

#### 1. **CLAUDE.md Documentation**

Store permanent project context in `CLAUDE.md`:
```markdown
# Project Context

## Architecture Decisions
- Using React with TypeScript
- State management via Zustand
- API layer with React Query

## Current Work
- Refactoring authentication flow
- Next: Implement refresh token rotation

## Important Constraints
- Never commit .env files
- Always run tests before commits
- Use conventional commit messages
```

This information loads automatically in every session, preventing repeated context buildup.

#### 2. **Markdown Checklists for Complex Tasks**

Create a `TODO.md` or GitHub issue as a working scratchpad:
```markdown
## Authentication Refactor Checklist

- [x] Extract auth logic to separate module
- [x] Implement token refresh mechanism
- [ ] Add error handling for expired tokens
- [ ] Update login component
- [ ] Write integration tests
- [ ] Update documentation

## Notes
- RefreshToken component in utils/auth.ts
- Tests failing on line 45 - need to mock localStorage
```

By constantly updating this checklist, Claude "recites its objectives into the end of the context," keeping goals in recent attention and avoiding "lost-in-the-middle" problems.

#### 3. **Git Commits as Checkpoints**

Make regular, atomic commits:
```bash
# After completing each discrete unit of work
git add src/auth/
git commit -m "feat: implement token refresh mechanism

- Add refreshToken function to auth service
- Handle 401 responses with automatic retry
- Store refresh token in secure httpOnly cookie"
```

This creates resumable checkpoints. If you need to `/clear` or restart a session, the commit history preserves exactly what was accomplished.

#### 4. **Memory Tool for Cross-Session Persistence**

The memory tool enables Claude to store information outside the context window through a file-based system. Claude can create, read, update, and delete files in a dedicated memory directory that persists across conversations.

Use cases:
- Architectural decisions that should influence all future work
- Bug patterns discovered during debugging
- Performance optimization notes
- Dependency relationships and constraints

### Pre-Compaction Workflow

When approaching context limits:

1. **Document current state**: Update TODO.md with progress
2. **Commit completed work**: Create a clean git checkpoint
3. **Note blocking issues**: Document any errors or obstacles
4. **Update CLAUDE.md**: Add any new architectural insights
5. **Then compact or clear**: Start fresh with state preserved

---

## Persistent Autonomous Operation

### Enabling Autonomous Workflows

Claude Code can operate autonomously for extended periods when properly configured. Key enablers:

#### 1. **Sandboxing**

Sandboxing creates pre-defined boundaries within which Claude can work freely without repeated permission prompts:

```bash
# Enable sandboxed environment
/sandbox
```

Benefits:
- **84% reduction in permission prompts** in internal testing
- Safer execution of exploratory or high-impact operations
- Faster autonomous workflows without interruptions

#### 2. **YOLO Mode for Trusted Environments**

For containerized or isolated environments, skip permission prompts entirely:

```bash
claude --dangerously-skip-permissions
```

**Warning**: Only use in controlled environments without internet access or sensitive data exposure.

#### 3. **CLAUDE.md as Operational Boundaries**

Define rules that Claude must follow autonomously:

```markdown
# Autonomous Operation Rules

## Never
- Modify database migration files directly
- Skip running tests before commits
- Force push to main/master branches

## Always
- Run `npm test` after code changes
- Use TypeScript strict mode
- Follow existing naming conventions

## Verification Steps
1. Run linter: `npm run lint`
2. Run type check: `npm run type-check`
3. Run tests: `npm test`
4. Build: `npm run build`
```

CLAUDE.md instructions are treated as **immutable system rules**, while user prompts are flexible requests that must work within those boundaries.

### Autonomous Task Patterns

#### Container-Based Workflow

1. **Issue Creation**: Write detailed GitHub issue with requirements
2. **Autonomous Implementation**: Claude works independently in secure container
3. **Review**: Inspect completed pull request

This three-step process maximizes productivity while maintaining quality control.

#### Parallel Instance Strategy

Use git worktrees to run multiple Claude instances simultaneously:

```bash
# Create separate worktrees for parallel work
git worktree add ../myproject-feature-a feature-a
git worktree add ../myproject-feature-b feature-b

# Launch Claude in each directory
cd ../myproject-feature-a && claude
cd ../myproject-feature-b && claude
```

Each instance operates independently with its own context window—no conflicts, no waiting.

### Verification Without Human Feedback

As autonomous task length grows, Claude needs self-verification mechanisms:
- Unit tests with clear pass/fail criteria
- Integration tests for complex workflows
- Linting and type checking for code quality
- Build processes that catch errors early

Configure these in CLAUDE.md so Claude knows to run them autonomously.

---

## Memory and Continuity Patterns

### External Memory Systems

#### Memory Tool Architecture

The memory tool stores information outside the context window in a persistent file-based system:

```
project/
├── .claude/
│   └── memory/
│       ├── architecture.md
│       ├── api-patterns.md
│       └── known-issues.md
```

Claude can create, read, update, and delete these files across sessions, building an accumulating knowledge base.

#### Combined Strategy Performance

According to Anthropic's evaluations:
- **Memory tool + context editing**: 39% performance improvement over baseline
- **Context editing alone**: 29% improvement
- **100-turn web search**: 84% reduction in token consumption while enabling workflows that would otherwise fail

### Continuity Through Subagents

#### When to Use Subagents

Subagents operate in separate context windows, preventing pollution of the main conversation:

- **Research tasks**: Deep investigation without cluttering primary context
- **Verification**: Check implementation details without context bloat
- **Parallel exploration**: Evaluate multiple approaches simultaneously

#### Subagent Resumption

Subagents can be resumed to continue previous conversations, particularly useful for:
- Long-running research that spans multiple sessions
- Complex analysis requiring incremental refinement
- Building specialized knowledge bases for specific domains

#### Subagent Strategy

Tell Claude to use subagents early in conversations to preserve main context:

```
"Before we begin implementation, please use a subagent to:
1. Research the current authentication implementation
2. Identify all files that need modification
3. Check for existing test coverage

Then report back with findings so we can plan our approach."
```

### The CLAUDE.md Supremacy Pattern

Rather than creating specialized subagents, use CLAUDE.md to provide consistent context:

> "Put all key context in the CLAUDE.md. Then, let the main agent decide when and how to delegate work to copies of itself. This gives all the context-saving benefits of subagents without the drawbacks."

This approach:
- Ensures all instances share core project knowledge
- Reduces redundant context loading
- Maintains consistency across parallel operations

---

## Strategies for Long-Running Tasks

### Task Decomposition

Break substantial work into discrete phases that fit within context windows:

#### Phase 1: Exploration & Planning
```
Goal: Understand current architecture and plan changes
- Map existing authentication flow
- Identify integration points
- Document dependencies
- Create implementation checklist
```

#### Phase 2: Implementation
```
Goal: Execute planned changes with focus
- Implement core functionality
- Handle error cases
- Update related components
```

#### Phase 3: Verification & Refinement
```
Goal: Ensure quality and completeness
- Run comprehensive tests
- Fix discovered issues
- Update documentation
```

After each phase, commit work and optionally `/compact` or `/clear` to start the next phase fresh.

### Checklist-Driven Development

For migrations, large refactors, or systematic changes:

1. **Generate comprehensive checklist** of all required changes
2. **Save to `PROGRESS.md`** in the repository
3. **Update after each completed item**
4. **Compact between sections** to maintain focus

Example:
```markdown
## Database Migration Checklist

### Phase 1: Schema Updates (COMPLETE)
- [x] Create migration file
- [x] Update User model
- [x] Add indexes for performance

### Phase 2: Code Updates (IN PROGRESS)
- [x] Update user service
- [x] Modify authentication middleware
- [ ] Update user controller
- [ ] Refactor admin dashboard queries

### Phase 3: Testing (PENDING)
- [ ] Unit tests for user service
- [ ] Integration tests for auth flow
- [ ] Performance testing with new indexes
```

This pattern keeps Claude focused on immediate next steps while maintaining awareness of overall progress.

### Context Window Hygiene

#### Frequent Clearing

Use `/clear` between distinct tasks:

```
Task 1: Fix authentication bug → /clear
Task 2: Add new API endpoint → /clear
Task 3: Update documentation → /clear
```

The `/clear` command wipes conversation history while preserving:
- Project files and structure
- CLAUDE.md configuration
- Terminal/REPL session
- Git state and working directory

#### Selective File Reading

Instead of having Claude read entire codebases, provide targeted context:

```
"To fix the authentication bug, please review:
- src/auth/authService.ts (the main service)
- src/middleware/authenticate.ts (the middleware)
- tests/auth.test.ts (existing tests)

Do not read other files unless you identify a specific need."
```

Large repositories (20,000+ lines) often produce inferior results including reinventing existing patterns and missing critical dependencies when loaded entirely into context.

### Avoiding the Last 20% Problem

Research consistently shows performance degradation as models approach context limits. **Avoid using the last 20% of the context window for anything touching multiple parts of your codebase.**

If approaching 80% context utilization:
1. Complete current focused task
2. Commit the work
3. `/compact` or `/clear` to free space
4. Continue with next task

---

## Best Practices for Context Efficiency

### 1. **Optimize CLAUDE.md Content**

Include only essential, stable information:

✅ **Do Include:**
- Core architectural patterns
- Technology stack and versions
- Code style guidelines
- Testing requirements
- Never-change constraints

❌ **Don't Include:**
- Temporary task details
- Specific bug descriptions
- Implementation notes for current work
- Verbose explanations of basic concepts

### 2. **Use Context Quality Over Quantity**

Instead of loading many files "just in case," provide:
- Precise file paths for specific problems
- Clear problem descriptions
- Focused objectives
- Explicit constraints

High-quality, targeted context produces better results than comprehensive but unfocused information.

### 3. **Thinking Budget Control**

Extended thinking consumes significant context. Use appropriately:

- `"think"`: Basic analysis (smallest budget)
- `"think hard"`: Complex problem-solving
- `"think harder"`: Multi-faceted decisions
- `"ultrathink"`: Extremely complex scenarios (largest budget)

Don't request extended thinking for simple tasks—it wastes context on unnecessary reasoning.

### 4. **Manual Compaction at Logical Boundaries**

Proactively compact when:
- ✅ Finishing a feature implementation
- ✅ Completing a bug fix and verification
- ✅ Reaching a deployment point
- ✅ Switching between unrelated tasks

Avoid compacting when:
- ❌ Mid-debugging session
- ❌ During multi-file refactoring
- ❌ While tests are running
- ❌ In the middle of complex decision-making

### 5. **Custom Compaction Instructions**

Provide specific guidance for `/compact`:

```bash
/compact "Preserve: the authentication refactor plan, all discovered bugs, and testing notes. Summarize: the earlier discussion about database options."
```

This ensures critical information survives summarization.

### 6. **Leverage Tool Result Clearing**

Modern context editing automatically clears stale tool results. To optimize:
- Batch related file reads together
- Complete tool-heavy operations before switching contexts
- Trust that completed tool outputs can be safely discarded

### 7. **Document, Don't Discuss**

Instead of long conversational exchanges about architecture:

```
❌ "What do you think about using Redux vs Zustand? Let's discuss the pros and cons of each approach for our use case..."

✅ "Decision: Using Zustand for state management.
Reasoning: Simpler API, smaller bundle size, good TypeScript support.
[Add to CLAUDE.md, no context-heavy discussion needed]"
```

### 8. **Exit and Restart for Fresh Start**

When experiencing context-related issues:
1. Document current state in files (TODO.md, commit message, etc.)
2. `/quit` the session
3. Restart with `claude`
4. Reference the documented state to continue

This is often faster than trying to salvage an overwhelmed session through compaction.

---

## Practical Workflows

### Workflow 1: Feature Development Session

```markdown
## Start of Session
1. Review CLAUDE.md for project context
2. Create feature branch: `git checkout -b feature/user-profile`
3. Define scope: "Add user profile page with avatar upload"

## Implementation Phase
4. Read only necessary files: UserService, ProfileComponent
5. Implement core functionality
6. Update TODO.md with progress
7. Run tests: `npm test`

## Checkpoint
8. Commit: `git commit -m "feat: add user profile page"`
9. Manual compact: `/compact "Preserve: remaining TODO items, testing notes"`

## Continue or Wrap
10. If more work: Continue with fresh context
11. If done: Push branch, create PR, `/quit`
```

### Workflow 2: Bug Investigation and Fix

```markdown
## Investigation Phase
1. Create bug reproduction checklist
2. Use subagent for deep stack trace analysis
3. Main agent: Read minimal files to understand bug
4. Document findings in INVESTIGATION.md

## Fix Phase
5. `/clear` to start fresh
6. Reference INVESTIGATION.md for context
7. Implement fix in focused session
8. Verify with tests

## Verification
9. Commit fix
10. Update bug tracker
11. `/clear` before next task
```

### Workflow 3: Large Migration Project

```markdown
## Planning Session
1. Analyze scope of migration
2. Generate comprehensive checklist
3. Save to MIGRATION_CHECKLIST.md
4. Commit initial plan
5. `/quit`

## Implementation Sessions (multiple)
For each session:
1. Read MIGRATION_CHECKLIST.md
2. Select next section (e.g., "Update models")
3. Implement that section completely
4. Update checklist with [x]
5. Commit changes
6. `/compact` or `/quit`

## Final Verification Session
1. Review completed checklist
2. Run full test suite
3. Performance testing
4. Update documentation
5. Final commit and PR
```

### Workflow 4: Autonomous Parallel Development

```bash
# Setup
git worktree add ../project-auth feature/auth-refactor
git worktree add ../project-api feature/api-v2
git worktree add ../project-tests test/integration

# Launch parallel instances
cd ../project-auth && claude --dangerously-skip-permissions &
cd ../project-api && claude --dangerously-skip-permissions &
cd ../project-tests && claude &

# Each instance works independently:
# Instance 1: Refactor authentication
# Instance 2: Implement API v2
# Instance 3: Write integration tests

# Later: Review and merge
git worktree remove ../project-auth
git worktree remove ../project-api
git worktree remove ../project-tests
```

---

## Key Takeaways

1. **Context is working memory, not just storage**: Reserve 20-25% for reasoning processes
2. **Proactive beats reactive**: Manual `/compact` at logical points prevents mid-task disruptions
3. **Document state externally**: CLAUDE.md, TODO.md, and git commits preserve continuity
4. **Quality over quantity**: Focused, relevant context outperforms comprehensive information dumps
5. **Clear frequently**: `/clear` between unrelated tasks maintains focus and performance
6. **Subagents preserve main context**: Use for research and verification without pollution
7. **Trust auto-optimization**: Context editing and tool result clearing work automatically
8. **Plan for limits**: Break large tasks into phases that fit within context windows
9. **CLAUDE.md is supreme**: Central configuration ensures consistency across sessions and instances
10. **Persistence enables autonomy**: Proper state management allows Claude to work independently for extended periods

## Performance Metrics

Based on Anthropic's research:
- **39% improvement**: Combined memory tool + context editing
- **29% improvement**: Context editing alone
- **84% token reduction**: On 100-turn workflows
- **84% fewer prompts**: With sandboxing enabled

## Conclusion

Effective context management transforms Claude Code from a helpful assistant into a persistent, autonomous development partner. By understanding context compaction mechanics, proactively managing context windows, preserving state externally, and following established patterns, you can maintain high-quality code generation throughout extended sessions.

The shift from reactive (waiting for auto-compact) to proactive (strategic manual compaction) represents a maturity in working with AI coding assistants. Combined with proper use of CLAUDE.md, subagents, and external memory systems, you can build complex software systems across multiple sessions without losing continuity or context.

Master these patterns, and you'll unlock Claude Code's full potential for long-running, autonomous software development.

---

## Sources

- [Managing context on the Claude Developer Platform](https://claude.com/blog/context-management)
- [How I Use Every Claude Code Feature - Shrivu Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)
- [Context editing - Claude Docs](https://platform.claude.com/docs/en/build-with-claude/context-editing)
- [Managing Claude Code's Context - CometAPI](https://www.cometapi.com/managing-claude-codes-context/)
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [What is Claude Code auto-compact - ClaudeLog](https://claudelog.com/faqs/what-is-claude-code-auto-compact/)
- [How Claude Code Got Better by Protecting More Context](https://hyperdev.matsuoka.com/p/how-claude-code-got-better-by-protecting)
- [What to Do When Claude Code Starts Compacting - Du'An Lightfoot](https://www.duanlightfoot.com/posts/what-to-do-when-claude-code-starts-compacting/)
- [Claude Code Compaction - Steve Kinney](https://stevekinney.com/courses/ai-development/claude-code-compaction)
- [Why Claude Forgets: Guide to Auto-Compact & Context Windows](https://www.arsturn.com/blog/why-does-claude-forget-things-understanding-auto-compact-context-windows)
- [Claude Code Autonomous Control and Best Practices](https://smartscope.blog/en/generative-ai/claude/claude-code-control-best-practices/)
- [Making Claude Code more secure and autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing)
- [The Claude Code Revolution: Autonomous Development](https://octospark.ai/blog/the-comprehensive-guide-to-claude-code)
- [Claude Code 2.0: Checkpoints, Subagents, and Autonomous Coding](https://skywork.ai/blog/claude-code-2-0-checkpoints-subagents-autonomous-coding/)
- [Feature Request: Structured Workflow for Complex Tasks](https://github.com/anthropics/claude-code/issues/5996)
- [Subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [ClaudeLog - Context Window Depletion](https://claudelog.com/mechanics/context-window-depletion/)
