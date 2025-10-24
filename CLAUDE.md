# AI Workflow Starter - Claude Code Configuration

## Mission

This project uses AI agents and structured workflows to take feature ideas from concept through comprehensive planning documentation, following best practices for context engineering and test-driven development.

## Project Overview

**Current Phase:** Phase 1 - Planning & Documentation
**Template Version:** 1.0.0
**Status:** Production Ready

This is a reusable template for AI-assisted software development. Clone this repository to start any new project with structured workflows, specialized agents, and automated documentation.

## Core Principles

1. **Structured Planning First:** Every feature goes through exploration → research → planning before implementation
2. **Documentation as Code:** All decisions, architecture, and acceptance criteria are documented and version-controlled
3. **Test-Driven Ready:** Test strategies and stubs created during planning, implementation follows
4. **Context Efficiency:** Just-in-time loading, minimal memory footprint, agent specialization
5. **Progressive Enhancement:** Works standalone, enhanced by optional MCPs (Archon, etc.)

## Agent System

This project uses 5 specialized sub-agents for planning and documentation, plus the main Claude Code agent for implementation (Phase 2).

### Active Sub-Agents (Phase 1)

@.claude/subagents/explorer.md
@.claude/subagents/researcher.md
@.claude/subagents/planner.md
@.claude/subagents/documenter.md
@.claude/subagents/reviewer.md

**Note:** Sub-agents focus on planning, research, and documentation. The main Claude Code agent handles all code implementation, guided by planning documents created by sub-agents.

## Workflow Commands

Use these slash commands to orchestrate multi-agent workflows:

- `/explore [topic]` - Explore new feature idea, create initial PRD
- `/plan [FEAT-ID]` - Create comprehensive planning documentation
- `/update-docs` - Maintain documentation index and cross-references

*Phase 2 commands: `/build`, `/test`, `/commit` (coming soon)*

## Documentation Structure

All documentation lives in `docs/` with automatic indexing:

- **docs/templates/** - Document templates for agents to follow
- **docs/system/** - Technical architecture (architecture, database, API, integrations, stack)
- **docs/sop/** - Standard Operating Procedures (git workflow, testing, code style, lessons learned)
- **docs/features/** - Feature-specific docs (one folder per feature: FEAT-XXX/)
- **docs/archive/** - Completed or deprecated documentation

**Key files:**
- `docs/README.md` - Auto-updated index of all documentation
- `AC.md` - Global acceptance criteria (append-only)
- `CHANGELOG.md` - Conventional changelog (auto-updated)

## Development Workflow (Phase 1)

```
User has feature idea
  ↓
/explore [topic]
  ↓ Explorer agent asks clarifying questions
  ↓ Researcher agent conducts research (optional Archon, WebSearch)
  ↓ Creates docs/features/FEAT-XXX/prd.md and research.md
  ↓
User reviews and refines PRD
  ↓
/plan FEAT-XXX
  ↓ Planner agent orchestrates planning workflow
  ↓ Creates architecture.md, acceptance.md, testing.md, manual-test.md
  ↓ Generates test stubs in tests/ directory
  ↓ Appends acceptance criteria to AC.md
  ↓ Reviewer agent validates completeness
  ↓
/update-docs
  ↓ Documenter agent updates docs/README.md index
  ↓ Updates CHANGELOG.md with planning entry
  ↓
Ready for Phase 2 implementation
```

## Test-Driven Development Approach

Phase 1 prepares for TDD:
1. **Test Strategy:** Documented in `docs/features/FEAT-XXX/testing.md`
2. **Test Stubs:** Empty test functions created in `tests/` with TODO comments
3. **Acceptance Criteria:** Clear pass/fail conditions in `AC.md`
4. **Manual Testing:** Human testing checklist in `manual-test.md`

Phase 2 will implement tests first, then code to pass them.

## Archon Integration (Optional)

This template works standalone but can be enhanced with the Archon MCP for knowledge management.

### If Archon MCP is Available:

**Researcher Agent Benefits:**
- Access pre-crawled framework documentation via Archon RAG
- **Targeted search**: Query specific documentation sources (e.g., only React docs)
- **Code discovery**: Find implementation examples alongside conceptual docs
- **Page browsing**: Systematically explore documentation structure
- **Full-text retrieval**: Read complete pages for deep understanding
- **Short query optimization**: 2-5 keyword queries get best results
- Faster research with locally cached knowledge base
- Consistent technical context across all research sessions
- Automatic source tracking and citation

**RAG Workflow Example:**
```
1. Get sources → Find "FastAPI Documentation" (id: src_123)
2. Search → rag_search_knowledge_base("dependency injection", "src_123")
3. Code → rag_search_code_examples("Depends", "src_123")
4. Deep dive → rag_read_full_page(page_id from results)
5. Fallback → WebSearch only for gaps
```

**To Enable Archon:**
1. Install and configure Archon MCP in Claude Code settings
2. Crawl relevant framework docs to Archon knowledge base
3. Researcher agent will automatically detect and use Archon
4. Falls back to WebSearch if Archon unavailable

### Without Archon:

Researcher agent uses WebSearch exclusively - fully functional workflow.

## Archon Task Management (Optional)

This template can integrate with Archon MCP for task tracking alongside git-based documentation.

### Task Management Philosophy

**Git = Source of Truth** for all documentation and code
**Archon = Tracker** for workflow progress and context

### One Project Per Repository

- Single Archon project represents this repository
- All features create tasks under one project
- Project name matches repository folder name
- Feature tasks use `feature` field for grouping

### Task Structure: Feature as Epic

Each FEAT-XXX is an epic with 4 workflow-phase tasks:

1. **Explore & Research** - Explorer + Researcher agents
   - Status: `done` after `/explore` completes
   - Links to: prd.md, research.md

2. **Plan Architecture & Tests** - Planner + Reviewer agents
   - Status: `done` after `/plan` completes
   - Links to: architecture.md, acceptance.md, testing.md, manual-test.md

3. **Build with TDD** - Main Claude agent following architecture
   - Status: `doing` during `/build`, `review` after
   - Links to: implementation files + test files

4. **Validate & Commit** - Main Claude agent testing + git workflow
   - Status: `doing` during `/test` + `/commit`, `done` after merge
   - Links to: test results + PR/commit

### Task Lifecycle

```
/explore FEAT-XXX
→ Create 4 tasks in Archon
→ Mark task 1 "done" (exploration complete)

/plan FEAT-XXX
→ Mark task 2 "done" (planning complete)

/build FEAT-XXX
→ Mark task 3 "doing" (building)
→ Mark task 3 "review" (build complete, needs testing)

/test FEAT-XXX
→ Validate acceptance criteria

/commit "message"
→ Mark task 4 "done" (feature complete)
```

### When Archon Unavailable

All workflows function identically without Archon:
- Git remains source of truth
- No task tracking, manual project management instead
- Graceful degradation with informational warnings

### Future Archon Features (Phase 3)

- Bidirectional sync: Start from Archon task → trigger workflows
- Session state persistence via Archon memory
- Collaborative knowledge building across sessions

## Context Management

### Memory Hierarchy
1. **CLAUDE.md** (this file) - Core orchestration, agent imports
2. **Subagents** - Specialized agent instructions (@imported above)
3. **Documentation** - JIT loading via Read tool when needed
4. **SOPs** - Referenced by agents for standards enforcement

### Session Recovery
The PreCompact hook (`.claude/hooks/pre_compact.py`) automatically saves session state before context compaction, including:
- Current feature being worked on
- Last agent invoked
- Pending tasks
- Links to key files

## Quality Gates

Every workflow includes validation:

1. **Reviewer Agent:** Validates all required sections exist
2. **Template Compliance:** Docs follow templates in `docs/templates/`
3. **SOP Enforcement:** Agents follow procedures in `docs/sop/`
4. **Completeness Checks:** No TODOs left in planning docs

## Standard Operating Procedures

Agents enforce SOPs documented in `docs/sop/`:
- **git-workflow.md** - Branching, commits, PRs
- **testing-strategy.md** - How we test
- **code-style.md** - Linting, formatting
- **mistakes.md** - Lessons learned, gotchas
- **github-setup.md** - Repository setup

## Project Customization

### For New Projects:
1. Clone this repository
2. Update this CLAUDE.md with project-specific details
3. Customize `docs/system/` with your architecture
4. Add project-specific SOPs to `docs/sop/`
5. Run `/explore` with your first feature

### Stack-Specific Profiles:
This template is generic. For stack-specific enhancements (TypeScript/React, Python/FastAPI, etc.), see `docs/future-enhancements.md`.

## Phase Roadmap

**Phase 1 (Current):** Planning & Documentation ✅
- Feature exploration and research
- Comprehensive planning documentation
- Test strategy and stubs
- Documentation maintenance

**Phase 2 (Next):** Implementation & Testing
- Main Claude Code agent implements features based on planning docs
- Test-first implementation (TDD Red-Green-Refactor cycle)
- Automated formatting and linting hooks
- Git workflow automation via slash commands

**Phase 3 (Future):** Automation & Profiles
- Full Archon integration
- Stack-specific profiles
- CI/CD integration
- Advanced hooks and automation

## Rules & Guardrails

1. **Phase 1 Agents:** Write ONLY to `docs/**`, `tests/**` (stubs only), and `AC.md`
2. **No Implementation:** Phase 1 agents must not write implementation code
3. **Template Compliance:** All feature docs must follow templates in `docs/templates/`
4. **Quality Gates:** Reviewer agent must validate before workflow completes
5. **Documentation Sync:** Documenter agent maintains docs/README.md index
6. **SOP Enforcement:** Agents must reference and follow SOPs

## Getting Help

- **Template Documentation:** See `docs/README.md` for complete documentation index
- **Workflow Examples:** See `docs/features/FEAT-001_example/` for sample workflow
- **Future Features:** See `docs/future-enhancements.md` for roadmap
- **Troubleshooting:** See `docs/sop/mistakes.md` for common issues

---

**Note:** This is Phase 1 (Planning & Documentation). Implementation agents, git automation, and advanced hooks come in Phase 2.
