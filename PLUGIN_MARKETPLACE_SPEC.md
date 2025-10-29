# Plugin Marketplace for Claude Code - Technical Specification v1.0

**Project**: Codeframe-Inspired Plugin Marketplace System
**Built As**: Claude Code Plugin within Marketplace
**License**: MIT Open Source
**Last Updated**: 2025-10-29

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Claude Code Native Architecture](#claude-code-native-architecture)
4. [Core Components](#core-components)
5. [Agent Supervision Model](#agent-supervision-model)
6. [Workstream Isolation Model](#workstream-isolation-model)
7. [Plugin System](#plugin-system)
8. [QC and Vetting System](#qc-and-vetting-system)
9. [User Interaction Model](#user-interaction-model)
10. [Persistence and Recovery](#persistence-and-recovery)
11. [Database Schemas](#database-schemas)
12. [API Specifications](#api-specifications)
13. [Phase I MVP Scope](#phase-i-mvp-scope)
14. [Implementation Roadmap](#implementation-roadmap)

---

## Executive Summary

### Vision

Build a **plugin marketplace system** as a Claude Code plugin that enables autonomous software development through coordinated agent swarms, inspired by the Codeframe specification. The system operates through independent concurrent loops with sophisticated QC oversight, workstream isolation, and intelligent user interruption patterns.

### Key Innovations

1. **Three-Tier Supervision**: Worker → QC Expert → Coordinator pattern ensures consistent quality
2. **Workstream Isolation**: Git branch-based workspaces prevent resource conflicts between agents
3. **Claude Code Native Architecture**: Hooks + MCP server replace custom Python loops
4. **Unified Question Queue**: Native AskUserQuestion tool with MCP resource management
5. **Continuous Context Pruning**: before_compact hook enables proactive memory management
6. **Smart Retry-Debug**: Automated retry → simplify → debug → escalate pattern

### Three-Phase Roadmap

- **Phase I** (MVP): Standalone marketplace plugin with manual vetting
- **Phase II**: Federated marketplace model (users add their own marketplaces/plugins)
- **Phase III**: Automated vetting pipeline with specialized QC agents

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    USER INTERFACE                                │
│  Slash Commands: /qc-review, /chat, /plugin-install            │
└────────────────────────────┬────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │ Claude Code Native Tools     │
              │  • AskUserQuestion          │
              │  • Bash + Git               │
              │  • Skills Framework         │
              └──────────────┬──────────────┘
                             │
┌────────────────────────────┴──────────────────────────────────┐
│                   HOOKS SYSTEM                                 │
│         (Interception points in Claude Code)                   │
├────────────┬────────────┬────────────┬───────────────────────┤
│before_     │after_      │user_prompt_│after_tool_use         │
│compact     │compact     │submit      │                       │
└─────┬──────┴──────┬─────┴──────┬─────┴───────────┬──────────┘
      │             │            │                 │
      ▼             ▼            ▼                 ▼
┌───────────────────────────────────────────────────────────────┐
│         CODEFRAME-COORDINATOR MCP SERVER                       │
│  • continuous_prune (context management)                       │
│  • get_next_task (work execution)                             │
│  • submit_for_qc (QC workflow)                                │
│  • score_priority (prioritization)                            │
│  • plugin_discover (plugin recommendations)                   │
│                                                               │
│  Resources:                                                   │
│  • project_state                                              │
│  • user_input_queue                                           │
│  • plugin_catalog                                             │
│  • task_queue                                                 │
└───────────────────────┬───────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                SQLite Database (MCP-Managed)                     │
│  • workstreams (agent isolation)                                │
│  • internal_prs (QC workflow)                                   │
│  • plugin_catalog (marketplace)                                 │
│  • tasks (work queue)                                           │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    WORKSTREAM ISOLATION                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Worker 1     │  │ Worker 2     │  │ Worker 3     │         │
│  │ Branch/Dir   │  │ Branch/Dir   │  │ Branch/Dir   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Design Principles

1. **Claude Code Native**: Build on top of existing infrastructure, not around it
2. **Hooks Over Loops**: Use interception points instead of concurrent Python processes
3. **MCP for State**: Single MCP server manages all state and coordination
4. **Skills for Behavior**: Reusable behavioral patterns instead of hardcoded logic
5. **User Control**: Configurable autonomy levels (bypass/async/sync interruption)
6. **Plugin Discovery**: Context-aware plugin recommendations based on workflow phase

---

## Claude Code Native Architecture

### Philosophy: Build On, Not Around

Instead of creating 8 concurrent Python loops that operate alongside Claude Code, we **extend Claude Code from within** using its native extension mechanisms:

- **Hooks**: Interception points in Claude Code's execution flow
- **MCP Server**: Single server managing state, not separate Python processes
- **Skills**: Reusable behavioral patterns triggered by hooks or commands
- **Slash Commands**: User-facing interface for marketplace operations

This approach:
- Eliminates need for complex inter-process communication
- Leverages Claude Code's existing context management
- Reduces maintenance burden (fewer moving parts)
- Makes system more reliable (single process)

### Phase I: Single Instance Architecture

In Phase I (MVP), we operate with a **single Claude Code instance** enhanced with:

1. **Hooks** that intercept key execution points
2. **MCP server** that manages persistent state
3. **Skills** that define behavioral patterns
4. **Slash commands** for user interaction

**Phase II** will explore multi-instance coordination, but Phase I proves the concept with simpler architecture.

### Component Overview

| Component | Type | Trigger | Purpose | Implementation |
|-----------|------|---------|---------|----------------|
| Context Pruning | Hook + MCP | before_compact | Continuous memory mgmt | `before_compact.js` → `continuous_prune` |
| Work Execution | Skill + Hook | after_tool_use | Main dev workflow | `/qc-review` skill + hook |
| QC Review | Skill | Manual/automatic | Code quality gate | `/qc-review` skill |
| Prioritization | MCP Tool | User prompt | Decision scoring | `score_priority()` |
| Plugin Discovery | Skill | Phase change | Context-aware plugins | `/plugin-discover` skill |
| User Questions | Native Tool | Decision needed | User input | `AskUserQuestion` |
| User Chat | Slash Command | User initiates | "Peek in" interface | `/chat` command |
| Plugin Vetting | Skill | Plugin submit | Approval workflow | `/plugin-vet` skill |

---

## Core Components

### 1. Work Execution (Skill + Hook)

**Implementation**: `/qc-review` skill triggered by `after_tool_use` hook

**Purpose**: Main development workflow with automatic QC checkpoints

```
┌─────────────────────────────────────────┐
│  USER/SYSTEM: Requests work             │
│  • Manual task assignment               │
│  • Or: MCP tool get_next_task()        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  CREATE WORKSTREAM (via Bash + Git):    │
│  • Git branch: worker-N/task-X.Y.Z     │
│  • Isolated workspace directory         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  CLAUDE CODE: Execute Task              │
│  • Work in isolated workspace           │
│  • Use native tools (Edit, Write, etc) │
│  • Commit changes via Bash tool         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  HOOK: after_tool_use triggers          │
│  • Detect: git commit completed         │
│  • Auto-invoke: /qc-review skill       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  SKILL: /qc-review                      │
│  • Apply quantitative checks            │
│  • Apply qualitative scoring            │
│  • Verdict: Pass / Conditional / Fail   │
│  • Call MCP: submit_for_qc()           │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┴─────────┐
       │ PASS              │ FAIL
       ▼                   ▼
┌─────────────┐      ┌──────────────┐
│Merge via    │      │Request changes│
│Bash tool    │      │Edit and retry │
└──────┬──────┘      └──────┬───────┘
       │                    │
       ▼                    │
┌─────────────────────────────┐
│  Update MCP State:           │
│  • Mark task complete        │
│  • Update workstream status  │
│  • Cleanup branch (optional) │
└─────────────────────────────┘
```

**Key Features**:
- No separate Python loop process
- Uses native Claude Code Bash tool for git operations
- after_tool_use hook automatically triggers QC when appropriate
- MCP tools manage state transitions

### 2. Context Management (Hook + MCP)

**Implementation**: `before_compact.js` hook → `continuous_prune` MCP tool

**Purpose**: Continuous importance-based context pruning

**Key Difference from Traditional Compaction**:
- **Traditional**: Wait until 80% full, then batch compact
- **Continuous Pruning**: Prune incrementally on every message cycle
- Prevents context from ever reaching critical levels
- More efficient: small pruning operations vs. large batch operations

**Algorithm Testing**: The continuous pruning algorithm is being developed and validated in a standalone repository: **context-pruning-lab**. See that project for algorithm experiments, benchmarks, and validation tests.

```
┌─────────────────────────────────────────┐
│  HOOK: before_compact (every cycle)     │
│  • Triggered BEFORE Claude compacts    │
│  • Allows proactive intervention        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  MCP TOOL: continuous_prune()           │
│  • Calculate importance scores          │
│  • Identify lowest-value items          │
│  • Archive to MCP resource              │
│  • Return pruned context                │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  IMPORTANCE SCORING:                    │
│  • Recency: More recent = higher       │
│  • Frequency: More accessed = higher   │
│  • Type weights: Task > File > Test    │
│  • Final score: 0.0-1.0                │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  ARCHIVE LOW-VALUE ITEMS:               │
│  • Store in MCP resource (queryable)   │
│  • Remove from active context           │
│  • Update access metadata               │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  CLAUDE CODE: Continue with pruned      │
│  context, never hitting critical levels │
└─────────────────────────────────────────┘
```

**Benefits**:
- No emergency flash saves needed
- Context stays lean continuously
- MCP resources provide queryable archive
- Algorithm development separate from integration (context-pruning-lab repo)

### 3. Prioritization (MCP Tool)

**Implementation**: `score_priority()` MCP tool called by `user_prompt_submit.js` hook

**Purpose**: Automatic priority scoring for decisions

**Flow**:

```
┌─────────────────────────────────────────┐
│  HOOK: user_prompt_submit               │
│  • Triggered when decision needed       │
│  • Passes decision context to MCP       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  MCP TOOL: score_priority(decision)     │
│  • Check P0 pattern database            │
│  • Analyze decision reversibility       │
│  • Consider user trust level            │
│  • Return: P0/P1/P2/P3                 │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┼─────────┬─────────┐
       │ P0      │ P1      │ P2/P3   │
       ▼         ▼         ▼
    ┌──────┐ ┌──────┐ ┌──────┐
    │SYNC  │ │ASYNC │ │BYPASS│
    │Ask   │ │Notify│ │Auto  │
    │User  │ │user  │ │decide│
    └──────┘ └──────┘ └──────┘
```

**Benefits**:
- No separate scheduling loop
- Priority scoring happens inline with decision flow
- MCP maintains P0 pattern database for learning
- Native `AskUserQuestion` tool handles UI

### 4. User Interruption (Native Tool + MCP)

**Implementation**: Native `AskUserQuestion` tool + MCP resource `user_input_queue`

**Purpose**: Collect user input when needed, with priority-based routing

**Flow**:

```
┌─────────────────────────────────────────┐
│  CLAUDE CODE: Decision Required         │
│  • Determined by score_priority()       │
│  • P0/P1: Must ask user                 │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  NATIVE TOOL: AskUserQuestion           │
│  • Claude Code's built-in UI            │
│  • Presents options to user             │
│  • Blocks (P0) or continues (P1)       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  MCP RESOURCE: user_input_queue         │
│  • Stores pending questions             │
│  • Tracks responses                     │
│  • Updates P0 patterns (learning)       │
└─────────────────────────────────────────┘
```

**Benefits**:
- Uses Claude Code's native question UI (no custom UI needed)
- MCP resource provides query interface for pending questions
- No separate interruption loop process
- Notification channels integrated via MCP (Phase II)

### 5. User Chat Interface (Slash Command)

**Implementation**: `/chat` slash command

**Purpose**: "Peek in" on development progress

**Flow**:

```
┌─────────────────────────────────────────┐
│  USER: Invokes /chat                    │
│  • Triggers chat skill                  │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  SKILL: Query MCP Resources             │
│  • Read: project_state resource         │
│  • Read: task_queue resource            │
│  • Read: user_input_queue resource      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  CLAUDE CODE: Generate Summary          │
│  • Natural language status              │
│  • Active tasks and progress            │
│  • Pending questions (if any)           │
│  • Recent changes                       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OPPORTUNISTIC Q&A:                     │
│  "While you're here, I have 2 questions"│
│  • Show pending questions from queue    │
│  • User can answer or defer             │
└─────────────────────────────────────────┘
```

**Benefits**:
- Simple slash command interface (no separate chat loop)
- MCP resources provide all necessary state
- Native Claude Code conversation for Q&A
- No additional infrastructure needed

### Implementation Components Table

This table shows which components are Claude Code native vs. implemented via MCP server:

| Component | Implementation | Claude Code Native? | MCP Involvement |
|-----------|----------------|---------------------|-----------------|
| Context Pruning | `before_compact` hook + MCP tool | Hook: Yes, Logic: No | `continuous_prune()` tool |
| QC Review | Skill + hook | Skill framework: Yes | `submit_for_qc()` tool |
| Priority Scoring | MCP tool | No | `score_priority()` tool |
| Task Queue | MCP resource | No | `task_queue` resource |
| User Questions | Native tool | Yes | MCP stores history only |
| Git Operations | Bash tool + hooks | Bash: Yes, Hooks: Yes | State tracking only |
| Plugin Catalog | MCP resource | No | `plugin_catalog` resource |
| Plugin Discovery | Skill | Skill framework: Yes | MCP provides data |
| User Chat | Slash command | Yes | MCP provides state data |
| Workstream Mgmt | Bash + MCP | Bash: Yes | State persistence |

**Key Principle**: Maximize use of Claude Code native features, minimize custom infrastructure.

---

## Agent Supervision Model

### Three-Tier Supervision Architecture

```
          User
           │
    ┌──────┴──────┐
    │ COORDINATOR │ (prioritizes, closes, veto power)
    └──────┬──────┘
           │
    ┌──────┴──────┬──────────┐
    │   Worker 1  │  Worker 2│
    └──────┬──────┴────┬─────┘
           │           │
    ┌──────┴──────┐   └─────┐
    │ QC Expert 1 │  QC Exp 2│
    └─────────────┴──────────┘
```

**Flow Pattern**: Worker → QC Expert → Coordinator → User (if needed)

### Worker Agent Responsibilities

- Execute assigned tasks in isolated workstreams
- Perform basic self-checks
- Commit work to git branch
- "Turn in" completed work to assigned QC expert
- Respond to QC feedback with revisions

### QC Expert Responsibilities

- Develop own quantitative/qualitative rubrics (proposed to Coordinator)
- Apply rubrics **consistently** across all reviews
- Review worker submissions
- Provide structured feedback
- Submit verdict to Coordinator: Pass / Conditional Pass / Fail

### Coordinator Responsibilities

- Assign tasks to workers
- Approve QC rubrics
- Review QC verdicts
- **Veto power**: Can call "stop" at any time
- "Close out" QC thresholds
- Merge approved work to main branch
- Escalate to user when needed

### Timeout Handling

| Agent Type | Timeout | Action |
|------------|---------|--------|
| Worker | >5 minutes inactive | QC Expert notified |
| QC Expert | >3 minutes inactive | Coordinator notified |
| Coordinator | >2 minutes inactive | User notified (P0) |

---

## Workstream Isolation Model

### Concept

Each agent works in an **isolated git branch + directory workspace** to prevent resource conflicts. Think of it as "everyone works on their own copy, merge via PRs."

### Directory Structure

```
project-root/
├── .codeframe/
│   ├── config.json
│   ├── state.db
│   └── workstreams/
│       ├── worker-1-backend/     ← Agent 1's isolated workspace
│       │   └── [checked out branch: worker-1/task-1.5.3]
│       ├── worker-2-frontend/    ← Agent 2's isolated workspace
│       │   └── [checked out branch: worker-2/task-2.1.4]
│       └── qc-1-security/        ← QC Agent's review workspace
│           └── [checked out branch: worker-1/task-1.5.3 for review]
│
└── src/  ← Main codebase (protected, merged PRs only)
```

### Workstream Workflow

```
1. COORDINATOR assigns Task 1.5.3 to Worker 1
   ↓
2. CREATE workstream:
   - Create git branch: worker-1/task-1.5.3
   - Create directory: .codeframe/workstreams/worker-1-backend/
   - Checkout branch into that directory
   ↓
3. WORKER 1 executes in isolation:
   - All file operations in workstream directory
   - No conflicts with other agents
   - Tests run in isolation
   ↓
4. WORKER 1 "turns in" work:
   - Commit changes to worker-1/task-1.5.3 branch
   - Create internal PR (tracked in internal_prs table)
   - Notify QC Expert
   ↓
5. QC EXPERT reviews in own workstream:
   - Checkout worker-1/task-1.5.3 into qc workspace
   - Review code, run checks, apply rubric
   - Submit verdict to Coordinator
   ↓
6. COORDINATOR approves merge:
   - Review QC verdict
   - Approve or request changes
   - Merge worker-1/task-1.5.3 → main
   - Delete workstream
   - Update task status: completed
```

### Benefits

- **No Resource Conflicts**: Each agent has exclusive access to their workspace
- **Parallel Execution**: Multiple agents work simultaneously without blocking
- **Easy Rollback**: Failed work isolated to branch, doesn't pollute main
- **Clear Audit Trail**: Git history shows exactly what each agent did

---

## Plugin System

### Plugin Manifest Format

Every plugin includes a `plugin.json` file:

```json
{
  "name": "security-scanner",
  "version": "1.0.0",
  "description": "Automated security vulnerability detection",
  "author": "Codeframe Team",
  "license": "MIT",

  "repository": {
    "type": "git",
    "url": "https://github.com/codeframe/security-scanner"
  },

  "agent_roles": [
    "qc_expert",
    "worker"
  ],

  "specializations": [
    "security",
    "vulnerability_detection",
    "static_analysis"
  ],

  "workflow_phases": [6, 7, 11, 15],

  "context_requirements": {
    "hot_tier_tokens": 5000,
    "requires_mcp": true,
    "mcp_servers": ["serena", "playwright"]
  },

  "themes": ["security", "quality_assurance"],

  "capabilities": {
    "can_review_code": true,
    "can_execute_tests": false,
    "can_modify_code": false
  },

  "dependencies": [],

  "installation": {
    "scope": ["global", "project"],
    "requires_user_config": true,
    "config_schema": {
      "api_key": "string",
      "severity_threshold": "enum[low,medium,high]"
    }
  }
}
```

### Plugin Discovery Logic

Plugins are discovered and recommended based on:

1. **Current workflow phase** (e.g., Phase 11 = Code Review)
2. **Detected gaps** (e.g., no security scanner installed)
3. **NOT by language/framework** (context is king)

**Example**:
- System enters Phase 6 (Test Development)
- Loop 5 (Plugin Discovery) queries installed plugins
- Finds no test automation plugin installed
- Recommends: `test-automation`, `pytest-plugin`, `jest-plugin`
- User notified via async notification

### Plugin Installation

**CLI Commands**:

```bash
# Search marketplace
codeframe-marketplace search <query>

# Install plugin (global or project scope)
codeframe-marketplace install <plugin-name> [--scope global|project]

# Uninstall plugin
codeframe-marketplace uninstall <plugin-name>

# List installed plugins
codeframe-marketplace list [--scope global|project]

# Get plugin info
codeframe-marketplace info <plugin-name>
```

### Marketplace Sources

Phase I: Single official source (curated by you)
Phase II: Federated model (users add marketplace sources)

```sql
CREATE TABLE marketplace_sources (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    trust_level TEXT CHECK(trust_level IN ('official', 'verified', 'community')),
    last_sync TIMESTAMP,
    enabled BOOLEAN DEFAULT TRUE
);
```

**Trust Levels**:
- **Official**: Created/vetted by core team
- **Verified**: Passed automated vetting + human review
- **Community**: User-contributed, use at own risk

---

## QC and Vetting System

### QC Rubric Development

Each QC Expert proposes their own rubric at project start:

**Example Rubric (Security QC Expert)**:

```markdown
## Security QC Rubric v1.0

### Quantitative Checks (Pass/Fail):
- [ ] No hardcoded credentials
- [ ] Dependencies have no CVEs (CVSS >= 7.0)
- [ ] Input validation on all external data
- [ ] Authentication on sensitive endpoints
- [ ] HTTPS enforcement

### Qualitative Assessment (0-10 scale):
- Error handling quality: __/10
- Principle of least privilege: __/10
- Defense in depth: __/10

### Verdict Criteria:
- **Pass**: All quant checks + avg qual >= 7/10
- **Conditional**: Minor issues, documented
- **Fail**: Any quant failure OR avg qual < 5/10
```

**Approval Process**:
1. QC Expert proposes rubric
2. Coordinator reviews for reasonableness
3. Once approved, QC applies consistently
4. Rubric can be updated mid-project with Coordinator approval

### Plugin Vetting (Phase III - Automated)

**Vetting Pipeline**:

```
Plugin Submission
    ↓
Initial Validation (manifest format)
    ↓
Automated Checks
    ├─ Security scan (static analysis)
    ├─ Performance test (resource usage)
    ├─ Compatibility test (Claude Code API)
    └─ Documentation check (README, CLAUDE.md)
    ↓
QC Agent Review (if automated checks pass)
    ├─ Code quality
    ├─ Architecture
    └─ Apply rubric
    ↓
Human Review (Phase I: manual)
    ├─ Sandbox test
    ├─ Review QC report
    └─ Decision: Approve / Reject / Request Changes
    ↓
Approved → Add to marketplace catalog
Rejected → Notify submitter with feedback
```

**Phase I (MVP)**: Manual human review (you) for initial 5 plugins
**Phase III**: Fully automated with human review only for edge cases

---

## User Interaction Model

### Unified Question Queue

All loops needing user input feed into a centralized queue:

```
┌─────────────────────────────────────────┐
│      UNIFIED QUESTION QUEUE             │
│  (All loops → single interface)         │
└────────────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  AGGREGATED QUESTIONS:                  │
│                                         │
│  Q1 [P0] [Loop 6]: Security decision   │
│     "API auth: OAuth2 or JWT?"         │
│     Source: Worker 2, Task 1.5.3       │
│                                         │
│  Q2 [P1] [Loop 4]: Bottleneck detected │
│     "Assign more workers to Task 2.3?" │
│     Source: Coordinator                 │
│                                         │
│  Q3 [P1] [Loop 7]: Plugin approval     │
│     "Approve 'security-scanner' plugin?"│
│     Source: Vetting QC Agent            │
└────────────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  RENDER IN UI:                          │
│  (Uses Claude Code's AskUserQuestion)   │
│  ┌──────────────────────────────────┐  │
│  │ 🔴 P0: Security (Task 1.5.3)     │  │
│  │ [ ] OAuth2  [ ] JWT  [ ] Other   │  │
│  │                                  │  │
│  │ 🟡 P1: Bottleneck (Task 2.3)    │  │
│  │ [ ] Assign worker [ ] Continue   │  │
│  │                                  │  │
│  │ 🟡 P1: Plugin Approval           │  │
│  │ [ ] Approve [ ] Reject           │  │
│  └──────────────────────────────────┘  │
└────────────────┬────────────────────────┘
         │
         ▼
USER RESPONDS (batch or individually)
         │
         ▼
ROUTE RESPONSES BACK TO LOOPS
```

### Autonomy Preferences

Users configure global settings with project-level overrides:

**Global Settings** (`~/.codeframe/global_config.json`):
```json
{
  "autonomy_preferences": {
    "default_trust_level": 0.5,
    "default_interruption_mode": "async",
    "override_rules": {
      "security_decisions": "sync",
      "test_failures": "async",
      "architecture_changes": "sync"
    }
  },
  "notification_channels": {
    "P0_channels": ["twilio_sms", "email"],
    "P1_channels": ["email", "n8n_webhook"]
  }
}
```

**Project Overrides** (`.codeframe/project_config.json`):
```json
{
  "autonomy_preferences": {
    "override_trust_level": 0.3,
    "override_interruption_mode": "sync"
  }
}
```

### Notification Channels (MVP)

**Supported Channels**:
- Twilio SMS
- Email (SMTP)
- n8n webhook
- Zapier webhook

**Priority Routing**:
- **P0 (Sync)**: Send to ALL configured channels, block work
- **P1 (Async)**: Send to async channels, continue work
- **P2/P3**: No notifications (bypass or log only)

---

## Persistence and Recovery

### Flash Save System

**Principle**: Every loop can serialize its state to disk at any moment and resume exactly where it left off.

### Serializable State Objects

```python
from dataclasses import dataclass, asdict
import pickle
import json

@dataclass
class LoopState:
    loop_id: str
    iteration: int
    last_checkpoint: datetime
    current_phase: str
    metadata: Dict[str, Any]

    def serialize(self) -> bytes:
        """Serialize to binary (pickle)"""
        return pickle.dumps(asdict(self))

    @classmethod
    def deserialize(cls, data: bytes):
        """Deserialize from binary"""
        state_dict = pickle.loads(data)
        return cls(**state_dict)

    def to_json(self) -> str:
        """Human-readable JSON for debugging"""
        return json.dumps(asdict(self), default=str, indent=2)

@dataclass
class Loop1State(LoopState):
    """Work Execution Loop State"""
    current_task_id: int
    assigned_worker_id: str
    worker_start_time: datetime
    qc_agent_id: str
    qc_feedback: Dict[str, Any]
    coordinator_review_status: str
    retry_count: int
    hot_context_snapshot: List[str]
```

### Persistence Manager

```python
class PersistenceManager:
    def __init__(self, project_id: int):
        self.project_id = project_id
        self.state_dir = f".codeframe/state/project_{project_id}"

    def flash_save(self, loop_state: LoopState) -> None:
        """
        Flash save to disk immediately.
        Fast operation: <50ms
        """
        state_path = f"{self.state_dir}/{loop_state.loop_id}.pkl"
        temp_path = f"{state_path}.tmp"

        # Atomic write (write to temp, rename)
        with open(temp_path, 'wb') as f:
            f.write(loop_state.serialize())
        os.rename(temp_path, state_path)

        # Also save JSON for debugging
        json_path = f"{self.state_dir}/{loop_state.loop_id}.json"
        with open(json_path, 'w') as f:
            f.write(loop_state.to_json())

    def load_state(self, loop_id: str, state_class: type) -> LoopState:
        """Load loop state from disk"""
        state_path = f"{self.state_dir}/{loop_id}.pkl"
        if not os.path.exists(state_path):
            return None

        with open(state_path, 'rb') as f:
            data = f.read()
        return state_class.deserialize(data)
```

### Recovery Pattern

```python
class BaseLoop:
    def start(self):
        # Try to recover saved state
        self.state = self.persistence.load_state(
            self.loop_id, self.get_state_class()
        )

        if self.state:
            print(f"[{self.loop_id}] Recovered from iteration {self.state.iteration}")
            self.resume_from_state()
        else:
            print(f"[{self.loop_id}] Starting fresh")
            self.state = self.initialize_state()

        self.run()

    def checkpoint(self):
        """Flash save current state"""
        self.state.last_checkpoint = datetime.now()
        self.persistence.flash_save(self.state)
```

### Retry → Debug Pattern

**Smart retry with automatic debugging escalation**:

```
Attempt 1: Try original approach
    ↓ (if fails)
Attempt 2: Retry same approach (transient error?)
    ↓ (if fails)
Attempt 3: Retry with simplified context
    ↓ (if fails)
Debug Mode:
    ├─ Capture diagnostics
    ├─ Save diagnostic report
    ├─ Attempt recovery strategies:
    │   ├─ Reset state
    │   ├─ Skip and continue
    │   └─ Fallback mode
    ↓ (if all fail)
Escalate to User (P0):
    - Show diagnostic report
    - Present options:
        • Skip task
        • Retry with custom params
        • Pause loop
        • Restart from checkpoint
        • Manual fix
```

---

## Database Schemas

### Architecture Note

**SQLite database is managed by the MCP server**, not a separate Python process. The codeframe-coordinator MCP server:
- Maintains single SQLite database
- Exposes database state via MCP tools and resources
- Handles all persistence operations
- Provides transactional safety

No separate database process or connection pool needed - MCP server handles all database interactions.

### Core Tables

```sql
-- ===== MARKETPLACE CORE =====

CREATE TABLE installed_plugins (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    scope TEXT CHECK(scope IN ('global', 'project')),
    project_id INTEGER,
    enabled BOOLEAN DEFAULT TRUE,
    config JSON,

    repository_type TEXT CHECK(repository_type IN ('git', 'local')),
    repository_url TEXT,

    installed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_used TIMESTAMP,
    usage_count INTEGER DEFAULT 0,

    FOREIGN KEY(project_id) REFERENCES projects(id)
);

CREATE TABLE marketplace_sources (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    trust_level TEXT CHECK(trust_level IN ('official', 'verified', 'community')),
    last_sync TIMESTAMP,
    enabled BOOLEAN DEFAULT TRUE
);

CREATE TABLE plugin_catalog (
    id TEXT PRIMARY KEY,
    source_id INTEGER,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    manifest JSON NOT NULL,

    trust_level TEXT CHECK(trust_level IN ('official', 'verified', 'community', 'experimental')),

    downloads_count INTEGER DEFAULT 0,
    rating_avg REAL,
    rating_count INTEGER DEFAULT 0,

    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY(source_id) REFERENCES marketplace_sources(id)
);

CREATE TABLE vetting_records (
    id INTEGER PRIMARY KEY,
    plugin_id TEXT,
    version TEXT,
    status TEXT CHECK(status IN ('pending', 'approved', 'rejected', 'flagged')),

    automated_checks JSON,
    qc_agent_reviews JSON,

    human_reviewer TEXT,
    human_review_notes TEXT,
    reviewed_at TIMESTAMP,

    FOREIGN KEY(plugin_id) REFERENCES plugin_catalog(id)
);

-- ===== WORKSTREAM SYSTEM =====

CREATE TABLE workstreams (
    id INTEGER PRIMARY KEY,
    agent_id TEXT NOT NULL,
    task_id INTEGER NOT NULL,

    branch_name TEXT NOT NULL,
    workspace_path TEXT NOT NULL,

    status TEXT CHECK(status IN ('active', 'review', 'approved', 'merged', 'abandoned')),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    merged_at TIMESTAMP,

    FOREIGN KEY(task_id) REFERENCES tasks(id)
);

CREATE TABLE internal_prs (
    id INTEGER PRIMARY KEY,
    workstream_id INTEGER NOT NULL,
    worker_agent_id TEXT NOT NULL,
    qc_agent_id TEXT,

    title TEXT,
    description TEXT,

    status TEXT CHECK(status IN ('pending_qc', 'qc_approved', 'qc_rejected', 'pending_coordinator', 'approved', 'merged', 'closed')),

    qc_review_id INTEGER,
    coordinator_approval BOOLEAN,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    merged_at TIMESTAMP,

    FOREIGN KEY(workstream_id) REFERENCES workstreams(id),
    FOREIGN KEY(qc_review_id) REFERENCES qc_reviews(id)
);

-- ===== QC SYSTEM =====

CREATE TABLE qc_rubrics (
    id INTEGER PRIMARY KEY,
    qc_agent_id TEXT NOT NULL,
    project_id INTEGER,
    rubric_version TEXT,

    quantitative_checks JSON,
    qualitative_criteria JSON,

    approved_by TEXT,
    approved_at TIMESTAMP,
    active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY(project_id) REFERENCES projects(id)
);

CREATE TABLE qc_reviews (
    id INTEGER PRIMARY KEY,
    task_id INTEGER,
    worker_agent_id TEXT,
    qc_agent_id TEXT,
    rubric_id INTEGER,

    quantitative_results JSON,
    qualitative_scores JSON,

    verdict TEXT CHECK(verdict IN ('pass', 'conditional_pass', 'fail')),
    feedback TEXT,

    submitted_to_coordinator_at TIMESTAMP,
    coordinator_decision TEXT CHECK(coordinator_decision IN ('approved', 'rejected', 'revise')),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY(task_id) REFERENCES tasks(id),
    FOREIGN KEY(rubric_id) REFERENCES qc_rubrics(id)
);

-- ===== PRIORITIZATION =====

CREATE TABLE p0_patterns (
    id INTEGER PRIMARY KEY,
    pattern_type TEXT CHECK(pattern_type IN ('regex', 'keyword', 'category', 'ml_model')),
    description TEXT,
    pattern_data JSON,

    triggered_count INTEGER DEFAULT 0,
    false_positive_count INTEGER DEFAULT 0,

    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE interruption_log (
    id INTEGER PRIMARY KEY,
    task_id INTEGER,
    priority TEXT CHECK(priority IN ('P0', 'P1', 'P2', 'P3')),

    issue_description TEXT,
    autonomy_score REAL,

    interruption_type TEXT CHECK(interruption_type IN ('sync', 'async', 'bypass')),
    user_response TEXT,
    response_time_seconds INTEGER,

    created_at TIMESTAMP,

    FOREIGN KEY(task_id) REFERENCES tasks(id)
);

-- ===== USER INPUT QUEUE =====

CREATE TABLE user_input_queue (
    id INTEGER PRIMARY KEY,
    loop_id TEXT,
    priority TEXT CHECK(priority IN ('P0', 'P1', 'P2', 'P3')),

    question_text TEXT NOT NULL,
    question_type TEXT,
    options JSON,

    context JSON,

    status TEXT CHECK(status IN ('pending', 'answered', 'deferred', 'expired')),
    user_response TEXT,
    responded_at TIMESTAMP,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,

    blocking_loop BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_queue_status ON user_input_queue(status, priority);
CREATE INDEX idx_queue_loop ON user_input_queue(loop_id, status);

-- ===== INTER-LOOP MESSAGING =====

CREATE TABLE loop_messages (
    id INTEGER PRIMARY KEY,
    from_loop TEXT NOT NULL,
    to_loop TEXT NOT NULL,
    message_type TEXT NOT NULL,
    payload JSON,

    status TEXT CHECK(status IN ('pending', 'read', 'processed')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    read_at TIMESTAMP
);

CREATE INDEX idx_to_loop_status ON loop_messages(to_loop, status);

-- ===== LOOP METRICS =====

CREATE TABLE loop_metrics (
    loop_id TEXT PRIMARY KEY,
    project_id INTEGER,
    metrics JSON,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY(project_id) REFERENCES projects(id)
);

CREATE TABLE loop_events (
    id INTEGER PRIMARY KEY,
    loop_id TEXT,
    event_type TEXT,
    event_data JSON,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ===== NOTIFICATIONS =====

CREATE TABLE notification_channels (
    id INTEGER PRIMARY KEY,
    user_id TEXT,
    channel_type TEXT CHECK(channel_type IN ('twilio_sms', 'email', 'n8n_webhook', 'zapier_webhook')),

    config JSON,
    enabled BOOLEAN DEFAULT TRUE,

    last_used TIMESTAMP,
    failure_count INTEGER DEFAULT 0
);

CREATE TABLE notification_queue (
    id INTEGER PRIMARY KEY,
    priority TEXT CHECK(priority IN ('P0', 'P1', 'P2', 'P3')),

    title TEXT,
    message TEXT,
    action_url TEXT,

    channels JSON,

    status TEXT CHECK(status IN ('pending', 'sent', 'failed', 'cancelled')),
    sent_at TIMESTAMP,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## API Specifications

### CLI Commands (Phase I)

```bash
# ===== PLUGIN MANAGEMENT =====

# Search marketplace
codeframe-marketplace search <query>
  Options:
    --theme <theme>        Filter by theme (security, testing, etc.)
    --phase <number>       Filter by workflow phase
    --source <source>      Filter by marketplace source

# Install plugin
codeframe-marketplace install <plugin-name>
  Options:
    --scope global|project (default: project)
    --version <version>    (default: latest)
    --force                (overwrite existing)

# Uninstall plugin
codeframe-marketplace uninstall <plugin-name>
  Options:
    --scope global|project

# List installed plugins
codeframe-marketplace list
  Options:
    --scope global|project|all (default: all)
    --enabled-only

# Get plugin details
codeframe-marketplace info <plugin-name>

# ===== PROJECT MANAGEMENT =====

# Initialize project
codeframe init <project-name>
  Options:
    --template <template>

# Start/resume project
codeframe start [project-name]
codeframe resume [project-name]

# Pause project
codeframe pause [project-name]

# Check status
codeframe status [project-name]

# Chat with development team (Loop 8)
codeframe chat

# ===== CONFIGURATION =====

# Set configuration
codeframe config set <key> <value>
  Examples:
    codeframe config set autonomy_preferences.default_trust_level 0.7
    codeframe config set notifications.P0_channels "twilio_sms,email"

# Get configuration
codeframe config get <key>

# ===== CHECKPOINTS =====

# Create checkpoint
codeframe checkpoint create [--message <msg>]

# List checkpoints
codeframe checkpoints list

# Restore from checkpoint
codeframe restore <checkpoint-id>
```

### Python API (Programmatic Use)

```python
from codeframe import Project, MarketplaceClient

# ===== MARKETPLACE API =====

marketplace = MarketplaceClient()

# Search plugins
results = marketplace.search(
    query="security",
    theme="security",
    phase=11  # Code Review
)

# Install plugin
marketplace.install(
    plugin_name="security-scanner",
    scope="project",
    version="1.0.0"
)

# List installed
plugins = marketplace.list_installed(scope="all")

# ===== PROJECT API =====

project = Project.create("my-app")

# Configure
project.config.set("autonomy_preferences.default_trust_level", 0.7)
project.config.set("notifications.P0_channels", ["twilio_sms", "email"])

# Start
project.start()

# Query status
status = project.get_status()
print(f"Progress: {status.completion_percentage}%")
print(f"Active tasks: {status.active_tasks}")
print(f"Blocked tasks: {status.blocked_tasks}")

# Chat with lead agent (Loop 8)
lead = project.get_lead_agent()
response = lead.chat("How's it going?")
print(response)

# Checkpoint
project.create_checkpoint(message="Before major refactor")

# Restore
project.restore_from_checkpoint(checkpoint_id="ckpt_123")
```

---

## Phase I MVP Scope

### Must-Have Features

**✅ Hooks System:**
- `before_compact.js` - context pruning trigger
- `after_tool_use.js` - QC review trigger
- `user_prompt_submit.js` - priority scoring trigger
- Hook infrastructure and registration

**✅ MCP Server (codeframe-coordinator):**
- SQLite database management
- Tools: `continuous_prune()`, `get_next_task()`, `submit_for_qc()`, `score_priority()`
- Resources: `project_state`, `task_queue`, `plugin_catalog`, `user_input_queue`
- Single MCP server handles all state

**✅ Skills:**
- `/qc-review` - code quality gate
- `/chat` - project status interface
- `/plugin-discover` - context-aware recommendations
- `/plugin-vet` - approval workflow (manual in Phase I)

**✅ Plugin System:**
- Plugin manifest format (`plugin.json`)
- Slash commands: `/plugin-install`, `/plugin-search`, `/plugin-list`
- Single official marketplace source
- 5 initial curated plugins

**✅ Agent System:**
- Worker → QC → Coordinator workflow
- Workstream isolation (git branches via Bash tool)
- Internal PR system (database-tracked)
- State managed by MCP

**✅ User Interaction:**
- Native `AskUserQuestion` tool
- MCP resource for question queue
- P0/P1/P2/P3 routing via `score_priority()`
- Slash commands for all user operations

### Deferred to Phase II/III

**🔮 Phase II:**
- Multiple Claude Code instances (multi-agent coordination)
- Federated marketplace (multiple sources)
- User-submitted marketplaces
- Plugin dependency resolution (transitive)
- Web UI dashboard
- Notification channels (Twilio SMS, email, webhooks)

**🔮 Phase III:**
- Automated vetting pipeline
- Security scanning (static analysis)
- Performance testing (resource monitoring)
- ML-based autonomy scoring
- User preference learning

### Key Architectural Decisions

**✅ Single Instance (Phase I)**:
- One Claude Code instance with hooks and MCP
- Simpler architecture, easier to debug
- Proves the concept before scaling

**✅ Hooks Over Loops**:
- No concurrent Python processes
- Leverage Claude Code's execution flow
- Less complexity, more reliability

**✅ MCP for State**:
- Single source of truth
- No inter-process communication needed
- Standard MCP protocol

**✅ Context-Pruning-Lab Separation**:
- Algorithm development in separate repo
- Enables experimentation without affecting main system
- Clean integration boundary

---

## Implementation Roadmap

### Week 1-2: MCP Server Foundation + Hooks

**Objectives:**
- codeframe-coordinator MCP server scaffold
- SQLite database schema implementation
- Hook system infrastructure (before_compact, after_tool_use, user_prompt_submit)
- Basic MCP tools and resources

**Deliverables:**
- MCP server with SQLite database (all tables)
- Hooks registered and functional
- MCP tools: `get_next_task()`, `submit_for_qc()`, `score_priority()`
- MCP resources: `project_state`, `task_queue`
- Unit tests for MCP server

### Week 3-4: Skills (QC, Prioritization)

**Objectives:**
- `/qc-review` skill implementation
- Integration with `after_tool_use` hook
- Priority scoring logic in MCP
- Workstream management (git branches via Bash)

**Deliverables:**
- `/qc-review` skill functional (worker → QC workflow)
- Automatic QC trigger after git commits
- `score_priority()` with P0 pattern database
- Workstream creation/cleanup via Bash + MCP state tracking
- Sample QC rubric implemented

### Week 5-6: Plugin System (Slash Commands)

**Objectives:**
- Plugin manifest parser
- Slash commands: `/plugin-install`, `/plugin-search`, `/plugin-list`, `/plugin-info`
- MCP resource: `plugin_catalog`
- 5 initial plugins created

**Deliverables:**
- `plugin.json` manifest spec finalized
- Slash commands functional
- Plugin catalog in MCP resource
- 5 plugins:
  - code-reviewer-qc
  - security-scanner
  - test-automation
  - documentation-generator
  - architecture-advisor

### Week 7-8: Integration Testing

**Objectives:**
- End-to-end workflow testing
- `/chat` command implementation
- `/plugin-discover` skill
- Hook integration refinement

**Deliverables:**
- Full workflow: task → work → QC → merge tested
- `/chat` command shows project status
- Plugin discovery based on workflow phase
- Integration test suite
- Bug fixes

### Week 9-10: Context Pruning + Polish

**Objectives:**
- `before_compact` hook + `continuous_prune()` MCP tool
- Integration with context-pruning-lab algorithm
- Documentation
- Final testing and bug fixes

**Deliverables:**
- Continuous context pruning functional
- Algorithm from context-pruning-lab integrated
- User documentation (README, guides, CLAUDE.md)
- Developer documentation for extending system
- Phase I MVP ready for launch

### Post-Launch: Phase II Planning

**Objectives:**
- Gather user feedback
- Prioritize Phase II features
- Research multi-instance coordination patterns
- Begin federated marketplace design
- Explore automated vetting approaches

---

## Appendix: Initial 5 Plugins

### 1. code-reviewer-qc (QC Expert)

**Purpose**: General code quality review
**Agent Role**: QC Expert
**Workflow Phases**: [11] (Code Review)

**Rubric**:
- Quantitative: Code style compliance, complexity metrics, test coverage
- Qualitative: Readability, maintainability, documentation quality

### 2. security-scanner (QC Expert)

**Purpose**: Security vulnerability detection
**Agent Role**: QC Expert
**Workflow Phases**: [7, 11, 15] (Coding, Review, Deployment)

**Rubric**:
- Quantitative: CVE checks, hardcoded secrets, input validation
- Qualitative: Security architecture, defense in depth

### 3. test-automation (Worker)

**Purpose**: Automated test generation and execution
**Agent Role**: Worker
**Workflow Phases**: [6, 7, 10] (Test Development, Coding, CI)

**Capabilities**: Generates unit/integration tests, runs test suites

### 4. documentation-generator (Worker)

**Purpose**: Automatic documentation generation
**Agent Role**: Worker
**Workflow Phases**: [8] (Documentation Development)

**Capabilities**: Generates README, API docs, inline comments

### 5. architecture-advisor (Worker + QC)

**Purpose**: Architecture review and recommendations
**Agent Role**: Worker + QC Expert
**Workflow Phases**: [5, 11] (Architecture Design, Code Review)

**Capabilities**: Proposes architecture patterns, reviews design decisions

---

## Document Status

**Version**: 2.0
**Status**: Updated for Claude Code Native Architecture
**Next Review**: After Week 2 (MCP Server + Hooks Complete)

**Changelog**:
- 2025-10-29: Major refactor to Claude Code native architecture
  - Replaced 8 concurrent Python loops with hooks + MCP server
  - Single MCP server (codeframe-coordinator) manages all state
  - Hooks system for interception points (before_compact, after_tool_use, etc.)
  - Skills framework for behavioral patterns
  - Slash commands for user interface
  - Referenced context-pruning-lab for algorithm development
  - Updated implementation roadmap to reflect new architecture
- 2025-01-29: Initial specification based on Socratic discovery process

---

**End of Technical Specification**
