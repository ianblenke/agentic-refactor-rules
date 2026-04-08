# Agentic Refactor Prompt: Context-Reset Architecture with BMAD + OpenSpec

> **Purpose**: This document is a formal prompt for an AI agent to refactor an existing project into a multi-agent, context-reset architecture. It synthesizes Anthropic's harness design findings for long-running applications with BMAD's role-based agent methodology and OpenSpec's spec-driven development contracts. The result is a system where every context reset is anchored to versioned specifications, every handoff is traceable to requirements, and every evaluation is grounded in measurable acceptance criteria.

---

## Role

You are a senior AI systems architect operating within the BMAD (Breakthrough Method for Agile AI-Driven Development) framework. Your task is to analyze an existing project and refactor it into a harness-driven, multi-agent system that uses **context resets with structured handoffs** instead of context compaction. All work is **spec-anchored**: specifications are the source of truth, code implements specifications, tests verify specifications, and handoffs reference specifications.

---

## Core Principles

### 1. Context Resets Over Compaction

Context compaction (summarizing earlier conversation in-place) preserves continuity but causes **context anxiety** — agents prematurely wrap up work believing they are near token limits, produce lower-quality output as the window fills, and lose coherence on lengthy tasks. Agents also systematically overestimate the quality of their own work when evaluating in the same context that produced it.

**Context resets** — clearing the context window entirely and starting a fresh agent with a structured handoff carrying the previous agent's state and next steps — address both issues.

### 2. Spec-Anchored Development

Every code change follows the chain: **OpenSpec requirement -> test -> implementation -> verification -> spec reconciliation**. Context resets do not break this chain because the spec artifacts persist on disk. A fresh agent can reconstruct full context by reading `openspec/capabilities/*/spec.md`, the current handoff, and the traceability matrix.

### 3. Generator-Evaluator Separation

Inspired by GANs: the agent that produces work must never be the agent that judges it. Independent evaluation with tuned skepticism is the only reliable quality gate. In BMAD terms, the Developer agent (Amelia) and the QA agent (Quinn) are always separate context windows.

---

## BMAD Role-to-Agent Mapping

The harness maps BMAD's virtual team roles to discrete agent contexts with full resets between them:

| BMAD Role | Agent | Context | Primary Artifacts |
|---|---|---|---|
| Analyst (Mary) | **Discovery Agent** | Fresh per analysis task | `_bmad/product-brief.md`, research notes |
| PM (John) | **Planner Agent** | Fresh per planning cycle | `_bmad/prd.md`, `epics/*.md`, `epics/stories/*.md` |
| Architect (Winston) | **Architect Agent** | Fresh per architecture decision | `_bmad/architecture.md`, ADRs, `openspec/capabilities/*/design.md` |
| Scrum Master (Bob) | **Orchestrator** | Stateless loop (script or SDK) | `.harness/handoffs/`, `.harness/contracts/` |
| Developer (Amelia) | **Generator Agent** | Fresh per sprint/story | Code, `openspec/capabilities/*/spec.md` updates, handoff artifacts |
| QA (Quinn) | **Evaluator Agent** | Fresh per evaluation | `.harness/evaluations/`, `ops/test-results.md` |
| UX Designer (Sally) | **Design Agent** (optional) | Fresh per design task | `_bmad/ux-spec.md`, design tokens |
| Skill Evolver (Trace2Skill) | **Skill Evolution Agent** | Fresh per evolution cycle | `.harness/skills/`, `.harness/patches/` |

**Critical rules**:
- No two BMAD roles share a context window. The orchestrator is a stateless script, not an LLM — it reads handoff files and launches the next agent.
- **Do not ask Claude to adopt BMAD personas by name.** Claude Code's hardcoded identity ("I am Claude Code") is a hard constraint that overrides external persona assignments. Instead, frame each role as a **task scope**: "Your task is architecture review. Focus exclusively on design decisions and ADRs. Do NOT write implementation code." The functional constraint achieves role separation without triggering the identity override. See "Claude Code Harness Adaptation" section for details.

---

## Directory Structure

The refactored project uses this layout, merging BMAD strategic docs, OpenSpec behavioral contracts, harness runtime artifacts, and operational tracking:

```
project-root/
  _bmad/                              # BMAD strategic layer
    prd.md                            # Product Requirements Document
    architecture.md                   # Architecture with ADRs, "Last Reconciled" date
    traceability.md                   # REQ-* -> implementation status matrix
    ux-spec.md                        # UX specification (optional)
    product-brief.md                  # Discovery output (optional)

  openspec/                           # OpenSpec behavioral contracts
    AGENTS.md                         # Instructions for AI agents working with specs
    project.md                        # Project conventions
    capabilities/                     # Source of truth: what the system does
      <capability-name>/
        spec.md                       # REQ-*, SCENARIO-* definitions
        design.md                     # Technical patterns and decisions
    change-proposals/                 # Delta specs for proposed changes
      <change-name>/
        proposal.md                   # Intent, scope, approach
        specs/                        # ADDED/MODIFIED/REMOVED requirements
          <capability-name>/spec.md
        tasks.md                      # Implementation checklist
      archive/                        # Completed proposals with audit trail

  epics/                              # Work decomposition
    epic-<N>.md                       # Epic definitions
    stories/
      story-<slug>.md                 # Individual stories with acceptance criteria

  .harness/                           # Runtime artifacts (gitignored or tracked per preference)
    config.yaml                       # Harness configuration
    prompts/                          # Agent role prompts (one per BMAD role)
      discovery.md                    # Analyst Mary
      planner.md                      # PM John
      architect.md                    # Architect Winston
      design.md                       # UX Designer Sally
      generator.md                    # Developer Amelia
      evaluator.md                    # QA Quinn
      skill-evolver.md                # Trace2Skill evolution agent
    handoffs/                         # Structured handoff artifacts
      handoff-<session-id>.yaml
    contracts/                        # Sprint contracts
      contract-sprint-<N>.yaml
    evaluations/                      # Evaluation reports
      eval-sprint-<N>.yaml
    skills/                           # Skill evolution artifacts (Trace2Skill)
      SKILL.md                        # Root skill document (evolved over time)
      scripts/                        # Executable helpers distilled from traces
      references/                     # Edge-case guidance (low-prevalence patterns)
      changelog.yaml                  # Skill version history with evolution rationale
    patches/                          # Trajectory-specific patches (intermediate artifacts)
      patch-sprint-<N>.yaml           # Per-sprint skill patches before consolidation

  .claude/                              # Claude Code harness adaptation layer
    rules/                            # Path-scoped rules (injected as fresh system-reminders)
      openspec.yaml                   # "Validate against spec before generating code for openspec/"
      src.yaml                        # "Read sprint contract and spec before modifying src/"
      tests.yaml                      # "Every test must reference REQ-* or SCENARIO-*"
      harness.yaml                    # "Never modify .harness/ files during generation sprints"

  ops/                                # Operational tracking
    status.md                         # What's working, what's next (session handoff doc)
    changelog.md                      # What was done, traceable to user instructions
    known-issues.md                   # Active defects and workarounds
    e2e-test-plan.md                  # End-to-end test plan
    test-results.md                   # E2E test execution results
    metrics.md                        # Session metrics, turn logs, token tracking
    server.md                         # Server and credentials info

  scripts/
    orchestrate.py                    # Harness orchestration loop
    session-metrics.py                # Token cost extraction
    validate-phase-gate.sh            # Hook script: enforce phase gates deterministically
    validate-task-completion.sh       # Hook script: reject task completion if tests fail
```

---

## Claude Code Harness Adaptation

> **Why this section exists**: Claude Code's architecture is deliberately engineered to prioritize its hardcoded system prompt over user-supplied `CLAUDE.md` instructions. BMAD and OpenSpec workflows depend on behavioral rules (phase discipline, spec-first development, role separation) that compete for attention with ~50 pre-existing system prompt directives. Understanding the failure modes — and routing critical constraints through deterministic enforcement mechanisms instead of probabilistic `CLAUDE.md` instructions — is essential for making spec-driven agentic workflows reliable.

### The Instruction Priority Problem

Claude Code assembles a three-layer API request:

```
{
  "system": [ ... ],      // Static, cached system prompt: ~2,300-3,600 tokens
  "tools": [ ... ],       // Tool definitions: 14,000-17,600 tokens
  "messages": [ ... ]     // Conversation history + system-reminders
}
```

Your `CLAUDE.md` is **not** part of the `system` block. It is injected into the `messages` array as a `<system-reminder>` XML tag attached to each user turn. The model's trained attention weighting treats `system` as authoritative and `messages` as potentially adversarial (to prevent prompt injection). This means:

1. **Structural subordination**: `CLAUDE.md` content occupies a lower-priority position than the hardcoded system prompt
2. **"May or may not be relevant" qualifier**: The harness wraps `CLAUDE.md` with language granting Claude permission to disregard it: *"this context may or may not be relevant to your tasks"*
3. **Instruction budget exhaustion**: The system prompt already contains ~50 behavioral directives, consuming 25-33% of Claude's reliable instruction-following capacity (~150-200 instructions) before `CLAUDE.md` is even read
4. **Positional bias**: `CLAUDE.md` sits in the middle of the messages array — a lower-attention zone between the high-attention peripheries (system prompt at the start, most recent user message at the end)
5. **Tool definition dilution**: 14,000-17,600 tokens of tool JSON schemas sit between the system prompt and your `CLAUDE.md`, further reducing attention
6. **37 dynamic system reminders**: Claude Code injects reactive `<system-reminder>` messages mid-conversation (token budget warnings, context compaction notices, task tool nudges). These carry the same trust level as `CLAUDE.md` but appear more recently in the message history, systematically outcompeting methodology instructions from turn 1
7. **Conciseness directive**: External users receive "IMPORTANT: Go straight to the point. Try the simplest approach first. Be extra concise." — actively hostile to the extended, multi-phase reasoning BMAD and OpenSpec require

### Specific Failure Modes for BMAD/OpenSpec

| Failure Mode | Mechanism | Impact |
|---|---|---|
| **Role adoption refusal** | Claude's identity is hardcoded: "I am Claude Code, Anthropic's official CLI" — treated as a hard constraint, not a soft preference | Agent refuses to adopt BMAD personas (Product Manager, Architect, etc.) defined in slash commands or prompts |
| **Phase skipping** | "Go straight to the point" + "lead with action, not reasoning" push Claude to skip research/design phases and generate code immediately | Discovery, Planning, and Architecture phases get collapsed into direct implementation |
| **Spec fidelity drift** | Context rot in long sessions causes divergence from specs established in earlier turns | Implementation drifts from OpenSpec requirements over multi-turn sessions |
| **Plan-before-code suppression** | "Lead with action, not reasoning" actively suppresses the deliberate planning phase BMAD requires | Generator starts implementing before fully reading sprint contracts |
| **Methodology instruction decay** | Dynamic system reminders injected at turn 50 outcompete BMAD methodology instructions from turn 1 | Spec-anchored discipline degrades as conversations grow |

### Mitigation Strategies

The following strategies move critical behavioral constraints out of the probabilistic `CLAUDE.md` channel and into deterministic enforcement mechanisms.

#### 1. Use Hooks for Behavioral Rules (Deterministic Enforcement)

`CLAUDE.md` instructions are probabilistic — the model chooses whether to follow them. Hooks (`PreToolUse`, `PostToolUse`, `SessionStart`) are **deterministic** — the harness executes them regardless of model state. Phase enforcement logic belongs in hooks, not `CLAUDE.md`.

```json
// settings.json (project-level: .claude/settings.json)
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "scripts/validate-phase-gate.sh",
            "description": "Block code generation if spec file doesn't exist for the capability being modified"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "scripts/validate-test-run.sh",
            "description": "After test execution, verify coverage thresholds are met"
          }
        ]
      }
    ]
  }
}
```

**`scripts/validate-phase-gate.sh`** (exit code 2 = block with feedback):

```bash
#!/bin/bash
# Enforce spec-first development: block code writes if no spec exists
# for the capability being modified.
# Called by PreToolUse hook on Write|Edit.

FILE_PATH="$1"  # The file being written/edited

# Extract capability from file path (e.g., src/auth/login.py -> auth)
CAP=$(echo "$FILE_PATH" | sed -n 's|.*src/\([^/]*\)/.*|\1|p')

if [ -z "$CAP" ]; then
    exit 0  # Not a src/ file — allow
fi

SPEC="openspec/capabilities/$CAP/spec.md"
if [ ! -f "$SPEC" ]; then
    echo "BLOCKED: No spec found at $SPEC. Create the OpenSpec capability spec before implementing code for '$CAP'. See Phase 1.2 of the refactor guide." >&2
    exit 2
fi

# Check that at least one REQ-* exists in the spec
if ! grep -q 'REQ-' "$SPEC"; then
    echo "BLOCKED: Spec at $SPEC contains no REQ-* identifiers. Add requirements before implementing." >&2
    exit 2
fi

exit 0
```

**What belongs in hooks vs. CLAUDE.md:**

| Constraint | Enforcement Channel | Why |
|---|---|---|
| "Don't write code without a spec" | **Hook** (PreToolUse on Write/Edit) | Must be deterministic — model will skip this under conciseness pressure |
| "Run tests before handoff" | **Hook** (PostToolUse on Bash or TaskCompleted) | Must be deterministic — model may declare done without testing |
| "Every test must reference REQ-*" | **Hook** (PostToolUse on Write for test files) | Can be validated by script — no model judgment needed |
| "Read the sprint contract first" | **`.claude/rules/`** path-scoped rule | Injected fresh when touching `src/` — better than stale CLAUDE.md |
| "Tech stack is Python 3.12 + FastAPI" | **CLAUDE.md** | Factual context — appropriate for probabilistic channel |
| "Build command is `make build`" | **CLAUDE.md** | Factual context |
| "Use Opus for implementation" | **Harness config** (`.harness/config.yaml`) | Orchestrator reads this, not the model |

#### 2. Keep CLAUDE.md Lean (<300 Lines, Factual Only)

`CLAUDE.md` should be a **session onboarding document**, not a methodology definition. Keep it under 300 lines focused exclusively on project-specific factual context: tech stack, directory structure, build commands, naming conventions.

Move BMAD/OpenSpec methodology into:
- **Hooks** for rules that must be enforced deterministically
- **`.claude/rules/`** for path-scoped rules injected fresh when relevant
- **`--append-system-prompt`** for critical workflow constraints that need system-prompt-level priority
- **Referenced documents** (`.harness/prompts/*.md`, `openspec/AGENTS.md`) that Claude loads on demand via the Progressive Disclosure pattern — methodology lives in files the agent reads when needed, not in the always-loaded `CLAUDE.md`

**CLAUDE.md template for refactored projects:**

```markdown
# CLAUDE.md

## Project
<1-2 sentence description>

## Tech Stack
<language, framework, key dependencies>

## Build / Test / Deploy
\`\`\`bash
<build command>
<test command>
<lint command>
<deploy command>
\`\`\`

## Directory Structure
| Path | Contents |
|------|----------|
| _bmad/ | BMAD strategic docs (prd.md, architecture.md, traceability.md) |
| openspec/ | OpenSpec capability specs — READ BEFORE MODIFYING CODE |
| .harness/ | Agent prompts, handoffs, contracts, evaluations |
| ops/ | Status, changelog, metrics, test results |

## Conventions
<naming, formatting, patterns specific to this project>

## Key Commands
<frequently used commands>

## When to Read Deeper
- Before modifying any capability: read `openspec/capabilities/<cap>/spec.md`
- Before architectural decisions: read `_bmad/architecture.md`
- Before starting a sprint: read the sprint contract in `.harness/contracts/`
- Before reporting done: read `ops/e2e-test-plan.md` and run E2E tests
```

#### 3. Use `.claude/rules/` for Path-Scoped Reminders

The `.claude/rules/` directory supports YAML files with `paths:` frontmatter. These rules are injected as fresh `<system-reminder>` messages when Claude touches relevant files — arriving at the **current** turn rather than stale from session start. This gives them recency advantage over `CLAUDE.md`.

```yaml
# .claude/rules/openspec.yaml
---
paths:
  - "openspec/**"
---
You are modifying OpenSpec specification files. These are the source of truth
for all requirements. Before editing:
1. Read the existing REQ-* and SCENARIO-* identifiers
2. Assign new REQ-*/SCENARIO-* IDs following the existing numbering scheme
3. Use Given/When/Then BDD format for all SCENARIO-* definitions
4. After editing, update _bmad/traceability.md to reflect changes
```

```yaml
# .claude/rules/src.yaml
---
paths:
  - "src/**"
---
Before modifying source code:
1. Read the sprint contract in .harness/contracts/ for this story
2. Read the relevant openspec/capabilities/<cap>/spec.md
3. Write or update tests FIRST — each test must reference REQ-* or SCENARIO-*
4. Implement code to make tests pass
5. Run the full test suite before declaring done
Think carefully about how this code satisfies the spec requirements.
```

```yaml
# .claude/rules/tests.yaml
---
paths:
  - "tests/**"
---
Every test file MUST include a traceability header:
# Tests for: openspec/capabilities/<cap>/spec.md
# REQ-<CAP>-NNN: <description>
# SCENARIO-<CAP>-NNN: <description>

Every test function must reference at least one REQ-* or SCENARIO-*.
Tests without spec references are orphans and must be linked or removed.
Think harder about whether test assertions actually match what the spec requires.
```

```yaml
# .claude/rules/harness.yaml
---
paths:
  - ".harness/**"
---
Harness artifacts (.harness/) are managed by the orchestrator and skill
evolution pipeline. During generation sprints, do NOT modify:
- .harness/prompts/ (evolved by Trace2Skill)
- .harness/config.yaml (managed by orchestrator)
- .harness/evaluations/ (written by evaluator only)
You MAY write to:
- .harness/handoffs/ (your handoff artifact)
```

#### 4. Use `--append-system-prompt` for High-Priority Methodology Flags

The `--append-system-prompt` flag injects text directly above the tool definitions in the system prompt — substantially better placement than a `<system-reminder>` in the messages array. Critical workflow constraints that must be universally respected belong here.

```bash
claude --append-system-prompt "$(cat <<'EOF'
WORKFLOW RULES (override default behavior):
- This project uses spec-anchored development. NEVER write implementation code
  without first reading the relevant openspec/capabilities/<cap>/spec.md.
- NEVER evaluate your own output quality. The evaluator agent handles QA.
- When working on a story, read the sprint contract FIRST.
- Write tests BEFORE implementation code. Tests must reference REQ-* identifiers.
- Use 'think harder' when analyzing specs or designing implementations.
EOF
)"
```

**Note**: This adds per-session overhead (not cacheable). Use sparingly for the 3-5 most critical constraints that the model most often violates.

#### 5. Explicitly Counteract the Conciseness Directive

The external user build actively suppresses reasoning with "Go straight to the point. Be extra concise." This is directly hostile to BMAD's multi-phase, deliberate reasoning requirements. Counteract by explicitly invoking extended thinking:

- Use `think` (4K token budget), `think harder` (10K tokens), or `ultrathink` (31,999 tokens) in phase-entry prompts
- Include thinking invocations in agent prompts and sprint contracts:

```markdown
<!-- In .harness/prompts/generator.md -->
Before writing any implementation code, think harder about:
1. What REQ-* and SCENARIO-* does the sprint contract require?
2. What tests need to exist before implementation begins?
3. What design patterns from design.md apply?
4. What edge cases does the spec define?
```

```markdown
<!-- In .harness/prompts/evaluator.md -->
Ultrathink about whether each test actually verifies what the spec requires.
Do not accept tests that merely check status codes when the spec mandates
schema validation or specific response content.
```

#### 6. Manage Context Window as a First-Class Concern

Context window degradation is not theoretical — quality noticeably degrades at ~147,000-152,000 tokens even in a 200K window. BMAD workflows that span many turns are particularly vulnerable because methodology instructions from early turns lose influence.

**Practices:**
- **Rotate sessions at phase boundaries**: Research → Design → Spec → Generation should each be a fresh session, not one marathon conversation. This is already the design of the harness (context resets between agents), but it must also apply when working interactively in Claude Code.
- **Monitor token usage**: Use `/context` to check usage. If approaching 65% capacity, write a handoff and start a fresh session.
- **Never rely on compaction to preserve methodology**: Context compaction summarizes away the spec details and methodology instructions that BMAD depends on. Prefer a context reset with a structured handoff over compaction.
- **Front-load critical context**: When starting a fresh session, load the sprint contract and relevant specs in the first message — they'll benefit from the high-attention position at the start of the conversation.

#### 7. Role Framing Without Identity Override

Claude resists adopting BMAD personas because its identity ("I am Claude Code") is a hard constraint. Instead of asking Claude to "become" a persona, frame roles as **task scoping with expertise emphasis**:

**Don't:**
```
You are Winston, the Architect agent. Your role is to make design decisions...
```

**Do:**
```
Your task is architecture review. Focus exclusively on design decisions,
ADRs, and technical pattern selection. Do NOT write implementation code.
Read the change proposal and produce updated design.md files.
Think harder about tradeoffs between approaches.
```

The functional constraint ("focus on design, don't implement") achieves role separation without triggering the identity override. The agent's behavior is scoped by what it's asked to do and what tools it has access to, not by what persona it claims to be.

---

## Phase 1: Project Analysis and BMAD Bootstrap

Before making any changes, analyze the existing project and bootstrap the BMAD + OpenSpec structure.

### 1.1 Current Architecture Inventory

- Map all major components, services, and their responsibilities
- Identify the current build/test/deploy pipeline
- Catalog existing state management (databases, caches, file stores, queues)
- Document existing error handling and recovery patterns
- Check if `_bmad/architecture.md` exists; if so, check "Last Reconciled" date — if >30 days old, flag to user before proceeding

### 1.2 Existing Spec Extraction

If the project has no OpenSpec structure, extract implicit specifications from the existing codebase:

- For each identifiable capability, create `openspec/capabilities/<capability>/spec.md`
- Assign `REQ-<CAP>-NNN` identifiers to each discovered requirement
- Write `SCENARIO-<CAP>-NNN` for each observable behavior (use Given/When/Then BDD format)
- Mark all extracted specs with `Status: EXTRACTED` (not yet verified against implementation)

If OpenSpec structure already exists, audit it:
- Are specs current with the implementation?
- Are there requirements with no corresponding tests?
- Are there tests with no corresponding requirement IDs in comments?

### 1.3 BMAD Document Bootstrap

Create or update the BMAD strategic layer:

- `_bmad/prd.md` — Product Requirements Document. Source: existing README, product docs, or user interview. Captures the WHAT and WHY at the product level.
- `_bmad/architecture.md` — Architecture document with ADRs. Source: existing code structure, infrastructure config, deployment scripts. Include "Last Reconciled: YYYY-MM-DD".
- `_bmad/traceability.md` — Requirement-to-implementation status matrix:

```markdown
| REQ ID | Capability | Description | Spec Status | Impl Status | Test Status |
|--------|-----------|-------------|-------------|-------------|-------------|
| REQ-AUTH-001 | auth | User login via JWT | Specified | Implemented | Covered |
| REQ-AUTH-002 | auth | Token refresh | Specified | Partial | Missing |
```

### 1.4 Quality Assessment Gaps

- Identify where the system relies on self-evaluation
- Document where quality criteria are implicit rather than explicit
- Note where failures go undetected until user interaction
- Map which REQ-* have no SCENARIO-* coverage

**Output**: `_bmad/prd.md`, `_bmad/architecture.md`, `_bmad/traceability.md`, populated `openspec/capabilities/` tree

---

## Phase 2: Agent Role Decomposition

All seven BMAD roles are mapped to discrete agent contexts. Each operates in a fresh context window — no two roles ever share a context.

### 2.1 Discovery Agent (BMAD: Analyst Mary)

**Context**: Fresh window per analysis task. Reads existing codebase, documentation, user-provided context.

**Responsibility**: Explore the problem space before planning begins. This is BMAD Phase 1 — optional for well-understood changes, mandatory for new product areas, unfamiliar domains, or when the user's request is ambiguous.

**When to invoke**:
- User request references a domain, technology, or integration the project hasn't touched before
- The scope is unclear and needs research before it can be planned
- Existing specs and architecture docs don't cover the area being changed
- User explicitly asks for analysis or exploration

**Process**:
1. Read user request and identify knowledge gaps
2. Research the problem space: read existing code, documentation, external references
3. Analyze competing approaches, tradeoffs, and risks
4. Interview the user (via structured questions in the handoff) if ambiguity remains
5. Produce a product brief that the Planner can consume

**Output artifacts**:
- `_bmad/product-brief.md` — Problem statement, target users, key constraints, competitive landscape, recommended approach with rationale
- Research notes (optional, stored in `.harness/handoffs/` as discovery handoff)

**Handoff artifact**:

```yaml
handoff:
  agent_role: "discovery"
  session_id: "<uuid>"
  timestamp: "<ISO-8601>"
  status: "completed" | "needs_user_input"

  problem_statement: "what we're trying to solve"

  findings:
    - topic: "area researched"
      summary: "what was learned"
      implications: "how this affects the plan"
      sources: ["where this came from"]

  open_questions:
    - question: "what still needs answering"
      why_it_matters: "impact on planning if unanswered"
      suggested_resolution: "who/what can answer this"

  recommendation:
    approach: "recommended path forward"
    rationale: "why this over alternatives"
    alternatives_rejected:
      - approach: "alternative considered"
        reason_rejected: "why not"

  ready_for_planning: true | false
```

### 2.2 Planner Agent (BMAD: PM John)

**Context**: Fresh window. Reads `_bmad/prd.md`, `_bmad/architecture.md`, prior handoffs.

**Responsibility**: Convert user intent into epics and stories anchored to OpenSpec.

**Process**:
1. Read user request (1-4 sentences)
2. Read `_bmad/prd.md` for product context and `_bmad/architecture.md` for constraints
3. Read existing `openspec/capabilities/*/spec.md` to understand current system
4. Produce or update:
   - `epics/epic-<N>.md` with scope and feature list
   - `epics/stories/story-<slug>.md` for each implementation unit
   - For new capabilities: `openspec/change-proposals/<change-name>/proposal.md` with delta specs (ADDED/MODIFIED/REMOVED requirements)
   - For existing capabilities: identify which REQ-* and SCENARIO-* are affected

**Story format** (each story becomes one generator sprint):

```markdown
# Story: <slug>

## Epic
epic-<N>

## Description
<what this story delivers>

## Acceptance Criteria (linked to OpenSpec)
- [ ] REQ-<CAP>-NNN: <description> — verified by SCENARIO-<CAP>-NNN
- [ ] REQ-<CAP>-NNN: <description> — verified by SCENARIO-<CAP>-NNN

## Technical Notes
<high-level approach, NOT granular implementation details — avoid cascade errors>

## Dependencies
- Requires: story-<other-slug> (if any)
- Blocked by: <blocker> (if any)

## Definition of Done
- All listed REQ-* implemented
- All listed SCENARIO-* pass as automated tests
- `openspec/capabilities/<cap>/spec.md` updated with implementation status
- Code committed with story-<slug> in commit message
```

**Output artifacts**: Epic files, story files, change proposals with delta specs

### 2.3 Architect Agent (BMAD: Winston)

**Context**: Fresh window. Reads `_bmad/architecture.md`, `_bmad/prd.md`, change proposals.

**Responsibility**: Technical design decisions for new capabilities or significant changes.

**Process**:
1. Read the change proposal from the planner
2. Read existing `openspec/capabilities/*/design.md` files for current patterns
3. Produce or update:
   - `openspec/capabilities/<capability>/design.md` — technical patterns, data models, API contracts
   - `_bmad/architecture.md` — new ADRs for significant decisions, update "Last Reconciled" date
4. Run an **implementation readiness check** (BMAD Phase 3 quality gate): return PASS / CONCERNS / FAIL
   - FAIL blocks progression to implementation
   - CONCERNS are documented and escalated to user

**Output artifacts**: Updated design docs, ADRs, readiness check result

### 2.4 Design Agent (BMAD: UX Designer Sally)

**Context**: Fresh window per design task. Reads `_bmad/prd.md`, `_bmad/ux-spec.md`, relevant capability specs.

**Responsibility**: Define the user experience, interaction patterns, and visual design language before implementation begins. This is the UX counterpart to the Architect — Sally defines HOW users interact while Winston defines HOW the system is built.

**When to invoke**:
- The work involves user-facing interfaces (web, mobile, CLI, API developer experience)
- New interaction patterns are needed that aren't covered by existing `_bmad/ux-spec.md`
- The evaluator has flagged UX coherence or usability issues in prior sprints
- User explicitly requests design work

**Process**:
1. Read `_bmad/prd.md` for product context and user personas
2. Read existing `_bmad/ux-spec.md` for established design patterns (if exists)
3. Read relevant `openspec/capabilities/*/spec.md` for functional requirements the UI must serve
4. Read evaluator feedback from prior sprints (if redesign is triggered by QA failure)
5. Define or update:
   - Interaction flows (user journeys mapped to SCENARIO-* sequences)
   - Component hierarchy and layout patterns
   - Design tokens (colors, typography, spacing) if applicable
   - Accessibility requirements (mapped to REQ-* identifiers)
   - Error states and edge case UX

**Output artifacts**:
- `_bmad/ux-spec.md` — Interaction patterns, component specs, design tokens, accessibility requirements
- Annotated wireframes or layout descriptions (stored as markdown with ASCII diagrams or referenced design files)

**Handoff artifact**:

```yaml
handoff:
  agent_role: "design"
  session_id: "<uuid>"
  timestamp: "<ISO-8601>"
  status: "completed" | "needs_user_input"

  design_decisions:
    - decision: "interaction pattern chosen"
      rationale: "why this serves the user need"
      req_ids: ["REQ-* this supports"]
      alternatives_rejected:
        - pattern: "alternative considered"
          reason: "why not"

  components_specified:
    - name: "component name"
      purpose: "what it does for the user"
      interactions: ["list of user actions and responses"]
      scenarios: ["SCENARIO-* it participates in"]

  accessibility_requirements:
    - req_id: "REQ-<CAP>-NNN"
      description: "accessibility requirement"
      wcag_level: "A" | "AA" | "AAA"

  open_questions:
    - question: "design decision needing user input"
      options: ["option A", "option B"]
      recommendation: "which and why"

  ready_for_implementation: true | false
```

**Integration with Generator**: The generator reads `_bmad/ux-spec.md` alongside `design.md`. UX specs constrain the generator's implementation choices for user-facing components. The evaluator grades UX coherence against Sally's specs, not against the generator's own aesthetic judgment.

### 2.5 Generator Agent (BMAD: Developer Amelia)

**Context**: Fresh window per story. Reads story file, relevant specs, design docs, last handoff.

**Responsibility**: Implement one story at a time, spec-anchored.

**Process**:
1. Read `epics/stories/story-<slug>.md` for acceptance criteria and REQ-* references
2. Read `openspec/capabilities/<cap>/spec.md` for full requirement context
3. Read `openspec/capabilities/<cap>/design.md` for technical patterns
4. Read last handoff artifact (if continuing from a prior session)
5. Implement the story:
   - Write tests FIRST — each test file references REQ-* and SCENARIO-* in comments
   - Implement code to satisfy tests
   - Run unit tests and type checks
6. Update spec artifacts:
   - If change proposal exists: apply delta specs (ADDED -> append, MODIFIED -> overwrite, REMOVED -> delete)
   - Update implementation status in `openspec/capabilities/<cap>/spec.md`
7. Commit code with `story-<slug>` in commit message
8. Write handoff artifact

**Handoff artifact** (written on completion, context limit, or blocker):

```yaml
handoff:
  agent_role: "generator"
  story: "story-<slug>"
  session_id: "<uuid>"
  timestamp: "<ISO-8601>"
  status: "completed" | "blocked" | "context_limit" | "failed"

  spec_references:
    implemented:
      - id: "REQ-AUTH-001"
        capability: "auth"
        verification: "SCENARIO-AUTH-001 passes"
    deferred:
      - id: "REQ-AUTH-002"
        reason: "dependency on story-<other>"

  completed_work:
    - description: "what was done"
      files_modified: ["list of files"]
      tests_added: ["list of test files"]

  current_state:
    build_status: "passing" | "failing"
    test_status: "passing" | "failing"
    failing_tests: ["test names if any"]
    known_issues: ["list"]

  remaining_work:
    - description: "what still needs to be done"
      req_ids: ["REQ-* still unimplemented"]
      context_needed: ["files the next agent should read"]

  decisions_made:
    - decision: "what was decided"
      rationale: "why"
      spec_impact: "none" | "spec update needed"

  blockers:
    - description: "what is blocking"
      suggested_resolution: "how to unblock"
```

### 2.6 Evaluator Agent (BMAD: QA Quinn)

**Context**: Fresh window. Receives ONLY the story spec, sprint contract, and produced artifacts — never the generator's conversation.

**Responsibility**: Independent quality judgment grounded in OpenSpec requirements.

**Process**:
1. Read `epics/stories/story-<slug>.md` for acceptance criteria
2. Read `openspec/capabilities/<cap>/spec.md` for SCENARIO-* definitions
3. Read the sprint contract for verification methods and thresholds
4. Read the generator's handoff artifact for claimed completions
5. **Actively test** — do not just read code:
   - Run the test suite; verify all SCENARIO-* tests pass
   - For web apps: browser automation (Playwright) against deployed system
   - For APIs: integration tests with real protocol exchanges
   - For CLI tools: end-to-end execution with expected inputs/outputs
   - Use proper DNS names when feasible, not just localhost
6. Grade against sprint contract criteria (see Phase 5)
7. Produce evaluation report

**Evaluation report**:

```yaml
evaluation:
  story: "story-<slug>"
  session_id: "<uuid>"
  timestamp: "<ISO-8601>"
  verdict: "pass" | "fail"
  overall_score: 0.0-5.0

  requirement_verification:
    - req_id: "REQ-AUTH-001"
      scenario_id: "SCENARIO-AUTH-001"
      status: "verified" | "failed" | "not_tested"
      evidence: "what was observed"

  criteria_scores:
    - criterion: "Completeness"
      score: 4
      evidence: "all REQ-* implemented except REQ-AUTH-002"
    - criterion: "Correctness"
      score: 3
      evidence: "SCENARIO-AUTH-003 fails: token expiry not enforced"

  bugs_found:
    - severity: "critical" | "major" | "minor"
      description: "what's wrong"
      req_id: "REQ-* violated, if applicable"
      reproduction: "steps to reproduce"

  feedback_for_generator:
    - "specific, actionable critique 1"
    - "specific, actionable critique 2"

  spec_discrepancies:
    - "spec says X but implementation does Y — spec or impl needs updating"
```

**Critical tuning requirement**: Out of the box, LLMs are poor QA agents. The evaluator prompt (`.harness/prompts/evaluator.md`) requires iterative tuning:
1. Run evaluator, read its logs
2. Find judgment divergences from human expectations
3. Update the evaluator prompt
4. Repeat until evaluator catches the classes of issues that matter

### 2.7 Orchestrator (BMAD: Scrum Master Bob)

**Context**: NOT an LLM agent. The Orchestrator is a **stateless script or SDK harness** that manages the lifecycle of all other agents. It has no context window, no memory, no judgment — it reads files on disk and makes deterministic decisions.

**Responsibility**: Launch agents in the correct sequence, pass handoff artifacts between them, enforce contracts, manage retries, and escalate failures.

**Why this is not an LLM**: The orchestrator's decisions are mechanical (did the evaluator pass? are retries remaining? what's the next story?). Using an LLM here would waste tokens, introduce non-determinism, and create an unnecessary context window that could accumulate anxiety. A shell script or Python script is the correct tool.

**Process**:
1. Read the plan (epics and stories in execution order)
2. For each story:
   a. Construct sprint contract from story acceptance criteria + evaluation config
   b. Launch Generator Agent with: story file + spec files + design docs + contract + last handoff
   c. Wait for generator handoff artifact
   d. Validate handoff artifact (schema check, referenced files exist)
   e. Launch Evaluator Agent with: story file + spec files + contract + handoff + produced artifacts
   f. Read evaluation report
   g. If pass: update traceability, proceed to next story
   h. If fail + retries remaining: construct retry handoff (include evaluator critique), goto (b)
   i. If fail + no retries: escalate to user, log in `ops/known-issues.md`
3. After all stories: launch final cross-story evaluation
4. Run reconciliation steps (update ops docs, traceability, architecture date)
5. **Before shutting down any agent, mark all of its tasks as complete** — stale in-progress tasks corrupt state for future sessions and crash-recovery

**Conditional agent invocation**: The orchestrator decides when to invoke optional agents:
- **Discovery Agent (Mary)**: Invoked when the user's request references an unfamiliar domain or when the planner's handoff includes `ready_for_planning: false`
- **Architect Agent (Winston)**: Invoked when a change proposal introduces a new capability or the planner flags architectural impact
- **Design Agent (Sally)**: Invoked when the work involves user-facing interfaces or when the evaluator flags UX issues

**Implementation**: `scripts/orchestrate.py` or equivalent. See Phase 5 for the full loop pseudocode.

**Orchestrator state file** (persisted to disk for crash recovery):

```yaml
orchestrator_state:
  session_id: "<uuid>"
  started: "<ISO-8601>"
  plan_file: "epics/epic-<N>.md"

  story_queue:
    - story: "story-<slug>"
      status: "pending" | "in_progress" | "passed" | "failed" | "escalated"
      attempt: 0
      max_attempts: 3
      current_handoff: ".harness/handoffs/handoff-<session-id>.yaml"
      current_evaluation: ".harness/evaluations/eval-sprint-<N>.yaml"

  agents_invoked:
    - agent: "discovery" | "planner" | "architect" | "design" | "generator" | "evaluator"
      story: "story-<slug> or null"
      session_id: "<uuid>"
      started: "<ISO-8601>"
      ended: "<ISO-8601> or null"
      status: "running" | "completed" | "failed"
      tasks_cleaned: true | false      # all tasks marked complete/failed before shutdown
      cost: "$X.XX"

  totals:
    stories_completed: 0
    stories_failed: 0
    total_cost: "$0.00"
    total_duration: "0h 0m"
```

---

## Phase 3: Sprint Contract Negotiation

Before each story implementation, the orchestrator constructs a contract from OpenSpec artifacts. The contract binds generator and evaluator to the same measurable criteria.

### 3.1 Contract Structure

```yaml
sprint_contract:
  sprint_id: "sprint-<N>"
  story: "story-<slug>"
  epic: "epic-<N>"

  requirements:
    - id: "REQ-AUTH-001"
      description: "User login via JWT"
      scenarios:
        - id: "SCENARIO-AUTH-001"
          given: "a user with valid credentials"
          when: "the user submits login form"
          then: "a JWT token is returned AND the user is redirected to dashboard"

  definition_of_done:
    - "All listed SCENARIO-* pass as automated tests"
    - "Build passes with no errors"
    - "No regressions in existing SCENARIO-* tests"

  verification_methods:
    - criterion: "Functional Completeness"
      method: "Run test suite, verify all SCENARIO-* in contract pass"
      threshold: "100% pass rate"
    - criterion: "Integration"
      method: "E2E test against deployed system per ops/e2e-test-plan.md"
      threshold: "All E2E scenarios pass"
    - criterion: "Spec Fidelity"
      method: "Compare implementation behavior against spec.md requirements"
      threshold: "No unresolved discrepancies"

  scope_boundaries:
    in_scope: ["explicit REQ-* list"]
    out_of_scope: ["explicit exclusions"]

  renegotiation_protocol:
    trigger: "generator discovers contract is infeasible"
    action: "write handoff with status 'blocked' and renegotiation request — never silently reduce scope"
```

---

## Phase 4: Evaluation Criteria Design

### 4.1 Criteria Template

```yaml
criterion:
  name: "Completeness" | "Correctness" | "Spec Fidelity" | "Craft" | "Robustness" | custom
  weight: 0.0-1.0

  scoring:
    5: "All REQ-* implemented, all SCENARIO-* pass, edge cases handled"
    4: "All REQ-* implemented, all SCENARIO-* pass, minor edge cases missed"
    3: "Most REQ-* implemented, some SCENARIO-* fail with minor issues"
    2: "Significant REQ-* missing or multiple SCENARIO-* failures"
    1: "Core functionality broken or missing"

  hard_fail_conditions:
    - "Build fails"
    - "Any SCENARIO-* marked critical fails"
    - "Security vulnerability introduced"
    - "Data loss possible"
    - "Spec explicitly contradicted without documented rationale"

  evaluation_method: "automated test + E2E verification"
```

### 4.2 Weighting Guidance

- **Weight heavily**: Spec fidelity (does the code match the spec?), completeness (are all REQ-* addressed?), integration correctness
- **Weight lightly**: Code style, formatting, minor craft issues
- **Hard-fail on**: Spec contradictions without rationale, broken builds, security issues, data loss

### 4.3 Prompt Wording as Steering

The language in evaluation criteria directly steers generator behavior. Include the criteria in the generator's prompt — they guide implementation, not just evaluation:

- "Every REQ-* must have a corresponding automated test referencing it by ID" steers toward test discipline
- "Implementations that diverge from spec.md must include a rationale comment and a spec update" steers toward spec fidelity
- "The evaluator will actively use the system, not just read the code" steers toward real functionality over display-only features

---

## Phase 5: Testing Architecture and Coverage Enforcement

Testing is not a step that happens after implementation — it is the enforcement mechanism that makes spec-anchored development real. Without it, REQ-* identifiers are just labels. This phase defines the test pyramid, the coverage model, the enforcement gates, and how each agent interacts with the test infrastructure.

### 5.1 Test Pyramid Mapped to OpenSpec

Every layer of the test pyramid traces back to OpenSpec artifacts:

```
                 +--------------------------+
                 |       E2E Tests          |  SCENARIO-* across capabilities
                 |  (Playwright, real       |  Full deployed stack, real DNS
                 |   protocol exchanges)    |  ops/e2e-test-plan.md
                 +--------------------------+
               +------------------------------+
               |      Contract Tests           |  API shape validation between
               |  (Schema-validated stubs,     |  components — catches silent
               |   consumer-driven contracts)  |  contract drift
               +------------------------------+
             +----------------------------------+
             |     Integration Tests             |  SCENARIO-* within capability
             |  (Real DB, real services,         |  Cross-component interactions
             |   no mocks at boundaries)         |  openspec/capabilities/*/spec.md
             +----------------------------------+
          +---------------------------------------+
          |          Unit Tests                    |  REQ-* individual behaviors
          |  (Single function/module,              |  Isolated logic, fast feedback
          |   mocks OK for external deps)          |  Referenced by REQ-<CAP>-NNN
          +---------------------------------------+

  (Orthogonal — applies when the project is itself a testing/compliance tool):

          +---------------------------------------+
          |    Conformance Fixture Tests           |  Golden fixtures (all pass) +
          |  (Known-good + known-bad mocks,        |  poison fixtures (targeted fail)
          |   false-positive/false-negative gates)  |  tests/conformance_fixtures/
          +---------------------------------------+
```

| Test Layer | What It Verifies | OpenSpec Anchor | Who Writes | Who Runs | When |
|---|---|---|---|---|---|
| **Unit** | Individual REQ-* behaviors in isolation | `REQ-<CAP>-NNN` in test comments | Generator (Amelia) | Generator (self-check) + Evaluator (verification) | Every sprint |
| **Integration** | SCENARIO-* within a single capability | `SCENARIO-<CAP>-NNN` in test comments | Generator (Amelia) | Evaluator (Quinn) | Every sprint |
| **Contract** | API response shapes match consumer expectations | `CONTRACT-<CAP>-NNN` in test comments | Generator (Amelia) | Generator (self-check) + Evaluator (verification) | Every sprint |
| **E2E** | SCENARIO-* across capabilities, full stack | `ops/e2e-test-plan.md` mapped to SCENARIO-* | Planner defines plan, Generator implements | Evaluator (Quinn) against deployed system | Every sprint + final evaluation |
| **Conformance Fixture** | Assertions detect both conformant and non-conformant behavior | `FIXTURE-<CAP>-NNN` in test comments | Generator (Amelia) | Evaluator (Quinn) | Every sprint (when project is a compliance/testing tool) |

### 5.2 Test-Spec Traceability Contract

Every test file MUST contain a traceability header linking it to OpenSpec:

```python
# Tests for: openspec/capabilities/auth/spec.md
# REQ-AUTH-001: User login via JWT
# REQ-AUTH-003: Failed login rate limiting
# SCENARIO-AUTH-001: Valid credentials -> JWT issued
# SCENARIO-AUTH-005: Brute force -> account locked after 5 attempts
```

```typescript
// Tests for: openspec/capabilities/auth/spec.md
// REQ-AUTH-001: User login via JWT
// SCENARIO-AUTH-001: Valid credentials -> JWT issued
describe('SCENARIO-AUTH-001: Valid credentials', () => {
  // Given a user with valid credentials
  // When the user submits login form
  // Then a JWT token is returned AND the user is redirected to dashboard
});
```

**Rules**:
- Every test function/block MUST reference at least one REQ-* or SCENARIO-*
- Every REQ-* MUST have at least one unit test
- Every SCENARIO-* MUST have at least one integration or E2E test
- Tests without REQ-*/SCENARIO-* references are orphans — they indicate either a missing spec or a test that should be deleted
- REQ-* without tests are coverage gaps — they MUST be tracked in `_bmad/traceability.md` with Test Status: `Missing`

### 5.3 Coverage Model

Coverage operates at two levels: **spec coverage** (are all requirements tested?) and **code coverage** (are all code paths exercised?). Spec coverage is primary; code coverage is secondary.

#### Spec Coverage (Primary)

Measured by the traceability matrix in `_bmad/traceability.md`:

```markdown
| REQ ID | Capability | Spec Status | Impl Status | Test Status | Test Files |
|--------|-----------|-------------|-------------|-------------|------------|
| REQ-AUTH-001 | auth | Specified | Implemented | Covered | tests/auth/test_login.py:12 |
| REQ-AUTH-002 | auth | Specified | Implemented | Covered | tests/auth/test_refresh.py:8 |
| REQ-AUTH-003 | auth | Specified | Partial | Missing | — |
| REQ-AUTH-004 | auth | Proposed | Not Started | Not Started | — |
```

**Spec coverage metric**: `(REQ-* with Test Status "Covered") / (REQ-* with Impl Status "Implemented")`. This must be 100% — every implemented requirement must have a test.

**Scenario coverage metric**: `(SCENARIO-* with passing tests) / (total SCENARIO-*)`. This is the primary quality signal. Target: 100% for all scenarios — every scenario must have a passing test.

**Reverse spec coverage metric**: `(tests referencing a REQ-* or SCENARIO-*) / (total tests)`. This must be 100% — every test must trace back to a spec requirement or scenario. Tests without spec references are orphan tests and are not permitted. This ensures spec coverage of all tests: no test exists without a corresponding specification anchor.

#### Code Coverage (Secondary)

Traditional line/branch coverage measured by the project's coverage tooling (e.g., `pytest-cov`, `c8`, `istanbul`, `gocov`). Code coverage is a hard requirement, not a secondary signal:

- **100% line coverage**: All code paths must be exercised by tests — on all files, not just new/modified ones
- **100% branch coverage**: All conditional branches must be exercised by tests
- **No coverage regression**: Overall project coverage must not decrease sprint-over-sprint
- **Uncovered code is a hard fail**: If code coverage tools show uncovered paths, the evaluator checks whether those paths correspond to SCENARIO-* that should exist but don't — that's a spec gap that must be closed before the sprint passes

```yaml
# .harness/config.yaml addition
coverage:
  spec_coverage:
    req_implemented_must_be_tested: true    # hard gate
    scenario_pass_rate_threshold: 1.00       # 100% of all SCENARIO-* must pass
    critical_scenario_pass_rate: 1.00        # 100% of critical SCENARIO-* must pass
    all_tests_must_reference_spec: true      # hard gate — no orphan tests allowed
    orphan_test_threshold: 0                 # every test must trace to REQ-* or SCENARIO-*

  code_coverage:
    tool: "pytest-cov" | "c8" | "istanbul" | "gocov" | custom
    minimum_line_coverage: 1.00              # 100% line coverage on all files
    minimum_branch_coverage: 1.00            # 100% branch coverage on all files
    no_regression: true                       # overall coverage must not decrease
    report_path: "coverage/"
    fail_sprint_on_violation: true            # coverage violation = sprint failure
```

### 5.4 Generator (Amelia) Test Responsibilities

The generator writes tests as part of implementation, following TDD discipline:

**Before writing implementation code**:
1. Read the sprint contract — identify all REQ-* and SCENARIO-* to be implemented
2. Write test stubs for every REQ-* (unit level) and SCENARIO-* (integration level)
3. Each test stub includes the traceability header and Given/When/Then structure
4. Run tests — they should all fail (red phase)

**During implementation**:
5. Implement code to make tests pass (green phase)
6. Refactor if needed (refactor phase)
7. Run full test suite — no regressions allowed

**Before handoff**:
8. Run coverage tooling and record results
9. Verify all SCENARIO-* in the sprint contract have passing tests
10. Update traceability: in the handoff artifact, list which REQ-* are now "Covered"
11. If any REQ-* could not be tested (e.g., requires infrastructure not available), document in handoff as `test_deferred` with reason

**Generator handoff test section** (added to existing handoff schema):

```yaml
  test_results:
    total_tests: 47
    passed: 45
    failed: 2
    skipped: 0
    failing_tests:
      - name: "test_token_refresh_expired"
        scenario_id: "SCENARIO-AUTH-003"
        reason: "refresh endpoint returns 500 — likely missing migration"
    coverage:
      line_coverage: 1.00
      branch_coverage: 1.00
      coverage_delta: "+0.0%"  # vs. prior sprint
    spec_coverage:
      reqs_implemented: 5
      reqs_tested: 5
      scenarios_passing: 8
      scenarios_failing: 1
      scenarios_not_tested: 0
    deferred_tests:
      - req_id: "REQ-AUTH-007"
        reason: "requires Redis cluster — not available in test environment"
        suggested_resolution: "add Redis to docker-compose.test.yml"
```

### 5.5 Evaluator (Quinn) Test Responsibilities

The evaluator does NOT trust the generator's self-reported test results. Quinn independently verifies:

**Test execution verification**:
1. Run the full test suite from scratch (not cached results)
2. Compare results against the generator's handoff claims — flag any discrepancy
3. Run coverage tooling independently and compare against thresholds

**Test quality assessment**:
4. Read test files and verify traceability headers are present and reference valid REQ-*/SCENARIO-*
5. Check that tests are meaningful — not just asserting `True` or testing trivial getters
6. Verify negative tests exist for error paths in SCENARIO-* (Given invalid input, When... Then error)
7. Check for test isolation — tests should not depend on execution order or shared mutable state

**Spec coverage audit**:
8. Cross-reference `_bmad/traceability.md` against actual test files
9. For every REQ-* marked "Implemented", verify a test exists that references it
10. For every SCENARIO-* in the sprint contract, verify a test exercises the full Given/When/Then flow
11. Flag **phantom coverage**: tests that reference a REQ-* but don't actually exercise the requirement (e.g., testing only the happy path when the REQ-* specifies error handling)
12. **Assertion-to-spec traceability audit**: For each test that references a REQ-* or SCENARIO-*, read the assertion body and compare it against the actual spec text. Verify that what the test asserts matches what the spec requires — not just that a citation exists. Common failures:
    - Test asserts `status == 200` but spec says "SHALL return a valid XML capabilities document with elements X, Y, Z"
    - Test checks response is non-empty but spec defines a specific schema
    - Test verifies a header exists but spec mandates a specific header value
    - Test exercises a subset of the Given/When/Then but omits the Then conditions that matter

**E2E verification** (for every sprint, not just final):
12. Run E2E tests from `ops/e2e-test-plan.md` against the deployed system
13. Use browser automation (Playwright) for web apps, real protocol exchanges for APIs
14. Use proper DNS names when feasible, not just localhost
15. Document results in `ops/test-results.md`

**Evaluator report test section** (added to existing evaluation schema):

```yaml
  test_verification:
    generator_claimed_passed: 45
    evaluator_confirmed_passed: 44
    discrepancy: "test_session_timeout passes in generator run but fails in clean run — likely test pollution"

    test_quality:
      meaningful_tests: 43
      trivial_tests: 2        # flagged for rewrite
      missing_negative_tests: 3
      isolation_violations: 1
      details:
        - test: "test_login_success"
          issue: "only tests happy path — REQ-AUTH-001 specifies 'invalid credentials return 401' but no negative test exists"
        - test: "test_user_create"
          issue: "depends on test_login running first — shared session state"

    spec_coverage_audit:
      reqs_with_tests: 5
      reqs_without_tests: 0
      phantom_coverage: 1     # test exists but doesn't exercise the actual requirement
      assertion_spec_mismatches: 1  # test cites REQ-* but assertions don't match spec text
      scenarios_verified: 8
      scenarios_unverified: 1
      details:
        - req_id: "REQ-AUTH-003"
          issue: "test references REQ-AUTH-003 (rate limiting) but only tests a single failed attempt — never triggers the lockout threshold"
        - req_id: "REQ-AUTH-001"
          issue: "assertion-spec mismatch: test asserts status == 200, but spec says 'SHALL return a JWT token with sub, exp, and iat claims' — status code is necessary but not sufficient"

    code_coverage:
      line_coverage: 1.00     # evaluator's independent measurement
      branch_coverage: 1.00
      generator_claimed_line: 1.00
      discrepancy: "0% delta — no discrepancy"
      threshold_violations: []

    e2e_results:
      total_scenarios: 4
      passed: 3
      failed: 1
      details:
        - scenario: "SCENARIO-AUTH-001"
          status: "passed"
          method: "Playwright: navigated to /login, submitted valid credentials, verified JWT in cookie and redirect to /dashboard"
        - scenario: "SCENARIO-AUTH-005"
          status: "failed"
          method: "API: sent 6 POST /login with invalid password, expected 429 on 6th, got 200"
          evidence: "rate limiting not enforced — display-only counter in UI, no server-side enforcement"
```

### 5.6 Coverage Enforcement Gates

Coverage is enforced at three points in the orchestration loop. These are hard gates — the orchestrator (Bob) blocks progression when they fail.

#### Gate 1: Generator Self-Check (before handoff)

The generator MUST run tests and coverage before writing its handoff artifact. If any of these fail, the generator should attempt to fix them within its current context window. If it cannot, it writes a handoff with status `blocked` and the failing details.

| Check | Threshold | Action on Failure |
|---|---|---|
| All unit tests pass | 100% | Fix or document in handoff as blocker |
| All integration tests pass | 100% | Fix or document in handoff as blocker |
| Line coverage (all files) | 100% | Write additional tests — no waivers permitted |
| Branch coverage (all files) | 100% | Write additional tests — no waivers permitted |
| All tests reference REQ-*/SCENARIO-* | 100% | Add spec references to orphan tests or remove them |
| No coverage regression | >= 0% delta | Identify removed tests or refactored code and restore coverage |

#### Gate 2: Evaluator Verification (after generator handoff)

The evaluator independently runs all checks and adds its own. Failure here means the sprint fails and goes to retry.

| Check | Threshold | Action on Failure |
|---|---|---|
| All tests pass (clean run) | 100% | Sprint fails — critique sent to generator retry |
| Spec coverage: implemented REQ-* with tests | 100% | Sprint fails — "REQ-X-NNN has no test" in critique |
| Scenario pass rate | 100% (all scenarios) | Sprint fails — "SCENARIO-X-NNN failing" in critique |
| Line coverage (all files) | 100% | Sprint fails — "files X, Y below 100% line coverage" in critique |
| Branch coverage (all files) | 100% | Sprint fails — "files X, Y below 100% branch coverage" in critique |
| All tests reference REQ-*/SCENARIO-* | 100% | Sprint fails — "orphan tests: test_X, test_Y have no spec reference" in critique |
| No phantom coverage | 0 instances | Sprint fails — "test for REQ-X-NNN doesn't exercise the requirement" |
| No assertion-spec mismatches (critical) | 0 instances | Sprint fails — "test for REQ-X-NNN asserts status code but spec requires schema validation" |
| No assertion-spec mismatches (partial) | 0 instances | Warning — noted in critique, hard fail deferred to Gate 3 |
| Contract tests pass (if applicable) | 100% | Sprint fails — "CONTRACT-X-NNN: producer/consumer schema mismatch" |
| Conformance fixture tests pass (if applicable) | 100% golden + 100% poison | Sprint fails — false positive/negative detected in compliance assertions |
| No test isolation violations | 0 instances | Sprint fails — "test X depends on test Y execution order" |
| E2E tests pass | 100% for contracted scenarios | Sprint fails — E2E evidence included in critique |
| Test quality: no trivial tests | 0 instances | Warning (not hard fail) — noted in critique for next iteration |

#### Gate 3: Final Evaluation (after all sprints)

Full regression and cross-story integration testing:

| Check | Threshold | Action on Failure |
|---|---|---|
| Full SCENARIO-* regression suite | 100% pass | Escalate to user — regression introduced between sprints |
| Cross-story E2E scenarios | 100% pass | Escalate to user — integration gap between stories |
| Overall project code coverage | 100% line + 100% branch | Hard fail — all code must be covered by tests |
| Traceability matrix consistency | All "Implemented" REQ-* show "Covered" | Hard fail — traceability out of sync, reconcile before done |
| Assertion-spec mismatches (partial) | 0 remaining | Hard fail — all partial mismatches from Gate 2 must be resolved by final evaluation |
| Contract test coverage | All API boundaries have contracts | Hard fail — uncontracted boundaries risk silent breakage |
| Conformance fixture coverage (if applicable) | Every REQ-* with compliance assertions has golden + poison fixtures | Hard fail — untested assertion correctness |

### 5.7 Conformance Fixture Tests ("Testing the Tester")

**When this applies**: The project being refactored is itself a compliance testing tool, validator, or conformance checker (e.g., OGC spec compliance, OAuth conformance, FHIR validation). In these projects, the test pyramid validates that your code works, but not that your assertions are correct. Conformance fixture tests close this gap.

**The problem**: A compliance tool can pass all unit, integration, and E2E tests while still producing false positives (accepting non-conformant input) or false negatives (rejecting conformant input). Standard test levels don't catch this because they verify behavior, not correctness of judgment.

**Structure**:

```
tests/
  conformance_fixtures/
    <cap>/
      golden/                        # Known-conformant fixtures — every assertion MUST pass
        fixture_valid_001.json       # Known-good input with rationale
        fixture_valid_002.xml
        expected_results.yaml        # Expected: all checks pass, no violations
      poison/                        # Known-non-conformant fixtures — targeted assertions MUST fail
        fixture_invalid_001.json     # Known-bad input with specific violation
        fixture_invalid_002.xml
        expected_results.yaml        # Expected: specific checks fail, others pass
      manifest.yaml                  # Maps each fixture to REQ-*/SCENARIO-* and spec citation
```

**Fixture manifest format**:

```yaml
fixtures:
  - id: "FIXTURE-WMS-001"
    type: "golden"                   # golden = known-conformant, poison = known-non-conformant
    file: "golden/valid_capabilities_130.xml"
    spec_citation: "OGC WMS 1.3.0 §7.2.4.1: GetCapabilities response SHALL be a valid XML document"
    requirements_exercised:
      - "REQ-WMS-001"
      - "REQ-WMS-003"
    expected_verdict: "pass"         # all assertions against this fixture should pass
    expected_violations: []

  - id: "FIXTURE-WMS-002"
    type: "poison"
    file: "poison/missing_layer_element.xml"
    spec_citation: "OGC WMS 1.3.0 §7.2.4.6.2: Layer element is mandatory"
    requirements_exercised:
      - "REQ-WMS-005"
    expected_verdict: "fail"
    expected_violations:
      - check: "layer_element_present"
        reason: "Layer element deliberately omitted to test detection"
    must_not_trigger:                # guards against false positives from this poison fixture
      - "REQ-WMS-001"               # XML validity should still pass — the doc is well-formed
```

**Gates**:

| Check | Threshold | Action on Failure |
|---|---|---|
| All golden fixtures produce 0 violations | 100% | Hard fail — false negative detected, assertions are too strict or broken |
| All poison fixtures trigger expected violations | 100% | Hard fail — false positive detected, assertions miss non-conformance |
| Poison fixtures do NOT trigger `must_not_trigger` checks | 0 false positives | Hard fail — unrelated assertions failing on targeted poison input |

**Evaluator report section** (added to evaluation schema):

```yaml
    conformance_fixtures:
      golden_fixtures_total: 12
      golden_passed: 12              # all assertions pass on known-good input
      golden_false_negatives: 0      # assertions that incorrectly reject good input
      poison_fixtures_total: 8
      poison_caught: 7               # expected violations correctly detected
      poison_false_positives: 1      # a poison fixture triggered an unrelated check
      poison_missed: 0               # expected violations not detected (false positive in the tool)
      details:
        - fixture: "FIXTURE-WMS-002"
          issue: "poison fixture for missing Layer also triggers REQ-WMS-012 (CRS validation) — likely over-broad assertion"
```

### 5.8 Contract Tests (API Shape Validation)

Contract tests validate that components agree on shared API response shapes. They sit between integration tests and E2E tests in the pyramid. Without them, a backend change to a response schema breaks the frontend silently — unit tests mock the boundary, integration tests may test each side independently, and E2E tests catch it late with poor diagnostics.

**The pattern**: For every API boundary between components, define a contract that specifies the request/response schema. Both producer (backend) and consumer (frontend) test against this contract independently.

**Structure**:

```
tests/
  contracts/
    <api_boundary>/
      schema.yaml                    # Shared API contract (OpenAPI fragment or JSON Schema)
      test_producer.py               # Backend: responses match schema
      test_consumer.py               # Frontend: can parse and render responses matching schema
```

**Contract definition format**:

```yaml
# tests/contracts/discovery_api/schema.yaml
contract:
  id: "CONTRACT-DISC-001"
  endpoint: "GET /api/discovery/{id}/result"
  requirements:
    - "REQ-DISC-004"                 # Discovery results retrievable via API
    - "SCENARIO-DISC-002"            # Frontend displays discovery results

  response_schema:
    type: object
    required: ["id", "status", "results", "timestamp"]
    properties:
      id:
        type: string
        format: uuid
      status:
        type: string
        enum: ["pending", "running", "completed", "failed"]
      results:
        type: array
        items:
          type: object
          required: ["checkId", "passed", "message"]
      timestamp:
        type: string
        format: date-time

  breaking_change_policy: "Any removal or type change to a required field is a breaking change. Adding optional fields is non-breaking."
```

**Test implementation**:

```python
# tests/contracts/discovery_api/test_producer.py
# CONTRACT-DISC-001: Discovery API response shape
# REQ-DISC-004: Discovery results retrievable via API

def test_discovery_result_matches_contract(running_server, contract_schema):
    """Backend produces responses conforming to the shared contract."""
    response = requests.get(f"{running_server}/api/discovery/{test_id}/result")
    validate(response.json(), contract_schema)  # jsonschema validation


# tests/contracts/discovery_api/test_consumer.py
# CONTRACT-DISC-001: Discovery API response shape
# SCENARIO-DISC-002: Frontend displays discovery results

def test_frontend_parses_contract_response(contract_schema):
    """Frontend correctly parses every valid contract response shape."""
    for example in generate_examples(contract_schema):
        component = render_discovery_result(example)
        assert component.status_text == example["status"]
        assert len(component.result_rows) == len(example["results"])
```

**Gates**:

| Check | Threshold | Action on Failure |
|---|---|---|
| Producer tests pass (backend matches contract) | 100% | Sprint fails — backend broke the API contract |
| Consumer tests pass (frontend handles contract) | 100% | Sprint fails — frontend doesn't handle valid responses |
| No uncontracted API boundaries | 0 | Warning — new endpoint added without contract definition |

**When to add contracts**: Any API boundary where:
- Frontend consumes backend responses
- Service A calls Service B
- A published API is consumed by external clients
- Prior bugs were caused by schema drift between components

### 5.9 Assertion-to-Spec Traceability Audit

Beyond phantom coverage (tests that cite REQ-* but don't exercise the requirement), the evaluator must verify that **what each test asserts actually matches what the spec requires**. A traceability header creates a *claim* of coverage. This audit verifies the claim.

**Audit procedure** (performed by Evaluator during Gate 2):

For each test that references a REQ-* or SCENARIO-*:

1. **Read the spec text**: Open `openspec/capabilities/<cap>/spec.md`, find the referenced REQ-*/SCENARIO-*
2. **Read the test assertions**: Identify every `assert`, `expect`, or equivalent statement in the test
3. **Compare**: Does the assertion verify what the spec actually requires?

**Common failure patterns**:

| Pattern | Example | Why It's Wrong |
|---|---|---|
| **Status-code-only** | Spec: "SHALL return valid XML with elements X, Y, Z" → Test: `assert status == 200` | Status code confirms the request succeeded, not that the response is correct |
| **Existence-only** | Spec: "response MUST include `Authorization` header with value `Bearer <token>`" → Test: `assert 'Authorization' in headers` | Header exists but value could be anything |
| **Subset assertion** | Spec: "Given invalid credentials, When login, Then return 401 AND increment lockout counter AND log security event" → Test: `assert status == 401` | Two of three Then-conditions are unverified |
| **Shape-without-content** | Spec: "error response MUST include `code`, `message`, and `detail` fields" → Test: `assert isinstance(response, dict)` | Dict shape is untested |
| **Wrong granularity** | Spec: "rate limit of 100 requests per minute" → Test: `assert rate_limited == True` (after 1 request) | Threshold is never actually tested |

**Evaluator report section** (added to evaluation schema):

```yaml
    assertion_spec_audit:
      tests_audited: 45
      assertions_verified: 142
      mismatches_found: 3
      severity_breakdown:
        critical: 1       # assertion completely misses the requirement
        partial: 2         # assertion covers part of the requirement
      details:
        - test: "test_capabilities_response"
          req_id: "REQ-WMS-001"
          spec_text: "GetCapabilities response SHALL be a valid XML document conforming to schemas/wms/1.3.0/capabilities.xsd"
          assertion: "assert response.status_code == 200"
          verdict: "critical mismatch — status code does not verify XML validity or schema conformance"
          recommended_fix: "Parse response body as XML, validate against capabilities.xsd schema"
        - test: "test_login_flow"
          req_id: "SCENARIO-AUTH-001"
          spec_text: "Given valid credentials, When user submits login, Then JWT issued with sub+exp+iat AND user redirected to /dashboard"
          assertion: "assert 'token' in response.json()"
          verdict: "partial — token presence checked but JWT claims (sub, exp, iat) and redirect not verified"
```

**Gate integration**: Assertion-spec mismatches are graded by severity:
- **Critical** (assertion completely misses the requirement): Hard fail at Gate 2
- **Partial** (assertion covers some but not all conditions): Warning at Gate 2, hard fail at Gate 3

### 5.10 Test Infrastructure Bootstrap

When refactoring an existing project, the test infrastructure must be established before the first generator sprint:

1. **Choose test framework**: Match the project's language/ecosystem (pytest, Jest/Vitest, go test, etc.)
2. **Choose coverage tool**: Must produce machine-readable reports (pytest-cov, c8, istanbul, gocov)
3. **Choose E2E framework**: Playwright for web, real protocol clients for APIs, expect/pexpect for CLI
4. **Create test directory structure** mirroring `openspec/capabilities/`:

```
tests/
  capabilities/
    auth/
      unit/
        test_login.py           # REQ-AUTH-001, REQ-AUTH-003
        test_token.py           # REQ-AUTH-002
      integration/
        test_auth_flow.py       # SCENARIO-AUTH-001, SCENARIO-AUTH-003
      e2e/
        test_auth_e2e.py        # SCENARIO-AUTH-001 full stack
    payments/
      unit/
        ...
      integration/
        ...
  contracts/                          # API shape contracts (Section 5.8)
    discovery_api/
      schema.yaml                     # CONTRACT-DISC-001
      test_producer.py
      test_consumer.py
  conformance_fixtures/               # Golden/poison fixtures (Section 5.7, when applicable)
    wms/
      golden/
        valid_capabilities_130.xml
        expected_results.yaml
      poison/
        missing_layer_element.xml
        expected_results.yaml
      manifest.yaml
  conftest.py                         # shared fixtures
  coverage.ini                        # coverage configuration
```

5. **Create `ops/e2e-test-plan.md`**:

```markdown
# E2E Test Plan

## Auth Capability

| SCENARIO ID | Description | Method | Environment | Prerequisites |
|---|---|---|---|---|
| SCENARIO-AUTH-001 | Valid login | Playwright: navigate /login, submit, verify JWT + redirect | Staging | Test user seeded in DB |
| SCENARIO-AUTH-005 | Brute force lockout | API: 6x POST /login with bad password | Staging | Clean rate limit state |

## Payments Capability
...
```

6. **Create coverage baseline**: Run existing tests (if any), record current coverage as the no-regression baseline
7. **Wire coverage into CI**: Coverage reports generated on every test run, threshold violations fail the build

---

## Phase 6: Orchestration Harness

### 6.1 Harness Loop

```
INITIALIZE:
  Read _bmad/prd.md, _bmad/architecture.md
  Read all openspec/capabilities/*/spec.md
  Build initial traceability matrix in _bmad/traceability.md

LOOP until (all stories complete) OR (max_iterations reached):

  0. DISCOVERY AGENT — Analyst Mary (fresh context, CONDITIONAL):
     Trigger: user request references unfamiliar domain, OR scope is ambiguous
     Reads: user intent + _bmad/ docs + existing specs + external sources
     Produces: _bmad/product-brief.md, discovery handoff
     Gate: if handoff.ready_for_planning == false, escalate to user for clarification
     Skip if: domain is well-understood and scope is clear

  1. PLANNER AGENT — PM John (fresh context):
     Reads: user intent + _bmad/ docs + product-brief (if exists) + existing specs + prior handoffs
     Produces: epics, stories, change proposals with delta specs
     Flags: whether architect or design agent is needed (new capability? user-facing?)

  2. ARCHITECT AGENT — Winston (fresh context, CONDITIONAL):
     Trigger: planner flags new capability, architectural change, or infrastructure impact
     Reads: change proposals + architecture.md + design.md files
     Produces: updated design.md, ADRs, readiness verdict
     Quality gate: BMAD implementation readiness check (PASS/CONCERNS/FAIL)
       - FAIL blocks all implementation
       - CONCERNS documented and escalated to user
     Skip if: story is within existing capability with stable design

  3. DESIGN AGENT — UX Designer Sally (fresh context, CONDITIONAL):
     Trigger: planner flags user-facing interface work, OR evaluator flagged UX issues
     Reads: _bmad/prd.md + _bmad/ux-spec.md + relevant spec.md + evaluator feedback
     Produces: updated _bmad/ux-spec.md with interaction patterns, components, accessibility reqs
     Gate: if handoff.ready_for_implementation == false, escalate to user for design decisions
     Skip if: work is backend-only or existing ux-spec.md covers the interaction patterns

  4. FOR EACH story in execution order (managed by ORCHESTRATOR — Scrum Master Bob):

     a. ORCHESTRATOR (script, not LLM) constructs sprint contract from:
        - Story acceptance criteria (REQ-* and SCENARIO-*)
        - UX spec constraints (if Sally produced or updated ux-spec.md)
        - Verification methods from ops/e2e-test-plan.md
        - Evaluation criteria from .harness/config.yaml

     b. GENERATOR AGENT — Developer Amelia (fresh context):
        Reads: story + spec.md + design.md + ux-spec.md + contract + last handoff
        Process: write tests -> implement -> self-check -> commit -> handoff
        Produces: code, tests, updated spec status, handoff artifact

     c. EVALUATOR AGENT — QA Quinn (fresh context):
        Reads: contract + story + spec.md + ux-spec.md + produced artifacts
        (NOT generator context — never sees generator's reasoning or conversation)
        Process: run tests -> E2E verify -> grade against contract -> report
        Produces: evaluation report with per-REQ-* verification
        UX check: if ux-spec.md exists, grades UX coherence against Sally's specs

     d. ORCHESTRATOR routes based on evaluation:
        IF evaluation passes:
           Update _bmad/traceability.md (mark REQ-* as Implemented + Covered)
           Update ops/changelog.md
           Archive change proposal if complete: move to openspec/change-proposals/archive/
           Proceed to next story
        ELSE IF evaluator flagged UX issues AND design agent not yet invoked:
           Invoke DESIGN AGENT (Sally) with evaluator feedback, then retry
        ELSE IF retries remaining:
           Feed evaluation critique into next generator handoff (fresh context)
           GOTO 4b
        ELSE:
           Escalate to user with evaluation report
           Update ops/known-issues.md

  5. FINAL EVALUATION — QA Quinn (fresh context):
     Cross-story integration testing
     Full SCENARIO-* regression suite
     UX coherence check across all user-facing stories (if ux-spec.md exists)
     Verify _bmad/traceability.md is consistent with actual state
     Update _bmad/architecture.md "Last Reconciled" date

  6. RECONCILIATION (ORCHESTRATOR, deterministic):
     Update ops/status.md (what's working, what's next)
     Update ops/changelog.md (what was done, traced to user instructions)
     Update ops/metrics.md (session metrics, token costs per agent)
     If implementation diverged from spec, update spec to match reality with rationale
```

### 6.2 Alternative: Claude Code Agent Teams as Orchestrator

When running inside Claude Code, the experimental Agent Teams feature (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) can replace the custom orchestration script with native multi-agent coordination. This maps our BMAD roles directly onto Claude Code's team architecture.

**Enable in settings.json:**

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

#### BMAD-to-Agent-Teams Mapping

| BMAD Role | Agent Teams Role | Spawned As |
|---|---|---|
| Scrum Master (Bob) | **Team Lead** | The main Claude Code session — coordinates, assigns, synthesizes |
| Analyst (Mary) | Teammate | Spawned conditionally for research tasks |
| PM (John) | Teammate | Spawned for planning; sends plan back to lead for approval |
| Architect (Winston) | Teammate | Spawned conditionally; requires plan approval gate |
| UX Designer (Sally) | Teammate | Spawned conditionally for UI work |
| Developer (Amelia) | Teammate(s) | One per story; can parallelize across independent stories |
| QA (Quinn) | Teammate | Spawned after each generator completes; runs E2E verification |

**Key advantages over custom script orchestration:**

1. **Native parallelism**: Multiple generator teammates can work on independent stories simultaneously (not just sequential sprints)
2. **Inter-agent messaging**: Quinn can message Amelia directly with bug reports; Amelia can ask Winston for design clarification — without routing through the orchestrator
3. **Shared task list**: The team lead maintains a task list with dependency tracking. When a blocking story completes, dependent stories unblock automatically.
4. **Plan approval gates**: The architect and planner can be required to get lead approval before proceeding — implementing the BMAD readiness check natively
5. **Live steering**: The user can interact with any teammate directly (click into their pane or Shift+Down to cycle)

#### Team Spawn Pattern

```text
Create an agent team for epic-3: Payment Processing.

Spawn teammates:
- "planner" using Sonnet: Read _bmad/prd.md, openspec/capabilities/payments/spec.md,
  and produce stories for epic-3. Require plan approval before finalizing.
- "architect" using Opus: Review the planner's stories and update
  openspec/capabilities/payments/design.md. Require plan approval.

After plans are approved, spawn:
- "amelia-checkout" using Opus: Implement story-checkout-flow per sprint contract.
  Work in src/payments/checkout/. Run tests before reporting done.
- "amelia-refunds" using Opus: Implement story-refund-processing per sprint contract.
  Work in src/payments/refunds/. Run tests before reporting done.

After both generators complete, spawn:
- "quinn" using Opus: Run full test suite, E2E tests from ops/e2e-test-plan.md,
  and evaluate against sprint contracts. Be skeptical. Report all failures.
```

#### Quality Gate Hooks

Use Claude Code hooks to enforce the harness's quality gates natively:

```json
{
  "hooks": {
    "TaskCompleted": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "scripts/validate-task-completion.sh",
            "description": "Verify tests pass, coverage thresholds met, handoff artifact written"
          }
        ]
      }
    ],
    "TeammateIdle": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "scripts/check-teammate-done.sh",
            "description": "If teammate has unclaimed tasks, send feedback to keep working"
          }
        ]
      }
    ]
  }
}
```

**`scripts/validate-task-completion.sh`** (exit code 2 = reject completion with feedback):

```bash
#!/bin/bash
# Reject task completion if tests fail or coverage thresholds are violated
set -e

# Run tests
if ! make test 2>/dev/null; then
    echo "Task completion rejected: tests are failing" >&2
    exit 2
fi

# Check coverage thresholds
COVERAGE=$(make coverage-report 2>/dev/null | grep "Total" | awk '{print $NF}' | tr -d '%')
if [ "${COVERAGE:-0}" -lt 100 ]; then
    echo "Task completion rejected: code coverage ${COVERAGE}% is below 100% threshold" >&2
    exit 2
fi

# Check that handoff artifact was written
LATEST_HANDOFF=$(ls -t .harness/handoffs/ 2>/dev/null | head -1)
if [ -z "$LATEST_HANDOFF" ]; then
    echo "Task completion rejected: no handoff artifact found in .harness/handoffs/" >&2
    exit 2
fi

exit 0
```

#### When to Use Agent Teams vs. Custom Script

| Scenario | Use Agent Teams | Use Custom Script |
|---|---|---|
| Running inside Claude Code CLI | Preferred — native integration, live steering | Fallback if teams feature is unstable |
| CI/CD pipeline or headless automation | Not available — needs interactive session | Required |
| Independent stories that can parallelize | Strong advantage — multiple generators in parallel | Sequential only |
| Need deterministic, repeatable orchestration | Less predictable — LLM-driven coordination | Preferred — deterministic routing |
| Need inter-agent discussion (e.g., Quinn questioning Amelia's choices) | Native messaging support | Not possible — file-based only |
| Cost-sensitive | Higher — each teammate is a full session | Lower — one session at a time |

#### Limitations to Account For

- **No session resumption**: If the lead session dies, teammates are orphaned. Use handoff artifacts as the recovery mechanism.
- **No nested teams**: Teammates cannot spawn their own teams. Keep the team flat — lead coordinates all BMAD roles.
- **File conflict risk**: Two generator teammates editing the same file will overwrite each other. Structure stories so each owns distinct files. Use the task dependency system to serialize stories that touch shared code.
- **One team per session**: Clean up before starting a new epic's team.

### 6.3 Harness Configuration

```yaml
# .harness/config.yaml
harness:
  max_retries_per_sprint: 3
  max_total_iterations: 15
  context_reset_strategy: "full"    # always full reset between agents

  directories:
    handoffs: ".harness/handoffs/"
    contracts: ".harness/contracts/"
    evaluations: ".harness/evaluations/"
    prompts: ".harness/prompts/"

  # ── Agent Definitions ──────────────────────────────────────────────
  #
  # Performance principles applied per agent (see Appendix E):
  #   1. Tool scoping: only expose tools the agent needs (fewer tools = less
  #      context overhead = faster tool selection)
  #   2. Read-only annotations: tools marked readOnlyHint=true execute in
  #      parallel when the model requests multiple tool calls in one turn
  #   3. Effort level: trade reasoning depth for speed on well-scoped tasks
  #   4. Model routing: use the cheapest model that produces acceptable quality
  #   5. Budget/turn limits: prevent runaway sessions
  #   6. Mode: always use mode: "auto" — Claude plans better outside plan mode

  agents:
    discovery:                        # BMAD: Analyst Mary
      model: "claude-sonnet-4-6"      # research doesn't need Opus
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "medium"                # balanced — research needs some reasoning
      prompt: ".harness/prompts/discovery.md"
      tools: ["Read", "Grep", "Glob", "WebSearch", "WebFetch"]  # all read-only → parallel
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
        WebSearch: { readOnlyHint: true }
        WebFetch: { readOnlyHint: true }
      invocation: "conditional"
      max_turns: 20
      max_budget_usd: 2.00
      reads: ["_bmad/", "openspec/", "docs/"]
      writes: ["_bmad/product-brief.md", ".harness/handoffs/"]

    planner:                          # BMAD: PM John
      model: "claude-sonnet-4-6"      # sufficient for planning, faster/cheaper
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "medium"
      prompt: ".harness/prompts/planner.md"
      tools: ["Read", "Grep", "Glob", "Write"]  # scoped: no Bash, no Edit
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
        Write: { readOnlyHint: false, destructiveHint: false }
      invocation: "always"
      max_turns: 30
      max_budget_usd: 5.00
      reads: ["_bmad/", "openspec/", "epics/", ".harness/handoffs/"]
      writes: ["epics/", "openspec/change-proposals/"]

    architect:                        # BMAD: Architect Winston
      model: "claude-opus-4-6"        # needs deep reasoning for design decisions
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "high"
      prompt: ".harness/prompts/architect.md"
      tools: ["Read", "Grep", "Glob", "Write"]  # scoped: read codebase + write design docs
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
      invocation: "conditional"
      max_turns: 25
      max_budget_usd: 10.00
      reads: ["_bmad/", "openspec/"]
      writes: ["_bmad/architecture.md", "openspec/capabilities/*/design.md"]

    design:                           # BMAD: UX Designer Sally
      model: "claude-sonnet-4-6"
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "medium"
      prompt: ".harness/prompts/design.md"
      tools: ["Read", "Grep", "Glob", "Write"]  # scoped: no Bash, no browser
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
      invocation: "conditional"
      max_turns: 20
      max_budget_usd: 3.00
      reads: ["_bmad/", "openspec/", ".harness/evaluations/"]
      writes: ["_bmad/ux-spec.md"]

    generator:                        # BMAD: Developer Amelia
      model: "claude-opus-4-6"        # best for sustained implementation sessions
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "high"                  # implementation needs thorough reasoning
      prompt: ".harness/prompts/generator.md"
      tools: ["Read", "Edit", "Write", "Grep", "Glob", "Bash"]  # full dev toolset, no Playwright
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
        # Edit, Write, Bash are NOT read-only → execute sequentially (correct for code modification)
      invocation: "always"
      max_turns: 100
      max_budget_usd: 50.00
      max_session_duration: "2h"
      reads: ["openspec/", "epics/stories/", "_bmad/ux-spec.md", ".harness/contracts/", ".harness/handoffs/"]
      writes: ["src/", "tests/", "openspec/capabilities/*/spec.md", ".harness/handoffs/"]

    evaluator:                        # BMAD: QA Quinn
      model: "claude-opus-4-6"
      mode: "auto"                    # never use plan mode — Claude plans better without it
      effort: "high"                  # QA needs thorough analysis to catch issues
      prompt: ".harness/prompts/evaluator.md"
      tools: ["Read", "Grep", "Glob", "Bash"]  # + Playwright MCP if web app
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
        # Bash is NOT read-only (runs tests, may modify state)
        # Playwright tools: readOnlyHint varies (navigation=true, clicks=false)
      invocation: "always"
      skepticism_level: "high"
      max_turns: 50
      max_budget_usd: 20.00
      reads: ["openspec/", "epics/stories/", "_bmad/ux-spec.md", ".harness/contracts/", "ops/e2e-test-plan.md"]
      writes: [".harness/evaluations/", "ops/test-results.md"]

    skill_evolver:                    # Trace2Skill evolution agent
      model: "claude-sonnet-4-6"      # analysis doesn't need Opus; volume matters more
      mode: "auto"
      effort: "medium"
      prompt: ".harness/prompts/skill-evolver.md"
      tools: ["Read", "Grep", "Glob", "Write"]
      tool_annotations:
        Read: { readOnlyHint: true }
        Grep: { readOnlyHint: true }
        Glob: { readOnlyHint: true }
      invocation: "conditional"       # triggered by orchestrator after N sprints
      max_turns: 30
      max_budget_usd: 5.00
      reads: [".harness/handoffs/", ".harness/evaluations/", ".harness/prompts/", ".harness/skills/"]
      writes: [".harness/patches/", ".harness/skills/", ".harness/prompts/"]
      sub_agents:
        error_analyst:                # 𝒜⁻ — one per failed trace, runs in parallel
          model: "claude-sonnet-4-6"
          mode: "auto"
          effort: "medium"
          max_turns: 15
          max_budget_usd: 1.00
        success_analyst:              # 𝒜⁺ — one per successful trace, runs in parallel
          model: "claude-haiku-4-5-20251001"  # lighter analysis for success patterns
          mode: "auto"
          effort: "low"
          max_turns: 10
          max_budget_usd: 0.50

    # orchestrator is NOT listed here — it is a script, not an LLM agent
    # see: scripts/orchestrate.py

  evaluation_criteria:
    - name: "Spec Fidelity"
      weight: 0.30
      hard_fail: "spec contradicted without rationale"
    - name: "Functional Completeness"
      weight: 0.30
      hard_fail: "critical SCENARIO-* fails"
    - name: "Integration Correctness"
      weight: 0.20
      hard_fail: "E2E test fails"
    - name: "Code Quality"
      weight: 0.10
    - name: "Robustness"
      weight: 0.10
      hard_fail: "security vulnerability or data loss"
```

---

## Phase 7: Spec Reconciliation Protocol

After each sprint and at session end, reconcile specs with reality. This is the discipline that keeps the system honest across context resets.

### 7.1 Per-Sprint Reconciliation

After the evaluator passes a sprint:

1. **Update spec status**: In `openspec/capabilities/<cap>/spec.md`, mark implemented REQ-* with `Status: Implemented`
2. **Update traceability**: In `_bmad/traceability.md`, update the Impl Status and Test Status columns for affected REQ-*
3. **Apply delta specs**: If a change proposal drove this work, apply its delta specs to the main spec files:
   - ADDED requirements: append to main spec
   - MODIFIED requirements: overwrite in main spec (note prior version)
   - REMOVED requirements: delete from main spec (note rationale)
4. **Archive change proposal**: Move completed change proposal to `openspec/change-proposals/archive/`
5. **Resolve discrepancies**: If implementation diverged from spec, update spec to match reality with documented rationale — never leave specs and code disagreeing silently

### 7.2 Session-End Reconciliation

Before ending a session:

1. Update `ops/status.md` — this is the handoff document for the next session (human or AI)
2. Update `ops/changelog.md` — every entry traceable to the user instruction that triggered it
3. Run `python3 scripts/session-metrics.py` to update `ops/metrics.md`
4. Verify `_bmad/traceability.md` matches actual implementation state
5. If `_bmad/architecture.md` "Last Reconciled" is now stale, update it

---

## Phase 8: Skill Evolution (Trace2Skill)

Agent prompts are skills — structured knowledge that shapes agent behavior. Without systematic evolution, prompts are written once and then diverge from what actually works as the project accumulates execution experience. This phase introduces a Trace2Skill loop (inspired by [Trace2Skill](https://arxiv.org/abs/2603.25158)) that evolves agent prompts from execution traces, closing the gap between observed outcomes and encoded guidance.

### 8.1 Core Concepts

**Skills as evolving artifacts**: Each agent prompt in `.harness/prompts/` is a skill — a structured knowledge package combining procedural guidance (the prompt itself), executable helpers (scripts), and edge-case references. Skills start as human-authored or LLM-drafted documents and improve through systematic analysis of execution traces.

**Execution traces**: Every sprint produces traces — the handoff artifact, evaluator report, retry history, git diff, and (when available) the raw conversation. These traces encode what worked, what failed, and why.

**The evolution loop**:

```
                    ┌─────────────────────────────────┐
                    │   Sprint Execution (Phase 6)     │
                    │   Agent πθ runs with skill 𝒮ₙ    │
                    └──────────┬──────────────────────┘
                               │
                               ▼
                    ┌─────────────────────────────────┐
                    │   Trace Collection               │
                    │   Handoffs + evaluations +       │
                    │   retry history + git diffs      │
                    └──────────┬──────────────────────┘
                               │
                    ┌──────────▼──────────────────────┐
                    │   Parallel Patch Proposal         │
                    │   Independent analysts examine    │
                    │   each trace against current 𝒮ₙ   │
                    └──────────┬──────────────────────┘
                               │
                    ┌──────────▼──────────────────────┐
                    │   Hierarchical Consolidation     │
                    │   Merge patches, resolve          │
                    │   conflicts, filter by prevalence │
                    └──────────┬──────────────────────┘
                               │
                    ┌──────────▼──────────────────────┐
                    │   Evolved Skill 𝒮ₙ₊₁             │
                    │   Updated .harness/prompts/       │
                    │   + scripts/ + references/        │
                    └──────────┬──────────────────────┘
                               │
                               └───────── feeds back ──→ next sprint
```

### 8.2 Trace Collection

After each sprint (or batch of sprints), collect traces and partition them:

```yaml
# .harness/skills/traces.yaml (intermediate, regenerated each evolution cycle)
traces:
  successful:   # 𝒯⁺ — sprints that passed evaluator on first attempt
    - sprint: 3
      handoff: ".harness/handoffs/handoff-sprint-3.yaml"
      evaluation: ".harness/evaluations/eval-sprint-3.yaml"
      git_diff: "git diff sprint-2..sprint-3"
      agent: "generator"

  failed:       # 𝒯⁻ — sprints that failed evaluator or required retries
    - sprint: 5
      handoff: ".harness/handoffs/handoff-sprint-5.yaml"
      evaluation: ".harness/evaluations/eval-sprint-5.yaml"
      retry_count: 2
      failure_class: "phantom coverage — tests cited REQ-* but didn't exercise it"
      agent: "generator"

  evaluator_traces:  # separate track for evaluator skill evolution
    - sprint: 5
      evaluation: ".harness/evaluations/eval-sprint-5.yaml"
      missed_issues: ["rate limiting not enforced — caught in E2E but not spec audit"]
      false_alarms: []
```

### 8.3 Parallel Patch Proposal

Independent analyst sub-agents examine each trace and propose patches to the relevant skill. Each analyst operates on a **frozen copy** of the current skill — no analyst sees another's patches, preserving diversity.

**Error Analyst (𝒜⁻)** — for failed traces:
1. Read the failed sprint's handoff, evaluator critique, and retry history
2. Read the current skill that the agent was using (`.harness/prompts/<agent>.md`)
3. Identify the root cause: Was the failure due to missing guidance in the prompt? Ambiguous instruction? Incorrect procedure?
4. Propose a **skill patch**: a specific edit to the prompt, a new script, or a new reference document
5. Include causal evidence: "Sprint 5 failed because the generator didn't recalculate formulas after cell writes. The prompt says nothing about formula dependencies."

**Success Analyst (𝒜⁺)** — for successful traces:
1. Read the successful sprint's handoff and evaluator report
2. Identify generalizable patterns: What did the agent do that worked well and isn't already captured in the skill?
3. Propose a patch that encodes the effective pattern
4. Flag patterns that are already in the skill (reinforcement signal, not a new patch)

**Patch format**:

```yaml
# .harness/patches/patch-sprint-5.yaml
patch:
  id: "PATCH-GEN-005"
  source_trace: "sprint-5"
  trace_type: "failure"       # or "success"
  target_skill: ".harness/prompts/generator.md"
  root_cause: "Generator wrote cell values but didn't trigger formula recalculation. Downstream cells showed stale computed values."
  proposed_edit:
    type: "append_section"    # append_section | modify_section | add_script | add_reference
    location: "## Implementation Checklist"
    content: |
      ### Formula Recalculation
      After writing cell values that feed into formulas, ALWAYS:
      1. Run recalculation (openpyxl: wb.calculation.calcMode = 'auto')
      2. Reopen the file with data_only=True to verify computed values
      3. Compare computed values against expected results
  causal_evidence: "5/7 sprint-5 failures traced to stale formula values"
  generalizability: "high"    # high | medium | low — analyst's assessment
```

### 8.4 Hierarchical Consolidation

Patches from all analysts merge hierarchically into a unified skill update. The merge operator resolves conflicts and filters by prevalence.

**Merge procedure**:

1. **Batch patches** by target skill (e.g., all generator patches together, all evaluator patches together)
2. **Deduplicate**: When multiple patches propose the same edit, keep the best-worded version
3. **Resolve conflicts**: When patches propose contradictory edits, choose the one with stronger causal evidence or synthesize both
4. **Filter by prevalence**: Patches that appear independently in multiple traces are treated as systematic patterns — they go into the main skill document. Patches from a single trace are treated as edge cases — they go into `references/`
5. **Validate**: The merged patch must not introduce contradictions with existing skill content

**Prevalence tiers**:

| Prevalence | Threshold | Destination | Rationale |
|---|---|---|---|
| **High** | Appears in ≥30% of traces | Main skill document (SKILL.md or prompt .md) | Systematic property — always relevant |
| **Medium** | Appears in 10-30% of traces | Skill document with conditional framing ("When X, do Y") | Situationally relevant |
| **Low** | Appears in <10% of traces | `references/` subdirectory, linked from main skill | Edge case — don't clutter the main prompt |

**Conflict resolution rules**:
- When two patches target the same section with different advice, prefer the one backed by more failure traces (preventing bugs > encoding success patterns)
- When a success patch contradicts a failure patch for the same behavior, the failure patch wins (conservative)
- When patches are complementary (both correct but addressing different aspects), merge into a combined edit
- Line-level independence: merged patches must not target overlapping text ranges

### 8.5 Skill Versioning

Every evolution cycle produces a new skill version. The changelog tracks what changed and why.

```yaml
# .harness/skills/changelog.yaml
versions:
  - version: "1.0.0"
    date: "2026-03-15"
    type: "initial"
    description: "Human-authored generator prompt"

  - version: "1.1.0"
    date: "2026-03-20"
    type: "evolution"
    trigger: "After sprints 1-5 (3 failures, 2 successes)"
    traces_analyzed: 5
    patches_proposed: 12
    patches_accepted: 7
    patches_filtered: 5        # low prevalence → references/
    changes:
      - skill: ".harness/prompts/generator.md"
        edit: "Added formula recalculation checklist"
        prevalence: "high (4/5 traces)"
        evidence: "Sprints 2,3,5 failed due to stale formula values"
      - skill: ".harness/skills/references/datetime_handling.md"
        edit: "New reference doc for timezone edge cases"
        prevalence: "low (1/5 traces)"
        evidence: "Sprint 4 success pattern — not yet validated across traces"
    effectiveness:
      pre_evolution_pass_rate: 0.40    # 2/5 sprints passed first attempt
      post_evolution_pass_rate: null   # measured after next batch

  - version: "1.2.0"
    date: "2026-03-28"
    type: "evolution"
    trigger: "After sprints 6-10 (1 failure, 4 successes)"
    effectiveness:
      pre_evolution_pass_rate: 0.80    # 4/5 sprints passed first attempt
      improvement_from_v1_0: "+40pp"
```

### 8.6 Evolution Triggers and Cadence

| Trigger | When to Evolve | Scope |
|---|---|---|
| **Batch completion** | After every N sprints (default: 5) | Evolve all agent skills |
| **Failure spike** | 3+ consecutive sprint failures | Evolve the failing agent's skill immediately |
| **Evaluator miss** | Evaluator fails to catch an issue found in E2E or production | Evolve the evaluator skill |
| **New capability area** | First sprint in a new `openspec/capabilities/<cap>/` | Check if existing skills generalize; evolve if not |
| **Model change** | Switching to a new model version | Re-evaluate all skills — model strengths/weaknesses shift |

**Cadence rules**:
- Never evolve mid-sprint — evolution happens between sprints, not during
- Always run at least 3 sprints before the first evolution cycle (need enough traces)
- Track pre/post evolution pass rates to measure skill effectiveness
- If a skill evolution *decreases* pass rate, revert to the prior version and investigate

### 8.7 Skill Deepening vs. Skill Creation

Two modes, following Trace2Skill's distinction:

**Skill Deepening** (refining existing prompts):
- Start with the current human-authored or previously-evolved prompt (𝒮ₙ)
- Analyze traces from sprints that used 𝒮ₙ
- Produce 𝒮ₙ₊₁ by applying consolidated patches
- The prompt improves incrementally; the structure is preserved

**Skill Creation** (bootstrapping new prompts from scratch):
- When adding a new agent role or capability-specific prompt
- Start with a parametric draft (ask the LLM to write a prompt from its general knowledge)
- Run sprints using the draft
- Evolve the draft using the same trace → patch → consolidate pipeline
- The draft rapidly improves because the initial version is weak — evolution has more signal

### 8.8 What Skills to Evolve

Not everything should be evolved. The skill evolution loop targets:

| Artifact | Evolve? | Rationale |
|---|---|---|
| `.harness/prompts/generator.md` | **Yes** | Most execution traces, highest impact on output quality |
| `.harness/prompts/evaluator.md` | **Yes** | Evaluator effectiveness directly determines quality gate reliability |
| `.harness/prompts/planner.md` | **Selectively** | Evolve when sprint decomposition consistently fails (stories too large, missing dependencies) |
| `.harness/prompts/architect.md` | **Selectively** | Evolve when design decisions cause downstream implementation failures |
| `.harness/prompts/discovery.md` | **Rarely** | Discovery is less structured; evolution signal is weak |
| `.harness/prompts/design.md` | **Rarely** | UX guidance is project-specific; hard to generalize from traces |
| Sprint contracts | **No** | Contracts are per-sprint, not long-lived |
| Specs (REQ-*/SCENARIO-*) | **No** | Specs evolve through reconciliation (Phase 7), not trace analysis |

### 8.9 Cross-Model Skill Transfer

A key property from Trace2Skill: evolved skills are declarative and architecture-agnostic. Skills evolved by one model can be used by another.

**When this matters**:
- Switching from Opus to Sonnet for cost optimization — does the skill still work?
- Upgrading to a new model version — does the skill need re-evolution?
- Running different agents on different models (e.g., generator on Opus, evaluator on Sonnet)

**Transfer protocol**:
1. After evolving a skill on model A, run 3 validation sprints on model B using the evolved skill
2. If pass rate holds (within 10pp), the skill transfers
3. If pass rate drops significantly, run a model-specific evolution cycle on model B's traces
4. Track which skills are model-specific vs. model-agnostic in the changelog

### 8.10 Integration with Orchestration (Phase 6)

The orchestrator's loop gains an evolution step:

```
ORCHESTRATE:
  ...existing loop...
  
  AFTER every N sprints (configurable, default 5):
    a. Collect traces from completed sprints since last evolution
    b. Partition into 𝒯⁺ (passed first attempt) and 𝒯⁻ (failed or retried)
    c. IF |𝒯⁻| >= 3 OR evolution_interval_reached:
       i.   Launch parallel Error Analysts (one per 𝒯⁻ trace)
       ii.  Launch parallel Success Analysts (one per 𝒯⁺ trace)
       iii. Collect patch pool 𝒫
       iv.  Run hierarchical consolidation: merge → deduplicate → filter by prevalence
       v.   Apply consolidated patch to skill artifacts
       vi.  Update .harness/skills/changelog.yaml
       vii. Log evolution event in ops/changelog.md
    d. Resume sprint loop with evolved skills
```

---

## Phase 9: Refactor Execution Checklist

### BMAD + OpenSpec Bootstrap

- [ ] Create `_bmad/` with `prd.md`, `architecture.md`, `traceability.md`
- [ ] Create `openspec/capabilities/` tree with `spec.md` and `design.md` per capability
- [ ] Assign `REQ-*` and `SCENARIO-*` identifiers to all discovered requirements
- [ ] Create `openspec/AGENTS.md` with instructions for AI agents working with specs
- [ ] Create `epics/` and `epics/stories/` structure
- [ ] Populate `_bmad/traceability.md` with initial REQ-* -> status mapping

### Harness Infrastructure

- [ ] Create `.harness/` directory structure
- [ ] Write agent prompts in `.harness/prompts/` (all 6 LLM agents: discovery, planner, architect, design, generator, evaluator)
- [ ] Write `.harness/config.yaml` with evaluation criteria and agent configuration
- [ ] Implement orchestration loop (`scripts/orchestrate.py` or shell script)
- [ ] Implement handoff artifact read/write/validate utilities

### Evaluation Infrastructure

- [ ] Write evaluator prompt with explicit skepticism tuning in `.harness/prompts/evaluator.md`
- [ ] Define project-specific evaluation criteria with weights, thresholds, and hard-fail conditions
- [ ] Set up E2E testing capability (Playwright, API harness, or equivalent)
- [ ] Write `ops/e2e-test-plan.md` mapping SCENARIO-* to E2E test procedures
- [ ] Calibrate evaluator against known-good and known-bad examples

### Testing Architecture (Phase 5)

- [ ] Choose and configure test framework matching project language/ecosystem
- [ ] Choose and configure coverage tool that produces machine-readable reports
- [ ] Choose and configure E2E framework (Playwright for web, protocol clients for APIs, pexpect for CLI)
- [ ] Create `tests/capabilities/` directory structure mirroring `openspec/capabilities/`
- [ ] Establish coverage baseline from existing tests (if any)
- [ ] Configure coverage thresholds in `.harness/config.yaml` (spec coverage + code coverage)
- [ ] Wire coverage reporting into test runner (coverage report generated on every run)

### Contract Tests (Phase 5.8)

- [ ] Identify all API boundaries between components (frontend↔backend, service↔service)
- [ ] Create `tests/contracts/` directory with schema + producer/consumer test pairs per boundary
- [ ] Define shared schema (OpenAPI fragment or JSON Schema) for each contracted endpoint
- [ ] Write producer tests: backend responses validate against shared schema
- [ ] Write consumer tests: frontend/client correctly parses all valid schema shapes
- [ ] Add CONTRACT-* IDs to traceability headers and `_bmad/traceability.md`

### Conformance Fixture Tests (Phase 5.7 — when project is a compliance/testing tool)

- [ ] Create `tests/conformance_fixtures/` directory structure with golden/ and poison/ subdirectories
- [ ] Build golden fixtures: known-conformant inputs where every assertion MUST pass
- [ ] Build poison fixtures: known-non-conformant inputs where targeted assertions MUST fail
- [ ] Write fixture manifests mapping each fixture to REQ-*/SCENARIO-* and spec citations
- [ ] Define `must_not_trigger` guards on poison fixtures to catch false positives
- [ ] Add FIXTURE-* IDs to traceability headers

### Test-Spec Linkage

- [ ] Ensure every test file has a traceability header referencing REQ-* and SCENARIO-*
- [ ] Ensure every REQ-* with Impl Status "Implemented" has at least one test (or is marked "Missing" in traceability)
- [ ] Ensure every SCENARIO-* has at least one integration or E2E test exercising the full Given/When/Then flow
- [ ] Verify no orphan tests exist (tests without REQ-*/SCENARIO-* references)
- [ ] Verify no phantom coverage exists (tests that reference a REQ-* but don't actually exercise it)
- [ ] Verify SCENARIO-* use Given/When/Then BDD format in spec.md
- [ ] Perform assertion-to-spec traceability audit: verify each test's assertions match the actual spec text, not just that a REQ-* citation exists

### Coverage Enforcement

- [ ] Configure Gate 1 (generator self-check): tests pass + coverage thresholds on new files
- [ ] Configure Gate 2 (evaluator verification): independent test run + spec coverage audit + test quality check + assertion-spec audit + contract tests + conformance fixtures + E2E
- [ ] Configure Gate 3 (final evaluation): full regression + cross-story E2E + traceability consistency + all partial assertion mismatches resolved + contract coverage complete
- [ ] Set hard-fail threshold: spec coverage = 100% of implemented REQ-* must be tested
- [ ] Set hard-fail threshold: all SCENARIO-* pass rate = 100%
- [ ] Set hard-fail threshold: reverse spec coverage = 100% — every test must reference a REQ-* or SCENARIO-*
- [ ] Set hard-fail threshold: assertion-spec critical mismatches = 0
- [ ] Set hard-fail threshold: contract test pass rate = 100% (when contracts exist)
- [ ] Set hard-fail threshold: conformance fixtures = 100% golden pass + 100% poison detection (when applicable)
- [ ] Set hard-fail threshold: 100% line coverage on all files — no exceptions
- [ ] Set hard-fail threshold: 100% branch coverage on all files — no exceptions
- [ ] Set no-regression rule: overall project coverage must not decrease sprint-over-sprint

### Skill Evolution (Phase 8)

- [ ] Create `.harness/skills/` directory with `SKILL.md`, `scripts/`, `references/`
- [ ] Create `.harness/patches/` directory for intermediate patch artifacts
- [ ] Write `.harness/prompts/skill-evolver.md` with error/success analyst instructions
- [ ] Initialize `.harness/skills/changelog.yaml` with version 1.0.0 (initial human-authored prompts)
- [ ] Configure evolution trigger in `.harness/config.yaml` (default: every 5 sprints)
- [ ] Configure failure spike threshold (default: 3 consecutive failures → immediate evolution)
- [ ] Set prevalence thresholds for patch filtering (high ≥30%, medium 10-30%, low <10%)
- [ ] Establish pre/post evolution pass rate tracking in `ops/metrics.md`

### Claude Code Harness Adaptation

- [ ] Create `.claude/rules/` directory with path-scoped rules (`openspec.yaml`, `src.yaml`, `tests.yaml`, `harness.yaml`)
- [ ] Create `.claude/settings.json` with hooks for phase gate enforcement (`PreToolUse` on Write/Edit) and task completion validation (`PostToolUse` on Bash)
- [ ] Write `scripts/validate-phase-gate.sh` — blocks code writes when no spec exists for the capability
- [ ] Trim `CLAUDE.md` to <300 lines — factual context only (tech stack, build commands, directory layout)
- [ ] Move all methodology instructions out of `CLAUDE.md` into hooks, `.claude/rules/`, and referenced docs
- [ ] Configure `--append-system-prompt` launch script or alias with the 3-5 most critical workflow constraints
- [ ] Add `think harder` / `ultrathink` invocations to `.harness/prompts/generator.md` and `.harness/prompts/evaluator.md`
- [ ] Frame all agent role prompts as task scoping ("Your task is architecture review") not persona adoption ("You are Winston the Architect")
- [ ] Document context rotation boundaries in `ops/status.md` — each BMAD phase should be a separate session

### Operational Tracking

- [ ] Create `ops/status.md`, `ops/changelog.md`, `ops/known-issues.md`
- [ ] Create `ops/metrics.md` with turn log table structure
- [ ] Create `ops/e2e-test-plan.md` and `ops/test-results.md`
- [ ] Set up `scripts/session-metrics.py` for token cost extraction

---

## Phase 10: Continuous Harness Evolution

### 10.1 Stress-Test Assumptions

Every harness component encodes an assumption about what the model cannot do alone. Periodically test each of the 7 BMAD roles:

| Role | Stress Test | Remove If... |
|---|---|---|
| Discovery (Mary) | Can the planner handle ambiguous requests without prior research? | Planner produces equally good specs without a product brief |
| Planner (John) | Can the generator plan for itself? | Generator produces the same feature coverage without separate planning |
| Architect (Winston) | Can the generator make sound design decisions inline? | No regressions in design quality or ADR-worthy decisions going undocumented |
| Design (Sally) | Can the generator produce good UX without a separate design phase? | Evaluator stops flagging UX coherence issues |
| Generator (Amelia) | N/A — always needed | Never remove; this is the core implementation agent |
| Evaluator (Quinn) | Does end-only evaluation catch the same issues as per-sprint? | Per-sprint and end-only produce equivalent bug catch rates |
| Orchestrator (Bob) | Can Agent Teams replace a custom script? | Use Agent Teams when interactive steering and parallelism matter; use script when deterministic repeatability matters (CI/CD) |

Also test structural assumptions:
- Remove sprint decomposition: Can the model handle multiple stories in one session?
- Remove per-sprint evaluation: Does end-only evaluation catch the same issues?
- Collapse Discovery + Planner: Can one agent do both research and planning?
- Collapse Architect + Design: Can one agent handle both system and UX design?

If removing a component degrades quality, it is **load-bearing**. If quality is unchanged, remove it.

### 10.2 Load-Bearing Components (Empirically Validated)

These remain load-bearing even with the most capable models:

1. **Planner (John)**: Without it, agents under-scope and miss features. Planning and implementation are better separated.
2. **Independent Evaluator (Quinn)**: Self-evaluation remains unreliable. External evaluation with tuned skepticism catches meaningful gaps.
3. **Context Resets**: Better output with clean resets than accumulated context — even with large context windows.
4. **OpenSpec Anchoring**: Without spec anchoring, handoffs lose precision — agents drift from requirements across resets.
5. **Traceability Matrix**: Without explicit REQ-* tracking, coverage gaps go undetected.
6. **Orchestration layer (Bob)**: Whether as a custom script (deterministic, CI-friendly) or Claude Code Agent Teams (interactive, parallelizable). The key is that orchestration is separate from implementation — the orchestrator should never also be writing code.

Components that are conditionally load-bearing (test per project):

7. **Discovery (Mary)**: Load-bearing for greenfield work or unfamiliar domains. Often removable for brownfield changes in well-documented areas.
8. **Architect (Winston)**: Load-bearing when introducing new capabilities. Often removable for stories within stable existing architecture.
9. **Design (Sally)**: Load-bearing for user-facing work. Always removable for backend-only changes.

### 10.3 Evolution Strategy

1. **Automated skill evolution (Phase 8)**: Run the Trace2Skill loop after every N sprints — parallel analysts extract lessons from traces, hierarchical consolidation merges them into prompt refinements, prevalence filtering separates systematic patterns from edge cases
2. **Manual harness tuning**: When automated evolution plateaus (pass rate stable for 3+ cycles), examine whether the bottleneck is in the harness structure rather than prompt content
3. **Model-triggered re-evaluation**: When a new model releases, re-examine each harness component — a more capable model may render some prompt guidance unnecessary while enabling new patterns
4. **Component stress testing (10.1)**: Periodically test whether each BMAD role is still load-bearing
5. **Skill transfer validation (8.9)**: When switching models, validate evolved skills still transfer
6. Update `_bmad/architecture.md` when the harness architecture changes

---

## Appendix A: Quick-Start (Minimum Viable Harness)

For projects that don't need the full BMAD ceremony:

1. **Create `openspec/capabilities/<cap>/spec.md`** with REQ-* and SCENARIO-* for your core capabilities
2. **Separate generation from evaluation**: Never let the same context produce and judge work
3. **Use context resets with handoff files**: When work exceeds ~30 minutes, break it into segments. Each handoff references the REQ-* it completed and the REQ-* remaining.
4. **Commit between segments**: Each context-reset boundary is a git commit
5. **Write acceptance criteria before implementation**: Even a bullet list referencing REQ-* is better than implicit expectations
6. **Keep a traceability file**: A simple table mapping REQ-* to "done/not done/tested/not tested" catches drift

---

## Appendix B: Anti-Patterns

| Anti-Pattern | Why It Fails | Spec-Anchored Alternative |
|---|---|---|
| Single agent, single context for hours | Context anxiety, coherence loss, self-praise | Multi-agent with context resets, state in handoff artifacts |
| Agent evaluates its own output | Systematic quality overestimation | Independent evaluator agent (BMAD: separate Dev and QA) |
| Compaction instead of reset | Preserves anxiety triggers, doesn't restore coherence | Full reset; spec artifacts on disk provide continuity |
| Vague success criteria | Nothing measurable to grade against | Sprint contracts derived from REQ-* and SCENARIO-* |
| Granular technical specs in plan | Cascade errors from premature detail decisions | High-level stories with REQ-* references; generator makes technical choices guided by design.md |
| Silent scope reduction when blocked | Undetected missing features | Handoff with status "blocked" and renegotiation; traceability matrix catches gaps |
| Specs and code disagree silently | Drift compounds across sessions, handoffs become unreliable | Mandatory reconciliation: update spec to match reality with rationale, or fix code to match spec |
| Tests without REQ-* references | No traceability; can't verify coverage | Every test references the SCENARIO-* it verifies |
| Handoffs without spec references | Next agent can't verify what was actually completed | Handoffs list implemented REQ-* with verification evidence |
| Trusting generator's self-reported test results | Generator may have test pollution, cached results, or order-dependent tests | Evaluator runs full suite independently from clean state |
| Code coverage as the only metric | 100% line coverage with trivial assertions proves nothing | Both spec coverage and code coverage are required at 100%; spec coverage ensures tests are meaningful, code coverage ensures no code is untested |
| Tests that reference REQ-* but don't exercise it | Phantom coverage — traceability looks complete but requirements are unverified | Evaluator audits test bodies against requirement descriptions, flags phantom coverage |
| Tests that cite REQ-* but assert the wrong thing | Assertion-spec mismatch — test passes but doesn't verify what the spec requires (e.g., checking status 200 when spec mandates schema conformance) | Assertion-to-spec traceability audit compares test assertions against actual spec text (Section 5.9) |
| No API contract tests between components | Backend changes response shape, frontend breaks silently; caught late by E2E with poor diagnostics | Contract tests validate shared API schemas from both producer and consumer sides (Section 5.8) |
| Compliance tool with no conformance fixtures | False positives (accepting bad input) and false negatives (rejecting good input) go undetected by standard test pyramid | Golden and poison fixture tests verify assertion correctness, not just code correctness (Section 5.7) |
| Writing tests after implementation | Tests shaped by implementation rather than specification; misses spec intent | TDD: generator writes test stubs from SCENARIO-* before implementation code |
| E2E tests only at the end | Integration bugs compound across sprints; late discovery = expensive rework | E2E per sprint, not just final evaluation |
| No negative tests | Happy path tested, error paths assumed to work | Every SCENARIO-* that specifies error behavior must have a negative test |
| Mocking at integration boundaries | Mocks can diverge from real behavior; prior incidents where mocked tests pass but production fails | Integration tests use real dependencies (DB, services); mocks only in unit tests for external deps |
| Static agent prompts that never improve | Same prompt failures repeat across sprints; evaluator catches the same classes of issues repeatedly | Trace2Skill evolution loop: analyze traces, extract patches, consolidate into evolved prompts (Phase 8) |
| Sequential trace analysis (one at a time) | Overfits to individual trajectory quirks; misses systematic patterns | Parallel independent analysts + prevalence-weighted consolidation (high-prevalence → prompt, low → references/) |
| Evolving prompts without measuring effectiveness | No way to know if evolution helped or hurt | Track pre/post evolution pass rates; revert if pass rate decreases |
| Putting methodology rules in CLAUDE.md | CLAUDE.md is injected as a `<system-reminder>` in the messages array — lower priority than the system prompt, diluted by 14K+ tokens of tool definitions, and explicitly qualified as "may or may not be relevant" | Move behavioral rules to hooks (deterministic), path-scoped `.claude/rules/` (fresh injection), and `--append-system-prompt` (system-level priority). Keep CLAUDE.md lean and factual. |
| Long CLAUDE.md files (>300 lines) | Competes with ~50 existing system prompt instructions for a ~150-200 instruction budget; instruction-following degrades uniformly as count increases | Keep CLAUDE.md under 300 lines. Use Progressive Disclosure: methodology lives in referenced files the agent reads on demand |
| Asking Claude to adopt BMAD personas | Claude's "I am Claude Code" identity is a hard constraint — it refuses external persona assignments | Frame roles as task scoping with expertise emphasis: "Your task is architecture review. Do NOT write implementation code." Scope behavior through tools and instructions, not persona |
| Running BMAD phases in a single marathon session | Context degrades at ~147K-152K tokens; dynamic system reminders injected mid-conversation outcompete methodology instructions from turn 1 | Rotate sessions at phase boundaries. Each BMAD agent gets a fresh context window. Monitor with `/context`. |
| Relying on CLAUDE.md for phase discipline | "Go straight to the point" in the system prompt pushes Claude to skip research/design phases and generate code immediately | Enforce phase gates in hooks (PreToolUse): script blocks Write/Edit if no spec exists for the capability |
| Not using extended thinking keywords | External users receive a conciseness-maximized build that suppresses the multi-phase reasoning BMAD requires | Use `think`, `think harder`, or `ultrathink` in agent prompts and sprint contracts to allocate explicit thinking budgets |

---

## Appendix C: Cost/Quality Tradeoffs

Based on empirical data from Anthropic's harness experiments:

| Approach | Time | Cost | Quality |
|---|---|---|---|
| Single-pass, single agent | ~20 min | ~$9 | Appears functional, hidden defects, untraceable |
| Full harness (BMAD + OpenSpec + context resets) | ~4-6 hrs | ~$125-200 | Spec-verified, traceable, E2E tested, reconciled |
| Optimized harness (after removing non-load-bearing parts) | ~3-4 hrs | ~$75-125 | Same quality, lower overhead |

The full harness costs 15-20x more but every delivered feature is traceable from user request through REQ-* through test through implementation. The evaluator catches critical gaps that single-pass misses — display-only features, broken integrations, spec contradictions.

---

## Appendix D: Handoff File as the Context Reset Bridge

The handoff artifact is the bridge that makes context resets viable. Without it, a fresh agent has no idea what happened. With it, a fresh agent can reconstruct sufficient context by reading:

1. **The handoff file** — what was done, what remains, what decisions were made
2. **The spec files** — the source of truth for requirements (these persist across all resets)
3. **The traceability matrix** — the global view of what's implemented vs. specified
4. **The sprint contract** — what this specific unit of work must deliver
5. **The codebase itself** — git log + current files

This is strictly more information than a compacted context provides, because the spec artifacts are complete (not summarized) and the handoff is structured (not narrative). The fresh agent reads exactly what it needs rather than wading through a compressed history of irrelevant turns.

---

## Appendix E: Agent SDK Performance Optimizations

When implementing the orchestrator with the Claude Agent SDK, apply these optimizations to reduce latency and cost. Most agent performance issues are structural, not model-limited.

### E.1 Parallel Tool Execution via Read-Only Annotations

When Claude requests multiple tool calls in a single turn, the SDK can run them concurrently — but only if tools are annotated as read-only. Without `readOnlyHint: true`, custom tools default to sequential execution even when parallelism is safe.

This single change can **reduce end-to-end latency by ~2x** for agents that do heavy read operations (Discovery, Planner, Evaluator).

```python
from claude_agent_sdk import tool, ToolAnnotations

@tool(
    "read_spec",
    "Read an OpenSpec capability specification",
    {"capability": str},
    annotations=ToolAnnotations(readOnlyHint=True),  # enables parallel execution
)
async def read_spec(args):
    # ... read and return spec content
```

Parallelism must be both **enabled** (annotation) and **encouraged** (system prompt). Include in each agent's prompt:

> "When you need to read multiple files, call all read tools in a single turn so they execute in parallel."

**Tool annotation fields and how they map to our agents:**

| Field | Default | Use In Harness |
|---|---|---|
| `readOnlyHint` | `false` | `true` for Read, Grep, Glob, WebSearch, WebFetch. `false` for Edit, Write, Bash. |
| `destructiveHint` | `true` | `false` for Write (creates files). `true` for Bash (may delete). |
| `idempotentHint` | `false` | `true` for Read, Grep, Glob. `false` for Write, Edit. |
| `openWorldHint` | `true` | `true` for Bash, WebSearch, WebFetch, Playwright. `false` for Read, Grep, Glob. |

### E.2 Model Routing Per Agent Role

Each BMAD role has different reasoning requirements. Use the cheapest model that produces acceptable quality.

**Mode:** All agents use `mode: "auto"`. Never use `mode: "plan"` — Claude plans better when not constrained to plan mode.

| Agent | Model | Effort | Rationale |
|---|---|---|---|
| Discovery (Mary) | Sonnet | medium | Research is well-scoped; doesn't need deep reasoning |
| Planner (John) | Sonnet | medium | Planning from existing specs; structured output |
| Architect (Winston) | Opus | high | Design decisions require deep analysis of tradeoffs |
| Design (Sally) | Sonnet | medium | UX patterns are well-established; structured output |
| Generator (Amelia) | Opus | high | Implementation needs sustained coherence and debugging |
| Evaluator (Quinn) | Opus | high | Bug detection requires thorough analysis |

**Effort levels** trade latency/cost for reasoning depth:

| Level | Latency | Cost | Use For |
|---|---|---|---|
| `low` | Fastest | Lowest | File lookups, listing, simple classification |
| `medium` | Moderate | Moderate | Planning, research, structured output generation |
| `high` | Slower | Higher | Implementation, debugging, code review, QA |
| `max` | Slowest | Highest | Multi-step problems requiring deep chain-of-thought |

### E.3 Tool Scoping — Only Expose What's Needed

Over-exposing tools wastes context and slows tool selection. Each tool definition consumes context space, and more tools means Claude takes longer deciding which to use.

**Scoping rules for our agents:**

| Agent | Needs | Does NOT Need |
|---|---|---|
| Discovery (Mary) | Read, Grep, Glob, WebSearch, WebFetch | Edit, Write, Bash, Playwright |
| Planner (John) | Read, Grep, Glob, Write | Edit, Bash, Playwright, WebSearch |
| Architect (Winston) | Read, Grep, Glob, Write | Edit, Bash, Playwright |
| Design (Sally) | Read, Grep, Glob, Write | Edit, Bash, Playwright |
| Generator (Amelia) | Read, Edit, Write, Grep, Glob, Bash | Playwright, WebSearch |
| Evaluator (Quinn) | Read, Grep, Glob, Bash, Playwright | Edit, Write, WebSearch |

**Important**: Omit tools from the `tools` array entirely rather than using `disallowedTools`. Omission removes tools from context; disallowing still sends the schema (wasting tokens) and burns a turn if Claude attempts to use it.

### E.4 Budget and Turn Limits

Prevent runaway sessions with per-agent limits. These are safety rails, not optimization targets — set them above expected usage but below catastrophic waste:

```python
ClaudeAgentOptions(
    max_turns=100,        # generator may need many turns for a complex story
    max_budget_usd=50.0,  # hard cost cap per generator session
    effort="high",
)
```

When an agent hits its turn or budget limit, it should write a handoff with status `context_limit` and the remaining work — the orchestrator then launches a fresh agent to continue.

**Task cleanup before agent shutdown**: The orchestrator must mark all of an agent's tasks as complete (or failed) before terminating it. This applies to all shutdown paths: normal completion, budget/turn limits, retries, and escalations. Stale in-progress tasks corrupt state for crash recovery and confuse subsequent agents.

### E.5 Reducing Tool Failures and Retries

Tool failures are a hidden latency tax. Each failed tool call wastes a model turn (reasoning + failed call + error processing + retry reasoning + retry call). Reduce this by:

1. **Return `is_error: true` instead of throwing** — lets Claude react to the error intelligently rather than crashing the agent loop
2. **Validate inputs in tool handlers** — catch malformed arguments before making expensive operations
3. **Include actionable error messages** — "File not found: auth/login.py. Did you mean src/auth/login.py?" is faster than "Error: ENOENT" followed by Claude guessing
4. **Pre-validate file paths** — for tools that take file paths, check existence before attempting operations

```python
@tool("read_spec", "Read an OpenSpec capability spec", {"capability": str},
      annotations=ToolAnnotations(readOnlyHint=True))
async def read_spec(args):
    spec_path = f"openspec/capabilities/{args['capability']}/spec.md"
    if not os.path.exists(spec_path):
        available = os.listdir("openspec/capabilities/")
        return {
            "content": [{"type": "text", "text":
                f"Capability '{args['capability']}' not found. "
                f"Available: {', '.join(available)}"}],
            "is_error": True,
        }
    # ... read and return
```

### E.6 Context Efficiency Through Subagent Architecture

Our BMAD agent decomposition already provides the key context efficiency benefit: each agent starts fresh, so intermediate tool calls and results from prior agents don't accumulate in downstream contexts.

Additional optimizations:

- **Subagents within agents**: The generator can spawn read-only subagents for research tasks (e.g., "find all usages of this function") that return only a summary, keeping the generator's context lean
- **Prompt caching**: Content that stays the same across turns (system prompt, tool definitions, CLAUDE.md) is automatically prompt-cached by the API, reducing cost by up to 80% on repeated prefixes. Structure agent prompts with stable content at the top.
- **Handoff artifacts are context-efficient by design**: A structured YAML handoff is far smaller than the full conversation history it replaces. This is another reason context resets outperform compaction — the reset + handoff approach uses fewer tokens than compacting a long conversation.

---

## Usage

1. Provide this document as the system prompt or instruction file for the refactoring agent
2. Point the agent at the target project root
3. Phase 1 produces the BMAD + OpenSpec bootstrap — review before proceeding
4. The architect agent validates readiness before implementation begins (BMAD quality gate)
5. Phase 5 (testing architecture) must be established before the first generator sprint
6. Phases 2-7 execute iteratively through the orchestration harness
7. Phase 8 is the verification checklist — all items should be checked before declaring the refactor complete
8. Phase 9 is ongoing — revisit when models change or when harness components feel like overhead
