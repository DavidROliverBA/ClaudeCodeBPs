# Claude Code Best Practices

A comprehensive collection of best practices, guides, and documentation for using Claude Code effectively. This repository covers everything from basic configuration to advanced workflows, compiled from Anthropic's official documentation, community expertise, and real-world usage patterns.

## Table of Contents

### Core Configuration

| # | Topic | Description |
|---|-------|-------------|
| 1 | [CLAUDE.md Files](docs/core-configuration/01-claude-md-files.md) | Creating project-specific context and instructions; hierarchical loading; coding standards and architecture documentation |
| 2 | [Settings Hierarchy](docs/core-configuration/02-settings-hierarchy.md) | User settings, project settings, and local settings configuration |
| 3 | [Permissions Management](docs/core-configuration/03-permissions-management.md) | Configuring allowed/denied tools, file access patterns, and balancing automation with safety |

### Prompting & Interaction

| # | Topic | Description |
|---|-------|-------------|
| 4 | [Effective Prompting](docs/prompting-interaction/04-effective-prompting.md) | Being explicit with instructions, providing context and motivation, specificity for first-attempt success |
| 5 | [Extended Thinking](docs/prompting-interaction/05-extended-thinking.md) | Using keywords ("think", "think harder", "ultrathink") for complex reasoning; Tab toggle; MAX_THINKING_TOKENS |
| 6 | [Output Format Control](docs/prompting-interaction/06-output-format-control.md) | Specifying code style, variable naming conventions, documentation formats, reducing verbose output |
| 7 | [Avoiding Over-engineering](docs/prompting-interaction/07-avoiding-over-engineering.md) | Prompting Claude to keep solutions minimal and focused |

### Automation & Workflows

| # | Topic | Description |
|---|-------|-------------|
| 8 | [Custom Slash Commands](docs/automation-workflows/08-custom-slash-commands.md) | Creating reusable commands in .claude/commands/ for common workflows |
| 9 | [Hooks](docs/automation-workflows/09-hooks.md) | Pre- and post-tool-use automation (linting, formatting, type-checking) |
| 10 | [Skills](docs/automation-workflows/10-skills.md) | Markdown-based guides for domain-specific tasks invoked via natural language |
| 11 | [CI/CD Integration](docs/automation-workflows/11-cicd-integration.md) | Using print mode (-p flag) for scripting and pipeline automation |

### Advanced Usage

| # | Topic | Description |
|---|-------|-------------|
| 12 | [MCP (Model Context Protocol)](docs/advanced-usage/12-mcp-model-context-protocol.md) | Connecting to external services, databases, APIs; configuring .mcp.json |
| 13 | [Subagents](docs/advanced-usage/13-subagents.md) | Configuring specialized agents (architect, builder, QA, reviewer) with defined responsibilities |
| 14 | [Git Worktrees](docs/advanced-usage/14-git-worktrees.md) | Running parallel Claude instances with isolated code states |
| 15 | [Session Management](docs/advanced-usage/15-session-management.md) | Using /clear for token efficiency, command history, conversation resumption (-c, -r flags) |

### Development Practices

| # | Topic | Description |
|---|-------|-------------|
| 16 | [Test-Driven Development](docs/development-practices/16-test-driven-development.md) | Leveraging Claude for robust test coverage and TDD workflows |
| 17 | [Prompt Planning](docs/development-practices/17-prompt-planning.md) | Using spec.md and prompt_plan.md files for structured multi-step implementations |
| 18 | [Code Review Integration](docs/development-practices/18-code-review-integration.md) | Configuring claude-code-review.yml for PR reviews; customizing review focus |
| 19 | [Image and Diagram Analysis](docs/development-practices/19-image-diagram-analysis.md) | Using screenshots and image files for visual context |

### Quality & Cost Optimization

| # | Topic | Description |
|---|-------|-------------|
| 20 | [Model Selection](docs/quality-cost/20-model-selection.md) | Strategic switching between Opus and Sonnet based on task complexity and cost |
| 21 | [Hallucination Reduction](docs/quality-cost/21-hallucination-reduction.md) | Prompting Claude to investigate files before answering |
| 22 | [Context Management](docs/quality-cost/22-context-management.md) | Handling compaction, saving state before context refresh, persistent autonomous operation |
| 23 | [File Cleanup](docs/quality-cost/23-file-cleanup.md) | Instructing Claude to remove temporary files after iteration |

## Quick Start

1. **New to Claude Code?** Start with [CLAUDE.md Files](docs/core-configuration/01-claude-md-files.md) and [Effective Prompting](docs/prompting-interaction/04-effective-prompting.md)

2. **Setting up a team project?** Read [Settings Hierarchy](docs/core-configuration/02-settings-hierarchy.md) and [Permissions Management](docs/core-configuration/03-permissions-management.md)

3. **Want to automate workflows?** Check out [Custom Slash Commands](docs/automation-workflows/08-custom-slash-commands.md) and [Hooks](docs/automation-workflows/09-hooks.md)

4. **Optimizing costs?** See [Model Selection](docs/quality-cost/20-model-selection.md) and [Context Management](docs/quality-cost/22-context-management.md)

## Document Statistics

| Category | Documents | Total Words (approx) |
|----------|-----------|---------------------|
| Core Configuration | 3 | ~10,000 |
| Prompting & Interaction | 4 | ~12,000 |
| Automation & Workflows | 4 | ~20,000 |
| Advanced Usage | 4 | ~22,000 |
| Development Practices | 4 | ~18,000 |
| Quality & Cost Optimization | 4 | ~16,000 |
| **Total** | **23** | **~98,000** |

## Sources

All documents are researched from authoritative sources including:

- **Official Anthropic Documentation**: [docs.anthropic.com](https://docs.anthropic.com), [code.claude.com](https://code.claude.com)
- **Anthropic Engineering Blog**: Best practices and technical deep-dives
- **Community Resources**: GitHub repositories, developer blogs, and tutorials
- **Real-world Usage**: Patterns from production implementations

Each document includes a comprehensive sources section with direct links to reference materials.

## Contributing

Found an error or have additional best practices to share? Contributions are welcome!

## License

This documentation is provided for educational purposes. Please refer to Anthropic's terms of service for Claude Code usage.

---

*Last updated: January 2026*
