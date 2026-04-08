# Agentic Refactor Rules

A formal prompt and methodology for refactoring any software project into a multi-agent, context-reset architecture using [BMAD](https://github.com/bmad-code-org/BMAD-METHOD) role-based agents and [OpenSpec](https://github.com/Fission-AI/OpenSpec) spec-driven contracts.

Based on findings from Anthropic's [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps).

## The Problem

AI agents working on long-running tasks hit two fundamental issues:

1. **Context anxiety** -- as the context window fills, agents prematurely wrap up work, lose coherence, and produce lower-quality output
2. **Self-evaluation bias** -- agents systematically overestimate the quality of work they produced in the same context

Context compaction (summarizing earlier conversation) doesn't fix either problem. It preserves the anxiety triggers and the self-evaluation bias stays intact.

## The Solution

**Context resets with structured handoffs.** Clear the context window entirely. Start a fresh agent. Pass it a structured handoff artifact containing the previous agent's state and next steps. Anchor everything to versioned specifications on disk so no information is lost.

This repo provides a comprehensive prompt document that guides an AI agent through refactoring any project to use this architecture.

## What's In This Repo

```
agentic-refactor-prompt.md    # The formal prompt (the core of this repo)
README.md                     # You are here
CLAUDE.md                     # Instructions for Claude Code sessions working in this repo
```

### What the Prompt Covers

| Phase | What It Does |
|-------|-------------|
| **Claude Code Adaptation** | Mitigate Claude Code's system prompt priority architecture: hooks for deterministic enforcement, path-scoped rules, lean CLAUDE.md, extended thinking, task-scoped role framing |
| **Phase 1** | Analyze the existing project, bootstrap BMAD docs and OpenSpec capability specs |
| **Phase 2** | Decompose into 7 BMAD agent roles (Discovery, Planner, Architect, Design, Generator, Evaluator, Orchestrator) |
| **Phase 3** | Define sprint contracts with measurable acceptance criteria linked to REQ-* and SCENARIO-* |
| **Phase 4** | Design weighted evaluation criteria with hard-fail conditions |
| **Phase 5** | Establish testing architecture, spec-to-test traceability, and coverage enforcement gates |
| **Phase 6** | Build the orchestration harness (custom script or Claude Code Agent Teams) |
| **Phase 7** | Spec reconciliation protocol -- keep specs and code in agreement across resets |
| **Phase 8** | Execution checklist for the full refactor |
| **Phase 9** | Continuous evolution -- stress-test harness components, remove what's not load-bearing |

Plus appendices on quick-start, anti-patterns, cost/quality tradeoffs, and Agent SDK performance optimizations.

## How to Use

### Option 1: Copy the Prompt Into Your Project

The simplest approach. Copy `agentic-refactor-prompt.md` into your target project and reference it from your `CLAUDE.md`:

```bash
cd /path/to/your-project
cp /path/to/agentic-refactor-rules/agentic-refactor-prompt.md .
```

Then add to your `CLAUDE.md`:

```markdown
## Refactoring Guide
Follow the methodology in `agentic-refactor-prompt.md` for all architectural work.
```

Start a Claude Code session and tell it to execute Phase 1:

```
Analyze this project per Phase 1 of agentic-refactor-prompt.md.
Produce the BMAD bootstrap and OpenSpec capability specs.
```

Review the Phase 1 output before proceeding. Then work through phases sequentially.

### Option 2: Use as a System Prompt

Feed the prompt directly to the Claude API or Agent SDK as part of a system message:

```python
from anthropic import Anthropic

client = Anthropic()

with open("agentic-refactor-prompt.md") as f:
    refactor_prompt = f.read()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16384,
    system=refactor_prompt,
    messages=[{
        "role": "user",
        "content": "Analyze this project and execute Phase 1. "
                   "The project root is at /path/to/project."
    }],
)
```

### Option 3: Use with Claude Code Agent Teams

For parallel execution across BMAD roles, enable Agent Teams and let the team lead coordinate the refactor:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
cd /path/to/your-project
claude
```

Then in the session:

```
Read agentic-refactor-prompt.md. Create an agent team following the BMAD
role mapping in Phase 2. Start with a planner teammate to execute Phase 1,
requiring plan approval before proceeding.
```

The team lead acts as the Orchestrator (Bob), spawning teammates for each BMAD role as needed. See Section 6.2 of the prompt for the full Agent Teams mapping.

### Option 4: Quick-Start (Minimum Viable Harness)

If you don't need the full BMAD ceremony, apply just the core principles:

1. **Create specs**: Write `openspec/capabilities/<cap>/spec.md` with `REQ-*` and `SCENARIO-*` for your core capabilities
2. **Separate generation from evaluation**: Never let the same context produce and judge work
3. **Use context resets**: When work exceeds ~30 minutes, break it into segments with handoff files
4. **Write acceptance criteria before implementation**: Reference `REQ-*` identifiers
5. **Commit between segments**: Each context-reset boundary is a git commit
6. **Keep a traceability file**: A simple table mapping `REQ-*` to done/not-done/tested/not-tested

## Directory Structure After Refactor

After applying the prompt, your project will have this additional structure:

```
your-project/
  _bmad/                              # BMAD strategic layer
    prd.md                            # Product Requirements Document
    architecture.md                   # Architecture with ADRs
    traceability.md                   # REQ-* -> implementation status matrix

  openspec/                           # OpenSpec behavioral contracts
    capabilities/
      <capability>/
        spec.md                       # REQ-*, SCENARIO-* definitions
        design.md                     # Technical patterns
    change-proposals/                 # Delta specs for changes in progress

  epics/                              # Work decomposition
    stories/

  .harness/                           # Runtime artifacts
    config.yaml                       # Agent configuration, evaluation criteria
    prompts/                          # Per-role agent prompts
    handoffs/                         # Structured handoff artifacts
    contracts/                        # Sprint contracts
    evaluations/                      # Evaluation reports

  tests/
    capabilities/                     # Test tree mirrors openspec structure

  ops/                                # Operational tracking
    status.md                         # Session handoff document
    changelog.md                      # What was done
    e2e-test-plan.md                  # SCENARIO-* -> E2E test procedures
    test-results.md                   # E2E execution results
```

## Key Concepts

### Context Resets

Instead of compacting (summarizing) a long conversation, clear the context entirely and start a fresh agent. The fresh agent reads structured artifacts from disk -- specs, handoffs, traceability -- which provide strictly more information than a compacted context because they're complete (not summarized) and structured (not narrative).

### Spec-Anchored Development

Every code change follows: **spec -> test -> implement -> verify -> reconcile**. Specs persist on disk across all context resets. Tests reference `REQ-*` and `SCENARIO-*` identifiers. The traceability matrix tracks what's specified, implemented, and tested.

### Generator-Evaluator Separation

The agent that writes code (Generator/Amelia) never evaluates its own output. A separate Evaluator agent (Quinn) receives only the specs, contracts, and produced artifacts -- never the generator's conversation. This breaks the self-evaluation bias.

### BMAD Roles

Seven specialized agent roles, each in its own context window:

| Role | Does | Doesn't |
|------|------|---------|
| **Discovery** (Mary) | Research unfamiliar domains | Plan or implement |
| **Planner** (John) | Produce epics, stories, change proposals | Make technical decisions |
| **Architect** (Winston) | Design decisions, ADRs, design.md | Write implementation code |
| **Design** (Sally) | UX specs, interaction patterns | Implement UI |
| **Generator** (Amelia) | Implement one story at a time, TDD | Evaluate quality or plan scope |
| **Evaluator** (Quinn) | Independent QA, E2E testing, grading | Fix bugs or write implementation |
| **Orchestrator** (Bob) | Route work, enforce gates, manage retries | Reason about code (script, not LLM) |

### Coverage Enforcement

Three gates enforce testing discipline:

1. **Generator self-check** -- tests pass + coverage thresholds before handoff
2. **Evaluator verification** -- independent test run + spec coverage audit + E2E
3. **Final evaluation** -- full regression + cross-story integration + traceability consistency

## Cost Expectations

From Anthropic's empirical data:

| Approach | Time | Cost | Quality |
|----------|------|------|---------|
| Single-pass, single agent | ~20 min | ~$9 | Appears functional, hidden defects |
| Full harness | ~4-6 hrs | ~$125-200 | Spec-verified, traceable, E2E tested |
| Optimized harness | ~3-4 hrs | ~$75-125 | Same quality, less overhead |

The harness costs more but every feature is traceable from user request through spec through test through implementation. The evaluator catches gaps that single-pass misses entirely.

## Evolution

The harness is not static. Every component encodes an assumption about what the model can't do alone. When models improve, stress-test each component:

- Remove it and measure quality impact
- If quality drops, it's **load-bearing** -- keep it
- If quality holds, it's overhead -- remove it

See Phase 9 of the prompt for the full stress-test table across all 7 BMAD roles.

## References

- [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) -- Anthropic Engineering
- [BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD) -- Breakthrough Method for Agile AI-Driven Development
- [OpenSpec](https://github.com/Fission-AI/OpenSpec) -- Spec-Driven Development for AI Coding Assistants
- [Making Your Agents Built Using Claude Agent SDK Run Faster](https://medium.com/@bayllama/making-your-agents-built-using-claude-agent-sdk-run-faster-2f2526a5cb42) -- Agent SDK performance optimizations
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams) -- Multi-agent coordination in Claude Code
- [Why Claude Code Overrides Your CLAUDE.md: A Technical Deep Dive](https://claude.ai/code) -- Analysis of Claude Code's system prompt priority architecture and why BMAD/OpenSpec instructions in CLAUDE.md are structurally subordinate. Informs the "Claude Code Harness Adaptation" section of the refactor prompt.

## License

This methodology document is provided as-is for use in your projects. No warranty expressed or implied.
