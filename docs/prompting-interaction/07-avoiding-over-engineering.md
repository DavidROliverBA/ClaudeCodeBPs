# Avoiding Over-Engineering When Using Claude Code

## Introduction

Claude Code is a powerful agentic coding assistant, but like many advanced AI models, it has a natural tendency toward over-engineering. Claude Opus 4.5, in particular, tends to create extra files, add unnecessary abstractions, or build in flexibility that wasn't explicitly requested. This comprehensive guide will help you recognize over-engineering patterns, prevent them through effective prompting, and recover when Claude goes overboard.

## Common Over-Engineering Patterns Claude Might Produce

Understanding how Claude tends to over-engineer is the first step in preventing it. Here are the most common patterns:

### 1. Excessive File Creation
Claude may split functionality across multiple files when a single file would suffice. For example, a simple feature might be implemented across 15 files when 3 would do the job, resulting in 1000+ lines when 120 lines would provide the same functionality.

### 2. Premature Abstraction
Claude often creates helper functions, utility classes, or abstractions for operations that only happen once. These abstractions add complexity without providing reusable value.

### 3. Speculative Error Handling
The model tends to add error handling, fallbacks, and validation for scenarios that can't actually happen in your system. This clutters code with unnecessary defensive programming.

### 4. Over-Configurability
Simple features often get unnecessary configuration options, environment variables, or customization parameters that weren't requested and won't be used.

### 5. Compatibility Layers
Claude may add backwards-compatibility shims, adapter patterns, or intermediate layers when you could simply change the existing code directly.

### 6. Future-Proofing
The model designs for hypothetical future requirements, adding extensibility points and architectural complexity for scenarios that may never materialize.

### 7. Unsolicited Refactoring
When asked to fix a bug, Claude might refactor surrounding code, clean up formatting, or "improve" nearby functions that weren't part of the request.

## Prompting Techniques to Keep Solutions Minimal

The most effective way to prevent over-engineering is through explicit, well-crafted prompts. Here are proven techniques:

### The Core Anti-Bloat Directive

Add this explicit instruction to your prompts or CLAUDE.md:

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused. Don't add features, refactor code, or make 'improvements' beyond what was asked.
```

### Specific Constraints to Include

Enhance your prompts with these specific constraints:

```
- A bug fix doesn't need surrounding code cleaned up
- A simple feature doesn't need extra configurability
- Don't add error handling for scenarios that can't happen
- Trust internal code and framework guarantees
- Only validate at system boundaries (user input, external APIs)
- Don't use backwards-compatibility shims when you can just change the code
- Don't create helpers, utilities, or abstractions for one-time operations
- Don't design for hypothetical future requirements
- The right amount of complexity is the minimum needed for the current task
- Reuse existing abstractions where possible
```

### Philosophy-Based Prompting

Communicate the underlying philosophy to guide Claude's decision-making:

```
Less code = fewer bugs. Mathematical fact.
Existing code = tested code. Practical reality.
Minimal changes = minimal risk. Engineering wisdom.

Every abstraction is complexity you maintain.
Every pattern is something you need to remember.
Keeping it minimal keeps it maintainable.
```

### Context-Specific Instructions

Tailor your constraints to your project phase and context:

**For Solo Developers/Early-Stage Startups:**
```
Prioritize simple, readable code with minimal abstraction. Avoid premature optimization.
Strive for elegant, minimal solutions that reduce complexity. I'm working alone, so
every abstraction is complexity I maintain alone.
```

**For Bug Fixes:**
```
Fix only the specific bug described. Do not refactor surrounding code, add tests for
unrelated functionality, or improve code style in nearby areas. Minimal changes only.
```

**For New Features:**
```
Implement only the feature as described. Do not add configuration options,
extensibility points, or additional related features. Follow existing patterns
in the codebase exactly.
```

## Specifying Scope and Constraints Clearly

Vague requests invite over-engineering. Clear scopes prevent it.

### Bad vs. Good Request Examples

**Bad (Invites Bloat):**
```
Add a calendar widget
```

**Good (Constrains Scope):**
```
Add a calendar widget following the existing pattern from HotDogWidget.php.
Use the same styling and data structure. Only implement month view.
```

**Bad (Open-Ended):**
```
Improve the authentication system
```

**Good (Specific and Bounded):**
```
Fix the authentication timeout bug in auth.py line 142. Do not refactor
the auth system or add new features.
```

### The "Follow Existing Patterns" Technique

One of the most effective scope constraints is referencing existing code:

```
Implement the user profile page following the exact same structure and patterns
as the dashboard page in dashboard.tsx. Don't add features that dashboard doesn't have.
```

This grounds Claude in concrete examples rather than abstract possibilities.

### Use Test-Driven Constraints

Writing tests first naturally constrains scope:

```
First, write tests that verify these specific behaviors: [list].
Then implement only the code necessary to pass these tests.
Do not add functionality beyond what the tests require.
```

## "Do Only What I Asked" Type Instructions

While the exact phrase "do only what I asked" doesn't appear in official Claude documentation, the concept is well-supported. Here are effective variations:

### Direct Constraint Instructions

```
IMPORTANT: Make only the changes I explicitly requested. Do not:
- Add related features I didn't mention
- Refactor code outside the scope of this request
- Add error handling beyond what I specified
- Create new abstractions or utilities
- Improve or optimize unrelated code

If you think something additional is needed, ask me first rather than implementing it.
```

### Surgical Change Request

```
Make a surgical change: modify only what's necessary to accomplish [specific goal].
Treat the surrounding code as immutable unless changing it is absolutely required.
```

### Confirmation-Based Approach

```
Before implementing, list exactly what you plan to change and why each change is
necessary. Wait for my confirmation before proceeding.
```

## Avoiding Premature Abstraction

Premature abstraction is one of the most common forms of over-engineering. Here's how to prevent it:

### The Three Uses Rule

```
Do not create abstractions (helper functions, utility classes, shared components)
unless the code will be used in at least three different places. For one or two uses,
accept the duplication.
```

### LLMs Change the DRY Calculus

With AI assistance, the traditional "Don't Repeat Yourself" (DRY) principle needs recalibration. Redoing work is now extremely cheap with AI, so you can afford to wait longer before abstracting. Some repetition is actually beneficial because it:

- Keeps code explicit and easy to understand
- Avoids the cost of maintaining abstractions
- Delays architectural decisions until you have more information
- Makes AI-assisted modifications easier (clear, concrete code is easier for AI to modify than abstract code)

### Instruction Example

```
It's okay to have some duplication. Don't create shared utilities or abstractions
unless the same code appears in at least 3 places. DRY is less important than clarity
and simplicity at this stage.
```

### Recognize True Abstraction Needs

Abstractions serve two legitimate purposes:
1. **Keeping patterns synchronized**: When the same logic must stay in sync across multiple locations
2. **Encapsulating complexity**: When hiding implementation details genuinely improves code clarity

If an abstraction doesn't serve one of these purposes, you don't need it yet.

## Keeping Refactoring Focused

Refactoring is necessary, but unfocused refactoring is a major source of bloat.

### Explicit Refactoring Boundaries

```
Refactor only the [specific module/function/class]. Do not refactor code that calls
this module or code that this module calls. Keep changes contained.
```

### The "Red-Green-Refactor" Constraint for AI

```
Follow strict TDD:
1. Red: Write failing test
2. Green: Write minimal code to pass
3. Refactor: Only clean up the code you just wrote

Do not refactor existing code outside this cycle.
```

### Prevent Scope Creep During Refactoring

```
Refactor [X] to improve [specific quality like performance/readability].
During refactoring:
- Do not change functionality
- Do not add features
- Do not refactor unrelated code you notice along the way
- Keep all existing tests passing
```

## Signs That Claude Is Over-Engineering

Recognizing over-engineering in progress helps you course-correct quickly. Watch for these warning signs:

### File and Code Volume Red Flags

- **Too many new files**: A simple feature shouldn't create 10+ new files
- **Excessive line counts**: If you asked for a small change and see hundreds of lines modified
- **Deep directory nesting**: Creating complex folder hierarchies for simple features

### Abstraction Warning Signs

- **Layers upon layers**: Multiple levels of abstraction (interfaces, base classes, adapters, facades) for simple functionality
- **Overuse of patterns**: Design patterns applied where direct code would be clearer
- **Generic naming**: Files or functions with names like `BaseHandler`, `AbstractProcessor`, `UtilityHelper` that don't reveal their actual purpose
- **Single-use utilities**: Helper functions or classes that are only called from one place

### Architecture Red Flags

- **Premature generalization**: Code that handles cases you don't have yet
- **Excessive configuration**: Configuration files, environment variables, or feature flags for simple behaviors
- **Dependency injection for everything**: DI containers and interfaces where direct instantiation would suffice

### Behavioral Indicators

- **Claude explains complex architecture**: If Claude starts explaining how its multi-layered design works, it's probably over-engineered
- **"This will make it easier to..."**: Watch for justifications about hypothetical future needs
- **Refactoring you didn't ask for**: Claude mentions "while I was there, I also improved..."

### Code Smell Indicators

Per the 6 warning signs of over-engineering:

1. **Hard-to-maintain code**: New team members struggle to understand it, or you spend more time debugging than adding features
2. **Unnecessarily complex architecture**: Abstraction upon abstraction without clear benefit
3. **Over-reliance on patterns**: Design patterns used where simpler code would work
4. **Premature optimization**: Performance optimizations for code that isn't slow
5. **Feature bloat**: More features than needed to solve the actual problem
6. **Gold plating**: Code that's "clever" rather than clear

## Recovery Techniques When Claude Goes Overboard

When Claude produces over-engineered code, you have several recovery options:

### 1. Built-in Rewind Feature (Fastest)

Claude Code has built-in checkpointing and rewind capabilities:

- **Quick rewind**: Press `Esc` twice (`Esc + Esc`) or use `/rewind` command
- **Selective restoration**: Choose what to restore:
  - **Conversation only**: Reset Claude's mental state but keep code changes
  - **Code only**: Revert file changes but keep conversation history
  - **Both**: Restore both conversation and code to a prior point

**When to use conversation-only restore**: When Claude's context is bloated with wrong-direction discussion and keeps referencing decisions you rejected. The conversation-only restore keeps your working code but resets Claude's mental state. When you continue, Claude reads your files fresh without the baggage of confused conversation history.

### 2. Redirect with Explicit Instructions

Instead of reverting, redirect Claude with strong constraints:

```
Stop. This is too complex. Let's simplify.

Remove all abstractions and helper functions. Implement this as a single,
straightforward function in [file]. No design patterns, no layers, no utilities.
Just direct, simple code.
```

### 3. Start Fresh with Better Constraints

If redirection doesn't work:

```
Let's start over with a simpler approach. Ignore the previous implementation.

Here's what I need: [specific requirement]

Constraints:
- Single file implementation
- No abstractions or helper classes
- No configuration systems
- Direct, straightforward code

Show me the plan first before implementing.
```

### 4. The "Show Me the Plan" Pattern

Prevent over-engineering before it happens:

```
Before writing any code, create a detailed plan showing:
1. Exactly which files you'll modify
2. What changes you'll make to each file
3. Why each change is necessary

Wait for my approval before proceeding.
```

This gives you a chance to catch over-engineering in the planning phase.

### 5. Git-Based Recovery

For more permanent recovery beyond session-level undo:

- Use standard Git operations: `git diff`, `git checkout`, `git reset`
- Claude Code checkpoints are for quick session recovery
- Git is for permanent history and collaboration
- Think of checkpoints as "local undo" and Git as "permanent history"

### 6. Incremental Simplification

If you want to salvage some work:

```
The current implementation is too complex. Simplify it step by step:

1. First, inline all single-use helper functions
2. Then, remove the [specific abstraction layer]
3. Then, consolidate files [A, B, C] into a single file
4. Finally, remove configuration options that aren't used

Do these one step at a time so I can review each.
```

### 7. Use Third-Party Tools

For more sophisticated rollback capabilities:

- **claude-code-rewind**: Captures every Claude Code action, allows selective rollback
- **ccundo**: Creates automatic snapshots at strategic points, supports pattern-based selective restoration

### 8. The Nuclear Option: Version Rollback

If Claude Code itself is behaving problematically after an update:

```bash
npm install @anthropic-ai/[email protected]
```

This downgrades Claude Code to a previous version.

## Using CLAUDE.md to Set Project-Wide Standards

The `CLAUDE.md` file is automatically pulled into context at the start of every conversation. Use it to establish project-wide minimalism standards:

### Example CLAUDE.md Section

```markdown
## Code Complexity Guidelines

**Philosophy**: Less code = fewer bugs. Keep it minimal.

**Rules**:
- Avoid creating abstractions until code is used in 3+ places
- Don't add error handling for scenarios that can't happen
- Simple features get simple implementations (single file preferred)
- Bug fixes don't refactor surrounding code
- Follow existing patterns exactly; don't "improve" them
- Don't design for hypothetical future requirements

**When making changes**:
1. Make only the changes explicitly requested
2. Reuse existing utilities rather than creating new ones
3. If you think something additional is needed, ask first

**Project Quirks**:
- [Document actual project-specific constraints that prevent over-engineering]
```

### Optimize CLAUDE.md Based on Your Codebase

Research shows that repository-specific prompt optimization yields significant improvements (+10.87% on SWE Bench Lite). Tailor your CLAUDE.md to your actual codebase:

- Document **real project quirks** rather than generic best practices
- Include **patterns specific to your code** that Claude should follow
- Use **LLM feedback on failures** to refine instructions over time
- Focus on **concrete examples** from your codebase

## Working Effectively Within Claude 4.x Behavior

Claude Opus 4.5 and Sonnet 4.5 have different behavior than earlier models. Understanding these changes helps you prompt effectively:

### More Conservative by Default

Claude 4.x models are more conservative about suggesting vs. implementing. Be explicit:

- **Less effective**: "Can you suggest improvements?"
- **More effective**: "Implement these specific changes: [list]"

### Responds to System Prompts More Aggressively

Claude Opus 4.5 responds more strongly to system-level instructions. This has two implications:

1. **Your anti-bloat instructions will be more effective**: The explicit constraints you add will have stronger effect
2. **Moderate your directive intensity**: Overly forceful language like "CRITICAL: You MUST" may cause over-triggering. The system prompt evolution shows that absolute commands got softened: "MUST" became "should," "NEVER" got qualified with "unless"

### Best Approach for Claude 4.x

Use clear, explicit, but measured language:

```
Keep solutions simple and minimal. Only make requested changes.
Don't add abstractions, extra features, or speculative error handling.
```

Rather than:

```
CRITICAL: You MUST NEVER add anything beyond what I asked! ABSOLUTELY NO abstractions!
```

## Multi-Agent Workflows and Complexity

When using subagents and multi-step workflows, be aware of complexity multiplication:

### When NOT to Use Multi-Agent Approaches

Keep it simple when:
- Your task is linear or involves unified reasoning
- Real-time responses and low latency are important
- The problem fits within a single context window
- You're working on a small, straightforward feature

### Signs Multi-Agent is Adding Unnecessary Complexity

- Repeated token overflows
- Conflicting outputs between agents
- Coordination overhead exceeds benefits
- You're spending more time debugging agent handoffs than actual code

### Keep Custom Agents Lightweight

- **Heavy custom agents** (25k+ tokens): Create bottlenecks
- **Lightweight custom agents** (under 3k tokens): Enable fluid orchestration

Start with one simple, focused custom agent that solves a specific problem, then expand only if you see measurable gains.

## The Explore-Plan-Code-Commit Workflow

Claude Code's recommended workflow inherently prevents scope creep when used correctly:

### 1. Explore Phase

```
Read these files: [specific files]
Don't write any code yet. Just understand the current implementation.
```

Use subagents to verify details without consuming main context.

### 2. Plan Phase

```
Create a minimal plan to accomplish [specific goal].
Show me exactly what you'll change and why.
```

This is your checkpoint to catch over-engineering before implementation.

### 3. Code Phase

```
Implement according to the approved plan. Don't add anything beyond the plan.
```

Having an explicit plan prevents "while I was there" additions.

### 4. Commit Phase

```
Review the changes. Do they match what we planned?
If yes, commit with message: [specific message]
```

Final verification that scope hasn't crept.

## Real-World Examples

### Example 1: The Calendar Widget

**User Request:**
```
Add a calendar widget to show events
```

**Over-Engineered Response:**
- Creates `CalendarWidget.tsx`, `CalendarTypes.ts`, `CalendarUtils.ts`, `CalendarHooks.ts`, `CalendarContext.tsx`
- Adds timezone handling, recurring events, multiple view modes, export functionality
- Creates configuration system for customizing appearance
- Implements caching layer for event data

**Properly Scoped Request:**
```
Add a calendar widget following the pattern from EventList.tsx.
Show events from the events array in a simple month grid view.
No timezone handling, no recurring events, no configuration options.
Just display dates and event titles.
```

**Minimal Response:**
- Single `CalendarWidget.tsx` file
- Displays events in month grid
- Uses existing event data structure
- ~80 lines of code

### Example 2: Authentication Bug Fix

**User Request:**
```
Fix the authentication bug
```

**Over-Engineered Response:**
- Refactors entire auth system to use dependency injection
- Adds comprehensive error handling and logging
- Creates AuthService, AuthProvider, AuthValidator classes
- Implements retry logic and fallback mechanisms
- Updates all files that touch authentication

**Properly Scoped Request:**
```
Fix the bug in auth.py line 142 where token expiry isn't checked correctly.
Change only that specific condition. Don't refactor the auth system.
```

**Minimal Response:**
- Changes single conditional in auth.py
- 1 line modified
- Bug fixed

## Conclusion

Over-engineering is Claude's natural tendency, but it's completely controllable through explicit prompting and workflow discipline. The key principles are:

1. **Be explicit about constraints**: Claude won't assume minimalism; you must request it
2. **Specify scope clearly**: Vague requests get bloated responses
3. **Plan before implementing**: Catch over-engineering in the planning phase
4. **Use recovery tools**: Rewind and reset when things go off track
5. **Optimize over time**: Use CLAUDE.md to encode lessons learned

Remember: The right amount of complexity is the minimum needed for the current task. Less code means fewer bugs, and existing code is tested code. Keep it minimal, keep it maintainable.

---

## Sources

- [Claude Code: Best practices for agentic coding - Anthropic](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Prompting best practices - Claude Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-4-best-practices)
- [How to Stop Claude Code From Overengineering Everything - Nathan Onn](https://www.nathanonn.com/how-to-stop-claude-code-from-overengineering-everything/)
- [CLAUDE.md: Best Practices Learned from Optimizing Claude Code - Arize](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/)
- [Checkpointing - Claude Code Docs](https://code.claude.com/docs/en/checkpointing)
- [The 6 warning signs of overengineering - LeadDev](https://leaddev.com/software-quality/the-6-warning-signs-of-overengineering)
- [Common pitfalls when building generative AI applications - Chip Huyen](https://huyenchip.com/2025/01/16/ai-engineering-pitfalls.html)
- [The Overlooked Costs of Agentic AI - AryaXAI](https://www.aryaxai.com/article/the-overlooked-costs-of-agentic-ai)
- [Managing AI coding assistants without losing control - LeadDev](https://leaddev.com/ai/managing-ai-coding-assistants-without-losing-control)
- [Claude Code Rewind - GitHub](https://github.com/holasoymalva/claude-code-rewind)
- [How Anthropic teams use Claude Code - PDF](https://www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf)
- [Builder.io: How I use Claude Code](https://www.builder.io/blog/claude-code)
- [ClaudeLog - Claude Code Documentation](https://claudelog.com/)
