# Product Requirements Document: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Status:** Exploring
**Created:** 2025-10-25
**Last Updated:** 2025-10-25

## Problem Statement

The current agent system relies on five fixed, general-purpose sub-agents (Explorer, Researcher, Planner, Documenter, Reviewer). When projects use specialized technologies like Supabase, PydanticAI, or FastAPI, agents must either conduct broad research every session or maintain expertise redundantly across multiple workflow runs. This creates inefficiency, increases context usage, and lacks domain-specific depth for framework-specific decisions. We need a systematic way to create, persist, and invoke specialist sub-agents that accumulate and reuse technology-specific expertise per project.

## Goals & Success Criteria

### Primary Goals
- Enable automatic suggestion and creation of specialist sub-agents during feature exploration when libraries/frameworks are mentioned
- Provide template-based specialist creation with comprehensive auto-population of agent instructions
- Allow specialists to be invoked as subordinates by Explorer and Researcher agents during workflows
- Persist specialists per-project in .claude/subagents/ for reuse across features

### Success Metrics
- Specialist sub-agents can be created in <2 minutes with user confirmation
- Specialists successfully answer domain-specific questions using Archon RAG → WebSearch → User input cascade
- Documentation index automatically reflects active specialists in project
- Test case: Successfully create and invoke a Database Specialist for Postgres/Supabase

## User Stories

### Story 1: Automatic Specialist Suggestion During Exploration
**As a** user running /explore for a new feature
**I want** the Explorer agent to detect mentioned libraries/frameworks and suggest creating specialists
**So that** I can build domain expertise incrementally without manual setup

**Acceptance Criteria:**
- [ ] Explorer agent detects library/framework mentions in user responses during Q&A
- [ ] After PRD creation, before invoking Researcher, Explorer suggests creating relevant specialists
- [ ] Suggestion includes scope choice (broad like "Database" vs narrow like "Supabase")
- [ ] User can confirm, skip, or defer specialist creation

### Story 2: Template-Based Specialist Creation
**As a** user confirming specialist creation
**I want** the system to auto-populate specialist agent instructions using TEMPLATE.md
**So that** specialists are comprehensive and follow agent architecture patterns

**Acceptance Criteria:**
- [ ] System uses .claude/subagents/TEMPLATE.md as base structure
- [ ] Auto-populates name, description, tools, and primary objective based on library/framework
- [ ] Creates file at .claude/subagents/[library-name]-specialist.md with kebab-case naming
- [ ] Specialist includes Archon RAG → WebSearch → User input cascade for knowledge gathering
- [ ] User can review and edit specialist before finalizing

### Story 3: Specialist Invocation by Main Agents
**As an** Explorer or Researcher agent
**I want** to invoke specialist sub-agents for domain-specific questions
**So that** I can leverage accumulated expertise without redundant research

**Acceptance Criteria:**
- [ ] Explorer can invoke specialists during discovery phase for clarifying questions
- [ ] Researcher can invoke specialists during research phase for deep-dive investigations
- [ ] Specialists are called using Task tool with @.claude/subagents/[name]-specialist.md reference
- [ ] Specialists return findings to calling agent (not directly to user)
- [ ] Main workflow agents can also invoke specialists directly if needed (hybrid pattern)

### Story 4: Specialist Knowledge Cascade
**As a** specialist sub-agent answering a question
**I want** to try Archon RAG first, fall back to WebSearch, then ask user
**So that** I prioritize local knowledge, leverage web when needed, and only interrupt user as last resort

**Acceptance Criteria:**
- [ ] Specialist checks Archon MCP availability and queries relevant sources first
- [ ] If Archon lacks coverage, specialist uses WebSearch with targeted queries
- [ ] If both sources insufficient, specialist formulates specific question for user
- [ ] Knowledge source is documented in specialist's response (RAG/Web/User)
- [ ] Specialist documents findings for future reference

### Story 5: Documentation Integration
**As a** user managing project documentation
**I want** active specialists to appear in docs/README.md index
**So that** I can track which domain expertise exists in the project

**Acceptance Criteria:**
- [ ] Documenter agent detects specialists in .claude/subagents/ directory
- [ ] docs/README.md includes "Active Specialists" section listing all specialists
- [ ] Each specialist entry shows name, scope, and creation date
- [ ] Specialist count shown in documentation index header

## Scope & Non-Goals

### In Scope
- Automatic suggestion during /explore after PRD creation, before Researcher invocation
- Template-based specialist creation with comprehensive auto-population
- User confirmation and scope choice (broad vs narrow) for specialist creation
- Specialist invocation by Explorer and Researcher as subordinates
- Hybrid invocation pattern (main workflow can also call specialists directly)
- Specialist file storage in .claude/subagents/[library-name]-specialist.md
- Archon RAG → WebSearch → User input knowledge cascade
- Documentation index integration showing active specialists
- Test case: Database Specialist for Postgres/Supabase

### Out of Scope (For This Version)
- Specialist-to-specialist invocation (future enhancement)
- Auto-updating specialists when library versions change
- Analytics/tracking of specialist usage frequency
- Cross-project specialist sharing or templates
- Dedicated /create-specialist slash command (only auto-suggestion in /explore)
- Specialist deprecation or archival workflow
- Specialist performance metrics or quality scoring

## Constraints & Assumptions

### Technical Constraints
- Must integrate with existing /explore workflow without breaking current functionality
- Specialist files must follow .claude/subagents/TEMPLATE.md structure for consistency
- Must work with or without Archon MCP (graceful degradation)
- Specialist invocation must use existing Task tool mechanism
- File naming must be unique to avoid collisions (kebab-case library names)

### Business Constraints
- Creation process must complete in <2 minutes to avoid workflow interruption
- User must be able to skip specialist creation without blocking /explore
- Specialists must be immediately usable after creation (no compilation/setup)

### Assumptions
- Users will mention libraries/frameworks explicitly during Explorer Q&A phase
- TEMPLATE.md provides sufficient structure for specialist agent instructions
- Archon MCP may or may not be available (must handle both cases)
- Documenter agent can be extended to detect and index specialists
- Specialists will be invoked <5 times per feature on average

## Open Questions

1. **What patterns exist for dynamic agent creation in AI workflow systems?**
   - Why it matters: Informs best practices for template population and validation
   - Who can answer: Research (AI agent systems, LangChain patterns, AutoGen)

2. **How should specialists handle library version specificity?**
   - Why it matters: Determines if specialists document version compatibility or stay version-agnostic
   - Who can answer: Research + User decision (trade-off between precision and maintenance)

3. **What is the optimal template population strategy?**
   - Why it matters: Static template vs. research-based auto-population affects creation time and quality
   - Who can answer: Research (template systems, code generation patterns)

4. **How should library/framework detection work during Explorer Q&A?**
   - Why it matters: Determines trigger mechanism reliability (regex, NLP, keyword matching)
   - Who can answer: Research (entity extraction, keyword detection libraries)

5. **Should specialists accumulate knowledge across invocations?**
   - Why it matters: Affects specialist file updates and knowledge persistence strategy
   - Who can answer: Research + User decision (stateless vs. stateful agents)

6. **What scope granularity works best for specialists?**
   - Why it matters: Determines whether "Database Specialist" or "Supabase Specialist" is optimal
   - Who can answer: User testing + Research (agent specialization vs. generalization trade-offs)

## Related Context

### Existing Features
- **FEAT-001 Example**: Demonstrates existing agent workflow patterns
- **Explorer Agent (.claude/subagents/explorer.md)**: Will be extended to suggest specialist creation
- **Researcher Agent (.claude/subagents/researcher.md)**: Will invoke specialists for deep-dive research
- **/explore Command (.claude/commands/explore.md)**: Workflow trigger where specialists are suggested
- **TEMPLATE.md (.claude/subagents/TEMPLATE.md)**: Base template for specialist creation

### References
- [Agent System Architecture](../../docs/system/architecture.md) - Current agent orchestration patterns
- [Archon Integration](../../docs/system/integrations.md) - RAG knowledge cascade approach
- [Git Workflow SOP](../../docs/sop/git-workflow.md) - File creation and commit standards

---

**Next Steps:**
1. Researcher agent will investigate open questions
2. After research completes, proceed to planning with `/plan FEAT-003`
