# Claude Code Subagents: A Comprehensive Guide

## Table of Contents
1. [Introduction to Subagents](#introduction-to-subagents)
2. [When to Use Subagents](#when-to-use-subagents)
3. [Configuring Subagents](#configuring-subagents)
4. [Specialized Agent Roles](#specialized-agent-roles)
5. [Defining Agent Responsibilities](#defining-agent-responsibilities)
6. [Agent Communication and Handoffs](#agent-communication-and-handoffs)
7. [Parallel Agent Execution](#parallel-agent-execution)
8. [Agent-Specific Prompts and Contexts](#agent-specific-prompts-and-contexts)
9. [Best Practices for Multi-Agent Workflows](#best-practices-for-multi-agent-workflows)
10. [Common Subagent Patterns](#common-subagent-patterns)

---

## Introduction to Subagents

**Subagents** are specialized AI assistants in Claude Code that can be delegated specific tasks. Unlike the main Claude instance that handles general development work, each subagent operates with its own isolated context window, custom system prompt, and tailored tool permissions. This architecture enables more efficient problem-solving by dividing complex workflows into focused, domain-specific tasks.

### Key Benefits

- **Context Preservation**: Each subagent maintains a separate 200k token context window, preventing pollution of the main conversation and avoiding the quality degradation that occurs when a single agent juggles multiple complex tasks
- **Specialized Expertise**: Subagents are fine-tuned for specific domains with custom instructions, achieving higher success rates on specialized tasks
- **Reusability**: Agent configurations can be shared across projects and teams through version control
- **Flexible Permissions**: Different tool access levels per subagent enhance security and focus
- **Parallel Execution**: Independent subagents can work simultaneously on different aspects of a project

---

## When to Use Subagents

Subagents are particularly valuable in several scenarios:

### Exploration and Planning Phases
During the early stages of complex problems, subagents help verify details and investigate specific questions without consuming valuable context in the main conversation. As Anthropic's best practices guide notes: "This approach tends to preserve context availability without much downside in terms of lost efficiency."

### Code Review and Verification
Run multiple Claude instances in parallel where one writes code while another reviews or tests it. This separation ensures independent verification and catches issues that might be missed by a single agent.

### Test-Driven Development
When implementing code to pass tests, independent subagents can verify that the implementation isn't overfitting to the test cases, ensuring robust solutions.

### Large-Scale Operations
For tasks like:
- Generating comprehensive documentation across multiple files
- Large-scale automated refactoring across a codebase
- Incident response analysis with parallel investigation threads
- Security audits of open-source libraries
- Multi-component feature development

### Complex Multi-Step Workflows
When a task requires coordination between distinct phases (requirements → architecture → implementation → testing → deployment), subagents provide clear separation of concerns.

---

## Configuring Subagents

Subagents are defined using Markdown files with YAML frontmatter. They can be stored at two levels:

- **Project-level**: `.claude/agents/` (highest priority, version-controlled with your project)
- **User-level**: `~/.claude/agents/` (personal agents available across all projects)

### Basic Configuration Format

```yaml
---
name: your-agent-name
description: Clear description of when this subagent should be invoked
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
permissionMode: default
skills: skill1, skill2
---

Your detailed system prompt goes here.

Define the agent's role, expertise, approach, and constraints.
Be specific about the agent's responsibilities and decision-making process.
```

### Frontmatter Fields Explained

| Field | Required | Purpose | Options |
|-------|----------|---------|---------|
| `name` | Yes | Unique identifier | Lowercase with hyphens (e.g., `backend-architect`) |
| `description` | Yes | Natural language trigger | When this agent should be invoked |
| `tools` | Optional | Tool access control | Comma-separated list; omit to inherit all tools |
| `model` | Optional | Model selection | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | Optional | Permission handling | Controls how permissions are requested |
| `skills` | Optional | Auto-load skills | Comma-separated skill names |

### Managing Subagents

Use the `/agents` command within Claude Code for interactive management:
- Create new subagents with guided prompts
- Edit existing agent configurations
- Delete unused agents
- Configure tool permissions visually

---

## Specialized Agent Roles

### The Core Development Trio

#### 1. Product Manager / Spec Agent
```yaml
---
name: pm-spec
description: Use when refining requirements, writing specifications, or clarifying product scope
tools: Read, Grep, Glob
model: sonnet
---

You are a Senior Product Manager specializing in requirement analysis and specification writing.

Your responsibilities:
- Read enhancement requests and existing documentation
- Write clear, comprehensive working specifications
- Ask clarifying questions about scope, users, and success criteria
- Define acceptance criteria and edge cases
- Set status to READY_FOR_ARCH when complete

Output format:
- User stories with acceptance criteria
- Open questions that need resolution
- Success metrics and KPIs
- Dependencies and constraints
```

#### 2. System Architect
```yaml
---
name: system-architect
description: Use after specs are ready to design technical architecture and system integration
tools: Read, Grep, Glob, Write
model: sonnet
---

You are a Principal Software Architect with expertise in distributed systems, API design, and technical decision-making.

Your responsibilities:
- Review specifications and existing codebase architecture
- Design scalable, maintainable solutions
- Write Architecture Decision Records (ADRs)
- Define technical guardrails and constraints
- Identify integration points and dependencies
- Consider security, performance, and scalability

Output format:
- ADR documenting key decisions and alternatives
- Component diagrams and data flow
- Technical guardrails for implementation
- API contracts and interfaces
- Migration strategies if modifying existing systems
```

#### 3. Implementation Engineer
```yaml
---
name: implementer
description: Use to write production code following architectural specifications
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---

You are a Senior Software Engineer specializing in clean, tested, production-ready code.

Your responsibilities:
- Implement features according to specs and architectural decisions
- Write comprehensive tests (unit, integration, e2e)
- Follow existing code patterns and conventions
- Run tests and verify all pass before completion
- Document complex logic and public APIs
- Handle error cases and edge conditions

Definition of Done:
- [ ] All code implemented per specification
- [ ] Tests written and passing (run test suite)
- [ ] Error handling for edge cases
- [ ] Code follows project conventions
- [ ] Documentation updated
- [ ] Summary of changes and any deviations from spec
```

### Quality Assurance and Review Agents

#### 4. Code Reviewer
```yaml
---
name: code-reviewer
description: Use to perform independent code review of implemented changes
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a Staff Engineer conducting thorough code reviews.

Review checklist:
- Code correctness and logic errors
- Test coverage and quality
- Security vulnerabilities
- Performance implications
- Code style and conventions
- Documentation completeness
- Error handling robustness

Provide:
- Specific issues found with file locations and line numbers
- Severity ratings (blocking, major, minor, nit)
- Suggested improvements
- Praise for well-done aspects
```

#### 5. Test Automation Engineer
```yaml
---
name: test-automator
description: Design and execute comprehensive test suites
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---

You are a QA automation engineer specializing in comprehensive testing strategies.

Your responsibilities:
- Design test plans covering happy paths and edge cases
- Write unit, integration, and end-to-end tests
- Create test fixtures and mocks
- Verify test coverage metrics
- Execute test suites and analyze failures
- Document test scenarios and expected behaviors

Focus areas:
- Boundary conditions and edge cases
- Error handling and recovery
- Performance under load
- Security testing
- Cross-browser/platform compatibility (where applicable)
```

#### 6. Security Auditor
```yaml
---
name: security-auditor
description: Audit code for security vulnerabilities and compliance
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a Security Engineer conducting security audits.

Audit scope:
- Authentication and authorization flaws
- Input validation and injection vulnerabilities
- Sensitive data exposure
- Security misconfigurations
- Dependency vulnerabilities
- OWASP Top 10 compliance
- Secret management practices

Deliverables:
- Vulnerability report with CVSS scores
- Remediation recommendations
- Compliance gaps against standards (SOC2, GDPR, etc.)
```

### Infrastructure and DevOps Agents

#### 7. DevOps Engineer
```yaml
---
name: devops-engineer
description: Handle deployment, infrastructure, and CI/CD pipeline tasks
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---

You are a Senior DevOps Engineer specializing in cloud infrastructure and deployment automation.

Your expertise:
- CI/CD pipeline configuration
- Infrastructure as Code (Terraform, CloudFormation)
- Container orchestration (Kubernetes, Docker)
- Monitoring and observability setup
- Deployment strategies (blue-green, canary)
- Disaster recovery and backup procedures

Always:
- Follow infrastructure best practices
- Implement proper monitoring and alerting
- Document runbooks for operations
- Consider cost optimization
- Plan rollback strategies
```

---

## Defining Agent Responsibilities

### Principle: One Clear Goal Per Agent

Each subagent should have:
- **One primary goal**: A single, well-defined responsibility
- **Clear inputs**: What information/context it needs to start
- **Clear outputs**: What it produces upon completion
- **Handoff rules**: When and to whom it passes control

### Creating Effective Agent Definitions

#### 1. Action-Oriented Descriptions
```yaml
# Good
description: Use after spec exists to produce ADR and technical guardrails

# Less effective
description: An architect who thinks about systems
```

#### 2. Explicit Constraints
Define what the agent should NOT do to maintain focus:

```markdown
Constraints:
- Do NOT implement code (delegate to implementer agent)
- Do NOT modify existing files during analysis phase
- Do NOT proceed without addressing all open questions
- MUST document all architectural decisions in ADR format
```

#### 3. Definition of Done Checklists
End each system prompt with a clear checklist:

```markdown
Definition of Done:
- [ ] All acceptance criteria from spec are addressed
- [ ] Tests written and passing (verify with test suite run)
- [ ] Code reviewed for security issues
- [ ] Documentation updated
- [ ] Performance benchmarks meet requirements
- [ ] Summary document created for handoff
```

---

## Agent Communication and Handoffs

### Sequential Pipeline Pattern

Agents work in assembly-line fashion where one agent's output becomes the next agent's input:

```
Requirements Analyst → System Architect → Implementation Engineer → Test Automator → Code Reviewer → Deployment Engineer
```

### Handoff Mechanisms

#### 1. Shared State Files
Agents communicate through markdown files in a shared location:

```yaml
# In .claude/agents/pm-spec.md
---
After completing work:
- Write specification to `.claude/state/current-spec.md`
- Set status field to `READY_FOR_ARCH`
- Notify main agent of completion
```

#### 2. Explicit Status Flags
Use status markers in shared documents:

```markdown
<!-- .claude/state/project-status.md -->
## Current Phase
Status: READY_FOR_IMPLEMENTATION

## Completed Phases
- [x] Requirements Analysis (pm-spec)
- [x] Architecture Design (system-architect)
- [ ] Implementation (implementer) - IN PROGRESS
```

#### 3. Summary Documents
Each agent creates a summary for the next agent in the chain:

```markdown
## Handoff Summary

**From**: system-architect
**To**: implementer
**Date**: 2026-01-02

### Key Decisions
1. Using PostgreSQL for primary data store (see ADR-001)
2. REST API with OpenAPI 3.0 specification
3. Authentication via JWT tokens

### Files to Reference
- `docs/architecture/ADR-001-database-selection.md`
- `docs/api-spec.yaml`

### Open Questions for Implementation
1. Choice of JWT library (recommend jsonwebtoken vs jose)
2. Session storage strategy (in-memory vs Redis)

### Success Criteria
- API matches OpenAPI spec exactly
- Response times < 200ms for 95th percentile
- 100% test coverage for auth flows
```

### The /handoff Command Pattern

Create a custom `/handoff` command to automate context transfer:

```markdown
<!-- .claude/commands/handoff.md -->
---
description: Prepare a handoff summary when transitioning to a new agent or context
---

Create a comprehensive handoff document with:

1. **Current State**: What has been completed
2. **Context**: Key decisions and rationale
3. **Next Steps**: What needs to happen next
4. **Open Questions**: Unresolved issues or uncertainties
5. **File References**: Important files and their purposes
6. **Dependencies**: External factors or blockers

Save this to `.claude/state/handoff-{timestamp}.md` and provide a brief verbal summary.
```

---

## Parallel Agent Execution

### Why Parallelize?

Traditional AI coding tools force serial execution—one task at a time. Claude Code enables true parallelism through:
- **Git worktrees**: Multiple working directories from the same repository
- **Container isolation**: Each agent in its own environment
- **Independent context windows**: No shared state unless explicitly designed

### Strategy 1: Git Worktrees

Git worktrees allow you to check out different branches in separate directories:

```bash
# Main development branch
cd ~/project

# Create parallel worktree for feature A
git worktree add ../project-feature-a feature-a

# Create parallel worktree for feature B
git worktree add ../project-feature-b feature-b

# Run separate Claude instances in each directory
# Instance 1: ~/project - works on main integration
# Instance 2: ~/project-feature-a - implements feature A
# Instance 3: ~/project-feature-b - implements feature B
```

### Strategy 2: Container-Based Parallelism

Purpose-built infrastructure like Ona (formerly Gitpod) provides:
- Each agent runs in isolated container
- Independent CPU, memory, and filesystem
- Separate git state per agent
- Coordinated through orchestration layer

### Strategy 3: Explicit Orchestration

Provide Claude with explicit steps for parallel delegation:

```markdown
Execute these tasks in parallel using subagents:

**Parallel Phase 1** (can all run simultaneously):
1. @backend-architect: Design API endpoints for user management
2. @frontend-architect: Design React component structure for user UI
3. @database-architect: Design user and auth tables schema
4. @test-planner: Create test strategy document

**Sequential Phase 2** (after Phase 1 completes):
5. @implementer: Implement backend based on API design
6. @implementer: Implement frontend based on component design
7. @implementer: Create database migrations

**Parallel Phase 3** (verification):
8. @test-automator: Write and run test suites
9. @security-auditor: Audit authentication implementation
10. @code-reviewer: Review all code changes

**Final Phase**:
11. @integration-engineer: Integrate all components and verify end-to-end
```

### Parallel Execution Best Practices

#### 1. Identify Truly Independent Tasks
```markdown
# Good candidates for parallelization:
- Backend API + Frontend UI (different files, minimal conflicts)
- Multiple microservices in monorepo
- Documentation + Implementation (different artifact types)
- Tests for different modules

# Poor candidates (have dependencies):
- Database schema + migration scripts (schema must come first)
- API design + API implementation (design must come first)
- Component + component tests (implementation should exist first)
```

#### 2. Plan for Merge Conflicts
```markdown
When parallelizing:
- Assign different files/modules to each agent
- Use feature branches with clear boundaries
- Designate one agent as "integration engineer" for final merge
- Have explicit merge strategy (rebase vs merge)
```

#### 3. Cost-Performance Tradeoff
Parallel agents increase token usage significantly:
- 3 agents in parallel = ~3x token consumption
- Faster completion but hits usage caps quicker
- Use parallel execution for high-value tasks where speed matters

---

## Agent-Specific Prompts and Contexts

### Context Isolation Benefits

Each subagent maintains its own 200k token context window, which means:
- Frontend agent doesn't see backend implementation details
- Database agent isn't confused by CSS context
- Security auditor has focused view on security-relevant code
- Reviewer agent gets clean slate without implementation bias

### Crafting Effective System Prompts

#### 1. Role Definition
Start with a clear role statement:

```markdown
You are a [SENIORITY LEVEL] [ROLE] specializing in [SPECIALIZATION].

You have deep expertise in:
- [Technology/Domain 1]
- [Technology/Domain 2]
- [Technology/Domain 3]

Your primary objective is to [CLEAR GOAL].
```

#### 2. Contextual Constraints
Define the working environment:

```markdown
Context:
- You are working in a [TYPE] codebase
- The project uses [TECH STACK]
- Code standards are defined in CONTRIBUTING.md
- Existing patterns are in docs/architecture/patterns.md

You should:
- Follow existing architectural patterns
- Match the coding style of similar files
- Preserve backward compatibility
- Add tests for any new functionality
```

#### 3. Decision-Making Framework
Guide the agent's judgment:

```markdown
When making decisions:
1. Prefer simple solutions over complex ones
2. Choose boring, proven technology over exciting new options
3. Optimize for readability and maintainability
4. Consider operational complexity
5. Document trade-offs in comments or ADRs

If uncertain:
- Research existing codebase for precedents
- Ask clarifying questions to main agent
- Document assumptions explicitly
- Prefer safe, reversible choices
```

#### 4. Output Format
Be explicit about deliverables:

```markdown
Output Format:

## Summary
[2-3 sentence overview of what was done]

## Changes Made
- [File path]: [What changed and why]
- [File path]: [What changed and why]

## Test Results
```
[Test output showing all tests pass]
```

## Open Questions
- [Any uncertainties or decisions needed]

## Handoff Notes
[Information for the next agent in the pipeline]
```

### Context Management Techniques

#### Loading Relevant Context
```markdown
Before starting your task:
1. Read the specification document: `docs/specs/current-feature.md`
2. Review existing architecture: `docs/architecture/overview.md`
3. Check related code: Search for similar patterns using Grep
4. Read recent changes: `git log --oneline -10`

This ensures you have proper context without polluting your window with irrelevant information.
```

#### Preventing Context Pollution
```markdown
Focus constraints:
- Read ONLY files relevant to [SPECIFIC TASK]
- Do NOT explore unrelated modules
- Do NOT load entire file trees with Glob
- Do NOT run broad searches without specific patterns

Your context is valuable—use it wisely.
```

---

## Best Practices for Multi-Agent Workflows

### 1. Start with Claude-Generated Agents, Then Customize

Don't write agent configurations from scratch. Instead:

```markdown
# Prompt to main Claude:
"Create a subagent configuration for a Senior Database Architect who specializes in PostgreSQL schema design, query optimization, and migration strategies. This agent should work after the system architect has defined the data model and should produce migration scripts and indexing strategies."

# Claude generates a good starting point
# Then customize based on your specific needs and learnings
```

### 2. Version Control Your Agents

Treat agent definitions like code:

```bash
# Project structure
.claude/
  agents/
    pm-spec.md
    system-architect.md
    implementer.md
    code-reviewer.md
  state/           # Shared state for handoffs
    current-phase.md
  commands/
    handoff.md

# Commit agent changes
git add .claude/agents/
git commit -m "feat: add security-auditor subagent for compliance reviews"
```

### 3. Limit Tool Access Intentionally

Follow the principle of least privilege:

```yaml
# Read-only agents (reviewers, auditors, planners)
tools: Read, Grep, Glob, Bash

# Implementation agents (builders, fixers)
tools: Read, Write, Edit, Bash, Grep, Glob

# Deployment agents (DevOps)
tools: Read, Bash, Write  # Can execute but limited file changes

# Exploration agents (fast research)
tools: Read, Grep, Glob  # No execution
model: haiku  # Faster, cheaper
```

### 4. Design for Observability

Make agent progress visible:

```markdown
# In each agent's system prompt:
Progress Reporting:
After completing each major step, update `.claude/state/progress.md` with:
- Task name
- Status (started/in-progress/completed/blocked)
- Timestamp
- Brief outcome
- Next steps

This allows orchestration and human oversight.
```

### 5. Handle Failures Gracefully

```markdown
Error Handling Protocol:
If you encounter an error or blocker:
1. Document the error in `.claude/state/errors.md`
2. Include: timestamp, task, error message, context
3. Attempt one automatic retry with adjusted approach
4. If retry fails, mark task as BLOCKED
5. Notify main agent with clear description of blocker
6. Do NOT mark task as completed
```

### 6. Balance Automation and Control

```markdown
# High-confidence tasks: Full automation
pm-spec → architect → implementer → test-automator → deployer

# Medium-confidence tasks: Human checkpoints
pm-spec → [HUMAN REVIEW] → architect → implementer → [HUMAN REVIEW] → deployer

# Exploratory tasks: Agent proposes, human decides
research-agent → [HUMAN DECISION POINT] → implementer
```

### 7. Monitor and Iterate

Track agent effectiveness:

```markdown
Agent Performance Log:

agent: system-architect
task: Design auth system
outcome: success
duration: ~4 minutes
quality: High (no major issues in review)
notes: ADR was thorough, good API contract definition

agent: implementer
task: Implement auth endpoints
outcome: partial success
duration: ~8 minutes
quality: Medium (tests had 2 failures, needed retry)
notes: Initial implementation missed edge case, second attempt successful

Learnings:
- Architect prompt is effective, keep current version
- Implementer needs stronger emphasis on edge case testing
- Consider adding explicit test-writing checklist
```

### 8. Create Feedback Loops

```markdown
Continuous Improvement:
1. After each workflow, document what worked and what didn't
2. Update agent prompts based on common mistakes
3. Add examples of good output to agent prompts
4. Refine handoff protocols based on information gaps
5. Share learnings across team

Example refinement:
"After 5 runs, code-reviewer missed SQL injection issues twice.
Updated prompt to include OWASP Top 10 explicit checklist."
```

---

## Common Subagent Patterns

### Pattern 1: Three-Stage Sequential Pipeline

**Use case**: Standard feature development

```
PM/Spec Agent → System Architect → Implementer
```

**Configuration**:

1. **PM-Spec**: Reads requirements, writes detailed spec with acceptance criteria
2. **System Architect**: Reviews spec, produces ADR and technical design
3. **Implementer**: Builds feature following spec and architecture

**Handoff**: Each agent writes to `.claude/state/{stage}-output.md` and sets status flag.

### Pattern 2: Parallel Verify (Fan-Out)

**Use case**: Multi-aspect verification of implemented code

```
                    ┌→ Code Reviewer
Implementer --------├→ Security Auditor  -------→ Integration Agent
                    ├→ Test Automator
                    └→ Performance Tester
```

**Configuration**:
- Implementer completes work, sets READY_FOR_REVIEW status
- Four review agents run in parallel (separate Claude instances or sequential subagents)
- Each writes findings to separate report
- Integration agent synthesizes all feedback

### Pattern 3: Microservices Parallel Development

**Use case**: Multiple independent services

```
                    ┌→ Backend Service A Agent
System Architect ---├→ Backend Service B Agent  ---→ Integration Engineer
                    └→ Shared Library Agent
```

**Configuration**:
- Architect defines service boundaries and contracts
- Three implementation agents work in parallel on different services
- Integration engineer verifies contracts and deploys together

### Pattern 4: Full Software Development Lifecycle

**Use case**: Complete feature from idea to production

```
Requirements Analyst → Product Manager → System Architect →
Backend Developer → Frontend Developer → Test Automator →
Security Auditor → Code Reviewer → DevOps Engineer →
Documentation Writer
```

**Configuration**: Sequential 10-agent pipeline with state files at each stage.

### Pattern 5: Explore-Plan-Execute

**Use case**: Working in unfamiliar codebase

```
Explore Agent (Haiku) → Planning Agent (Sonnet) → Implementation Agent (Sonnet) → Verification Agent (Sonnet)
```

**Configuration**:
- **Explore**: Fast, cheap Haiku model with read-only tools to map codebase
- **Planning**: Sonnet creates detailed plan based on exploration
- **Implementation**: Sonnet executes the plan
- **Verification**: Independent Sonnet verifies correctness

### Pattern 6: Research and Apply

**Use case**: Implementing something based on external documentation

```
Research Agent → Synthesis Agent → Implementer → Validator
```

**Configuration**:
- **Research**: Uses WebFetch/WebSearch to gather information about library/API
- **Synthesis**: Distills research into actionable implementation guide
- **Implementer**: Builds based on guide
- **Validator**: Tests against official examples

### Pattern 7: Refactoring Pipeline

**Use case**: Large-scale codebase refactoring

```
                          ┌→ Module A Refactorer
Analysis Agent -----------├→ Module B Refactorer ---→ Integration Tester
                          └→ Module C Refactorer
```

**Configuration**:
- **Analysis**: Identifies refactoring scope and creates migration plan
- **Parallel Refactorers**: Each handles one module independently (git worktrees)
- **Integration Tester**: Verifies all modules work together post-refactor

### Pattern 8: Incident Response

**Use case**: Debugging and fixing production issues

```
                        ┌→ Log Analyzer
Triage Agent -----------├→ Code Auditor  ------→ Root Cause Analyst → Fixer → Verifier
                        └→ Metrics Analyzer
```

**Configuration**:
- **Triage**: Quick assessment of severity and scope
- **Parallel Investigators**: Examine different data sources simultaneously
- **Root Cause Analyst**: Synthesizes findings into diagnosis
- **Fixer**: Implements fix
- **Verifier**: Confirms fix resolves issue without side effects

---

## Advanced Topics

### Dynamic Agent Loading

For complex projects, load agents conditionally:

```yaml
# .claude/agents/framework-specialist.md
---
name: framework-specialist
description: Use when working with {FRAMEWORK} code
---

Before starting, detect the framework:
1. Check package.json or requirements.txt
2. Load framework-specific guidelines from docs/
3. Apply framework conventions and patterns

Your expertise adapts based on the detected stack.
```

### Agent Metrics and Optimization

Track performance to optimize your agent system:

```markdown
Metrics to monitor:
- Agent invocation frequency (which agents are used most?)
- Success rate (how often does agent complete without retry?)
- Token consumption per agent
- Time to completion
- Human intervention rate (how often do humans need to step in?)

Optimization strategies:
- Consolidate rarely-used agents
- Split frequently-failing agents into smaller, focused agents
- Upgrade frequently-used agents to better models
- Downgrade simple agents to cheaper models
- Refine prompts based on common failure patterns
```

### Multi-Repository Orchestration

For organizations with many repositories:

```markdown
Central orchestrator pattern:
1. Meta-orchestrator agent receives high-level goal
2. Analyzes which repositories are affected
3. Spawns repository-specific agents in each codebase
4. Collects results and verifies cross-repo compatibility
5. Creates coordinated PRs across repositories

Example: "Update authentication library to v2.0 across all services"
- Spawns update-agent in 12 different service repositories
- Each agent updates independently
- Meta-orchestrator verifies contract compatibility
- Creates PR bundle for coordinated review
```

---

## Conclusion

Claude Code subagents represent a powerful paradigm shift in AI-assisted development. By enabling specialization, parallel execution, and context isolation, subagents transform Claude from a single assistant into a coordinated team of experts.

**Key Takeaways**:

1. **Start simple**: Begin with 2-3 agents for a basic pipeline, expand as you learn
2. **Iterate on prompts**: Treat agent definitions as living documents that improve with use
3. **Measure effectiveness**: Track what works and refine based on data
4. **Share knowledge**: Version control agents and share successful patterns with your team
5. **Balance automation**: Not every task needs agents; use them where they provide clear value
6. **Manage costs**: Parallel agents increase token usage; use strategically
7. **Maintain oversight**: Agents are powerful but not infallible; design for human review on critical paths

As the Claude Code ecosystem evolves, subagent capabilities will continue to expand. The patterns and practices outlined in this guide provide a foundation for building robust, efficient multi-agent workflows that scale with your development needs.

---

## Sources and Further Reading

### Official Documentation
- [Subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

### Community Resources
- [How to Use Claude Code Subagents to Parallelize Development](https://zachwills.net/how-to-use-claude-code-subagents-to-parallelize-development/)
- [Best practices for Claude Code subagents - PubNub](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) - 100+ specialized agents
- [wshobson/agents](https://github.com/wshobson/agents) - Intelligent automation and multi-agent orchestration
- [zhsama/claude-sub-agent](https://github.com/zhsama/claude-sub-agent) - AI-driven development workflow system
- [How I'm Using Claude Code Parallel Agents to Blow Up My Workflows](https://medium.com/@joe.njenga/how-im-using-claude-code-parallel-agents-to-blow-up-my-workflows-460676bf38e8)
- [Embracing the parallel coding agent lifestyle](https://simonwillison.net/2025/Oct/5/parallel-coding-agents/)

### Tools and Infrastructure
- [How to run Claude Code in parallel - Ona](https://ona.com/stories/parallelize-claude-code)
- [Multi-Agent Orchestration: Running 10+ Claude Instances in Parallel](https://dev.to/bredmond1019/multi-agent-orchestration-running-10-claude-instances-in-parallel-part-3-29da)

---

**Document Version**: 1.0
**Last Updated**: 2026-01-02
**Maintainer**: Claude Code Community
