# Model Selection in Claude Code: A Comprehensive Guide

## Introduction

Choosing the right Claude model for your coding tasks is crucial for balancing performance, cost, and development velocity. Claude Code supports multiple models from the Claude 4 family, each optimised for different use cases. This guide provides strategic insights for selecting and switching between Opus, Sonnet, and Haiku models to maximise efficiency and minimise costs.

**Key Recommendation:** If you're unsure which model to use, Anthropic recommends starting with Claude Sonnet 4.5, as it offers the best balance of intelligence, speed, and cost for most use cases, with exceptional performance in coding and agentic tasks.

## Available Models Overview

Claude Code supports three primary model tiers, each with distinct characteristics:

| Model | Primary Use Case | Performance | Cost Efficiency |
|-------|-----------------|-------------|-----------------|
| **Opus 4.5** | Maximum intelligence, complex reasoning, architectural decisions | Highest capability, moderate speed | Most expensive |
| **Sonnet 4.5** | Daily coding tasks, complex agents, production workflows | High capability, fast speed | Best value |
| **Haiku 4.5** | Simple tasks, high-volume operations, speed-critical applications | Good capability, fastest speed | Most economical |

### Model Aliases

Claude Code provides convenient aliases for easy switching:
- `default` - Adapts based on your account type (recommended)
- `opus` - Latest Opus model (currently 4.5)
- `sonnet` - Latest Sonnet model (currently 4.5)
- `haiku` - Latest Haiku model (currently 4.5)
- `opusplan` - Hybrid mode (Opus for planning, Sonnet for execution)
- `sonnet[1m]` - Extended 1 million token context window (Console/API only)

## Strategic Model Switching

### The Hybrid Approach: OpusPlan

The most sophisticated strategy is using the `opusplan` alias, which automatically switches models based on the task phase:

- **Plan Mode**: Uses Opus 4.5 for complex reasoning and architectural decisions
- **Execution Mode**: Automatically switches to Sonnet 4.5 for code generation and implementation

This approach provides "the best of both worlds: Opus's superior reasoning for planning, and Sonnet's efficiency for execution," optimising both quality and cost.

### Task-Based Switching Strategy

The recommended workflow for mixing models is:

1. **Haiku** - Initial setup, file reads, basic content extraction
2. **Sonnet** - Building features, writing logic, managing state, connecting APIs
3. **Opus** - Code reviews, optimisation analysis, architectural decisions

This combination makes workflows "faster and safer" while controlling costs effectively.

### Mid-Session Switching Considerations

When switching models during a conversation, be aware that token consumption increases because the new model must process the entire conversation history. For long conversations, consider starting a fresh session when changing models to minimise token usage.

## When to Use Each Model

### Opus 4.5: Maximum Capability

**Best For:**
- Complex architectural decisions requiring deep reasoning
- Comprehensive code reviews catching subtle issues
- Refactoring large codebases with complex dependencies
- Optimisation analysis requiring trade-off evaluation
- Problems requiring genuine development partnership

**Characteristics:**
- Demonstrates the ability to internalize objectives
- Works persistently on complex problems
- Proactively identifies and resolves obstacles
- Catches issues other models miss
- Slower but more thorough

**Real-World Use Case:** "If you want a full review, optimisation tips, or a deep analysis of your code structure, Opus is your model."

### Sonnet 4.5: The All-Rounder

**Best For:**
- Daily development work and feature implementation
- Multi-file operations and state management
- API integration and complex logic
- Agentic workflows and production tasks
- Most coding scenarios requiring reliability

**Characteristics:**
- Faster and cheaper than Opus
- Reliable and consistent performance
- Handles complex tasks with explicit guidance
- Trusted for production workflows
- Rarely freezes or fails

**Real-World Use Case:** "Sonnet is the all-rounder and is trusted for daily work — writing logic, managing state, connecting APIs, and handling multiple files."

### Haiku 4.5: Speed and Efficiency

**Best For:**
- Simple file reads and content extraction
- Routine formatting and style corrections
- Basic syntax validation and linting
- Simple text transformations
- High-volume operations requiring speed
- Lightweight agent tasks

**Characteristics:**
- Fastest model available (3x faster than others)
- Most cost-effective option
- 90% capability at significant savings
- Ideal for routine, non-complex tasks

**Performance Example:** In testing, Haiku completed a task in 27 seconds compared to Opus at 76 seconds and Sonnet at 86 seconds.

## Cost vs Capability Trade-offs

### Pricing Comparison

Understanding the cost differences is essential for budget optimisation:

| Model | Input Cost | Output Cost | Cost Ratio |
|-------|-----------|-------------|------------|
| **Haiku 4.5** | $1/MTok | $5/MTok | 1x (baseline) |
| **Sonnet 4.5** | $3/MTok | $15/MTok | 3x more than Haiku |
| **Opus 4.5** | $5/MTok | $25/MTok | 5x more than Haiku |

**Key Insight:** Opus is approximately 5 times more expensive than Sonnet and about 20 times more expensive than Haiku for equivalent workloads.

### Cost Optimisation Techniques

**1. Prompt Caching**
Claude Code automatically uses prompt caching to optimise performance and reduce costs. Testing shows 70-80% cost savings are possible through caching, with batch processing and caching potentially reducing costs by up to 90% in some cases.

**2. Context Management**
- Use `/clear` frequently between tasks to reset the context window
- Prevent irrelevant conversation history from consuming tokens
- Create focused `CLAUDE.md` files with concise documentation

**3. Strategic Model Selection**
- Don't use Opus for tasks Haiku can handle
- Reserve Opus for reviews and complex reasoning
- Use Sonnet as the default workhorse

**4. Extended Context Considerations**
The 1 million token context window (available via `[1m]` suffix) carries different pricing considerations. Use it only when truly necessary for large codebase operations.

## Configuration Options

### Four Methods to Configure Models

Listed by priority order:

#### 1. During Session (Highest Priority)
Switch models mid-conversation:
```
/model opus
/model sonnet
/model haiku
/model opusplan
```

#### 2. At Startup
Launch Claude Code with a specific model:
```bash
claude --model opus
claude --model sonnet
claude --model haiku
```

#### 3. Environment Variable
Set persistent default in `~/.bashrc` or `~/.zshrc`:
```bash
export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"
```

#### 4. Settings File (Lowest Priority)
Configure permanently in your settings file:
```json
{
  "permissions": { ... },
  "model": "opus"
}
```

### Advanced Environment Variables

Control specific model alias mappings:
- `ANTHROPIC_DEFAULT_OPUS_MODEL` - Override default Opus version
- `ANTHROPIC_DEFAULT_SONNET_MODEL` - Override default Sonnet version
- `ANTHROPIC_DEFAULT_HAIKU_MODEL` - Override default Haiku version
- `CLAUDE_CODE_SUBAGENT_MODEL` - Configure subagent model

### Prompt Caching Control

Disable prompt caching globally or selectively:
```bash
# Disable globally
export DISABLE_PROMPT_CACHING=1

# Disable per model tier
export DISABLE_PROMPT_CACHING_HAIKU=1
export DISABLE_PROMPT_CACHING_SONNET=1
export DISABLE_PROMPT_CACHING_OPUS=1
```

### Checking Current Configuration

View your active model:
```
/status
```

This displays both your current model and account information.

## Task Complexity Assessment

Matching the right model to task complexity is essential for cost-effective development:

### Simple Tasks → Haiku
- File reading and basic content extraction
- Syntax validation and linting
- Formatting and style corrections
- Simple text transformations
- Routine, repetitive operations

### Moderate Complexity → Sonnet
- Feature implementation
- API integration
- Multi-file refactoring
- State management
- Bug fixes requiring context understanding

### High Complexity → Opus
- Architectural design decisions
- Complex refactoring with trade-offs
- Performance optimisation requiring deep analysis
- Security reviews
- Problems requiring creative problem-solving

### Using Extended Thinking

For particularly complex problems, combine model selection with extended thinking modes:
- `think` - Basic extended reasoning
- `think hard` - Deeper analysis
- `ultrathink` - Maximum reasoning depth

**Best Practise:** "Claude performs best when it has a clear target to iterate against." Use test-driven development or visual mockups as clear targets for complex tasks.

## Benchmarks and Performance

### Coding Performance: SWE-bench Verified

On real-world coding tasks, Claude models demonstrate exceptional performance:

| Model | SWE-bench Score | Comparison |
|-------|----------------|------------|
| **Sonnet 4** | 72.7% | Industry leading |
| **Opus 4** | 72.5% | Near-parity with Sonnet |
| Claude 3.7 Sonnet | 62.3% | Previous generation |
| GPT-4.1 | 54.6% | Competitor |
| Gemini 2.5 Pro | 63.2% | Competitor |

**Key Insight:** Sonnet 4 slightly edges out Opus 4 on coding tasks, making it the preferred choice for most development work.

### Additional Benchmarks (Sonnet 4)

- **GPQA Diamond** (graduate-level reasoning): 75.4%
- **TAU-bench** (agentic tool use): 80.5% Retail / 60.0% Airline
- **MMLU** (multilingual QA): 86.5%
- **AIME** (math competition): 70.5%

### Speed Comparison

Relative performance on identical tasks:

| Model | Time to Completion | Speed Rating |
|-------|-------------------|--------------|
| Haiku | 27 seconds | Fastest (1x baseline) |
| Opus | 76 seconds | 2.8x slower |
| Sonnet | 86 seconds | 3.2x slower |

**Note:** While Haiku is fastest, task completion time also depends on quality requirements and whether multiple iterations are needed.

### Context Window Capabilities

All current Claude 4 models support:
- **Standard**: 200K tokens
- **Extended** (beta): 1M tokens (available with `[1m]` suffix)
- **Maximum output**: 64K tokens

## Best Practices for Cost-Effective Usage

### 1. Start with Sonnet, Scale as Needed

Begin with Sonnet 4.5 as your default model. Only upgrade to Opus when you encounter tasks requiring deeper reasoning, and downgrade to Haiku for simple operations.

### 2. Implement the Explore-Plan-Code-Commit Workflow

This workflow reduces wasted tokens by having Claude research and plan first rather than jumping to implementation:
1. **Explore**: Understand the codebase (Sonnet or Haiku)
2. **Plan**: Design the solution (Opus or Sonnet)
3. **Code**: Implement features (Sonnet)
4. **Commit**: Review and finalise (Opus for critical code, Sonnet otherwise)

### 3. Use Clear, Specific Instructions

Claude performs best with clear targets. Provide:
- Specific acceptance criteria
- Test cases to pass
- Visual mockups or examples
- Explicit constraints and requirements

This reduces iterations and token consumption across all models.

### 4. Leverage Bash Tools and Custom Commands

Create custom slash commands for repetitive tasks to avoid redundant explanations and reduce token usage.

### 5. Monitor Usage Patterns

Track which tasks consume the most tokens and optimise your model selection strategy accordingly. Some users report 70-80% cost reductions through strategic model switching.

### 6. Consider Account-Specific Behaviours

For certain Max plan users, Claude Code automatically falls back to Sonnet if you hit usage thresholds with Opus, helping manage costs automatically.

### 7. Use Safe YOLO Mode for Simple Tasks

For straightforward work like fixing lint errors, use `--dangerously-skip-permissions` to streamline simple operations without sacrificing quality.

### 8. Batch Similar Tasks

When possible, batch similar operations together before switching models to minimise the token overhead of model switching.

## Model Knowledge Cutoffs

Understanding knowledge cutoffs helps set appropriate expectations:

| Model | Reliable Cutoff | Training Data Cutoff |
|-------|----------------|---------------------|
| Opus 4.5 | May 2025 | August 2025 |
| Sonnet 4.5 | January 2025 | July 2025 |
| Haiku 4.5 | February 2025 | July 2025 |

For questions about recent events or technologies, consider model knowledge limitations.

## Common Model Switching Scenarios

### Scenario 1: Starting a New Feature
```
1. /model sonnet (or keep default)
2. Implement the feature
3. /model opus
4. Review and optimise
```

### Scenario 2: Bug Investigation
```
1. /model haiku
2. Read error logs and gather context
3. /model sonnet
4. Fix the bug
```

### Scenario 3: Architecture Design
```
1. /model opus (or opusplan)
2. Design architecture and make key decisions
3. Implementation proceeds with sonnet (automatic with opusplan)
```

### Scenario 4: High-Volume Testing
```
1. /model haiku
2. Run simple validation across many files
3. Switch to sonnet only if complex issues found
```

## Troubleshooting Model Configuration

### Known Issues

**Default Model Persistence**: When using the `/model` command to switch models, the default model may not persist correctly upon re-entering Claude Code. To fix this, manually edit the `settings.json` file in the `.claude` directory.

**Context Window Reset**: Switching models mid-session increases token consumption. For long conversations, start a fresh session instead.

### Verification Steps

1. Check current model: `/status`
2. Verify environment: `echo $ANTHROPIC_MODEL`
3. Inspect settings file: Check `.claude/settings.json`
4. Review model mappings: Check model-specific environment variables

## Future Considerations

As of 2026, model selection is becoming increasingly important as organisations adopt AI agents, copilots, and automated workflows. The choice between models is "less about raw intelligence and more about reliability, reasoning style, safety constraints, and how well each model fits into real production systems."

Key trends to watch:
- Continued improvement in cost-performance ratios
- Enhanced caching mechanisms
- Better automatic model selection
- More granular usage controls

## Conclusion

Effective model selection in Claude Code requires understanding the strengths and costs of each model tier. By strategically switching between Opus, Sonnet, and Haiku based on task complexity, you can achieve optimal results while maintaining cost efficiency.

**Core Principles:**
1. Default to Sonnet 4.5 for most work
2. Use Opus 4.5 for complex reasoning and reviews
3. Leverage Haiku 4.5 for simple, high-volume tasks
4. Consider the OpusPlan hybrid approach for complex projects
5. Manage context and use caching to reduce costs
6. Match model capability to actual task requirements

By following these guidelines, you can maximise development velocity while minimising costs, creating a sustainable and effective coding workflow with Claude Code.

---

## Sources and Further Reading

- [Model configuration - Claude Code Docs](https://code.claude.com/docs/en/model-config)
- [Claude Code Model Configuration | Claude Help Centre](https://support.claude.com/en/articles/11940350-claude-code-model-configuration)
- [Models overview - Claude Docs](https://platform.claude.com/docs/en/about-claude/models/overview)
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Anthropic Claude Models Complete Guide | CodeGPT](https://www.codegpt.co/blog/anthropic-claude-models-complete-guide)
- [Anthropic API Pricing: The 2026 Guide](https://www.nops.io/blog/anthropic-api-pricing/)
- [Claude Code Model Selection | Steve Kinney](https://stevekinney.com/courses/ai-development/claude-code-model-selection)
- [Sonnet 4.5 vs Haiku 4.5 vs Opus 4.1 Real Projects Comparison | Medium](https://medium.com/@ayaanhaider.dev/sonnet-4-5-vs-haiku-4-5-vs-opus-4-1-which-claude-model-actually-works-best-in-real-projects-7183c0dc2249)
- [Anthropic Claude Review 2026: Testing Results](https://hackceleration.com/anthropic-review/)
- [A complete guide to model configuration in Claude Code - eesel AI](https://www.eesel.ai/blog/model-configuration-claude-code)
