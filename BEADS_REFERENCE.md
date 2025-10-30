# Beads Issue Tracking Reference

**Database**: `.beads/cf.db`
**Prefix**: `cf` (codeframe)
**Total Issues**: 48
**Critical Path Issues**: 30 (P0 priority)

---

## Quick Commands

```bash
# See what's ready to work on (no blockers)
bd ready

# List all issues
bd list

# Show issue details
bd show cf-1

# View dependency tree for an issue
bd dep tree cf-14

# Update issue status
bd update cf-10 --status in_progress
bd close cf-10 --reason "Completed MCP server scaffold"

# Check for circular dependencies
bd dep cycles
```

---

## Issue Structure

### Phase Epics (9 total)

| ID | Title | Priority | Description |
|----|-------|----------|-------------|
| **cf-1** | Phase 1: Foundation | P0 | MCP server + database foundation (Weeks 1-2) |
| **cf-2** | Phase 2: Core Integration | P0 | Hooks + MCP integration (Weeks 2-3) - HIGH RISK |
| **cf-3** | Phase 3: Workflows | P0 | QC workflow + prioritization (Weeks 3-5) |
| **cf-4** | Phase 4: Marketplace | P1 | Plugin system + catalog (Weeks 5-7) |
| **cf-5** | Phase 5: Integration Testing | P0 | E2E validation (Weeks 7-8) |
| **cf-6** | Phase 6: Context Management | P0 | Continuous pruning integration (Weeks 9-10) |
| **cf-7** | Track A: Context-Pruning-Lab | P0 | Algorithm validation (Weeks 1-6, parallel) |
| **cf-8** | Track B: Plugin Development | P1 | Initial 3-5 plugins (Weeks 3-6, parallel) |
| **cf-9** | Track C: Documentation | P2 | User/developer docs (Weeks 1-10, incremental) |

---

## Critical Path

```
cf-1 (Phase 1) → cf-2 (Phase 2) → cf-3 (Phase 3) → cf-5 (Phase 5) → cf-6 (Phase 6)
     ↓                ↓                ↓                ↓                ↓
  Weeks 1-2        Weeks 2-3        Weeks 3-5        Weeks 7-8        Weeks 9-10
```

**Zero Slack**: Any delay on critical path cascades to launch.

---

## Phase 1: Foundation (cf-1)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-10** | WP1.1: MCP Server Scaffold | P0 | 3 days | None (START HERE) |
| **cf-11** | WP1.2: SQLite Database - Core Tables | P0 | 4 days | None (START HERE) |
| **cf-12** | WP1.3: Basic MCP Tools | P0 | 2 days | cf-10, cf-11 |
| **cf-13** | WP1.4: Basic MCP Resources | P0 | 2 days | cf-10 |
| **cf-14** | WP1.5: Unit Testing - Foundation | P0 | 2 days | cf-11, cf-12, cf-13 |

**Total**: 13 days (2.6 weeks)

**Start Now**: cf-10, cf-11 are ready and can be done in parallel

**Quality Gate**: MCP server functional, all core tables created, 90%+ test coverage

---

## Phase 2: Core Integration (cf-2)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-15** | WP2.1: Hook Infrastructure | P0 | 4 days | cf-1 (Phase 1) |
| **cf-16** | WP2.2: Hook ↔ MCP Integration | P0 | 3 days | cf-15 |
| **cf-17** | WP2.3: State Management | P0 | 2 days | cf-15 |
| **cf-18** | WP2.4: Integration Testing - Hooks | P0 | 2 days | cf-16, cf-17 |

**Total**: 11 days (2.2 weeks) + 4 days buffer (HIGH RISK)

**Blocks**: Phase 3, Phase 4, Phase 5, Phase 6

**Quality Gate**: Hooks trigger correctly without breaking Claude Code, hook overhead < 100ms

**⚠️ HIGH RISK**: Hook system integration complexity unknown. May need Claude Code team support.

---

## Phase 3: Workflows (cf-3)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-19** | WP3.1: QC Skills Implementation | P0 | 5 days | cf-2 (Phase 2) |
| **cf-20** | WP3.2: Automatic QC Triggering | P0 | 3 days | cf-19 |
| **cf-21** | WP3.3: Priority Scoring Logic | P0 | 4 days | cf-2 (Phase 2) |
| **cf-22** | WP3.4: Workstream Isolation | P0 | 4 days | cf-2 (Phase 2) |
| **cf-23** | WP3.5: Sample QC Rubric | P1 | 2 days | cf-19 |

**Total**: 18 days (3.6 weeks)

**Blocks**: Phase 4, Phase 5, Phase 6

**Quality Gate**: Worker → QC → Coordinator workflow completes successfully, workstreams isolated

---

## Phase 4: Marketplace (cf-4)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-24** | WP4.1: Plugin Manifest Specification | P1 | 2 days | cf-3 (Phase 3) |
| **cf-25** | WP4.2: Plugin Catalog System | P1 | 3 days | cf-24 |
| **cf-26** | WP4.3: Slash Commands | P1 | 4 days | cf-25 |
| **cf-27** | WP4.4: Plugin Implementations | P1 | 10 days | cf-24, cf-8 (Track B) |
| **cf-28** | WP4.5: Plugin Testing | P1 | 2 days | cf-26, cf-27 |

**Total**: 21 days (4.2 weeks) sequential, 14 days with parallelization

**Blocks**: None (off critical path)

**Quality Gate**: 3-5 plugins functional, plugin discovery works, catalog persists

---

## Phase 5: Integration Testing (cf-5)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-29** | WP5.1: End-to-End Workflow Tests | P0 | 3 days | cf-3 (Phase 3) |
| **cf-30** | WP5.2: /chat Command | P1 | 2 days | cf-29 |
| **cf-31** | WP5.3: /plugin-discover Skill | P1 | 2 days | cf-25 |
| **cf-32** | WP5.4: Bug Fixing & Refinement | P0 | 5 days | cf-29 |

**Total**: 12 days (2.4 weeks)

**Blocks**: Phase 6

**Quality Gate**: All E2E tests pass, no critical bugs, performance targets met

---

## Phase 6: Context Management (cf-6)

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-33** | WP6.1: Context-Pruning-Lab Algorithm Validation | P0 | - | cf-40 (Track A) |
| **cf-34** | WP6.2: Continuous Pruning Integration | P0 | 4 days | cf-33 |
| **cf-35** | WP6.3: Performance Testing | P0 | 2 days | cf-34 |
| **cf-36** | WP6.4: Documentation | P1 | 3 days | cf-5 (Phase 5) |
| **cf-37** | WP6.5: Final Testing & Polish | P0 | 2 days | cf-35 |

**Total**: 11 days (2.2 weeks)

**Blocks**: Launch

**Quality Gate**: Context pruning prevents degradation over 100+ interactions, all docs complete

---

## Track A: Context-Pruning-Lab (cf-7)

**Parallel Track** - Runs independently from Phase 1-6

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-38** | Track A: Algorithm Implementation | P0 | Weeks 1-2 | None (START HERE) |
| **cf-39** | Track A: Experiments & Benchmarks | P0 | Weeks 3-4 | cf-38 |
| **cf-40** | Track A: Validation & Optimization | P0 | Weeks 5-6 | cf-39 |

**Total**: 6 weeks parallel development

**Feeds Into**: cf-33 (WP6.1 Algorithm Validation)

**Quality Gate**: All experiments pass, performance < 50ms per prune, no degradation observed

**Start Now**: cf-38 is ready and independent

---

## Track B: Plugin Development (cf-8)

**Parallel Track** - Can be parallelized with 2-3 developers

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-41** | Track B: code-reviewer-qc Plugin | P1 | 2 days | cf-24 |
| **cf-42** | Track B: security-scanner Plugin | P1 | 2 days | cf-24 |
| **cf-43** | Track B: test-automation Plugin | P1 | 2 days | cf-24 |
| **cf-44** | Track B: documentation-generator Plugin | P2 | 2 days | cf-24 |
| **cf-45** | Track B: architecture-advisor Plugin | P2 | 2 days | cf-24 |

**Total**: 10 days sequential, 4-5 days with parallelization

**Feeds Into**: cf-27 (WP4.4 Plugin Implementations)

**Priority Order**: cf-41 (CRITICAL), cf-42/43 (HIGH), cf-44/45 (MEDIUM)

---

## Track C: Documentation (cf-9)

**Parallel Track** - Incremental documentation throughout project

### Work Packages

| ID | Title | Priority | Duration | Dependencies |
|----|-------|----------|----------|--------------|
| **cf-46** | Track C: User Documentation | P2 | Incremental | None |
| **cf-47** | Track C: Developer Documentation | P2 | Incremental | None |
| **cf-48** | Track C: Architecture Documentation | P2 | Incremental | None |

**Total**: Weeks 1-10, written incrementally

**Feeds Into**: cf-36 (WP6.4 Final Documentation)

**Start Now**: All three can start immediately and be updated incrementally

---

## Dependency Types

### `blocks` - Sequential Dependency
One task must complete before another can start.

**Examples**:
- cf-16 (Hook Integration) blocks on cf-15 (Hook Infrastructure)
- cf-2 (Phase 2) blocks on cf-1 (Phase 1)

### `parent-child` - Hierarchical Relationship
Work packages belong to phase epics.

**Examples**:
- cf-10, cf-11, cf-12, cf-13, cf-14 are children of cf-1 (Phase 1)

### `related` - Soft Connection
Issues are related but don't block each other.

**Examples**:
- cf-27 (Plugin Implementations) related to cf-8 (Track B epic)

---

## What's Ready to Start Now?

Run `bd ready` to see current ready work. As of initialization:

**Ready to Start (No Blockers)**:
1. **cf-10**: WP1.1: MCP Server Scaffold (P0) - 3 days
2. **cf-11**: WP1.2: SQLite Database - Core Tables (P0) - 4 days
3. **cf-38**: Track A: Algorithm Implementation (P0) - Weeks 1-2
4. **cf-46**: Track C: User Documentation (P2) - Incremental
5. **cf-47**: Track C: Developer Documentation (P2) - Incremental
6. **cf-48**: Track C: Architecture Documentation (P2) - Incremental

**Recommended Start**:
1. Start **cf-10** and **cf-11** in parallel (critical path)
2. Start **cf-38** in parallel (separate track)
3. Begin **cf-46, cf-47, cf-48** incrementally

---

## Workflow Examples

### Starting Work on an Issue

```bash
# Claim the issue
bd update cf-10 --status in_progress

# Work on it...

# When complete
bd close cf-10 --reason "MCP server scaffold complete with TypeScript setup"
```

### Checking What's Unblocked

```bash
# See all ready work
bd ready

# See ready work by priority
bd list --status open --priority 0 | grep "ready"

# See what's blocked
bd list --status open | grep "blocked"
```

### Visualizing Dependencies

```bash
# See dependency tree for Phase 1
bd dep tree cf-1

# See dependency tree for WP1.5 (shows all prerequisites)
bd dep tree cf-14

# Check for circular dependencies (should be none)
bd dep cycles
```

### Tracking Progress

```bash
# List all open issues
bd list --status open

# List by phase (using grep)
bd list | grep "Phase 1"

# Show completed work
bd list --status closed

# Show in-progress work
bd list --status in_progress
```

---

## Cross-Reference to Implementation Plan

### Document Sections → Beads Issues

| Plan Section | Beads Issues |
|--------------|--------------|
| Phase 1: Foundation | cf-1 (epic), cf-10 to cf-14 (work packages) |
| Phase 2: Core Integration | cf-2 (epic), cf-15 to cf-18 (work packages) |
| Phase 3: Workflows | cf-3 (epic), cf-19 to cf-23 (work packages) |
| Phase 4: Marketplace | cf-4 (epic), cf-24 to cf-28 (work packages) |
| Phase 5: Integration Testing | cf-5 (epic), cf-29 to cf-32 (work packages) |
| Phase 6: Context Management | cf-6 (epic), cf-33 to cf-37 (work packages) |
| Track A: Context-Pruning-Lab | cf-7 (epic), cf-38 to cf-40 (tasks) |
| Track B: Plugin Development | cf-8 (epic), cf-41 to cf-45 (plugins) |
| Track C: Documentation | cf-9 (epic), cf-46 to cf-48 (docs) |

### Critical Path Visualization

```
START
  ↓
cf-10, cf-11 (Phase 1 Foundation - parallel start)
  ↓
cf-12, cf-13 (Phase 1 MCP Tools/Resources)
  ↓
cf-14 (Phase 1 Testing - Quality Gate 1)
  ↓
cf-15 (Phase 2 Hook Infrastructure - HIGH RISK)
  ↓
cf-16, cf-17 (Phase 2 Integration + State)
  ↓
cf-18 (Phase 2 Testing - Quality Gate 2)
  ↓
cf-19, cf-21, cf-22 (Phase 3 QC + Priority + Workstreams - parallel)
  ↓
cf-20 (Phase 3 Auto QC Triggering)
  ↓
cf-23 (Phase 3 QC Rubric - Quality Gate 3)
  ↓
cf-29 (Phase 5 E2E Tests)
  ↓
cf-32 (Phase 5 Bug Fixing - Quality Gate 5)
  ↓
[WAIT: cf-40 (Track A Validation) must complete]
  ↓
cf-33 (Phase 6 Algorithm Validation)
  ↓
cf-34 (Phase 6 Pruning Integration)
  ↓
cf-35 (Phase 6 Performance Testing)
  ↓
cf-37 (Phase 6 Final Polish - Quality Gate 6)
  ↓
LAUNCH
```

---

## Tips for Working with Beads

### For Solo Developer
1. Focus on critical path (P0 issues)
2. Use `bd ready` daily to see what's unblocked
3. Start parallel tracks early (cf-38 context-pruning-lab)
4. Document incrementally (cf-46, cf-47, cf-48)

### For Team (2-3 Developers)
1. **Developer A**: MCP backend (cf-10, cf-11, cf-12, cf-13, cf-14)
2. **Developer B**: Hooks + Skills (cf-15, cf-16, cf-17, cf-18, cf-19, cf-20)
3. **Developer C**: Context-pruning-lab + Plugins (cf-38, cf-39, cf-40, cf-41, cf-42, cf-43)

### Dealing with Blocked Issues
If an issue is blocked but you think it can start:
```bash
# Check dependencies
bd dep tree cf-XX

# If dependency is incorrect, remove it
bd dep remove cf-XX cf-YY

# If dependency should be 'related' not 'blocks'
bd dep remove cf-XX cf-YY
bd dep add cf-XX cf-YY --type related
```

---

## Integration with Git

Beads automatically syncs with git:
- ✅ Auto-exports to JSONL after changes (5s debounce)
- ✅ Auto-imports from JSONL after git pull
- ✅ No manual export/import needed

**Git workflow**:
```bash
# Make changes in beads
bd update cf-10 --status in_progress

# Changes automatically exported to .beads/*.jsonl
# Commit everything
git add .beads/
git commit -m "chore: Update beads status for cf-10"
git push

# On another machine
git pull
# Changes automatically imported!
```

---

## Document Status

**Version**: 1.0
**Last Updated**: 2025-10-29
**Total Issues**: 48
**Database**: `.beads/cf.db`

**Sync with Implementation Plan**: This document is a cross-reference to `PHASE_I_IMPLEMENTATION_PLAN.md`. Keep both updated:
- Implementation Plan: Detailed execution strategy, time estimates, risk analysis
- Beads Reference: Issue tracking, dependencies, status updates

**For Updates**:
- When creating new issues, add them to this reference
- When closing issues, document completion in git commit messages
- When dependencies change, update both the tree diagrams here and in beads

---

**END OF BEADS REFERENCE**
