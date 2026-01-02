# Extended Thinking in Claude Code: A Comprehensive Guide

## Overview

Extended Thinking is a powerful feature in Claude Code that allows the AI to spend additional computation time reasoning through complex problems before generating responses. Claude 3.7 Sonnet and Claude 4 models function as both standard LLMs and reasoning models, giving you the flexibility to choose when Claude should think longer before responding.

When Extended Thinking is enabled, Claude self-reflects before answering, which significantly improves performance on mathematics, physics, coding, instruction-following, and other complex reasoning tasks. This mode allocates more "reasoning budget," resulting in better multi-step planning, stronger adherence to instructions, and more reliable tool usage—at the cost of higher latency and increased token consumption.

## Thinking Mode Keywords

Claude Code includes a sophisticated preprocessing layer that intercepts specific keywords and maps them to different thinking budget levels. These keywords only work in Claude Code—they have no special significance in the Claude web interface or API.

### The Thinking Intensity Spectrum

The thinking modes follow this progression, from least to most intensive:

**"think"** → **"think hard"** → **"think harder"** → **"ultrathink"**

Each level allocates progressively more thinking budget for Claude to use.

### Keyword Details

#### Basic Level: "think"
Triggers basic extended thinking mode, giving Claude additional computation time to evaluate alternatives more thoroughly. This is the entry-level thinking mode suitable for moderately complex tasks.

**Alternative phrases that trigger basic thinking:**
- "think about it"
- "think a lot"
- "think deeply"
- "think more"

#### Intermediate Level: "think hard" / "megathink"
Provides approximately 10,000 tokens of thinking budget. This level is recommended for:
- API design decisions
- Database schema planning
- Performance optimization
- Complex refactoring across multiple files

#### Advanced Level: "think harder"
Activates a more intensive thinking mode with a substantial token budget for thorough evaluation of alternatives.

**Additional trigger phrases:**
- "think intensely"
- "think longer"
- "think really hard"
- "think super hard"
- "think very hard"

#### Maximum Level: "ultrathink"
The most intensive thinking mode, providing approximately 31,999 tokens of thinking budget for maximum depth of reasoning. This is the highest level of cognitive processing available in Claude Code.

**Best use cases for ultrathink:**
- System architecture redesigns
- Critical debugging sessions
- Complex database migrations
- Performance bottleneck analysis in unfamiliar codebases
- Architectural decisions with long-term implications

**Important note:** Ultrathink both allocates the maximum thinking budget AND semantically signals to Claude to reason more thoroughly, which may result in deeper thinking than necessary for your task.

## Tab Toggle for Extended Thinking

Claude Code provides a convenient keyboard shortcut to toggle thinking mode on and off during interactive sessions.

### How to Toggle

**Press the Tab key** during any Claude Code session to toggle thinking mode on or off. The current thinking state is displayed directly in the Claude Code UI, making it easy to see when extended reasoning is active.

**Note:** In newer versions (v2.0.66+), the keyboard shortcut was changed to **Alt+T** (or **Option+T** on Mac).

### UI Indicators

Claude Code displays clear visual indicators for:
- Current thinking mode status (on/off)
- Current thinking budget being utilized
- A "Thinking" indicator with timer when processing

When you use thinking trigger words like "ultrathink" in your prompts, Claude Code automatically indicates that maximum thinking budget is being utilized.

### Known Issues

- The status message "Thinking on" or "Thinking off" appears only briefly (1-2 seconds) before disappearing, making it difficult to verify the current state
- The Option+T shortcut doesn't work on German Mac keyboards due to key mapping conflicts
- Some versions have reported bugs where the Tab toggle doesn't function properly

## MAX_THINKING_TOKENS Configuration

The `MAX_THINKING_TOKENS` environment variable provides fine-grained control over extended thinking behavior across all Claude Code requests.

### What It Does

`MAX_THINKING_TOKENS` sets a global thinking budget that applies to all requests when configured. This setting takes priority over keyword-based thinking triggers.

**Important constraint:** The `max_tokens` parameter must always be greater than `thinking.budget_tokens`.

### Recommended Settings

For general Claude Code usage:
```bash
export MAX_THINKING_TOKENS=1024
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096
```

**Rationale:**
- `MAX_THINKING_TOKENS=1024` provides space for extended thinking without cutting off tool use responses, while maintaining focused reasoning chains
- This balance helps prevent trajectory changes that aren't always helpful for coding tasks specifically
- `CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096` ensures adequate headroom for tasks involving significant file creation or Write tool usage

### Amazon Bedrock Configuration

For Claude Code on Amazon Bedrock:
```bash
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096
export MAX_THINKING_TOKENS=1024
```

Setting `CLAUDE_CODE_MAX_OUTPUT_TOKENS` lower won't reduce costs but may cut off long tool uses, causing the Claude Code agent loop to fail persistently.

### Making Settings Permanent

Add the configuration to your shell's startup file:

**For Bash:**
```bash
echo 'export MAX_THINKING_TOKENS=1024' >> ~/.bashrc
source ~/.bashrc
```

**For Zsh:**
```bash
echo 'export MAX_THINKING_TOKENS=1024' >> ~/.zshrc
source ~/.zshrc
```

### Configuration File

You can also configure thinking mode in `~/.claude/settings.json`:
```json
{
  "alwaysThinkingEnabled": true
}
```

Use the `/config` command to verify your settings and see "Thinking Mode: True" status.

### Important Behavior Notes

- When `MAX_THINKING_TOKENS` is set, it takes priority and controls the thinking budget for all requests
- Keyword triggers like "ultrathink" only work when `MAX_THINKING_TOKENS` is NOT set
- There's a known bug where setting `MAX_THINKING_TOKENS` may force every request to be a thinking request
- The minimum thinking budget is 1,024 tokens

## Impact on Response Quality

Extended thinking can dramatically improve response quality for appropriate tasks, but understanding when and how to use it is crucial.

### Performance Improvements

Extended thinking enhances Claude's ability to:

**Mathematical Reasoning:**
- Multi-step calculations and proofs
- Complex equation solving
- Statistical analysis

**Coding Excellence:**
- Architectural changes across multiple files
- Complex refactoring operations
- Performance bottleneck identification
- Multi-step debugging sessions
- Nuanced error handling

**Planning and Analysis:**
- Comprehensive project planning
- Detailed document analysis
- Balancing multiple competing constraints
- Evaluating alternative approaches

**Instruction Following:**
- Better adherence to complex requirements
- More consistent behavior across multi-turn conversations
- Improved tool use reliability

### Quality vs. Budget Relationship

The model's performance can vary significantly at different thinking budget settings:

- **1,024 tokens (minimum):** Suitable for moderately complex tasks
- **10,000 tokens:** Good for API design, database planning, performance optimization
- **16,000+ tokens:** Recommended starting point for complex architectural tasks
- **32,000 tokens (maximum recommended):** System architecture redesigns, critical debugging

Larger token counts enable more comprehensive and nuanced reasoning, though there can be diminishing returns depending on the task. The thinking budget is a target rather than a strict limit—actual token usage may vary.

### When Quality Suffers

**Overthinking simple tasks:** Extended thinking can actually make Claude MORE verbose and LESS accurate on basic tasks. For simple syntax fixes, formatting tasks, or well-specified problems with clear instructions, extended thinking adds unnecessary complexity.

**Loss of focus:** On straightforward tasks, extended thinking may introduce tangential reasoning that detracts from the direct solution.

## Cost and Latency Considerations

Extended thinking provides powerful reasoning capabilities, but comes with measurable tradeoffs that should inform your usage patterns.

### Latency Impact

Extended thinking adds processing time before Claude begins generating visible responses:

- **Typical range:** 500ms to several seconds per request
- **Complex queries:** Can extend to 156+ seconds for highly complex reasoning tasks
- **Network factors:** Thinking budgets above 32K tokens can cause long-running requests that hit system timeouts and connection limits

**Best practice:** For live settings (pair programming, live chat support), limit thinking mode to critical segments to avoid excessive latency.

### Cost Implications

**Token consumption:** Extended reasoning consumes approximately 20-50% more tokens per response due to the thinking steps.

**Estimated cost range per task:**
- Basic thinking: ~$0.06
- Intermediate thinking: ~$0.15-$0.25
- Ultrathink: ~$0.48

**Cost optimization strategies:**
- Start with the minimum thinking budget (1,024 tokens) and increase incrementally
- Reserve ultrathink for tasks where correctness outweighs token costs
- Toggle thinking off for simple, routine queries
- Use batch processing for thinking budgets above 32K tokens

### Performance Benchmarking

To find the optimal balance for your use case:

1. Start with minimum thinking budget (1,024 tokens)
2. Progressively increase for tasks requiring more depth
3. Benchmark end-to-end task accuracy vs. latency and token usage
4. Identify sweet spots for different task categories

## Best Practices for Complex Reasoning Tasks

### Workflow Integration

**The Explore → Plan → Code → Commit Pattern**

Extended thinking is particularly valuable during the planning phase:

1. **Explore:** Ask Claude to research and understand the problem space
2. **Plan:** Use "think" or "think harder" to request a detailed approach
3. **Code:** Implement the solution with standard thinking or thinking off
4. **Commit:** Review and finalize

Steps 1-2 are crucial—without them, Claude tends to jump straight to coding a solution. Asking Claude to research and plan first with extended thinking significantly improves performance for problems requiring deeper thinking upfront.

### Prompting Strategies

**High-level over prescriptive:**
Claude often performs better with high-level instructions to "think deeply about a task" rather than step-by-step prescriptive guidance. Give Claude room to explore.

**Example:**
```
Good: "think harder about the best architecture for this microservices system"
Less effective: "first analyze X, then consider Y, then evaluate Z..."
```

**Constraint-rich prompts:**
Extended thinking excels at balancing multiple competing requirements simultaneously. Include all relevant constraints in your prompt.

**Example:**
```
"Design a caching system that:
- Minimizes latency
- Stays within 512MB memory
- Handles 10K requests/second
- Maintains consistency across distributed nodes
- Minimizes infrastructure cost

Think harder about the tradeoffs."
```

**Self-verification:**
Instruct Claude to verify work through testing before completion:

```
"Implement the authentication flow and test it thoroughly before showing me the results. Think about edge cases."
```

**Few-shot examples:**
Include example reasoning patterns using XML tags to demonstrate desired problem-solving approaches:

```xml
<thinking>
First I need to understand the existing authentication mechanism...
Then I'll identify potential security vulnerabilities...
Finally I'll propose improvements with minimal breaking changes...
</thinking>

Now apply this reasoning approach to analyze our payment processing system.
```

### For Large Outputs

When requesting 20,000+ word outputs, ask for a detailed outline with word counts per section first. This helps maintain structure and accuracy across long-form content.

### Multi-Turn Conversations

**Always pass thinking blocks back to the API.** This is critical for maintaining the model's reasoning flow across conversation turns. Don't truncate or remove thinking blocks when continuing a conversation.

## When NOT to Use Extended Thinking

Understanding when to disable extended thinking is as important as knowing when to enable it.

### Avoid Extended Thinking For:

**Simple, straightforward tasks:**
- Syntax corrections
- Code formatting
- Basic information retrieval
- Simple text transformations
- Routine file operations

**Well-specified problems:**
- Tasks with clear, step-by-step instructions already provided
- Problems with obvious, single-path solutions
- Repetitive operations following established patterns

**Speed-critical scenarios:**
- Live pair programming sessions (except for critical decisions)
- Real-time chat support
- Quick iterations during development
- Interactive debugging of simple issues

**Low-stakes decisions:**
- Variable naming
- Comment additions
- Minor refactoring within a single function

**General conversations:**
- Casual questions
- Basic explanations
- Simple documentation requests

### Why Avoid It?

**Unnecessary overhead:** Extended thinking adds latency (500ms to several seconds) and costs (20-50% more tokens) that provide no benefit for simple tasks.

**Potential overthinking:** Claude may introduce unnecessary complexity or verbosity when overthinking simple problems, reducing clarity and directness.

**Iteration speed:** In exploratory workflows where you're trying different approaches rapidly, extended thinking slows you down without proportional benefit.

### The Selective Approach

**Best practice:** Remain in default mode during exploration and only enable extended thinking for the "hard parts":
- Critical migration scripts
- Complex architectural decisions
- Performance optimization analysis
- Multi-system integration design
- Security-critical implementations

This selective approach optimizes for both speed and accuracy while minimizing unnecessary costs.

## Advanced Configuration and API Usage

### API Implementation

To enable extended thinking via the API, add a `thinking` object with the `type` parameter set to `enabled` and specify `budget_tokens`:

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  },
  "max_tokens": 16000,
  "messages": [...]
}
```

### Interleaved Thinking (Beta - Claude 4 Only)

Interleaved thinking is an advanced mode where Claude can pause during response generation to think, rather than only thinking before the response.

**To enable:**
Add the beta header `interleaved-thinking-2025-05-14` to your API request.

**Important difference:**
With interleaved thinking, `budget_tokens` can exceed `max_tokens`, as it represents the total budget across all thinking blocks within one assistant turn.

**Supported models:**
- Claude 4 Opus
- Claude 4 Sonnet
- Claude 4 Haiku

### Budget Guidelines by Task Complexity

| Task Complexity | Recommended Budget | Use Case Examples |
|----------------|-------------------|-------------------|
| Low | 1,024 tokens | Moderate problem-solving, basic planning |
| Medium | 5,000-10,000 tokens | API design, database schema, multi-file refactoring |
| High | 16,000+ tokens | System architecture, complex algorithms |
| Critical | 32,000 tokens | Major migrations, security reviews, performance optimization |

**For budgets above 32K:** Use batch processing to avoid networking issues, timeouts, and connection limits.

### Language Considerations

Extended thinking performs optimally in English, though final outputs support any language Claude supports. For best results, submit prompts in English even if you want responses in another language.

## Extended Thinking vs. "Think" Tool

It's important to understand the distinction between Extended Thinking and Claude's "think" tool:

**Extended Thinking:**
- What Claude does BEFORE starting to generate a response
- Deeply considers and iterates on its plan before taking action
- More comprehensive reasoning
- Better for complex planning and multi-step problems

**"Think" Tool:**
- Used AFTER Claude starts generating a response
- Allows Claude to pause mid-response to gather more information
- Less comprehensive than extended thinking
- More focused on new information the model discovers during execution
- Useful for realizing mid-task that more context is needed

**Recommendation:** Use extended thinking for simpler tool use scenarios like non-sequential tool calls or straightforward instruction following. Use the "think" tool when Claude might need to dynamically adjust its approach based on intermediate results.

## What to Avoid

### Don't Pass Thinking Back as User Text

Never copy Claude's extended thinking output and pass it back in user messages. This degrades performance and can confuse the model's reasoning process.

**Bad:**
```
User: Here's what you thought earlier: [extended thinking content]
Now apply it to this new problem...
```

**Good:**
```
[Simply pass back the complete assistant message including thinking blocks via the API]
```

### Don't Manually Prefill or Edit Thinking

Don't try to guide Claude by prefilling or editing the thinking content. Let Claude generate its own reasoning naturally.

### Don't Chase Token Volume

Avoid pushing token output merely for volume's sake. More thinking tokens don't always mean better results—focus on providing Claude with clear context and constraints instead.

### Avoid Below-Minimum Budgets

If you need thinking below the minimum budget (1,024 tokens), use standard mode with thinking turned off and employ traditional chain-of-thought prompting with XML tags like `<thinking>...</thinking>` instead.

## Practical Examples

### Example 1: System Architecture Decision

```
Prompt: "ultrathink - I need to design a real-time notification system that can handle
100K concurrent users, integrate with our existing PostgreSQL database, support both
WebSocket and SSE protocols, and maintain message delivery guarantees. Consider
scalability, cost, operational complexity, and failure modes."

Result: Claude will use ~32K thinking tokens to thoroughly evaluate architectures
(WebSocket servers, message queues, database polling, event sourcing), analyze
tradeoffs, and recommend an approach with detailed justification.
```

### Example 2: Complex Debugging

```
Prompt: "think harder about why this authentication middleware is intermittently
failing for about 2% of requests. The logs show token validation passes, but users
still get 401 errors. Race condition? Distributed cache issue? Network timing?"

Result: Claude will methodically reason through potential causes, examining timing
windows, distributed system behaviors, and edge cases that might explain the
intermittent failures.
```

### Example 3: When NOT to Use Thinking

```
Bad: "think - please add a comment to this function explaining what it does"

Good: "please add a comment to this function explaining what it does"

Reason: This is a straightforward task that doesn't benefit from extended reasoning.
Extended thinking would add latency and cost without improving quality.
```

## Summary and Quick Reference

### Quick Decision Matrix

| Scenario | Use Extended Thinking? | Recommended Level |
|----------|----------------------|-------------------|
| Syntax fix, formatting | ❌ No | N/A |
| Complex algorithm design | ✅ Yes | think harder |
| System architecture | ✅ Yes | ultrathink |
| Adding comments | ❌ No | N/A |
| Multi-file refactoring | ✅ Yes | think hard |
| Performance optimization | ✅ Yes | think harder |
| Simple variable rename | ❌ No | N/A |
| Security review | ✅ Yes | ultrathink |
| Live pair programming | ⚠️ Selective | think (for critical decisions) |
| Database migration | ✅ Yes | ultrathink |

### Key Takeaways

1. **Start small:** Begin with 1,024 tokens and increase incrementally
2. **Be selective:** Use extended thinking for complex tasks, not simple ones
3. **Balance costs:** Weigh the 20-50% token increase against quality improvements
4. **Mind latency:** Extended thinking adds 500ms to several seconds per request
5. **Toggle wisely:** Use Tab (or Alt+T) to enable thinking only when needed
6. **Set budgets appropriately:** Match thinking budget to task complexity
7. **Pass thinking blocks:** Always include thinking blocks in multi-turn conversations
8. **Avoid overthinking:** More thinking doesn't always mean better results

### Environment Variables Cheat Sheet

```bash
# Recommended general configuration
export MAX_THINKING_TOKENS=1024
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096

# For complex tasks (temporary override)
export MAX_THINKING_TOKENS=16000

# Maximum (use batch processing)
export MAX_THINKING_TOKENS=32000
```

### Keyboard Shortcuts

- **Tab** (older versions) or **Alt+T** / **Option+T** (newer versions): Toggle thinking mode on/off
- Look for UI indicators showing current thinking state

---

## Sources

This guide was compiled from the following authoritative sources:

- [Building with extended thinking - Claude Docs](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
- [Claude's extended thinking - Anthropic News](https://www.anthropic.com/news/visible-extended-thinking)
- [Claude Code: Best practices for agentic coding - Anthropic Engineering](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Extended thinking - Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-extended-thinking.html)
- [Extended thinking tips - Claude Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/extended-thinking-tips)
- [Using extended thinking - Claude Help Center](https://support.claude.com/en/articles/10574485-using-extended-thinking)
- [The ultrathink mystery: does Claude really think harder? - ITECS Blog](https://itecsonline.com/post/the-ultrathink-mystery-does-claude-really-think-harder)
- [Claude Code Thinking Levels: From Think to Ultra-Think - Goat Review](https://goatreview.com/claude-code-thinking-levels-think-ultrathink/)
- [What is UltraThink in Claude Code - ClaudeLog](https://claudelog.com/faqs/what-is-ultrathink/)
- [How to Toggle Thinking in Claude Code - ClaudeLog](https://claudelog.com/faqs/how-to-toggle-thinking-in-claude-code/)
- [Thinking mode in Claude 4.5: All You need to Know - DEV Community](https://dev.to/anna001/thinking-mode-in-claude-45-all-you-need-to-know-561d)
- [Extended Thinking in Claude: Practical Settings for Productivity - Claude AI](https://claude-ai.chat/guides/extended-thinking/)
- [Claude 4.5 vs 3.5/3.7: Speed vs Accuracy Comparison (2025)](https://skywork.ai/blog/claude-4-5-vs-3-5-3-7-speed-vs-accuracy-comparison-2025/)

---

*Last updated: January 2026*
*Document version: 1.0*
