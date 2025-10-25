# CHANGELOG

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **Phase 1.5: Archon MCP Integration** ✅
  - RAG knowledge base integration for Researcher agent
  - 5 modern RAG tools (get_sources, search_kb, search_code, list_pages, read_page)
  - Task management across all workflow commands (/explore, /plan, /build, /test, /commit)
  - One Archon project per repository model
  - Feature-as-epic: 4 workflow-phase tasks per FEAT-XXX
  - Graceful degradation when Archon unavailable
- **FEAT-003: Specialist Sub-Agent Creation System** ✅ - Implementation and testing complete ([docs](docs/features/FEAT-003_specialist_subagent_creation/))
  - **Slash Command:** `/create-specialist [library-name] [scope]` for creating specialist sub-agents
  - **Specialist Creator Agent:** Auto-populates specialists in <2 minutes (template + research)
  - **Explorer Enhancement:** Library detection and specialist suggestion after PRD creation
  - **Researcher Enhancement:** Specialist invocation for targeted domain research
  - **TEMPLATE.md Enhancement:** Specialist creation guidance and conventions
  - **Test Report:** 28/28 automated tests passed (100% coverage) - [view report](docs/features/FEAT-003_specialist_subagent_creation/test-report.md)
  - Knowledge cascade: Archon RAG → WebSearch → User input
  - 24 acceptance criteria met across creation, integration, and workflow
  - Documentation-based implementation (markdown files only, Phase 1 compliant)
  - Ready for manual runtime testing

### Changed
- Researcher agent updated from deprecated `mcp__archon__search` to modern RAG tools
- Framework-agnostic examples (no specific tool bias)
- CLAUDE.md expanded with Archon integration documentation
- All slash commands now update Archon task status
- future-enhancements.md updated to reflect Phase 1.5 completion

### Phase Roadmap
- Phase 1 (Complete): Planning & Documentation ✅
- **Phase 1.5 (Complete): Archon Integration** ✅
- Phase 2 (Next): Implementation workflow (main agent), testing automation, git workflow
- Phase 3 (Future): Enhanced Archon (bidirectional sync), stack profiles, advanced automation

## [1.0.1] - 2025-10-25

### Added
- Archon MCP RAG integration for knowledge base searches
- Archon task management across workflow
- Targeted documentation search by source
- Code example discovery capabilities
- Task lifecycle tracking (todo → doing → review → done)

### Changed
- Researcher agent tools updated to modern RAG API
- Examples made framework-agnostic
- Documentation index updated with Archon features

## [1.0.0] - 2025-10-24

### Added
- Initial AI Workflow Starter template v1.0.0
- Phase 1: Planning & Documentation system
- 5 specialized sub-agents (Explorer, Researcher, Planner, Documenter, Reviewer)
- 6 slash commands (3 active: /explore, /plan, /update-docs + 3 Phase 2 stubs: /build, /test, /commit)
- PreCompact hook for session state recovery
- 6 documentation templates (PRD, research, architecture, acceptance, testing, manual-test)
- 6 Standard Operating Procedures (SOPs)
- 5 system documentation files
- FEAT-001 example feature demonstrating complete workflow
- Comprehensive README and documentation index
- Test-driven development support with test stub generation

## [1.0.0] - 2025-10-24

### Added
- Initial template release
- Complete Phase 1 planning workflow
- Agent system with specialization
- Documentation automation
- Session recovery via PreCompact hook
- Quality gates with Reviewer agent
- Example feature for reference

### Documentation
- CLAUDE.md orchestration
- Complete agent definitions
- Slash command workflows
- SOP documentation
- System architecture templates
- Testing strategy
- Git workflow standards

### Infrastructure
- Directory structure
- Hook system configuration
- Template system
- Documentation index automation

---

**Note:** This changelog is automatically updated by the Documenter agent when running `/update-docs` or during feature planning. Feature-specific entries will be appended as development progresses.

**Convention:** We follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages, which feeds into this changelog.
