# Implementing Claude Code Across Your Development Organisation

A comprehensive guide for technology leaders, engineering managers, and practice leads on rolling out Claude Code effectively across development teams, establishing standards, and creating rapid improvement cycles.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Phase 1: Assessment and Planning](#phase-1-assessment-and-planning)
3. [Phase 2: Foundation and Infrastructure](#phase-2-foundation-and-infrastructure)
4. [Phase 3: Pilot Programme](#phase-3-pilot-programme)
5. [Phase 4: Standards and Governance](#phase-4-standards-and-governance)
6. [Phase 5: Organisation-Wide Rollout](#phase-5-organisation-wide-rollout)
7. [Phase 6: Continuous Improvement](#phase-6-continuous-improvement)
8. [Governance Framework](#governance-framework)
9. [Metrics and Measurement](#metrics-and-measurement)
10. [Common Challenges and Solutions](#common-challenges-and-solutions)
11. [Templates and Checklists](#templates-and-checklists)

---

## Executive Summary

Implementing Claude Code across a development organisation requires more than installing a CLI tool—it demands establishing shared practices, governance structures, and feedback loops that enable teams to leverage AI assistance consistently while continuously improving their approaches.

### Key Success Factors

1. **Centralised Configuration, Decentralised Execution** — Establish organisation-wide standards while allowing teams autonomy in implementation
2. **Iterative Rollout** — Start with pilots, gather learnings, refine, then scale
3. **Living Documentation** — Treat CLAUDE.md files and configurations as evolving assets
4. **Measurable Outcomes** — Define and track metrics from day one
5. **Community of Practice** — Create forums for sharing learnings across teams

### Expected Outcomes

| Timeframe | Expected Outcome |
|-----------|------------------|
| 1-2 months | Pilot teams operational with initial standards |
| 3-4 months | Refined standards, 50% team adoption |
| 5-6 months | Full organisation rollout, improvement cycles established |
| 6-12 months | Measurable productivity gains, mature governance |

---

## Phase 1: Assessment and Planning

**Duration**: 2-4 weeks

### 1.1 Current State Assessment

Before implementation, understand your organisation's readiness:

#### Technical Readiness Checklist

- [ ] Development environments support CLI tool installation
- [ ] Teams have access to Anthropic API or enterprise agreement in place
- [ ] Git workflows are established and consistent
- [ ] CI/CD pipelines exist and are modifiable
- [ ] Security review process for new tools exists

#### Cultural Readiness Assessment

| Factor | Questions to Ask | Red Flags |
|--------|------------------|-----------|
| Change appetite | How do teams respond to new tools? | Strong resistance to tooling changes |
| Documentation culture | Do teams maintain READMEs? | No existing documentation practices |
| Collaboration | Do teams share learnings? | Siloed teams with no cross-pollination |
| Quality focus | Are code reviews thorough? | Rubber-stamp reviews, no standards |

#### Skills Inventory

Assess current team capabilities:

- **Prompt engineering experience** — Has anyone used AI coding assistants?
- **CLI proficiency** — Are developers comfortable with terminal-based tools?
- **Configuration management** — Do teams manage shared configurations effectively?

### 1.2 Stakeholder Alignment

#### Key Stakeholders to Engage

| Stakeholder | Concerns to Address | Value Proposition |
|-------------|---------------------|-------------------|
| Engineering Leadership | ROI, productivity metrics | Faster delivery, reduced context-switching |
| Security/Compliance | Data handling, API access | Configurable permissions, audit trails |
| Team Leads | Learning curve, workflow disruption | Gradual adoption, productivity gains |
| Developers | Job impact, tool fatigue | Augmentation not replacement, reduced toil |
| Finance | Cost management | Token optimisation, model selection strategies |

#### Alignment Workshop Agenda

1. **Vision Setting** (30 mins) — What does successful AI-assisted development look like?
2. **Concerns Mapping** (30 mins) — Surface and address objections
3. **Success Metrics** (30 mins) — Agree on measurable outcomes
4. **Governance Model** (30 mins) — Establish decision-making structure
5. **Timeline Agreement** (30 mins) — Commit to phased rollout plan

### 1.3 Resource Planning

#### Team Structure

| Role | Responsibilities | FTE Allocation |
|------|------------------|----------------|
| Programme Lead | Overall implementation, stakeholder management | 0.5 FTE |
| Technical Lead | Standards, configuration templates, troubleshooting | 0.5 FTE |
| Champions (per team) | Local adoption, feedback collection | 0.1 FTE each |
| Security Reviewer | Permissions audit, compliance verification | 0.2 FTE |

#### Budget Considerations

- **API Costs** — Estimate based on team size and usage patterns
- **Training Time** — Allow 4-8 hours per developer for initial onboarding
- **Tooling** — Consider MCP server hosting, CI/CD integration costs
- **Support** — Allocate time for champions and technical leads

---

## Phase 2: Foundation and Infrastructure

**Duration**: 2-3 weeks

### 2.1 Enterprise Configuration Architecture

Establish a hierarchical configuration structure that enables consistency while allowing flexibility:

```
Organisation Level (Managed/Enterprise)
    ├── Security policies (non-overridable)
    ├── Approved MCP servers
    └── Cost controls

Practice/Division Level (~/.claude/settings.json templates)
    ├── Common tool permissions
    ├── Shared hooks
    └── Model preferences

Project Level (.claude/settings.json)
    ├── Project-specific tools
    ├── Team conventions
    └── CI/CD integration

Developer Level (.claude/settings.local.json)
    └── Personal preferences (gitignored)
```

### 2.2 Create Organisation-Wide CLAUDE.md Template

Establish a base template that all projects extend:

```markdown
# [Project Name] — Claude Code Context

## Organisation Standards

> This project follows [Organisation Name] development standards.
> See: [link to central standards documentation]

## Project Overview

[One paragraph describing what this project does and its business context]

## Technology Stack

- **Language**: [e.g., TypeScript 5.x]
- **Framework**: [e.g., Next.js 14]
- **Database**: [e.g., PostgreSQL 15]
- **Infrastructure**: [e.g., AWS, Kubernetes]

## Architecture

[Brief description of architecture patterns used]

Key directories:
- `src/` — Application source code
- `tests/` — Test suites
- `docs/` — Documentation

## Development Workflow

### Branch Strategy
[e.g., GitFlow, trunk-based development]

### Commit Conventions
[e.g., Conventional Commits]

### Code Review Requirements
[e.g., Minimum reviewers, required checks]

## Coding Standards

### Style Guide
[Link to or summary of coding standards]

### Testing Requirements
- Unit test coverage minimum: [X]%
- Integration tests required for: [criteria]

### Documentation Requirements
- Public APIs must have JSDoc/docstrings
- Complex logic requires inline comments

## Claude-Specific Instructions

### Preferred Behaviours
- Always run tests after code changes
- Use existing patterns found in the codebase
- Keep changes minimal and focused

### Prohibited Actions
- Do not modify files in `src/generated/`
- Do not commit directly to main branch
- Do not add new dependencies without discussion

## Common Tasks

### Running Tests
```bash
npm test
```

### Starting Development Server
```bash
npm run dev
```

### Building for Production
```bash
npm run build
```

## Team Contacts

- **Tech Lead**: [Name]
- **Product Owner**: [Name]
```

### 2.3 Establish Shared Configuration Repository

Create a central repository for Claude Code configurations:

```
claude-code-standards/
├── README.md
├── templates/
│   ├── CLAUDE.md.template
│   ├── settings.json.template
│   └── hooks/
│       ├── pre-commit-lint.sh
│       ├── post-edit-format.sh
│       └── security-check.sh
├── commands/
│   ├── review-pr.md
│   ├── fix-issue.md
│   ├── write-tests.md
│   └── document-code.md
├── skills/
│   ├── tdd-workflow/
│   ├── security-review/
│   └── performance-audit/
├── mcp-servers/
│   ├── approved-servers.json
│   └── setup-guides/
└── docs/
    ├── onboarding-guide.md
    ├── best-practices.md
    └── troubleshooting.md
```

### 2.4 Security and Permissions Framework

#### Permission Tiers

Define organisation-wide permission tiers:

**Tier 1: Restricted (Default for new developers)**
```json
{
  "permissions": {
    "allow": [],
    "deny": ["Bash(*)", "computer(*)"],
    "ask": ["Read", "Edit", "Write", "Glob", "Grep"]
  }
}
```

**Tier 2: Standard (After training completion)**
```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Glob", "Grep"],
    "deny": ["Bash(rm -rf*)", "Bash(sudo*)"],
    "ask": ["Write", "Bash"]
  }
}
```

**Tier 3: Trusted (Senior developers, leads)**
```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Write", "Glob", "Grep", "Bash"],
    "deny": ["Bash(rm -rf /*)"]
  }
}
```

#### Mandatory Security Hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "scripts/check-sensitive-files.sh \"$FILE_PATH\"",
        "description": "Block edits to sensitive files"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "scripts/security-scan.sh \"$FILE_PATH\"",
        "description": "Scan for secrets in modified files"
      }
    ]
  }
}
```

### 2.5 Cost Management Infrastructure

#### Model Selection Policy

| Task Type | Recommended Model | Rationale |
|-----------|-------------------|-----------|
| Code exploration, simple queries | Haiku | Cost-effective, fast |
| Standard development tasks | Sonnet | Balanced capability/cost |
| Complex architecture, debugging | Opus | Maximum capability |
| Extended reasoning tasks | Opus + ultrathink | Complex problem-solving |

#### Token Budget Guidelines

```json
{
  "environment": {
    "CLAUDE_MODEL": "sonnet",
    "MAX_THINKING_TOKENS": "10000",
    "ANTHROPIC_RATE_LIMIT_TOKENS_PER_MINUTE": "100000"
  }
}
```

#### Cost Monitoring

- Set up API usage dashboards per team/project
- Establish monthly budget alerts
- Review high-usage patterns weekly

---

## Phase 3: Pilot Programme

**Duration**: 4-6 weeks

### 3.1 Pilot Team Selection

#### Ideal Pilot Team Characteristics

- [ ] 4-8 developers (manageable feedback collection)
- [ ] Mix of senior and junior developers
- [ ] Active development project (not maintenance mode)
- [ ] Tech lead willing to champion adoption
- [ ] Existing documentation practices
- [ ] Collaborative team culture

#### Anti-Patterns to Avoid

- Teams under delivery pressure with no slack time
- Teams resistant to process changes
- Projects with high security classification (start simpler)
- Teams with no existing code review practices

### 3.2 Pilot Onboarding Programme

#### Week 1: Foundation

| Day | Activity | Duration | Owner |
|-----|----------|----------|-------|
| 1 | Tool installation and setup | 2 hours | Each developer |
| 1 | Configuration walkthrough | 1 hour | Technical Lead |
| 2 | Basic prompting workshop | 2 hours | Technical Lead |
| 3 | Project CLAUDE.md creation | 2 hours | Team |
| 4-5 | Supervised usage with support | Ongoing | Champion |

#### Week 2: Practice

| Day | Activity | Duration | Owner |
|-----|----------|----------|-------|
| 1-2 | Paired programming with Claude | 4 hours | Pairs |
| 3 | Extended thinking workshop | 1 hour | Technical Lead |
| 4 | Hooks and automation setup | 2 hours | Team |
| 5 | Week 2 retrospective | 1 hour | Team |

#### Weeks 3-4: Independent Usage

- Daily stand-up check-ins on Claude usage
- Champion available for questions
- Collect usage patterns and pain points
- Document learnings in shared space

#### Weeks 5-6: Refinement

- Refine CLAUDE.md based on learnings
- Adjust permissions and hooks
- Create team-specific slash commands
- Prepare recommendations for rollout

### 3.3 Pilot Feedback Collection

#### Daily Micro-Feedback

Quick Slack/Teams poll:
- "How effective was Claude Code today?" (1-5 scale)
- "Any blockers or frustrations?" (free text)

#### Weekly Structured Feedback

```markdown
## Weekly Claude Code Feedback

### What worked well this week?
[Free text]

### What was frustrating or ineffective?
[Free text]

### Specific prompts that worked/failed?
[Examples]

### Suggestions for CLAUDE.md improvements?
[Free text]

### Time saved estimate (hours)?
[Number]

### Configuration changes needed?
[Specific requests]
```

#### End-of-Pilot Assessment

| Metric | Target | Actual |
|--------|--------|--------|
| Developer satisfaction (1-10) | ≥7 | |
| Estimated time savings (%) | ≥15% | |
| Code quality impact | Neutral or positive | |
| Security incidents | 0 | |
| Configuration stability | Minimal changes in final 2 weeks | |

### 3.4 Pilot Learnings Documentation

Create a comprehensive pilot report:

1. **Executive Summary** — Key outcomes and recommendations
2. **What Worked** — Practices to scale
3. **What Didn't Work** — Practices to avoid or modify
4. **Configuration Refinements** — Changes to templates
5. **Training Improvements** — Gaps identified
6. **Recommendations** — Adjustments for broader rollout

---

## Phase 4: Standards and Governance

**Duration**: 2-3 weeks (parallel with late pilot phase)

### 4.1 Establish the Claude Code Centre of Excellence

#### Structure

```
Claude Code Centre of Excellence
├── Steering Committee (quarterly)
│   ├── Engineering Director
│   ├── Security Lead
│   └── Finance Representative
├── Working Group (bi-weekly)
│   ├── Technical Lead
│   ├── Team Champions (rotating)
│   └── Training Lead
└── Community of Practice (weekly)
    └── All interested developers
```

#### Responsibilities

| Group | Responsibilities | Meeting Cadence |
|-------|------------------|-----------------|
| Steering Committee | Strategy, budget, policy decisions | Quarterly |
| Working Group | Standards evolution, issue resolution | Bi-weekly |
| Community of Practice | Knowledge sharing, peer support | Weekly |

### 4.2 Standards Documentation

Create and maintain living documentation:

#### Core Standards Document

```markdown
# Claude Code Organisation Standards

Version: 1.0
Last Updated: [Date]
Owner: [Centre of Excellence]

## 1. Configuration Standards

### 1.1 Required Settings
All projects MUST include:
- CLAUDE.md at project root
- .claude/settings.json with team configuration
- Security hooks enabled

### 1.2 Prohibited Configurations
The following are NOT permitted:
- Disabling security hooks
- Bypassing permission prompts in production code
- Using personal API keys for team projects

## 2. Usage Standards

### 2.1 Code Review Requirements
All Claude-generated code must:
- Pass existing CI/CD checks
- Be reviewed by a human before merge
- Include appropriate tests

### 2.2 Documentation Requirements
Claude-generated code should:
- Follow existing documentation patterns
- Not include AI attribution comments unless required

## 3. Security Standards

### 3.1 Data Handling
- Never paste production credentials into prompts
- Use environment variables for sensitive data
- Review Claude's file access before approval

### 3.2 Code Security
- Run security scans on generated code
- Review for common vulnerabilities (OWASP Top 10)
- Validate dependency additions

## 4. Cost Management

### 4.1 Model Selection
- Default to Sonnet for standard tasks
- Use Haiku for simple queries
- Reserve Opus for complex problems

### 4.2 Token Efficiency
- Use /clear regularly to manage context
- Leverage CLAUDE.md for persistent instructions
- Avoid uploading large files unnecessarily
```

### 4.3 Governance Processes

#### Configuration Change Process

```
Developer Request
       ↓
Champion Review (1 day)
       ↓
Working Group Discussion (bi-weekly meeting)
       ↓
Security Review (if needed)
       ↓
Standards Update
       ↓
Communication & Training
```

#### Exception Process

For legitimate deviations from standards:

1. **Request** — Developer documents need and justification
2. **Review** — Champion and Technical Lead assess risk
3. **Approval** — Working Group or Steering Committee (based on scope)
4. **Documentation** — Exception recorded with expiry date
5. **Monitoring** — Regular review of active exceptions

### 4.4 Training Programme

#### Curriculum Structure

**Level 1: Foundation (All developers)**
- Duration: 4 hours
- Format: Self-paced + workshop
- Content:
  - Installation and setup
  - Basic prompting
  - Understanding CLAUDE.md
  - Security and permissions
  - Common workflows

**Level 2: Proficient (After 2 weeks usage)**
- Duration: 2 hours
- Format: Workshop
- Content:
  - Advanced prompting (extended thinking)
  - Custom slash commands
  - Hooks configuration
  - Troubleshooting

**Level 3: Champion (Selected developers)**
- Duration: 4 hours
- Format: Workshop + mentoring
- Content:
  - Standards development
  - Team onboarding delivery
  - Feedback collection
  - Escalation handling

#### Training Materials Repository

```
training/
├── level-1/
│   ├── slides.md
│   ├── exercises/
│   ├── assessment.md
│   └── resources.md
├── level-2/
│   ├── slides.md
│   ├── exercises/
│   └── advanced-topics.md
├── level-3/
│   ├── facilitation-guide.md
│   ├── champion-handbook.md
│   └── escalation-procedures.md
└── quick-reference/
    ├── cheat-sheet.md
    ├── common-issues.md
    └── prompt-examples.md
```

---

## Phase 5: Organisation-Wide Rollout

**Duration**: 8-12 weeks

### 5.1 Rollout Waves

Structure rollout in manageable waves:

#### Wave Planning

| Wave | Teams | Duration | Focus |
|------|-------|----------|-------|
| 1 | 2-3 teams (early adopters) | 3 weeks | Validate refined approach |
| 2 | 4-6 teams (mainstream) | 3 weeks | Scale training, refine support |
| 3 | Remaining teams | 4 weeks | Full adoption, reduce support |

#### Wave Criteria

**Wave 1 Selection:**
- Teams expressing interest
- Similar tech stacks to pilot
- Strong champions available

**Wave 2 Selection:**
- Broader tech stack representation
- Mix of team sizes
- Include one "challenging" team

**Wave 3:**
- All remaining teams
- Focus on efficiency
- Self-service emphasis

### 5.2 Team Onboarding Checklist

```markdown
## Team Onboarding Checklist

### Pre-Onboarding (1 week before)
- [ ] Identify team champion
- [ ] Review team's tech stack for any special considerations
- [ ] Schedule training sessions
- [ ] Ensure API access provisioned
- [ ] Create team channel for support

### Week 1: Setup
- [ ] All developers complete Level 1 training
- [ ] Tool installed on all machines
- [ ] Project CLAUDE.md created
- [ ] .claude/settings.json configured
- [ ] Hooks enabled and tested
- [ ] Champion completes Level 3 training

### Week 2: Guided Usage
- [ ] Daily check-ins with champion
- [ ] First code review with Claude-assisted code
- [ ] Common issues documented
- [ ] CLAUDE.md refinements made

### Week 3: Independent Operation
- [ ] Team operating independently
- [ ] Champion handling most questions
- [ ] Feedback collected for improvements
- [ ] Handover to BAU support

### Post-Onboarding
- [ ] 30-day satisfaction survey
- [ ] Usage metrics reviewed
- [ ] Lessons learned documented
- [ ] Standards updates proposed (if any)
```

### 5.3 Support Structure

#### Tiered Support Model

```
Tier 1: Self-Service
├── Documentation
├── Quick reference guides
└── FAQ

Tier 2: Peer Support
├── Team champion
├── Community of Practice Slack channel
└── Weekly drop-in sessions

Tier 3: Expert Support
├── Technical Lead
├── Working Group members
└── Scheduled consultation

Tier 4: Escalation
├── Steering Committee
└── Vendor support (Anthropic)
```

#### Support Channels

| Channel | Purpose | Response Time |
|---------|---------|---------------|
| #claude-code-help (Slack) | Quick questions | < 4 hours |
| Weekly drop-in (Zoom) | Complex discussions | Next session |
| support@... (Email) | Formal requests | 1-2 days |
| Emergency hotline | Critical blockers | < 1 hour |

### 5.4 Communication Plan

#### Stakeholder Communications

| Audience | Content | Frequency | Channel |
|----------|---------|-----------|---------|
| All developers | Tips, updates, success stories | Weekly | Newsletter |
| Team leads | Metrics, roadmap, decisions | Bi-weekly | Email |
| Leadership | ROI, risks, strategic updates | Monthly | Report |
| Champions | Detailed updates, early previews | Weekly | Slack |

#### Communication Templates

**Weekly Developer Newsletter:**
```markdown
# Claude Code Weekly — [Date]

## Tip of the Week
[Practical tip with example]

## Usage Spotlight
[Team/developer success story]

## Common Question
Q: [Frequently asked question]
A: [Answer]

## Updates
- [Configuration changes]
- [New resources]
- [Upcoming sessions]

## Metrics
- Active users: [X]
- Tasks completed: [Y]
- Satisfaction score: [Z]
```

---

## Phase 6: Continuous Improvement

### 6.1 Feedback Loop Architecture

```
Individual Developers
    ↓ (Daily micro-feedback)
Team Champions
    ↓ (Weekly summaries)
Working Group
    ↓ (Bi-weekly analysis)
Standards Updates
    ↓ (Monthly releases)
All Teams
    ↓ (Adoption & feedback)
[Cycle repeats]
```

### 6.2 Improvement Cycle Cadence

#### Daily

- Developers report blockers via Slack
- Champions monitor team channels
- Quick fixes deployed for critical issues

#### Weekly

- Champions submit feedback summaries
- Community of Practice shares learnings
- Quick reference updates published

#### Bi-weekly

- Working Group analyses patterns
- Prioritises improvement backlog
- Approves minor standards changes

#### Monthly

- Standards release (semantic versioning)
- Training materials updated
- Metrics report published
- Retrospective on improvement process

#### Quarterly

- Steering Committee strategy review
- Budget assessment
- Roadmap planning
- External benchmarking

### 6.3 Standards Evolution Process

#### Versioning Scheme

```
MAJOR.MINOR.PATCH

MAJOR: Breaking changes requiring action from all teams
MINOR: New features or significant improvements
PATCH: Clarifications, bug fixes, minor updates
```

#### Release Process

1. **Proposal** — Champion or Working Group member raises RFC
2. **Discussion** — 1-2 week comment period
3. **Decision** — Working Group votes
4. **Documentation** — Standards updated
5. **Communication** — Change announced
6. **Training** — Materials updated if needed
7. **Rollout** — Phased deployment with migration guide

### 6.4 Knowledge Sharing Mechanisms

#### Internal

- **Weekly Lightning Talks** — 10-minute sessions sharing discoveries
- **Prompt Library** — Curated collection of effective prompts
- **Case Studies** — Detailed write-ups of significant wins
- **Office Hours** — Expert Q&A sessions

#### Cross-Organisation

- **Community Channels** — Participate in Claude Code communities
- **Conference Talks** — Share learnings externally
- **Blog Posts** — Publish experiences (with approval)
- **Vendor Feedback** — Contribute to Anthropic's improvement

### 6.5 Experimentation Framework

#### Safe Experimentation Guidelines

```markdown
## Experimentation Guidelines

### Approved Experiment Types
- New slash commands
- Alternative prompting approaches
- Workflow optimisations
- Tool integrations

### Experimentation Process
1. **Propose** — Document hypothesis and approach
2. **Review** — Champion assesses safety
3. **Isolate** — Run in personal/feature branch
4. **Measure** — Collect data on effectiveness
5. **Share** — Report findings to Community of Practice
6. **Standardise** — If successful, propose for standards

### Boundaries
- No security setting modifications without approval
- No production code without standard review
- No cost increases without budget approval
```

---

## Governance Framework

### 7.1 Decision Rights Matrix

| Decision Type | Team | Champion | Working Group | Steering |
|---------------|------|----------|---------------|----------|
| Personal settings.local.json | ✓ | | | |
| Project CLAUDE.md content | ✓ | Consult | | |
| Team settings.json | | ✓ | Inform | |
| Slash commands (team) | | ✓ | Inform | |
| Slash commands (org) | | | ✓ | |
| Security settings | | | ✓ | Approve |
| Model/cost policies | | | Propose | ✓ |
| Standards changes | | | ✓ | Inform |
| Budget allocation | | | | ✓ |
| Vendor relationships | | | | ✓ |

### 7.2 Policy Framework

#### Core Policies

1. **Acceptable Use Policy**
   - What Claude Code may be used for
   - Prohibited uses
   - Data handling requirements
   - Intellectual property considerations

2. **Security Policy**
   - Permission requirements
   - Code review requirements
   - Incident reporting procedures
   - Audit requirements

3. **Cost Management Policy**
   - Budget allocation model
   - Usage monitoring requirements
   - Escalation thresholds
   - Optimisation expectations

4. **Quality Policy**
   - Testing requirements for AI-generated code
   - Review standards
   - Documentation requirements
   - Technical debt management

### 7.3 Audit and Compliance

#### Regular Audits

| Audit Type | Frequency | Scope | Owner |
|------------|-----------|-------|-------|
| Security settings | Monthly | All projects | Security Lead |
| Usage patterns | Weekly | Cost anomalies | Technical Lead |
| Standards compliance | Quarterly | Sample projects | Working Group |
| Satisfaction | Quarterly | All developers | Programme Lead |

#### Audit Checklist

```markdown
## Project Compliance Audit

### Configuration
- [ ] CLAUDE.md exists and follows template
- [ ] settings.json uses approved configuration
- [ ] Security hooks are enabled
- [ ] No prohibited tools enabled

### Usage
- [ ] Code reviews include AI-generated code verification
- [ ] No sensitive data in prompt history
- [ ] Cost within budget allocation

### Documentation
- [ ] Team has completed required training
- [ ] Champion is active and engaged
- [ ] Feedback is being collected

### Outcomes
- [ ] Quality metrics stable or improving
- [ ] No security incidents
- [ ] Team satisfaction acceptable
```

---

## Metrics and Measurement

### 8.1 Key Performance Indicators

#### Adoption Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Developer activation rate | 90% within 30 days | API usage per developer |
| Daily active users | 70% of developers | Daily session count |
| Feature utilisation | 50% using advanced features | Command/hook usage |

#### Productivity Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time savings | 15% reduction in task time | Self-reported surveys |
| Code velocity | 10% increase | PRs merged per sprint |
| Context switch reduction | 20% reduction | Tool switch telemetry |

#### Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Defect rate | No increase | Bugs per release |
| Test coverage | Stable or increasing | Coverage reports |
| Security findings | No increase | SAST/DAST scans |

#### Satisfaction Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Developer satisfaction | ≥7/10 | Quarterly surveys |
| Net Promoter Score | ≥30 | Would you recommend? |
| Training effectiveness | ≥8/10 | Post-training surveys |

#### Cost Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Cost per developer | Within budget | API costs / headcount |
| Token efficiency | Improving trend | Tokens per task type |
| Model mix | 70% Sonnet, 20% Haiku, 10% Opus | API analytics |

### 8.2 Measurement Dashboard

```
┌────────────────────────────────────────────────────────────┐
│                  Claude Code Metrics Dashboard              │
├────────────────────────────────────────────────────────────┤
│  Adoption                    │  Quality                     │
│  ┌────────────────────────┐  │  ┌────────────────────────┐  │
│  │ Active Users: 142/160  │  │  │ Defect Rate: -2%       │  │
│  │ Activation: 89%        │  │  │ Test Coverage: +3%     │  │
│  │ Daily Active: 73%      │  │  │ Security: 0 incidents  │  │
│  └────────────────────────┘  │  └────────────────────────┘  │
├────────────────────────────────────────────────────────────┤
│  Productivity                │  Cost                        │
│  ┌────────────────────────┐  │  ┌────────────────────────┐  │
│  │ Time Saved: 18%        │  │  │ Monthly: £12,400       │  │
│  │ PRs/Sprint: +12%       │  │  │ Per Dev: £78           │  │
│  │ Satisfaction: 7.8/10   │  │  │ vs Budget: -8%         │  │
│  └────────────────────────┘  │  └────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

### 8.3 Reporting Cadence

| Report | Audience | Frequency | Content |
|--------|----------|-----------|---------|
| Usage Snapshot | Champions | Weekly | Adoption, issues |
| Team Dashboard | Team Leads | Bi-weekly | Team-specific metrics |
| Executive Summary | Leadership | Monthly | ROI, risks, strategy |
| Trend Analysis | Steering Committee | Quarterly | Long-term patterns |

---

## Common Challenges and Solutions

### 9.1 Adoption Challenges

| Challenge | Symptoms | Solutions |
|-----------|----------|-----------|
| Resistance to change | Low activation, complaints | Champion engagement, success stories, training |
| Tool fatigue | "Another tool to learn" | Demonstrate clear value, reduce friction |
| Fear of job replacement | Underutilisation, anxiety | Emphasise augmentation, celebrate human judgment |
| Skill gaps | Frustration, poor results | Targeted training, paired programming |

### 9.2 Technical Challenges

| Challenge | Symptoms | Solutions |
|-----------|----------|-----------|
| Poor prompt results | "Claude doesn't understand" | Prompt engineering training, CLAUDE.md improvement |
| Context overload | Slow responses, compaction | Regular /clear, modular CLAUDE.md |
| Integration issues | CI/CD failures | Dedicated integration support, testing |
| Cost overruns | Budget alerts | Model selection guidance, efficiency training |

### 9.3 Governance Challenges

| Challenge | Symptoms | Solutions |
|-----------|----------|-----------|
| Inconsistent practices | Divergent configurations | Stronger templates, audits |
| Slow decision-making | Bottlenecks, frustration | Clearer delegation, faster cycles |
| Standards stagnation | Outdated practices | Regular reviews, experimentation framework |
| Champion burnout | Declining engagement | Rotate champions, recognition, reduced load |

### 9.4 Quality Challenges

| Challenge | Symptoms | Solutions |
|-----------|----------|-----------|
| Hallucinated code | Bugs from incorrect assumptions | "Read before write" culture, verification |
| Over-engineering | Complex solutions for simple problems | Explicit constraints, code review focus |
| Reduced code ownership | "Claude wrote it, not me" | Review requirements, understanding emphasis |
| Documentation drift | CLAUDE.md becomes outdated | Regular reviews, ownership assignment |

---

## Templates and Checklists

### 10.1 Project Onboarding Checklist

```markdown
## Claude Code Project Setup Checklist

### Prerequisites
- [ ] Project repository exists
- [ ] CI/CD pipeline established
- [ ] Team champion identified
- [ ] API access provisioned

### Configuration
- [ ] CLAUDE.md created from template
- [ ] .claude/settings.json configured
- [ ] .claude/settings.local.json in .gitignore
- [ ] Security hooks enabled
- [ ] Custom slash commands created (if applicable)

### Documentation
- [ ] README references Claude Code usage
- [ ] Contributing guide updated
- [ ] Team conventions documented

### Testing
- [ ] Claude Code tested with common tasks
- [ ] Hooks verified working
- [ ] CI/CD integration tested

### Training
- [ ] All team members completed Level 1
- [ ] Champion completed Level 3
- [ ] Support channels communicated

### Go-Live
- [ ] Soft launch with subset of team
- [ ] Feedback collection mechanism active
- [ ] Escalation path documented
```

### 10.2 CLAUDE.md Review Checklist

```markdown
## CLAUDE.md Quality Review

### Completeness
- [ ] Project overview present and accurate
- [ ] Technology stack documented
- [ ] Architecture described
- [ ] Development workflow included
- [ ] Coding standards linked or summarised
- [ ] Common tasks documented

### Claude-Specific
- [ ] Preferred behaviours specified
- [ ] Prohibited actions listed
- [ ] Project-specific constraints included

### Maintainability
- [ ] Owner identified
- [ ] Last review date noted
- [ ] Version controlled
- [ ] Not excessively long (< 500 lines)

### Effectiveness
- [ ] Tested with common prompts
- [ ] Reduces need for repeated instructions
- [ ] Team feedback incorporated
```

### 10.3 Quarterly Review Template

```markdown
## Claude Code Quarterly Review — Q[X] [Year]

### Executive Summary
[2-3 paragraph overview of the quarter]

### Metrics Summary

| Metric | Q[X-1] | Q[X] | Target | Status |
|--------|--------|------|--------|--------|
| Active users | | | | |
| Satisfaction | | | | |
| Time savings | | | | |
| Cost per dev | | | | |

### Key Achievements
1. [Achievement with impact]
2. [Achievement with impact]
3. [Achievement with impact]

### Challenges Encountered
1. [Challenge and resolution/mitigation]
2. [Challenge and resolution/mitigation]

### Standards Changes
- [v1.X released with changes...]
- [Upcoming v1.Y will include...]

### Training and Adoption
- [X] developers completed Level 1
- [Y] new champions trained
- [Z] teams fully onboarded

### Cost Analysis
- Total spend: £[X]
- Cost per developer: £[Y]
- vs. Budget: [+/-Z%]
- Optimisation initiatives: [Details]

### Next Quarter Priorities
1. [Priority with success criteria]
2. [Priority with success criteria]
3. [Priority with success criteria]

### Recommendations
1. [Recommendation for Steering Committee]
2. [Recommendation for Steering Committee]

### Appendices
- A: Detailed metrics
- B: Team-by-team breakdown
- C: Standards changelog
```

### 10.4 Incident Report Template

```markdown
## Claude Code Incident Report

### Incident Details
- **Date/Time**: [When discovered]
- **Severity**: [Critical/High/Medium/Low]
- **Category**: [Security/Quality/Cost/Availability]
- **Reporter**: [Name]

### Description
[What happened]

### Impact
- **Users affected**: [Number/teams]
- **Duration**: [How long]
- **Business impact**: [Description]

### Root Cause
[Why it happened]

### Resolution
[How it was fixed]

### Preventive Measures
1. [Action to prevent recurrence]
2. [Action to prevent recurrence]

### Timeline
| Time | Event |
|------|-------|
| [Time] | [Event] |
| [Time] | [Event] |

### Lessons Learned
[What we learned and how we'll improve]
```

---

## Conclusion

Implementing Claude Code across a development organisation is a journey, not a destination. Success requires:

1. **Clear vision and stakeholder alignment** from the outset
2. **Solid foundations** in configuration, security, and governance
3. **Iterative rollout** that learns and adapts
4. **Strong community** of champions and practitioners
5. **Continuous improvement** cycles that rapidly propagate learnings

By following this guide and adapting it to your organisation's specific context, you can establish a sustainable, effective AI-assisted development practice that evolves with both your needs and the technology itself.

### Next Steps

1. **Assess** your organisation's readiness using the checklists in Phase 1
2. **Align** stakeholders with a workshop
3. **Plan** your pilot programme
4. **Execute** iteratively, measuring and learning
5. **Scale** with confidence based on proven patterns

---

## Additional Resources

- [CLAUDE.md Files Guide](core-configuration/01-claude-md-files.md)
- [Settings Hierarchy](core-configuration/02-settings-hierarchy.md)
- [Permissions Management](core-configuration/03-permissions-management.md)
- [Effective Prompting](prompting-interaction/04-effective-prompting.md)
- [Custom Slash Commands](automation-workflows/08-custom-slash-commands.md)
- [Hooks](automation-workflows/09-hooks.md)
- [CI/CD Integration](automation-workflows/11-cicd-integration.md)
- [Model Selection](quality-cost/20-model-selection.md)
- [Context Management](quality-cost/22-context-management.md)

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Owner: Claude Code Centre of Excellence*
