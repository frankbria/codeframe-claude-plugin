# Plugin Marketplace for Claude Code - Technical Specification v1.0

**Project**: Codeframe-Inspired Plugin Marketplace System
**Built As**: Claude Code Plugin within Marketplace
**License**: MIT Open Source
**Last Updated**: 2025-01-29

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Concurrent Loop System](#concurrent-loop-system)
4. [Agent Supervision Model](#agent-supervision-model)
5. [Workstream Isolation Model](#workstream-isolation-model)
6. [Plugin System](#plugin-system)
7. [QC and Vetting System](#qc-and-vetting-system)
8. [User Interaction Model](#user-interaction-model)
9. [Persistence and Recovery](#persistence-and-recovery)
10. [Database Schemas](#database-schemas)
11. [API Specifications](#api-specifications)
12. [Phase I MVP Scope](#phase-i-mvp-scope)
13. [Implementation Roadmap](#implementation-roadmap)

---

## Executive Summary

### Vision

Build a **plugin marketplace system** as a Claude Code plugin that enables autonomous software development through coordinated agent swarms, inspired by the Codeframe specification. The system operates through independent concurrent loops with sophisticated QC oversight, workstream isolation, and intelligent user interruption patterns.

### Key Innovations

1. **Three-Tier Supervision**: Worker → QC Expert → Coordinator pattern ensures consistent quality
2. **Workstream Isolation**: Git branch-based workspaces prevent resource conflicts between agents
3. **Concurrent Loop Architecture**: 8 independent loops communicate through coordinator pattern
4. **Unified Question Queue**: All loops feed into Claude Code's existing user interaction system
5. **Flash Save Persistence**: Object serialization enables instant checkpoint and recovery
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
│  CLI: codeframe-marketplace [cmd] / Chat: Loop 8                │
└────────────────────────────┬────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │   Unified Question Queue    │
              │  (Claude Code Integration)  │
              └──────────────┬──────────────┘
                             │
┌────────────────────────────┴──────────────────────────────────┐
│                   LOOP COORDINATORS                            │
│         (Central communication points)                         │
├────────────┬────────────┬────────────┬───────────┬───────────┤
│  Coord 1   │  Coord 2   │  Coord 4   │  Coord 6  │  Coord 7  │
│  (Work)    │ (Context)  │  (Sched)   │  (Inter)  │  (Vet)    │
└─────┬──────┴──────┬─────┴──────┬─────┴─────┬─────┴─────┬─────┘
      │             │            │           │           │
┌─────▼──────┐ ┌───▼─────┐ ┌───▼─────┐ ┌───▼─────┐ ┌──▼──────┐
│  LOOP 1    │ │ LOOP 2  │ │ LOOP 4  │ │ LOOP 6  │ │ LOOP 7  │
│  Work      │ │ Context │ │Schedule │ │Interrupt│ │ Vetting │
│  ALWAYS    │ │ REQUEST │ │ REQUEST │ │ON-DEMAND│ │ON-DEMAND│
│  RUNNING   │ │ INTAKE  │ │ INTAKE  │ │         │ │         │
└─────┬──────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘
      │             │           │           │           │
      └─────────────┴───────────┴───────────┴───────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                DATABASE MESSAGE BUS                              │
│  • loop_messages (inter-loop communication)                     │
│  • user_input_queue (unified questions)                         │
│  • workstreams (agent isolation)                                │
│  • internal_prs (QC workflow)                                   │
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

1. **Autonomous Operation**: Loops run independently with minimal human oversight
2. **Consistent Quality**: QC experts develop and apply rubrics consistently
3. **Isolation**: Workstreams prevent resource conflicts between agents
4. **Persistence**: Flash save at any moment enables instant recovery
5. **User Control**: Configurable autonomy levels (bypass/async/sync interruption)
6. **Plugin Discovery**: Context-aware plugin recommendations based on workflow phase

---

## Concurrent Loop System

### Loop Overview

The system operates through **8 independent concurrent loops**, each with its own coordinator:

| Loop | Name | Type | Trigger | Purpose |
|------|------|------|---------|---------|
| 1 | Work Execution | Always Running | System start | Main development workflow |
| 2 | Context Management | Request Intake | Context >80% | Compaction and tiering |
| 3 | Health Check | Request Intake | Manual / Timer | Agent timeout detection |
| 4 | Prioritization | Request Intake | Task complete | Scheduling and bottlenecks |
| 5 | Plugin Discovery | Request Intake | Phase change | Context-aware recommendations |
| 6 | Interruption | On-Demand | Decision needed | User input scoring |
| 7 | Vetting | On-Demand | Plugin submit | Plugin approval workflow |
| 8 | User Chat | On-Demand | User initiates | "Peek in" interface |

### Loop 1: Work Execution (Always Running)

**Frequency**: Continuous
**Purpose**: Main development loop - drives entire project

```
┌─────────────────────────────────────────┐
│  COORDINATOR: Select Next Task          │
│  • Query ready tasks (dependencies met) │
│  • Apply priority (P0 > P1 > P2 > P3)  │
│  • Match task to available worker       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  CREATE WORKSTREAM:                     │
│  • Git branch: worker-N/task-X.Y.Z     │
│  • Isolated workspace directory         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  WORKER: Execute Task                   │
│  • Work in isolated workspace           │
│  • Commit changes to branch             │
│  • "Turn in" to QC Expert              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  QC EXPERT: Review Work                 │
│  • Checkout worker's branch             │
│  • Apply quantitative checks            │
│  • Apply qualitative scoring            │
│  • Verdict: Pass / Conditional / Fail   │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┴─────────┐
       │ PASS              │ FAIL
       ▼                   ▼
┌─────────────┐      ┌──────────────┐
│Submit to    │      │Request changes│
│Coordinator  │      │Worker fixes   │
└──────┬──────┘      └──────┬───────┘
       │                    │
       ▼                    │
┌─────────────────────────────┐
│  COORDINATOR: Final Review   │
│  • Approve or call "stop"   │
│  • Merge branch → main      │
│  • Update task status       │
│  • Cleanup workstream       │
│  • Flash save               │
└────────────────┬────────────┘
                 │
                 ▼
          ┌──────────┐
          │  REPEAT  │
          └──────────┘
```

**Key Checkpoints**:
- Worker timeout (>5 min) → QC/Coordinator intervenes
- QC deadlock → Coordinator veto power
- P0 detected → Immediate sync interruption (breaks loop)

### Loop 2: Context Management (Request Intake)

**Trigger**: Context window >80% full
**Purpose**: Importance-based compaction and tiering

**IMPORTANT DISTINCTION**:
- **Flash Save** = Emergency backup of FULL context (120K+ tokens) → Only used for crash recovery
- **Compaction** = Selective reload with ONLY HOT tier (~20-30% of original context)
- **New conversation starts at ~20-30% capacity, NOT 80%**

```
┌─────────────────────────────────────────┐
│  TRIGGER: Context >80% Full             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  FLASH SAVE: Emergency Backup ONLY      │
│  • Save FULL conversation (120K tokens) │
│  • Git commit current work              │
│  • DB snapshot                          │
│  • NOT restored in normal operation     │
│  • Only for crash recovery              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  IMPORTANCE SCORING:                    │
│  • Calculate recency score              │
│  • Calculate access frequency           │
│  • Apply type weights                   │
│  • Final score: 0.0-1.0                │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  TIER ASSIGNMENT:                       │
│  • HOT (0.8-1.0): Active context       │
│  • WARM (0.4-0.8): Queryable archive   │
│  • COLD (0.0-0.4): Long-term storage   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  COMPACTION: Start NEW Conversation     │
│  • Load ONLY HOT tier (~40K tokens)    │
│  • Archive WARM tier to DB (queryable) │
│  • Archive COLD tier to filesystem     │
│  • Start fresh conversation at ~27%    │
│  • 73% capacity available for growth   │
└────────────────┬────────────────────────┘
                 │
                 ▼
          ┌──────────┐
          │  RESUME  │
          └──────────┘
```

### Compaction Example: Token Counts

This example demonstrates the context reduction process when compaction is triggered at 80% capacity (assuming a 150K token limit).

**Before Compaction (80% full - 120,000 tokens):**

| Context Item | Tokens | Importance Score | Tier Assignment |
|-------------|--------|------------------|-----------------|
| Current task spec | 10,000 | 0.95 | HOT (stays in context) |
| Active files (3) | 20,000 | 0.90 | HOT (stays in context) |
| Recent test results | 10,000 | 0.85 | HOT (stays in context) |
| Related files (10) | 30,000 | 0.60 | WARM (archived to DB) |
| Old test results | 30,000 | 0.30 | COLD (archived to filesystem) |
| Completed tasks | 20,000 | 0.20 | COLD (archived to filesystem) |
| **TOTAL** | **120,000** | - | - |

**After Compaction (27% full - 40,000 tokens):**

| Context Item | Tokens | Status | Location |
|-------------|--------|--------|----------|
| Current task spec | 10,000 | ✅ Loaded | New conversation |
| Active files (3) | 20,000 | ✅ Loaded | New conversation |
| Recent test results | 10,000 | ✅ Loaded | New conversation |
| Related files (10) | 30,000 | 📊 Queryable | Database (loaded on-demand) |
| Old test results | 30,000 | 📁 Archived | Filesystem (rarely accessed) |
| Completed tasks | 20,000 | 📁 Archived | Filesystem (rarely accessed) |
| **HOT TIER TOTAL** | **40,000** | - | **In active context** |
| **WARM TIER TOTAL** | **30,000** | - | **Available via DB query** |
| **COLD TIER TOTAL** | **50,000** | - | **Long-term storage** |

**Result**: Context reduced from 120K (80%) to 40K (27%), leaving **73% capacity available** for new work.

**Hot Tier Selection Algorithm:**

```python
def select_hot_tier(context_items: List[ContextItem],
                    target_tokens: int = 40000) -> List[ContextItem]:
    """
    Select the most important context items to keep in active memory.

    Args:
        context_items: All items in current context
        target_tokens: Target token count for HOT tier (~25-30% of max)

    Returns:
        List of items to keep in HOT tier
    """
    # Calculate importance scores for all items
    for item in context_items:
        item.importance_score = calculate_importance(item)

    # Sort by importance (descending)
    sorted_items = sorted(context_items,
                         key=lambda x: x.importance_score,
                         reverse=True)

    # Select items until target token count reached
    hot_tier = []
    current_tokens = 0

    for item in sorted_items:
        if current_tokens + item.token_count <= target_tokens:
            hot_tier.append(item)
            current_tokens += item.token_count
        else:
            # Token budget exceeded - remaining items go to WARM/COLD
            break

    return hot_tier

def calculate_importance(item: ContextItem) -> float:
    """
    Calculate importance score (0.0-1.0) based on multiple factors.

    Score components:
    - Recency: More recent = higher score (0.0-0.4)
    - Access frequency: More accesses = higher score (0.0-0.3)
    - Type weight: Some types inherently more important (0.0-0.3)
    """
    # Recency score (exponential decay)
    age_hours = (datetime.now() - item.last_accessed).total_seconds() / 3600
    recency_score = 0.4 * math.exp(-age_hours / 24)  # Half-life: 24 hours

    # Access frequency score (logarithmic)
    frequency_score = 0.3 * min(1.0, math.log(item.access_count + 1) / 5)

    # Type weight score
    type_weights = {
        'current_task': 0.30,      # Always highest priority
        'active_file': 0.25,       # Currently editing
        'test_result': 0.20,       # Recent test output
        'completed_task': 0.05,    # Historical only
        'dependency': 0.15,        # Related files
        'documentation': 0.10      # Reference material
    }
    type_score = type_weights.get(item.type, 0.10)

    # Combine scores
    total_score = recency_score + frequency_score + type_score
    return min(1.0, total_score)  # Cap at 1.0
```

**Tier Thresholds:**
- **HOT (0.8-1.0)**: Must stay in active context for immediate access
- **WARM (0.4-0.8)**: Important but can be loaded on-demand from database
- **COLD (0.0-0.4)**: Historical data, archived to filesystem, rarely needed

**WARM Tier Access Pattern:**
When an agent needs WARM tier data, the system:
1. Queries database for specific items by ID or pattern
2. Loads requested items into temporary context
3. Updates access count and recency (may promote to HOT on next compaction)
4. Removes from context after task completion

**COLD Tier Access Pattern:**
COLD tier data is typically only accessed for:
- Historical analysis requests
- Debugging old issues
- Compliance/audit requirements
- Manual user requests

### Loop 4: Prioritization & Scheduling (Request Intake)

**Trigger**: Task completion OR every 2 minutes
**Purpose**: Dependency resolution, bottleneck detection

```
┌─────────────────────────────────────────┐
│  SCAN TASK QUEUE: All Pending Tasks    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  DEPENDENCY CHECK:                      │
│  • Filter where predecessors complete   │
│  • Check circular dependencies          │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PRIORITY SCORING:                      │
│  • P0 (showstopper) → SYNC             │
│  • P1 (high) → ASYNC                   │
│  • P2 (medium) → BYPASS                │
│  • P3 (low) → BYPASS                   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  BOTTLENECK DETECTION:                  │
│  • Count tasks blocked by each task     │
│  • If blocking ≥ 3 → Flag bottleneck   │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┴─────────┐
       │ Bottleneck?       │
       ▼ YES               ▼ NO
┌─────────────┐      ┌──────────────┐
│ Rebalance:  │      │ Assign tasks │
│ • Extra     │      │ to idle      │
│   worker    │      │ workers      │
│ • Escalate  │      └──────────────┘
└─────────────┘
       │
       └────────────────┐
                        ▼
                 ┌──────────┐
                 │  REPEAT  │
                 └──────────┘
```

### Loop 6: Autonomous Interruption (On-Demand)

**Trigger**: Decision point requiring user input
**Purpose**: Score autonomy and route to bypass/async/sync

```
┌─────────────────────────────────────────┐
│  TRIGGER: Decision Required             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AUTONOMY SCORING:                      │
│  • Check P0 pattern database            │
│  • Check decision reversibility         │
│  • Check user trust level               │
│  • Calculate: P0/P1/P2/P3              │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┼─────────┬─────────┐
       │ P0      │ P1      │ P2      │ P3
       ▼         ▼         ▼         ▼
    ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
    │SYNC  │ │ASYNC │ │BYPASS│ │LOG   │
    │Stop  │ │Notify│ │Auto  │ │Only  │
    │work  │ │user  │ │decide│ │      │
    └───┬──┘ └───┬──┘ └───┬──┘ └──────┘
        │        │        │
        └────────┴────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  SEND NOTIFICATION:                     │
│  • Compose with context                 │
│  • Route to channels (SMS/email/webhook)│
│  • Log in interruption_log              │
└────────────────┬────────────────────────┘
                 │
       ┌─────────┴─────────┐
       │ SYNC              │ ASYNC
       ▼                   ▼
┌─────────────┐      ┌──────────────┐
│Block & wait │      │Continue work,│
│for user     │      │check for     │
│             │      │response/5min │
└──────┬──────┘      └──────┬───────┘
       │                    │
       └────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  USER RESPONDS:                         │
│  • Parse decision                       │
│  • Update tasks/settings                │
│  • Log response time                    │
│  • Update P0 patterns (learning)        │
└────────────────┬────────────────────────┘
                 │
                 ▼
          ┌──────────┐
          │  RESUME  │
          └──────────┘
```

### Loop 8: User Chat Interface (On-Demand)

**Trigger**: User initiates chat
**Purpose**: "Peek in" on development team, opportunistic question answering

```
┌─────────────────────────────────────────┐
│  LISTEN: User Initiates Chat            │
│  • CLI: codeframe chat                  │
│  • User types question                  │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  QUERY SYSTEM STATE:                    │
│  • Active tasks (Loop 1)                │
│  • Pending questions (Loop 6)           │
│  • Recent failures                      │
│  • Context health (Loop 2)              │
│  • Bottlenecks (Loop 4)                 │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  GENERATE RESPONSE:                     │
│  • Natural language summary             │
│  • Show metrics                         │
│  • Highlight blockers                   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OPPORTUNISTIC QUESTIONS:               │
│  "While you're here, I have 3 questions"│
│  • Show unified question queue          │
│  • User can answer or defer             │
└────────────────┬────────────────────────┘
                 │
                 ▼
          ┌──────────┐
          │  REPEAT  │
          └──────────┘
```

### Loop Lifecycle Management

**Startup Sequence**:

```python
def startup():
    # ALWAYS start Loop 1 (Work Execution)
    start_loop('loop_1', WorkExecutionLoop, always_running=True)

    # Register triggers for on-demand loops
    register_trigger('loop_6', InterruptionLoop, trigger_type='on_demand')
    register_trigger('loop_7', VettingLoop, trigger_type='on_demand')

    # Register triggers for request-intake loops
    register_trigger('loop_2', ContextManagementLoop,
                    trigger_fn=check_context_threshold)
    register_trigger('loop_4', PrioritizationLoop,
                    trigger_fn=check_scheduling_needs)
    register_trigger('loop_5', PluginDiscoveryLoop,
                    trigger_fn=check_plugin_discovery_needs)
    register_trigger('loop_8', UserChatLoop,
                    trigger_fn=check_user_chat_request)
```

**Shutdown Sequence**:

```python
def shutdown():
    # Flash save all loops (emergency checkpoint)
    for loop_id, loop_info in active_loops.items():
        loop_info['coordinator'].emergency_checkpoint()

    # Stop non-essential loops first
    for loop_id, loop_info in active_loops.items():
        if not loop_info['always_running']:
            loop_info['loop'].stop()

    # Finally stop Loop 1
    active_loops['loop_1']['loop'].stop()
```

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

**✅ Core Loops:**
- Loop 1 (Work Execution) - fully functional
- Loop 2 (Context Management) - basic tiering
- Loop 4 (Prioritization) - P0/P1/P2/P3 scoring
- Loop 6 (Interruption) - basic autonomy scoring
- Loop 8 (User Chat) - "peek in" interface

**✅ Agent System:**
- Worker → QC → Coordinator workflow
- Workstream isolation (git branches)
- Internal PR system
- Timeout detection

**✅ Plugin System:**
- Plugin manifest format
- CLI installation (global/project scope)
- Single official marketplace source
- 5 initial curated plugins

**✅ QC System:**
- QC rubric proposal/approval
- Quantitative + qualitative checks
- Coordinator veto power

**✅ Persistence:**
- Flash save (object serialization)
- Recovery from checkpoint
- Database state management

**✅ Notifications:**
- Twilio SMS
- Email (SMTP)
- Basic n8n/Zapier webhook

**✅ User Interaction:**
- Unified question queue
- Claude Code AskUserQuestion integration
- P0/P1/P2/P3 routing

### Deferred to Phase II/III

**🔮 Phase II:**
- Federated marketplace (multiple sources)
- User-submitted marketplaces
- Plugin dependency resolution (transitive)
- Web UI dashboard
- Badge/gamification system

**🔮 Phase III:**
- Automated vetting pipeline
- Security scanning (static analysis)
- Performance testing (resource monitoring)
- ML-based autonomy scoring
- User preference learning

### Research Items (Phase I)

**⏳ Context Management Research:**
- Evaluate: SQLite vs. Redis vs. Filesystem vs. In-memory
- Goal: <10ms plugin metadata lookup
- Decision criteria: Speed + effectiveness
- Deliverable: Recommendation document

---

## Implementation Roadmap

### Week 1-2: Foundation

**Objectives:**
- Database schema implementation
- Persistence manager (flash save)
- Base loop class with recovery
- Loop coordinator pattern

**Deliverables:**
- SQLite database created with all tables
- `PersistenceManager` class functional
- `BaseLoop` and `LoopCoordinator` base classes
- Unit tests for serialization/deserialization

### Week 3-4: Core Loops

**Objectives:**
- Loop 1 (Work Execution) - basic workflow
- Workstream isolation (git branch model)
- Internal PR system
- Basic QC rubric system

**Deliverables:**
- Loop 1 running end-to-end (worker → QC → coordinator)
- Workstream creation/cleanup functional
- Internal PR tracked in database
- Sample QC rubric implemented

### Week 5-6: Plugin System

**Objectives:**
- Plugin manifest parser
- CLI commands (search, install, uninstall, list, info)
- Plugin catalog database
- 5 initial plugins created

**Deliverables:**
- `plugin.json` manifest spec finalized
- CLI commands functional
- 5 plugins:
  - code-reviewer-qc
  - security-scanner
  - test-automation
  - documentation-generator
  - architecture-advisor

### Week 7-8: User Interaction & Loops

**Objectives:**
- Loop 4 (Prioritization)
- Loop 6 (Interruption) with autonomy scoring
- Loop 8 (User Chat)
- Unified question queue
- Notification system (Twilio, Email)

**Deliverables:**
- P0/P1/P2/P3 prioritization functional
- Autonomy scoring with P0 pattern database
- Chat interface ("peek in")
- SMS + Email notifications working

### Week 9-10: Polish & Testing

**Objectives:**
- Loop 2 (Context Management) - basic tiering
- Retry → Debug pattern
- Integration testing
- Documentation
- Bug fixes

**Deliverables:**
- Context tiering functional (hot/warm/cold)
- Smart retry with debug escalation
- Integration test suite
- User documentation (README, guides)
- Phase I MVP ready for launch

### Post-Launch: Phase II Planning

**Objectives:**
- Gather user feedback
- Prioritize Phase II features
- Begin federated marketplace design
- Research automated vetting approaches

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

**Version**: 1.0
**Status**: Draft for Phase I Implementation
**Next Review**: After Week 2 (Foundation Complete)

**Changelog**:
- 2025-01-29: Initial specification based on Socratic discovery process

---

**End of Technical Specification**
