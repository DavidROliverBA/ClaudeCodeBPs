# Hallucination Reduction in Claude Code: A Comprehensive Guide

## Introduction

Hallucination in AI code generation—where models generate plausible but incorrect code, non-existent APIs, or fabricated references—represents one of the most critical challenges in automated software development. Research shows that hallucinations affect up to 44% of failed code completion tasks and 16.4% of generated tests. This guide provides evidence-based strategies for minimizing hallucinations when using Claude Code, grounded in Anthropic's official best practices, peer-reviewed research, and industry standards.

---

## 1. Prompting Claude to Investigate Files Before Answering

The foundation of hallucination reduction in Claude Code is the **investigate-first principle**: Claude must read and understand relevant code before making any claims or proposing changes.

### The Investigation Mandate

According to Anthropic's official Claude Code best practices, the core rule is unambiguous:

> **"Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers."**

This principle should be embedded in your prompting strategy through explicit instructions:

```xml
<investigate_before_answering>
You MUST read and understand relevant files before proposing code edits.
Never speculate about code you have not inspected.
If the user references a specific file/path, you MUST open and inspect it before explaining or proposing fixes.
Do not make any claims about code before investigating unless you are certain of the correct answer.
Give grounded and hallucination-free answers.
</investigate_before_answering>
```

### Structured Exploration Workflow

Anthropic recommends a phased approach following the **Explore → Plan → Code → Commit** pattern:

**Explore Phase:**
- Ask Claude to read relevant files, images (like UI mockups), or URLs
- Provide either general pointers ("read the file that handles logging") or specific filenames ("read logging.py")
- **Explicitly tell Claude not to write any code yet**—the goal is pure information gathering

**Example prompt:**
```
Read the authentication module files in /src/auth/ and the user management
files in /src/users/. DO NOT write any code yet. Tell me what authentication
methods are currently implemented and how user sessions are managed.
```

**Plan Phase:**
After investigation, request explicit planning: "Ask Claude to make a plan for how to approach a specific problem." Research shows that "asking Claude to research and plan first significantly improves performance for problems requiring deeper thinking upfront."

### Extended Thinking for Complex Investigations

For non-trivial tasks, activate Claude's extended thinking modes to allocate additional computation time:

- `"think"` - Basic extended thinking
- `"think hard"` - Medium extended thinking
- `"think harder"` - High extended thinking
- `"ultrathink"` - Maximum extended thinking

These phrases map directly to increasing levels of cognitive budget, allowing Claude to evaluate alternatives more thoroughly before generating code.

### Subagent Delegation for Verification

A powerful grounding technique involves distributing investigation across multiple agents:

> **"Telling Claude to use subagents to verify details or investigate particular questions it might have, especially early on in a conversation or task, tends to preserve context availability without much downside in terms of lost efficiency."**

Example:
```
Use a subagent to investigate how error handling is currently implemented
across the codebase. Have the subagent check at least 5 different modules
and report back with patterns it finds.
```

---

## 2. Verification Strategies

Verification acts as a critical checkpoint between code generation and execution. Multiple complementary strategies create defense-in-depth against hallucinations.

### Test-Driven Development (TDD) as Verification

TDD becomes extraordinarily powerful with Claude Code because tests provide objective, executable verification:

> **"Claude performs best when it has a clear target to iterate against—a visual mock, a test case, or another kind of output."**

**Best practice workflow:**
1. Write tests first describing the desired functionality
2. Be explicit: "We are doing TDD. Write the tests for a function that does X. These tests should fail initially."
3. Have Claude implement to make tests pass
4. Tests serve as grounding anchors preventing hallucinated behaviors

### Best-of-N Verification

Run Claude through the same prompt multiple times and compare outputs:

- **Inconsistencies across outputs indicate potential hallucinations**
- Simple but effective for critical code sections
- Research from Anthropic shows this can catch hallucinations that slip through single-run validation

**Implementation:**
```bash
# Generate same component 3 times with identical prompts
# Compare outputs - differences suggest uncertainty/hallucination
for i in {1..3}; do
  claude-code "Implement user authentication" > output_$i.txt
done
diff output_1.txt output_2.txt
```

### Iterative Refinement with Self-Verification

Use Claude's outputs as inputs for follow-up verification prompts:

```
Here's the code you just generated. Now verify:
1. Do all imported packages actually exist?
2. Do all method calls match the actual API signatures?
3. Are there any deprecated patterns?
4. Cite specific documentation for any external APIs used.
```

This forces the model to re-evaluate its own output with a verification lens, catching inconsistencies.

### External Knowledge Restriction

Explicitly constrain Claude to provided context:

```
IMPORTANT: Only use information from the files in this repository.
Do NOT rely on general knowledge about libraries or frameworks.
If you're unsure whether a function exists, search the codebase first.
```

Research shows that 2025 prompt-based mitigation reduced GPT-4o's hallucination rate from 53% to 23% through similar constraint techniques.

### Cross-Model Validation

For critical systems, verify using multiple models:

> **"Using another AI model to see if it generates a similar response can help, since different generative AI models don't all use the same models and may help detect or remove hallucinations."**

Different models have different training data and failure modes—consensus increases confidence.

---

## 3. Grounding Responses in Actual Code

Grounding—anchoring responses in verified, existing code—is the most direct hallucination prevention mechanism.

### The De-Hallucinator Approach

Research from the De-Hallucinator paper (arXiv:2401.01701v3) demonstrates a powerful iterative grounding technique:

**Three-stage prompting strategy:**

1. **Initial Prompt**: Standard code context without augmentation
2. **RAG Prompt**: Add API references retrieved from the initial prompt
3. **Iterative Prompt**: Use the model's initial predictions to retrieve increasingly relevant APIs through multiple rounds

**Results across five LLMs:**
- 23.3–50.6% improvement in edit distance
- 23.9–61.0% improvement in API recall
- 17.9% more passing tests
- 63.2% of initially failing tests fixed

### Retrieval-Augmented Generation (RAG) for Code

RAG systems can decrease hallucination rates by 60–80% by grounding responses in verified documents. However, simple document fetching isn't sufficient—Stanford's 2025 legal RAG reliability work found that even well-curated retrieval pipelines can fabricate citations.

**Best practice: Span-level verification**
Each generated claim must be matched against retrieved evidence:

```
For each API or function you generate:
1. Retrieve its actual definition from the codebase
2. Quote the exact signature
3. Verify your usage matches the signature
4. If no match found in codebase, explicitly state: "This appears to be a new API not currently in the codebase"
```

### Direct Quotation for Factual Grounding

Anthropic's official guidance emphasizes:

> **"For tasks involving long documents (>20K tokens), ask Claude to extract word-for-word quotes first before performing its task. This grounds its responses in the actual text, reducing hallucinations."**

**Example prompt:**
```
First, extract the exact function signatures for all authentication-related
functions in auth.py. Quote them verbatim with line numbers.

Then, using ONLY those quoted signatures, implement the password reset feature.
```

### Context Window Utilization

Modern Claude models have extensive context windows—use them:

```
Include the entire relevant module in your context. If Claude doesn't know
a particular library, dump in example code from that library. LLMs are
incredibly good at imitating patterns from limited examples.
```

This reduces reliance on potentially outdated or incorrect training data.

---

## 4. "Read Before Write" Patterns

Claude Code enforces a **read-before-write mechanism** that prevents operations on files that haven't been examined in the current session.

### Enforcement Mechanism

The system maintains session tracking of files read and will **fail write operations** if an existing file hasn't been read first. This architectural decision prevents blind edits that often lead to hallucinated changes.

### Best Practices for Read-Before-Write

**Always prefer Edit over Write:**
- Use the **Edit tool** for existing files (requires prior Read)
- Reserve **Write tool** only for genuinely new files
- This enforces examination before modification

**Explicit read verification:**
```
Before making ANY changes to database.py:
1. Read database.py in full
2. Identify all existing connection pooling logic
3. Quote the current implementation
4. THEN propose changes that integrate with existing patterns
```

### Hooks for Read Validation

Claude Code supports lifecycle hooks that can enforce read-before-write at the system level:

**PreToolUse hook example:**
```bash
# .claude/hooks/pre-tool-use.sh
# Validate that files are read before Edit/Write operations
if [[ "$TOOL_NAME" == "Edit" ]] || [[ "$TOOL_NAME" == "Write" ]]; then
  # Check if file was read in this session
  if ! session_has_read "$FILE_PATH"; then
    echo "ERROR: Must read $FILE_PATH before editing"
    exit 1
  fi
fi
```

### Incremental Verification Pattern

For large refactors, read and verify incrementally:

```
Phase 1: Read and understand the current implementation
Phase 2: Read the test suite
Phase 3: Read dependent modules
Phase 4: Propose changes with explicit references to what you read
Phase 5: Implement changes
Phase 6: Verify tests still pass
```

Each phase provides grounding for the next, preventing cascading hallucinations.

---

## 5. Cross-Referencing and Validation

Cross-referencing involves verifying claims against multiple sources within and outside the codebase.

### Internal Cross-Referencing

**API consistency checking:**
```
For every function call in the generated code:
1. Find its definition in the codebase
2. Verify parameter count matches
3. Verify parameter types match
4. Check return type expectations
5. Flag any mismatches explicitly
```

**Pattern consistency:**
```
Before implementing new authentication logic:
1. Find 3 existing authentication implementations in the codebase
2. Document the common patterns
3. Ensure new implementation follows those patterns
4. If deviating, explicitly explain why
```

### External Documentation Verification

When using external libraries, mandate citations:

> **"A developer should ask for citations or API reference wherever possible to minimize hallucinations."**

**Example prompt:**
```
Implement OAuth2 authentication using the library found in package.json.
For every OAuth2-related method you use:
- Cite the specific documentation page
- Quote the example from official docs
- Verify the method signature matches current library version
```

### Dependency Verification

**Package hallucination** is a critical risk—approximately 45% of hallucinated packages are consistently regenerated with the same prompt. Mitigate this:

```python
# Before suggesting any import, verify it exists
import pkg_resources

def verify_package(package_name):
    try:
        pkg_resources.get_distribution(package_name)
        return True
    except pkg_resources.DistributionNotFound:
        return False

# Prompt instruction:
"Before importing any package, verify it exists in package.json or
requirements.txt. If not found, explicitly state: 'This package would
need to be added as a new dependency.'"
```

### Multi-Agent Cross-Validation

Deploy multiple agents for consensus-building:

> **"Run multiple agents independently on the same query, comparing outputs to flag disagreements. This redundancy surfaces potential hallucinations that might otherwise slip through."**

**Implementation pattern:**
```
Agent 1: Implement feature X
Agent 2: Implement feature X independently
Agent 3: Compare both implementations and identify:
  - Differences in approach
  - Discrepancies in API usage
  - Conflicting assumptions
  - Present findings for human review
```

---

## 6. Signs of Hallucination

Recognizing hallucination patterns enables early intervention before errors propagate.

### Common Hallucination Signatures in Code

Microsoft engineer Mithilesh Ramaswamy identifies key indicators:

1. **Generated code that doesn't compile** - Syntax errors, undefined symbols
2. **Overly convoluted or inefficient code** - Unnecessarily complex solutions
3. **Functions or algorithms that contradict themselves** - Internal logical inconsistencies
4. **Ambiguous behavior** - Code whose purpose or effects are unclear

### API-Specific Hallucinations

**Non-existent functions:**
> **"AI hallucinations sometimes just make up nonexistent functions"** that look plausible but don't exist in any package.

**Warning signs:**
- Function names that seem "too perfect" for your needs
- API patterns that don't match the library's style
- Methods that combine multiple concerns unusually
- Parameters that seem overly convenient

**Detection method:**
```
For each external API call:
1. Search the library's actual source code
2. Check official documentation
3. If not found in either, flag as potential hallucination
```

### Documentation Mismatches

> **"Generated code may reference documentation, but the described behavior doesn't match what the code does."**

**Verification:**
```
After Claude generates code with documentation:
1. Read the generated docstring
2. Trace through the code logic
3. Verify claimed behavior matches actual implementation
4. Check for edge cases not mentioned in docs
```

### Package Hallucination Patterns

Research shows **60% of hallucinated packages reappear at least once in 10 subsequent prompts**, indicating systematic rather than random hallucination.

**High-risk scenarios:**
- Niche or specialized libraries
- Recently updated packages (training data lag)
- Cross-language dependencies
- Platform-specific modules

**Detection:**
Self-detection works well: "ChatGPT and DeepSeek can self-detect their hallucinations effectively, exhibiting an 80% accuracy rate in identifying hallucinated packages."

**Prompt for self-detection:**
```
Review the packages you just imported. For each one:
1. State your confidence it actually exists (0-100%)
2. If confidence < 95%, verify it in package.json/requirements.txt
3. If not found, acknowledge it may be hallucinated
```

### Consistency Violations

**Temporal inconsistencies:**
- Different answers to the same question in one conversation
- Contradictory implementation details
- Changing "facts" about APIs or functionality

**Prompt for detection:**
```
You previously said function X takes 2 parameters.
Now you're calling it with 3 parameters.
Explain this discrepancy. Which is correct?
```

### Confidence Score Analysis

> **"Many AI systems assign internal scores to each word they generate. By analyzing these likelihood scores, organizations can apply filters or thresholds to identify low-confidence responses that may be more prone to hallucination."**

While not directly exposed in Claude Code, you can proxy confidence through:
- Request multiple generations and measure variance
- Ask Claude to rate its own confidence: "On a scale of 1-10, how confident are you this function exists?"

---

## 7. Recovery from Incorrect Assumptions

When hallucinations occur, structured recovery minimizes damage and restores grounding.

### Immediate Error Feedback Loop

> **"The moment you run LLM generated code, any hallucinated methods will be instantly obvious: you'll get an error. You can fix that yourself or you can feed the error back into the LLM and watch it correct itself."**

**Recovery workflow:**
```bash
# 1. Execute generated code
python generated_module.py

# 2. Capture error
# AttributeError: module 'requests' has no attribute 'get_with_retry'

# 3. Feed back to Claude with context
"The code you generated produces this error:
AttributeError: module 'requests' has no attribute 'get_with_retry'

This suggests you hallucinated a non-existent method.
Please:
1. Read the actual requests library documentation
2. Find the correct method for retry logic
3. Regenerate the code using only verified methods"
```

### Course Correction with Context Preservation

Anthropic's guidance emphasizes real-time intervention:

> **"Press Escape to interrupt Claude during any phase (thinking, tool calls, file edits), preserving context so you can redirect or expand instructions."**

This prevents wasted computation on hallucinated paths while maintaining conversation continuity.

### Iterative Error Correction

Research shows iterative prompting with ground-truth references fixes issues:

**De-Hallucinator results:** 63.2% of initially failing tests were fixed through iterative prompting with retrieved API references.

**Implementation:**
```
Iteration 1: Initial code generation (may contain hallucinations)
Iteration 2: Add actual API documentation to prompt → regenerate
Iteration 3: Add error messages from test failures → regenerate
Iteration 4: Add code examples from existing codebase → regenerate
```

### Root Cause Analysis

When hallucinations occur, investigate systematically:

```
A hallucination occurred. Let's diagnose:

1. What did Claude claim? [Quote the specific claim]
2. What's actually true? [Quote ground truth from code/docs]
3. What was the prompt context? [Review what information was provided]
4. What information was missing? [Identify gaps]
5. How should the prompt be improved? [Specific changes]
```

This converts failures into learning opportunities that improve future prompts.

### Fine-Tuning on Hallucination-Free Datasets

For repeated hallucination patterns in specific domains:

> **"Fine-tuning on a hallucination-free dataset dramatically reduced the hallucination rate by over 80%."**

While most users won't fine-tune Claude directly, you can create domain-specific examples:

```
# .claude/examples/database-patterns.md
# Verified Database Patterns in Our Codebase

## Connection Pooling
[Include actual working examples from your codebase]

## Transaction Handling
[Include actual working examples from your codebase]

# Instruction to Claude:
"Use ONLY the patterns documented in database-patterns.md.
Do not invent new patterns or import new libraries."
```

### Human-in-the-Loop Checkpoints

> **"Keeping a human in the loop—someone who can review both the AI-generated content and any system flags—adds an important layer of oversight, particularly when accuracy is critical."**

**Critical checkpoints:**
1. Before committing generated code
2. Before deploying to production
3. When introducing new dependencies
4. When modifying security-critical paths
5. When tests fail unexpectedly

---

## 8. Best Practices for Reliable Outputs

Synthesizing all strategies into actionable best practices for day-to-day Claude Code usage.

### Prompt Engineering Essentials

**Specificity reduces hallucination:**
> **"Claude Code's success rate improves significantly with more specific instructions, especially on first attempts."**

**Bad prompt:** "Add authentication"
**Good prompt:**
```
Add JWT-based authentication to the Express.js API in /src/api/server.js.
Requirements:
- Use the existing jsonwebtoken package (already in package.json)
- Follow the pattern established in /src/api/middleware/auth.js
- Token expiration: 24 hours
- Store tokens in HTTP-only cookies
- First READ both server.js and auth.js before proposing changes
```

### Bounded Questions for Accuracy

> **"It's best to ask bounded questions and critically examine the results. Outputs tend to be more accurate for questions with a smaller scope, and most developers will be better at catching errors by frequently examining small blocks of code."**

**Unbounded (risky):** "Build a complete authentication system"
**Bounded (safer):** "Write a single function that validates JWT tokens using our existing middleware pattern"

### Permission to Express Uncertainty

Anthropic's guidance emphasizes allowing Claude to admit knowledge gaps:

> **"Explicitly give Claude permission to admit uncertainty. This simple technique can drastically reduce false information."**

**Prompt instruction:**
```
IMPORTANT: If you're unsure about any API, function, or implementation detail:
- Say "I don't know" rather than guessing
- Suggest where to find the correct information
- Propose investigating before implementing

It is ALWAYS better to admit uncertainty than to hallucinate plausible-sounding but incorrect code.
```

### Temperature Control

> **"Claude's hallucinations can sometimes be solved by lowering the temperature of responses. Temperature is a measurement of answer creativity between 0 and 1."**

**Recommended settings:**
- **Temperature 0-0.3**: Code generation, API calls, factual tasks (minimize hallucination)
- **Temperature 0.4-0.7**: Architecture decisions, creative problem-solving
- **Temperature 0.8-1.0**: Brainstorming, exploratory ideation (expect more hallucination)

### CLAUDE.md Convention

Anthropic recommends creating a `.claude/CLAUDE.md` file (or `CLAUDE.md` in root):

```markdown
# CLAUDE.md - Project Context for Claude Code

## Build Commands
npm run build
npm test
npm run lint

## Critical Rules
- ALWAYS read files before editing them
- Use ONLY dependencies in package.json
- Follow patterns in /src/patterns/ for new features
- Run tests after every change
- Never modify database schema without migration

## Common Hallucination Risks
- The `utils` package was renamed to `helpers` in v2.0
- We use custom auth, NOT passport.js (despite what training data might suggest)
- Database uses TypeORM v0.3, not v0.2 (API changed significantly)
```

This file is automatically added to context, providing grounding for every session.

### Execution-Based Validation

> **"Potential mitigation strategies include using execution-based evaluation, incorporating safety checks, and leveraging human feedback."**

**Never trust, always verify:**
```bash
# After Claude generates code
npm run lint     # Catch syntax issues
npm test         # Verify behavior
npm run build    # Ensure compilation
npm run security-scan  # Check for vulnerabilities
```

Execution is the ultimate hallucination detector for code.

### Detection Tool Integration

Modern tools assist with hallucination detection:

- **SelfCheckGPT**: Consistency-based detection
- **Trustworthy Language Model**: Confidence scoring
- **Aimon**: Real-time production monitoring
- **Static Analysis**: Tools like CodeQL for API verification

Integrate these into CI/CD pipelines for automated hallucination detection.

### Model Selection and Context Awareness

> **"Learn how to use the context. If an LLM doesn't know a particular library you can often fix this by dumping in a few dozen lines of example code."**

**Context injection pattern:**
```
I'll be working with the custom GraphQL library at /src/graphql/custom.js.
Here's the usage pattern from existing code:

[Paste 20-30 lines of working examples]

Now, using EXACTLY this pattern, implement a new query for user profiles.
```

### Regular Assumption Audits

Periodically challenge Claude's assumptions:

```
List every external library, framework version, and API you've assumed
exists in this codebase. For each one:
1. Cite where you found evidence of it
2. Quote the specific code or config file
3. If you can't cite evidence, mark it as an assumption to verify
```

This meta-level verification catches systematic hallucinations early.

---

## Conclusion

Hallucination reduction in Claude Code is not a single technique but a comprehensive defensive strategy combining:

1. **Pre-generation grounding** through mandatory file investigation
2. **Generation-time constraints** via specific prompts and context injection
3. **Post-generation verification** through testing and execution
4. **Recovery mechanisms** for when hallucinations slip through

The 2025 research landscape shows hallucination mitigation is rapidly improving. Claude 4.x models include internal self-critique loops that actively flag suspect snippets. Temperature control, RAG with span-level verification, and multi-agent consensus are becoming standard practices. Fine-tuning on hallucination-free datasets can achieve >80% reduction in error rates.

Most critically: **treat hallucination prevention as a systems problem, not just a prompting problem.** Use architectural enforcement (read-before-write), automated validation (tests, linters), human checkpoints (code review), and cultural norms (permission to say "I don't know") together.

The result is AI-assisted development that combines the productivity of code generation with the reliability of human-verified, grounded implementation.

---

## Sources

1. [Reduce hallucinations - Claude Docs](https://docs.claude.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)
2. [Reduce hallucinations - Claude Platform Docs](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)
3. [Claude 2.1 Announcement - Anthropic](https://www.anthropic.com/news/claude-2-1)
4. [Avoiding Hallucinations Course - Anthropic GitHub](https://github.com/anthropics/courses/blob/master/prompt_engineering_interactive_tutorial/Anthropic%201P/08_Avoiding_Hallucinations.ipynb)
5. [Preventing Hallucination in AI: Industry Standards](https://www.virtuallycaffeinated.com/2025/04/01/preventing-hallucination-in-ai-a-guide-based-on-industry-standards/)
6. [Hallucinations in code are the least dangerous form](https://simonwillison.net/2025/Mar/2/hallucinations-in-code/)
7. [How to Prevent AI Hallucinations - Enkrypt AI](https://www.enkryptai.com/blog/how-to-prevent-ai-hallucinations)
8. [LLM Hallucinations Guide - Lakera](https://www.lakera.ai/blog/guide-to-hallucinations-in-large-language-models)
9. [Stop AI Hallucinations Guide 2025 - Infomineo](https://infomineo.com/artificial-intelligence/stop-ai-hallucinations-detection-prevention-verification-guide-2025/)
10. [How to keep AI hallucinations out of your code - InfoWorld](https://www.infoworld.com/article/3822251/how-to-keep-ai-hallucinations-out-of-your-code.html)
11. [Package Hallucinations - USENIX](https://www.usenix.org/publications/loginonline/we-have-package-you-comprehensive-analysis-package-hallucinations-code)
12. [Claude Code: Best practices for agentic coding - Anthropic](https://www.anthropic.com/engineering/claude-code-best-practices)
13. [Prompting best practices - Claude Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-4-best-practices)
14. [Claude Code Beginners' Guide - Apidog](https://apidog.com/blog/claude-code-beginners-guide-best-practices/)
15. [How I use Claude Code - Builder.io](https://www.builder.io/blog/claude-code)
16. [De-Hallucinator: Mitigating LLM Hallucinations via Iterative Grounding](https://arxiv.org/html/2401.01701v3)
17. [When LLMs day dream - Red Hat](https://www.redhat.com/en/blog/when-llms-day-dream-hallucinations-how-prevent-them)
18. [Extrinsic Hallucinations in LLMs - Lilian Weng](https://lilianweng.github.io/posts/2024-07-07-hallucination/)
19. [Stop LLM Hallucinations - Master of Code](https://masterofcode.com/blog/hallucinations-in-llms-what-you-need-to-know-before-integration)
20. [What is grounding and hallucinations in AI? - K2view](https://www.k2view.com/blog/what-is-grounding-and-hallucinations-in-ai/)
21. [Detecting Hallucinations in Generative AI - Codecademy](https://www.codecademy.com/article/detecting-hallucinations-in-generative-ai)
22. [What are AI Hallucinations? - AI21](https://www.ai21.com/knowledge/ai-hallucinations/)
23. [The Mirage of AI Programming - Trend Micro](https://www.trendmicro.com/vinfo/us/security/news/vulnerabilities-and-exploits/the-mirage-of-ai-programming-hallucinations-and-code-integrity)
24. [Survey on hallucination in LLMs - Frontiers](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2025.1622292/full)
25. [The State of AI Hallucinations in 2025 - Maxim AI](https://www.getmaxim.ai/articles/the-state-of-ai-hallucinations-in-2025-challenges-solutions-and-the-maxim-ai-advantage/)
