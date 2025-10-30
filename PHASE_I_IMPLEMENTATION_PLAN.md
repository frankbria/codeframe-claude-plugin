# Phase I Implementation Plan - Codeframe Plugin Marketplace

**Version**: 1.0
**Date**: 2025-10-29
**Status**: Ready for Execution
**Target Duration**: 10-12 weeks (team) / 12-16 weeks (solo)

---

## Executive Summary

### Objective
Build the Phase I MVP of the Codeframe Plugin Marketplace - a Claude Code plugin that enables autonomous software development through coordinated agent swarms with continuous context management.

### Critical Path (10 weeks minimum, zero slack)
```
MCP Server Foundation → Hooks Infrastructure → QC Skills → Integration Testing → Context Pruning → Launch
   (Weeks 1-2)           (Weeks 2-3)         (Weeks 3-4)    (Weeks 7-8)        (Weeks 9-10)
```

### Key Success Criteria
- ✅ MCP server manages all state via SQLite
- ✅ Hooks trigger correctly without interfering with Claude Code
- ✅ Worker → QC → Coordinator workflow functions end-to-end
- ✅ 3-5 curated plugins operational in marketplace
- ✅ Continuous context pruning prevents agent degradation
- ✅ All quality gates passed with 80%+ test coverage

### Top 3 Risks
1. **Hook System Integration** (High) - Unknown complexity, may need Claude Code team support
2. **Context Pruning Effectiveness** (Medium) - Mitigated by separate validation in context-pruning-lab
3. **SQLite Performance** (Medium) - Concurrent access patterns need careful locking strategy

---

## Dependency Graph

```
                    ┌────────────────────────────────────────┐
                    │   FOUNDATION (Weeks 1-2)               │
                    │   • MCP Server Scaffold                │
                    │   • SQLite Schema (Core Tables)        │
                    │   • Basic Tools/Resources              │
                    └──────────────┬─────────────────────────┘
                                   │
                    ┌──────────────┴─────────────────────────┐
                    │   CORE INTEGRATION (Weeks 2-3)         │
                    │   • Hook Infrastructure                │
                    │   • Hook Registration                  │
                    │   • MCP Tool Integration               │
                    └──────────────┬─────────────────────────┘
                                   │
                    ┌──────────────┴─────────────────────────┐
                    │   WORKFLOWS (Weeks 3-4)                │
                    │   • QC Skills                          │
                    │   • Priority Scoring                   │
                    │   • Workstream Isolation               │
                    └──────────────┬─────────────────────────┘
                                   │
                    ┌──────────────┴─────────────────────────┐
                    │   MARKETPLACE (Weeks 5-6)              │
                    │   • Plugin System                      │
                    │   • Slash Commands                     │
                    │   • Plugin Catalog                     │
                    └──────────────┬─────────────────────────┘
                                   │
                    ┌──────────────┴─────────────────────────┐
                    │   INTEGRATION (Weeks 7-8)              │
                    │   • End-to-End Testing                 │
                    │   • /chat Command                      │
                    │   • Bug Fixes                          │
                    └──────────────┬─────────────────────────┘
                                   │
                    ┌──────────────┴─────────────────────────┐
                    │   CONTEXT MANAGEMENT (Weeks 9-10)      │
                    │   • Continuous Pruning Integration     │
                    │   • Performance Testing                │
                    │   • Final Polish                       │
                    └────────────────────────────────────────┘


PARALLEL TRACKS (Not on Critical Path):
─────────────────────────────────────────────────────────────
Track A: Context-Pruning-Lab (Weeks 1-6)
  • Algorithm development
  • Experiments & benchmarks
  • Validation testing

Track B: Plugin Development (Weeks 3-6)
  • Plugin manifest design
  • Initial 3-5 plugin implementations
  • Plugin testing

Track C: Documentation (Weeks 1-10, Incremental)
  • Developer documentation
  • User guides
  • API reference
  • Architecture decision records
```

---

## Phase-by-Phase Breakdown

### Phase 1: Foundation (Weeks 1-2)

**Goal**: Establish MCP server and database foundation

#### Work Packages

**WP1.1: MCP Server Scaffold** (3 days)
- Set up Node.js/TypeScript project structure
- Implement MCP protocol server skeleton
- Create configuration management
- Set up development environment
- **Deliverable**: Running MCP server that responds to basic queries

**WP1.2: SQLite Database - Core Tables** (4 days)
- Implement core schemas:
  - `workstreams`
  - `internal_prs`
  - `qc_reviews`
  - `p0_patterns`
  - `user_input_queue`
- Create migration system
- Implement connection pooling and locking
- **Deliverable**: Database with core tables, migrations, and query helpers

**WP1.3: Basic MCP Tools** (2 days)
- Implement `get_next_task()` stub
- Implement `submit_for_qc()` stub
- Implement `score_priority()` stub
- **Deliverable**: MCP tools return mock data correctly

**WP1.4: Basic MCP Resources** (2 days)
- Implement `project_state` resource
- Implement `task_queue` resource
- **Deliverable**: Resources queryable via MCP protocol

**WP1.5: Unit Testing** (2 days)
- Database operation tests
- MCP tool tests
- Resource tests
- **Deliverable**: 90%+ coverage on database layer

#### Quality Gate 1: Foundation Validation
- [ ] MCP server starts without errors
- [ ] All core tables created successfully
- [ ] MCP tools respond with correct structure
- [ ] Unit tests pass with 90%+ coverage
- [ ] Manual verification: Query MCP server from test client

**Estimated Duration**: 13 days (2.6 weeks)
**Risk Buffer**: +2 days for learning MCP protocol

---

### Phase 2: Core Integration (Weeks 2-3)

**Goal**: Integrate hooks with MCP server

#### Work Packages

**WP2.1: Hook Infrastructure** (4 days)
- Research Claude Code hook system
- Create hook registration mechanism
- Implement `before_compact.js` stub
- Implement `after_tool_use.js` stub
- Implement `user_prompt_submit.js` stub
- **Deliverable**: Hooks trigger at correct lifecycle points

**WP2.2: Hook ↔ MCP Integration** (3 days)
- Connect hooks to MCP tools
- Implement error handling and fallbacks
- Add logging and debugging
- **Deliverable**: Hooks successfully call MCP tools

**WP2.3: State Management** (2 days)
- Implement workstream creation/cleanup
- Implement task state transitions
- Add database transaction support
- **Deliverable**: State persisted correctly across hook calls

**WP2.4: Integration Testing** (2 days)
- Test hook triggers with real Claude Code operations
- Test error conditions
- Performance testing (hook overhead < 100ms)
- **Deliverable**: Integration test suite

#### Quality Gate 2: Core Integration Validation
- [ ] Hooks trigger without breaking Claude Code functionality
- [ ] MCP tools receive correct data from hooks
- [ ] State persists across Claude Code sessions
- [ ] Integration tests pass
- [ ] Performance: hook overhead < 100ms per call

**Estimated Duration**: 11 days (2.2 weeks)
**Risk Buffer**: +4 days for hook system unknowns (HIGH RISK)

---

### Phase 3: Workflows (Weeks 3-5)

**Goal**: Implement QC workflow and prioritization

#### Work Packages

**WP3.1: QC Skills Implementation** (5 days)
- Create `/qc-review` skill structure
- Implement worker submission workflow
- Implement QC expert review workflow
- Implement coordinator approval workflow
- **Deliverable**: `/qc-review` skill functional

**WP3.2: Automatic QC Triggering** (3 days)
- Integrate with `after_tool_use` hook
- Detect git commit operations
- Trigger QC for appropriate operations
- **Deliverable**: QC automatically triggered after commits

**WP3.3: Priority Scoring Logic** (4 days)
- Implement P0 pattern database
- Implement decision analysis algorithm
- Implement user trust level tracking
- Add learning from user corrections
- **Deliverable**: `score_priority()` returns P0/P1/P2/P3 correctly

**WP3.4: Workstream Isolation** (4 days)
- Implement git branch creation via Bash
- Implement workstream directory structure
- Implement workstream cleanup
- Add workstream state tracking in database
- **Deliverable**: Isolated workstreams per agent

**WP3.5: Sample QC Rubric** (2 days)
- Create code-quality rubric template
- Implement rubric scoring algorithm
- Add rubric customization support
- **Deliverable**: Working rubric for code reviews

#### Quality Gate 3: Workflow Validation
- [ ] Worker can submit work for QC review
- [ ] QC expert can review and approve/reject
- [ ] Coordinator can veto QC decisions
- [ ] Priority scoring returns sensible P0/P1/P2/P3 levels
- [ ] Workstreams isolated (no file conflicts)
- [ ] End-to-end workflow test passes

**Estimated Duration**: 18 days (3.6 weeks)
**Risk Buffer**: +2 days for git bash complexity

---

### Phase 4: Marketplace (Weeks 5-7)

**Goal**: Build plugin system and catalog

#### Work Packages

**WP4.1: Plugin Manifest Specification** (2 days)
- Finalize `plugin.json` schema
- Create manifest validator
- Document manifest fields
- **Deliverable**: Plugin manifest spec document

**WP4.2: Plugin Catalog System** (3 days)
- Implement `plugin_catalog` table operations
- Implement plugin discovery algorithm
- Add context-aware recommendations
- **Deliverable**: Plugin catalog MCP resource

**WP4.3: Slash Commands** (4 days)
- Implement `/plugin-install`
- Implement `/plugin-search`
- Implement `/plugin-list`
- Implement `/plugin-info`
- **Deliverable**: All plugin slash commands functional

**WP4.4: Plugin Implementations** (10 days, parallelizable)
- Create 3-5 plugins:
  1. **code-reviewer-qc** (QC Expert) - 2 days
  2. **security-scanner** (Worker) - 2 days
  3. **test-automation** (Worker) - 2 days
  4. **documentation-generator** (Worker) - 2 days
  5. **architecture-advisor** (Worker + QC) - 2 days
- Each includes: manifest, skills, documentation
- **Deliverable**: 3-5 working plugins in catalog

**WP4.5: Plugin Testing** (2 days)
- Test plugin installation
- Test plugin discovery
- Test plugin activation
- **Deliverable**: Plugin test suite

#### Quality Gate 4: Marketplace Validation
- [ ] Plugins can be installed via slash command
- [ ] Plugin discovery recommends relevant plugins
- [ ] Plugin catalog persists across sessions
- [ ] At least 3 plugins fully functional
- [ ] Plugin documentation complete

**Estimated Duration**: 21 days (4.2 weeks)
**Risk Buffer**: +3 days for plugin complexity

**Note**: WP4.4 can be parallelized if multiple developers/subagents available. Solo path: 21 days sequential. Team path: 14 days with 2-3 parallel plugin developers.

---

### Phase 5: Integration Testing (Weeks 7-8)

**Goal**: End-to-end validation and hardening

#### Work Packages

**WP5.1: End-to-End Workflow Tests** (3 days)
- Test: task → work → QC → merge
- Test: priority scoring → user interruption
- Test: plugin discovery → installation → usage
- Test: workstream isolation under load
- **Deliverable**: E2E test suite

**WP5.2: /chat Command** (2 days)
- Implement project status queries
- Show active workstreams
- Show pending QC reviews
- Show recent activity log
- **Deliverable**: `/chat` command functional

**WP5.3: /plugin-discover Skill** (2 days)
- Analyze current workflow phase
- Identify capability gaps
- Recommend relevant plugins
- **Deliverable**: Context-aware plugin recommendations

**WP5.4: Bug Fixing & Refinement** (5 days)
- Fix issues found in E2E testing
- Performance optimization
- UX improvements
- **Deliverable**: Bug-free baseline

#### Quality Gate 5: Integration Validation
- [ ] Full workflow executes without errors
- [ ] No file conflicts between workstreams
- [ ] QC workflow handles edge cases
- [ ] /chat provides accurate status
- [ ] Plugin discovery recommends appropriately
- [ ] All E2E tests pass

**Estimated Duration**: 12 days (2.4 weeks)
**Risk Buffer**: +2 days for bug fixing

---

### Phase 6: Context Management & Polish (Weeks 9-10)

**Goal**: Integrate continuous pruning and finalize MVP

#### Work Packages

**WP6.1: Context-Pruning-Lab Algorithm Validation** (Prerequisites)
- Algorithm must pass all experiments (weeks 1-6)
- Benchmarks confirm no degradation
- Performance acceptable (< 50ms per prune)
- **Deliverable**: Validated pruning algorithm

**WP6.2: Continuous Pruning Integration** (4 days)
- Implement `continuous_prune()` MCP tool
- Integrate with `before_compact` hook
- Port algorithm from context-pruning-lab
- **Deliverable**: Continuous pruning functional

**WP6.3: Performance Testing** (2 days)
- Test with 100+ interactions
- Measure context size over time
- Validate no degradation after 10+ prunes
- **Deliverable**: Performance test results

**WP6.4: Documentation** (3 days)
- User guide (installation, usage, slash commands)
- Developer guide (extending, adding plugins)
- API reference (MCP tools, hooks, skills)
- Troubleshooting guide
- **Deliverable**: Complete documentation set

**WP6.5: Final Testing & Polish** (2 days)
- Manual user acceptance testing
- Polish UX issues
- Final bug fixes
- **Deliverable**: Production-ready MVP

#### Quality Gate 6: Launch Readiness
- [ ] Context pruning prevents degradation over 100+ interactions
- [ ] Context utilization stays 20-30% (not 80%+)
- [ ] All documentation complete and accurate
- [ ] No critical bugs remaining
- [ ] Manual UAT passed by project owner
- [ ] Performance acceptable (no noticeable lag)

**Estimated Duration**: 11 days (2.2 weeks)
**Risk Buffer**: +2 days for algorithm integration

---

## Parallel Work Streams

### Track A: Context-Pruning-Lab (Weeks 1-6, Independent)

**Goal**: Validate continuous pruning algorithm

#### Milestones
- **Week 1-2**: Initial algorithm implementation
- **Week 3-4**: Experiments and benchmarking
- **Week 5-6**: Validation and optimization
- **Deadline**: Algorithm proven by end of Week 6

#### Deliverables
- ✅ ContinuousPruner class with 4-tier architecture
- ✅ Experiment 1: Linear growth prevention
- ✅ Experiment 2: Importance preservation
- ✅ Experiment 3: Steady-state oscillation
- ✅ Benchmarks showing < 50ms per prune
- ✅ Documentation of algorithm behavior

**Dependencies**: None (fully independent)
**Team**: Can be separate developer or subagent

---

### Track B: Plugin Development (Weeks 3-6, Parallelizable)

**Goal**: Implement 3-5 initial plugins

#### Approach
- **Solo**: Sequential development (2 days per plugin = 10 days)
- **Team**: Parallel development (2-3 developers, 4-5 days)
- **Subagents**: Parallel delegation (3-5 subagents, 3-4 days)

#### Plugin Priority Order
1. **code-reviewer-qc** (CRITICAL) - QC expert functionality
2. **test-automation** (HIGH) - Enables TDD workflow
3. **security-scanner** (HIGH) - Security validation
4. **documentation-generator** (MEDIUM) - Developer productivity
5. **architecture-advisor** (MEDIUM) - Design guidance

**Dependencies**: Requires plugin manifest spec (WP4.1)
**Team**: Can parallelize with 2-3 developers/subagents

---

### Track C: Documentation (Weeks 1-10, Incremental)

**Goal**: Comprehensive documentation for users and developers

#### Approach
Document incrementally at each phase:
- **Phase 1**: MCP server architecture, database schema
- **Phase 2**: Hook system, integration patterns
- **Phase 3**: QC workflow, prioritization logic
- **Phase 4**: Plugin system, manifest specification
- **Phase 5**: End-to-end workflows, troubleshooting
- **Phase 6**: Final polish, examples, guides

#### Deliverables
- README.md (user-facing)
- DEVELOPER.md (extending system)
- API_REFERENCE.md (MCP tools, hooks, skills)
- PLUGIN_DEVELOPMENT.md (creating plugins)
- ARCHITECTURE.md (system design, ADRs)
- TROUBLESHOOTING.md (common issues)

**Dependencies**: Follows implementation phases
**Team**: Can be separate technical writer or subagent

---

## Time Estimates & Confidence Levels

### Solo Developer Path (Conservative)

| Phase | Optimistic | Realistic | Pessimistic | Confidence |
|-------|-----------|-----------|-------------|------------|
| Phase 1: Foundation | 10 days | 15 days | 20 days | 70% |
| Phase 2: Core Integration | 9 days | 15 days | 21 days | 50% (HIGH RISK) |
| Phase 3: Workflows | 16 days | 20 days | 26 days | 70% |
| Phase 4: Marketplace | 18 days | 24 days | 30 days | 75% |
| Phase 5: Integration | 10 days | 14 days | 18 days | 80% |
| Phase 6: Context Mgmt | 9 days | 13 days | 17 days | 75% |
| **TOTAL** | **72 days** | **101 days** | **132 days** | **68%** |
| **In Weeks** | **14.4 weeks** | **20.2 weeks** | **26.4 weeks** | |

**Recommended Solo Timeline**: 16-18 weeks with buffer

---

### Team Path (2-3 Developers)

| Phase | Optimistic | Realistic | Pessimistic | Confidence |
|-------|-----------|-----------|-------------|------------|
| Phase 1: Foundation | 8 days | 10 days | 14 days | 75% |
| Phase 2: Core Integration | 7 days | 11 days | 15 days | 55% (HIGH RISK) |
| Phase 3: Workflows | 12 days | 16 days | 20 days | 75% |
| Phase 4: Marketplace (parallel) | 10 days | 14 days | 18 days | 80% |
| Phase 5: Integration | 8 days | 12 days | 15 days | 85% |
| Phase 6: Context Mgmt | 7 days | 11 days | 14 days | 80% |
| **TOTAL** | **52 days** | **74 days** | **96 days** | **75%** |
| **In Weeks** | **10.4 weeks** | **14.8 weeks** | **19.2 weeks** | |

**Recommended Team Timeline**: 12-14 weeks with buffer

---

## Risk Management

### High Risks

**R1: Hook System Integration Complexity**
- **Probability**: 60%
- **Impact**: 1-2 week delay
- **Mitigation**:
  - Research Claude Code hook examples early (Week 0)
  - Engage Claude Code team for support
  - Build hook testing harness
  - Allocate 4-day buffer in Phase 2
- **Contingency**: If hooks unavailable, use polling + file watchers (reduces elegance but maintains functionality)

**R2: Context Pruning Algorithm Ineffective**
- **Probability**: 30%
- **Impact**: 2-3 week delay
- **Mitigation**:
  - Separate validation in context-pruning-lab (Weeks 1-6)
  - Multiple experiments before integration
  - Deadline: Algorithm proven by Week 6
  - Weekly check-ins on algorithm progress
- **Contingency**: Fall back to batch compaction with manual flash saves (Phase II enhancement)

---

### Medium Risks

**R3: SQLite Concurrent Access Issues**
- **Probability**: 40%
- **Impact**: 3-5 days
- **Mitigation**:
  - Use WAL mode from day 1
  - Implement proper connection pooling
  - Stress test with concurrent operations
  - Have migration path to PostgreSQL ready
- **Contingency**: Switch to PostgreSQL (1 day migration effort)

**R4: Plugin Development Delays**
- **Probability**: 50%
- **Impact**: 1 week delay
- **Mitigation**:
  - Prioritize: code-reviewer-qc, test-automation, security-scanner
  - Start with 3 plugins minimum (not 5)
  - Add remaining plugins post-launch
  - Parallelize if team available
- **Contingency**: Launch with 3 plugins, add 2 more in Phase II

---

### Low Risks

**R5: MCP Protocol Compliance Issues**
- **Probability**: 20%
- **Impact**: 2-3 days
- **Mitigation**:
  - Use official MCP SDK if available
  - Reference existing MCP server implementations
  - Test with MCP Inspector tool
- **Contingency**: Manual debugging with MCP logs

---

## Quality Gates & Validation Criteria

### Automated Testing Requirements

| Component | Coverage Target | Test Types |
|-----------|----------------|------------|
| MCP Tools | 90%+ | Unit, integration |
| Database Operations | 95%+ | Unit, stress |
| Hooks | 80%+ | Integration, E2E |
| Skills | 70%+ | Integration, E2E |
| Plugins | 75%+ | Unit, integration |

### Manual Validation Checklist

**User Experience**:
- [ ] Slash commands respond within 2 seconds
- [ ] Error messages are clear and actionable
- [ ] QC workflow feels natural and not intrusive
- [ ] Plugin discovery surfaces relevant recommendations
- [ ] /chat provides meaningful project status

**Performance**:
- [ ] Hook overhead < 100ms per call
- [ ] Context pruning < 50ms per prune
- [ ] Database queries < 50ms (95th percentile)
- [ ] Plugin installation < 5 seconds
- [ ] No memory leaks over 8-hour session

**Reliability**:
- [ ] No crashes during 100-interaction stress test
- [ ] Graceful degradation if MCP server unavailable
- [ ] State recovery after Claude Code restart
- [ ] No data corruption under concurrent access
- [ ] Workstreams cleaned up properly after completion

---

## Success Criteria for MVP Launch

### Minimum Viable Product Requirements

**Core Functionality**:
1. ✅ MCP server manages all state reliably
2. ✅ Hooks trigger at correct lifecycle points without errors
3. ✅ Worker → QC → Coordinator workflow completes successfully
4. ✅ Priority scoring routes decisions appropriately (P0/P1/P2/P3)
5. ✅ Workstream isolation prevents file conflicts
6. ✅ At least 3 plugins functional in marketplace
7. ✅ Continuous context pruning prevents degradation over 100+ interactions

**Quality Standards**:
1. ✅ 80%+ test coverage on critical paths
2. ✅ Zero critical bugs (P0)
3. ✅ < 5 known P1 bugs (documented, not blocking)
4. ✅ Performance targets met (hooks < 100ms, pruning < 50ms)
5. ✅ Documentation complete (user + developer guides)

**User Validation**:
1. ✅ Project owner successfully completes end-to-end workflow
2. ✅ QC review feels natural and adds value
3. ✅ Plugin discovery surfaces relevant recommendations
4. ✅ No showstopper UX issues

**Technical Validation**:
1. ✅ MCP server passes protocol compliance tests
2. ✅ Hooks don't interfere with Claude Code core functionality
3. ✅ Database handles concurrent access without corruption
4. ✅ Context pruning maintains agent capability over time

---

## Execution Strategy

### Solo Developer Approach

**Timeline**: 16-18 weeks

**Focus**: Sequential execution of critical path with strategic parallelization

**Week-by-Week**:
- **Weeks 1-2**: Foundation (MCP + Database)
- **Weeks 3-4**: Core Integration (Hooks) + Start context-pruning-lab
- **Weeks 5-7**: Workflows (QC, Priority) + Finish context-pruning-lab
- **Weeks 8-11**: Marketplace (Plugins, sequential implementation)
- **Weeks 12-13**: Integration Testing
- **Weeks 14-16**: Context Management + Polish
- **Weeks 17-18**: Buffer for unknowns

**Key Strategies**:
- Use subagents for documentation (parallel track)
- Use subagents for plugin scaffolding (parallel track)
- Defer nice-to-haves to post-launch
- Focus on 3 plugins minimum (not 5)
- Weekly self-review against quality gates

---

### Team Approach (2-3 Developers)

**Timeline**: 12-14 weeks

**Focus**: Parallel execution of independent work streams

**Team Structure**:
- **Developer A**: MCP Backend + Database + Context Pruning
- **Developer B**: Hooks + Skills + QC Workflow
- **Developer C**: Plugins + Documentation + Testing

**Week-by-Week**:
- **Weeks 1-2**: Foundation (A: MCP, B: Research hooks, C: Plugin design)
- **Weeks 3-4**: Integration (A: Database, B: Hooks, C: Context-pruning-lab)
- **Weeks 5-6**: Workflows (A: Priority scoring, B: QC skills, C: Plugins 1-3)
- **Weeks 7-8**: Marketplace (A: Catalog, B: Slash commands, C: Plugins 4-5)
- **Weeks 9-10**: Integration (A+B+C: E2E testing)
- **Weeks 11-12**: Context Mgmt (A: Pruning integration, B: Testing, C: Docs)
- **Weeks 13-14**: Buffer for unknowns

**Key Strategies**:
- Daily standup to sync dependencies
- Shared MCP protocol contracts defined early
- Integration testing starts Week 5 (continuous)
- Rotate who handles bug fixes weekly
- Code reviews required for all PRs

---

## Documentation Plan

### User Documentation (Weeks 1-10, Incremental)

**README.md** (Updated Weekly)
- Installation instructions
- Quick start guide
- Basic usage examples
- FAQ

**USER_GUIDE.md** (Week 9-10)
- Detailed slash command reference
- Workflow examples
- QC review process
- Plugin management
- Troubleshooting

**PLUGIN_CATALOG.md** (Week 6-7)
- Available plugins
- Plugin descriptions
- Installation instructions
- Usage examples

---

### Developer Documentation (Weeks 1-10, Incremental)

**DEVELOPER.md** (Week 9-10)
- Architecture overview
- Setting up development environment
- Running tests
- Debugging techniques
- Contributing guidelines

**API_REFERENCE.md** (Week 9-10)
- MCP tools specification
- Hook system API
- Skills framework
- Database schema reference

**PLUGIN_DEVELOPMENT.md** (Week 6-7)
- Plugin manifest specification
- Creating a plugin from scratch
- Testing plugins
- Publishing to marketplace

**ARCHITECTURE.md** (Week 9-10)
- System design overview
- Component interactions
- Data flow diagrams
- Architecture decision records (ADRs)

---

## Post-Launch Activities

### Week 11+: Phase II Planning

**Objectives**:
1. Gather user feedback from Phase I usage
2. Prioritize Phase II features based on user needs
3. Research multi-instance coordination patterns
4. Begin federated marketplace design
5. Explore automated vetting approaches

**Key Questions to Answer**:
- Is continuous context pruning effective in practice?
- Which plugins get the most usage?
- What additional plugins are most requested?
- Are there performance bottlenecks?
- What UX improvements are needed?

---

## Appendix A: Detailed Task List

### Phase 1: Foundation

**Task 1.1**: Set up Node.js/TypeScript project
- [ ] Initialize npm project
- [ ] Configure TypeScript
- [ ] Set up ESLint + Prettier
- [ ] Configure build scripts

**Task 1.2**: Implement MCP server skeleton
- [ ] Install MCP SDK or implement protocol
- [ ] Create server initialization
- [ ] Implement basic request/response handling
- [ ] Add logging framework

**Task 1.3**: Create SQLite database
- [ ] Define schema for core tables
- [ ] Implement migration system
- [ ] Create connection pooling
- [ ] Add query helper functions

**Task 1.4**: Implement MCP tools
- [ ] `get_next_task()` stub
- [ ] `submit_for_qc()` stub
- [ ] `score_priority()` stub
- [ ] Error handling for all tools

**Task 1.5**: Implement MCP resources
- [ ] `project_state` resource
- [ ] `task_queue` resource
- [ ] Resource update mechanisms

**Task 1.6**: Write unit tests
- [ ] Database tests
- [ ] MCP tool tests
- [ ] Resource tests
- [ ] Set up CI pipeline

---

### Phase 2: Core Integration

**Task 2.1**: Research Claude Code hooks
- [ ] Read Claude Code hook documentation
- [ ] Examine example hooks
- [ ] Understand hook lifecycle
- [ ] Identify constraints

**Task 2.2**: Implement hook infrastructure
- [ ] Create hook registration system
- [ ] Implement `before_compact.js`
- [ ] Implement `after_tool_use.js`
- [ ] Implement `user_prompt_submit.js`

**Task 2.3**: Connect hooks to MCP
- [ ] Hook → MCP tool communication
- [ ] Error handling and fallbacks
- [ ] Logging and debugging
- [ ] Performance monitoring

**Task 2.4**: Implement state management
- [ ] Workstream creation/cleanup
- [ ] Task state transitions
- [ ] Database transactions

**Task 2.5**: Write integration tests
- [ ] Test hooks with real Claude Code
- [ ] Test error conditions
- [ ] Performance tests

---

### Phase 3: Workflows

**Task 3.1**: Implement `/qc-review` skill
- [ ] Worker submission workflow
- [ ] QC expert review workflow
- [ ] Coordinator approval workflow
- [ ] Feedback mechanisms

**Task 3.2**: Integrate with `after_tool_use` hook
- [ ] Detect git commit operations
- [ ] Trigger QC automatically
- [ ] Handle QC responses

**Task 3.3**: Implement priority scoring
- [ ] P0 pattern database
- [ ] Decision analysis algorithm
- [ ] User trust level tracking
- [ ] Learning from corrections

**Task 3.4**: Implement workstream isolation
- [ ] Git branch creation via Bash
- [ ] Workstream directory structure
- [ ] Workstream cleanup
- [ ] State tracking in database

**Task 3.5**: Create sample QC rubric
- [ ] Define rubric template
- [ ] Implement scoring algorithm
- [ ] Add customization support

---

### Phase 4: Marketplace

**Task 4.1**: Finalize plugin manifest spec
- [ ] Define `plugin.json` schema
- [ ] Create manifest validator
- [ ] Write specification document

**Task 4.2**: Implement plugin catalog
- [ ] `plugin_catalog` table operations
- [ ] Plugin discovery algorithm
- [ ] Context-aware recommendations

**Task 4.3**: Implement slash commands
- [ ] `/plugin-install`
- [ ] `/plugin-search`
- [ ] `/plugin-list`
- [ ] `/plugin-info`

**Task 4.4**: Implement initial plugins
- [ ] code-reviewer-qc plugin
- [ ] security-scanner plugin
- [ ] test-automation plugin
- [ ] documentation-generator plugin (optional)
- [ ] architecture-advisor plugin (optional)

**Task 4.5**: Test plugin system
- [ ] Installation tests
- [ ] Discovery tests
- [ ] Activation tests

---

### Phase 5: Integration Testing

**Task 5.1**: Write E2E tests
- [ ] Task → work → QC → merge
- [ ] Priority scoring → interruption
- [ ] Plugin discovery → install → use
- [ ] Workstream isolation under load

**Task 5.2**: Implement `/chat` command
- [ ] Project status queries
- [ ] Active workstreams display
- [ ] Pending QC reviews display
- [ ] Recent activity log

**Task 5.3**: Implement `/plugin-discover` skill
- [ ] Analyze workflow phase
- [ ] Identify capability gaps
- [ ] Recommend plugins

**Task 5.4**: Bug fixing
- [ ] Fix issues from E2E tests
- [ ] Performance optimization
- [ ] UX improvements

---

### Phase 6: Context Management & Polish

**Task 6.1**: Validate context-pruning-lab algorithm
- [ ] All experiments pass
- [ ] Benchmarks acceptable
- [ ] Performance < 50ms per prune

**Task 6.2**: Integrate continuous pruning
- [ ] Implement `continuous_prune()` MCP tool
- [ ] Integrate with `before_compact` hook
- [ ] Port algorithm from context-pruning-lab

**Task 6.3**: Performance testing
- [ ] Test with 100+ interactions
- [ ] Measure context size over time
- [ ] Validate no degradation

**Task 6.4**: Write documentation
- [ ] User guide
- [ ] Developer guide
- [ ] API reference
- [ ] Troubleshooting guide

**Task 6.5**: Final testing & polish
- [ ] Manual UAT
- [ ] UX polish
- [ ] Final bug fixes

---

## Appendix B: Database Schema Quick Reference

### Core Tables (Phase 1)

```sql
-- Workstreams (agent isolation)
CREATE TABLE workstreams (
    id INTEGER PRIMARY KEY,
    agent_id TEXT NOT NULL,
    branch_name TEXT NOT NULL,
    working_dir TEXT NOT NULL,
    status TEXT CHECK(status IN ('active', 'merged', 'abandoned')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Internal PRs (QC workflow)
CREATE TABLE internal_prs (
    id INTEGER PRIMARY KEY,
    workstream_id INTEGER NOT NULL,
    worker_id TEXT NOT NULL,
    qc_agent_id TEXT,
    coordinator_id TEXT,
    status TEXT CHECK(status IN ('submitted', 'in_review', 'approved', 'rejected', 'merged')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (workstream_id) REFERENCES workstreams(id)
);

-- QC Reviews
CREATE TABLE qc_reviews (
    id INTEGER PRIMARY KEY,
    pr_id INTEGER NOT NULL,
    reviewer_id TEXT NOT NULL,
    score REAL,
    feedback TEXT,
    approved BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (pr_id) REFERENCES internal_prs(id)
);

-- Priority Patterns
CREATE TABLE p0_patterns (
    id INTEGER PRIMARY KEY,
    pattern_type TEXT CHECK(pattern_type IN ('regex', 'keyword', 'category')),
    pattern_value TEXT NOT NULL,
    description TEXT,
    confidence REAL DEFAULT 0.5
);

-- User Input Queue
CREATE TABLE user_input_queue (
    id INTEGER PRIMARY KEY,
    priority TEXT CHECK(priority IN ('P0', 'P1', 'P2', 'P3')),
    question TEXT NOT NULL,
    context TEXT,
    asked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    answered_at TIMESTAMP,
    answer TEXT
);
```

### Plugin Tables (Phase 4)

```sql
-- Plugin Catalog
CREATE TABLE plugin_catalog (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    description TEXT,
    agent_roles TEXT,
    workflow_phases TEXT,
    manifest_json TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Installed Plugins
CREATE TABLE installed_plugins (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    installed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Appendix C: MCP Tools Quick Reference

### Core Tools (Phase 1)

**`get_next_task()`**
- Purpose: Retrieve next task from queue
- Parameters: `agent_id` (string)
- Returns: `{ task_id, description, priority, context }`

**`submit_for_qc()`**
- Purpose: Submit work for QC review
- Parameters: `{ workstream_id, description, files_changed }`
- Returns: `{ pr_id, status }`

**`score_priority()`**
- Purpose: Score decision priority
- Parameters: `{ decision_context, question }`
- Returns: `{ priority: 'P0'|'P1'|'P2'|'P3', confidence }`

### Advanced Tools (Phase 6)

**`continuous_prune()`**
- Purpose: Prune context based on importance
- Parameters: `{ context_items, target_size }`
- Returns: `{ pruned_items, tokens_removed }`

---

## Appendix D: Skills Quick Reference

### Core Skills (Phase 3)

**`/qc-review`**
- Purpose: Initiate QC review workflow
- Triggers: Automatic (after git commit) or Manual
- Workflow: Worker submit → QC review → Coordinator approval

### User Skills (Phase 5)

**`/chat`**
- Purpose: Query project status
- Commands:
  - `/chat status` - Overall project health
  - `/chat workstreams` - Active workstreams
  - `/chat qc` - Pending QC reviews
  - `/chat activity` - Recent activity log

### Marketplace Skills (Phase 4)

**`/plugin-install [plugin-id]`**
- Purpose: Install plugin from marketplace

**`/plugin-search [query]`**
- Purpose: Search marketplace for plugins

**`/plugin-list`**
- Purpose: List installed plugins

**`/plugin-info [plugin-id]`**
- Purpose: Show plugin details

**`/plugin-discover`**
- Purpose: Context-aware plugin recommendations

---

## Document Status

**Version**: 1.0
**Status**: Ready for Execution
**Next Review**: After Phase 1 completion (Week 2)

**Changelog**:
- 2025-10-29: Initial comprehensive implementation plan created
  - 6 phases with detailed work packages
  - Dependency graph and critical path analysis
  - Solo vs team execution paths
  - 3 parallel work tracks identified
  - Risk management strategies
  - Quality gates and validation criteria
  - Time estimates with confidence intervals
  - Success criteria for MVP launch

---

**END OF IMPLEMENTATION PLAN**
