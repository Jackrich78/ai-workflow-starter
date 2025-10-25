# Testing Strategy: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Created:** 2025-10-25
**Test Coverage Goal:** 85%

## Test Strategy Overview

This feature enables dynamic specialist sub-agent creation during /explore workflows. Testing focuses on workflow integration, template processing, research auto-population, and specialist invocation patterns. We'll use unit tests for template logic and file operations, integration tests for agent workflow interactions, and manual tests for end-to-end creation and invocation flows. Priority is validating <2 minute creation time, comprehensive auto-population quality, and graceful degradation when Archon unavailable.

**Testing Levels:**
- ✅ Unit Tests: Template loading, placeholder substitution, file writing, library detection regex
- ✅ Integration Tests: Explorer workflow integration, Researcher workflow integration, Archon/WebSearch cascade
- ✅ E2E Tests: N/A (workflow-based testing via manual scenarios)
- ✅ Manual Tests: Full /explore with specialist creation, specialist invocation, documentation indexing

## Unit Tests

### Test Files to Create

#### `tests/features/specialist-creation/template-processing.test.md`

**Purpose:** Test template loading, placeholder substitution, and specialist file generation logic

**Test Stubs:**
1. **Test:** Load TEMPLATE.md successfully (AC-FEAT-003-401)
   - **Given:** TEMPLATE.md exists at .claude/subagents/TEMPLATE.md
   - **When:** Template loading function called
   - **Then:** Template content returned with all required sections intact
   - **Mocks:** File system read operation

2. **Test:** Substitute placeholders correctly (AC-FEAT-003-003)
   - **Given:** Template content with {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}} placeholders
   - **When:** Substitution function called with values: name="supabase", description="Supabase database specialist", library="Supabase"
   - **Then:** All placeholders replaced with correct values, no remaining {{ }} markers
   - **Mocks:** None (pure function)

3. **Test:** Generate kebab-case filename from library name (AC-FEAT-003-204)
   - **Given:** Library names with various cases: "Supabase", "PydanticAI", "FastAPI", "Next.js"
   - **When:** Filename generation function called
   - **Then:** Correct kebab-case filenames: "supabase-specialist.md", "pydantic-ai-specialist.md", "fast-api-specialist.md", "next-js-specialist.md"
   - **Mocks:** None (pure function)

4. **Test:** Write specialist file to correct location (AC-FEAT-003-003)
   - **Given:** Specialist content and filename
   - **When:** File write function called
   - **Then:** File written to .claude/subagents/[filename] with correct content
   - **Mocks:** File system write operation

5. **Test:** Validate YAML frontmatter structure (AC-FEAT-003-003)
   - **Given:** Generated specialist file content
   - **When:** YAML parser validates frontmatter
   - **Then:** Frontmatter contains required fields: name, description, tools, phase, status, color
   - **Mocks:** None (validation function)

---

#### `tests/features/specialist-creation/library-detection.test.md`

**Purpose:** Test library/framework detection regex patterns during Explorer Q&A

**Test Stubs:**
1. **Test:** Detect well-known libraries case-insensitively (AC-FEAT-003-001)
   - **Given:** User responses: "We'll use Supabase", "I'm using FASTAPI", "fastapi for backend"
   - **When:** Library detection regex applied
   - **Then:** Correctly identifies: "Supabase", "FastAPI", "FastAPI" (normalized)
   - **Mocks:** None (regex matching)

2. **Test:** Avoid false positives on partial matches (AC-FEAT-003-001)
   - **Given:** User responses: "We'll use super fast databases", "I like Paris", "The base implementation"
   - **When:** Library detection regex applied
   - **Then:** No libraries detected (no false positives for "super", "Paris", "base")
   - **Mocks:** None (regex matching)

3. **Test:** Detect multiple libraries in single response (AC-FEAT-003-101)
   - **Given:** User response: "We'll use Supabase for database and FastAPI for backend"
   - **When:** Library detection regex applied
   - **Then:** Both "Supabase" and "FastAPI" detected as separate libraries
   - **Mocks:** None (regex matching)

4. **Test:** Detect libraries with special characters (AC-FEAT-003-001)
   - **Given:** User responses: "Using Next.js", "Tailwind CSS framework", "Vue.js 3"
   - **When:** Library detection regex applied
   - **Then:** Correctly identifies: "Next.js", "Tailwind CSS", "Vue.js"
   - **Mocks:** None (regex matching)

---

#### `tests/features/specialist-creation/research-population.test.md`

**Purpose:** Test research auto-population logic for Archon RAG and WebSearch

**Test Stubs:**
1. **Test:** Extract content from Archon RAG results (AC-FEAT-003-004)
   - **Given:** Mock Archon RAG response with Supabase authentication patterns
   - **When:** Content extraction function called
   - **Then:** Relevant patterns extracted and formatted for specialist "Core Responsibilities" section
   - **Mocks:** Archon RAG API response

2. **Test:** Extract content from WebSearch results (AC-FEAT-003-005)
   - **Given:** Mock WebSearch response with Supabase documentation links
   - **When:** Content extraction function called
   - **Then:** Documentation content extracted and formatted with source URLs
   - **Mocks:** WebSearch API response

3. **Test:** Fall back to WebSearch when Archon unavailable (AC-FEAT-003-203)
   - **Given:** Archon MCP not available (tool detection returns false)
   - **When:** Research auto-population executes
   - **Then:** WebSearch used directly, no Archon query attempted
   - **Mocks:** MCP availability check, WebSearch API

4. **Test:** Handle research failure gracefully (AC-FEAT-003-103)
   - **Given:** Archon returns no results and WebSearch returns empty
   - **When:** Research auto-population executes
   - **Then:** Minimal static template returned with TODO markers for user population
   - **Mocks:** Archon RAG empty response, WebSearch empty response

5. **Test:** Document knowledge sources correctly (AC-FEAT-003-010)
   - **Given:** Content extracted from Archon RAG
   - **When:** Source attribution added
   - **Then:** Citation format: "Retrieved via Archon RAG (Supabase Documentation, 2025-10-25)"
   - **Mocks:** None (formatting function)

---

#### `tests/features/specialist-creation/performance-timing.test.md`

**Purpose:** Validate specialist creation completes in <2 minutes (AC-FEAT-003-201)

**Test Stubs:**
1. **Test:** Static scaffolding completes in <30 seconds
   - **Given:** TEMPLATE.md and library name "Supabase"
   - **When:** Template loading and substitution executes
   - **Then:** Completes in <30 seconds, file structure ready
   - **Mocks:** File operations (measure elapsed time)

2. **Test:** Archon RAG query completes in <30 seconds
   - **Given:** Archon MCP available and query "Supabase tools patterns"
   - **When:** RAG query executes
   - **Then:** Results returned in <30 seconds
   - **Mocks:** Archon RAG with latency simulation

3. **Test:** WebSearch query completes in <40 seconds
   - **Given:** WebSearch query "Supabase documentation best practices 2025"
   - **When:** Search executes
   - **Then:** Results returned in <40 seconds
   - **Mocks:** WebSearch with latency simulation

4. **Test:** Full creation workflow completes in <120 seconds (AC-FEAT-003-201)
   - **Given:** Full specialist creation workflow for "FastAPI"
   - **When:** User confirms creation → workflow executes → file written
   - **Then:** Total elapsed time <120 seconds measured
   - **Mocks:** All external dependencies (Archon, WebSearch, file I/O)

---

## Integration Tests

### Test Files to Create

#### `tests/integration/specialist-creation/explorer-workflow.integration.test.md`

**Purpose:** Test Explorer agent integration for library detection and specialist creation suggestion

**Test Stubs:**
1. **Test:** Explorer detects library and prompts user (AC-FEAT-003-002)
   - **Components:** Explorer agent, library detection logic, user prompt system
   - **Setup:** Simulate /explore conversation, user mentions "Supabase" in response
   - **Scenario:** Explorer Q&A → library detected → PRD created → specialist creation prompt shown
   - **Assertions:** Prompt appears after PRD, before Researcher invocation; offers narrow/broad/skip choices

2. **Test:** User confirms narrow scope specialist creation (AC-FEAT-003-003)
   - **Components:** Explorer agent, specialist creation workflow, file writer
   - **Setup:** Explorer prompts specialist creation, user chooses "Yes, narrow"
   - **Scenario:** User confirms → creation workflow executes → specialist file written
   - **Assertions:** File exists at .claude/subagents/supabase-specialist.md, contains narrow scope content

3. **Test:** User skips specialist creation, workflow continues (AC-FEAT-003-105)
   - **Components:** Explorer agent, Researcher agent invocation
   - **Setup:** Explorer prompts specialist creation, user chooses "Skip"
   - **Scenario:** User skips → Explorer immediately invokes Researcher → /explore completes normally
   - **Assertions:** No specialist file created, Researcher invoked, PRD and research.md created

4. **Test:** Explorer reuses existing specialist (AC-FEAT-003-102)
   - **Components:** Explorer agent, file existence check
   - **Setup:** Specialist file already exists at .claude/subagents/supabase-specialist.md
   - **Scenario:** User mentions "Supabase" → Explorer checks file → skips creation prompt → uses existing
   - **Assertions:** No duplicate creation prompt, existing specialist referenced, workflow continues

---

#### `tests/integration/specialist-creation/researcher-workflow.integration.test.md`

**Purpose:** Test Researcher agent integration for specialist invocation during research phase

**Test Stubs:**
1. **Test:** Researcher invokes specialist for domain-specific question (AC-FEAT-003-008)
   - **Components:** Researcher agent, Task tool, specialist agent
   - **Setup:** Specialist exists, PRD contains Supabase questions
   - **Scenario:** Researcher identifies Supabase question → invokes specialist via Task tool → receives answer
   - **Assertions:** Specialist invoked with correct question, answer returned, findings incorporated into research.md

2. **Test:** Specialist uses Archon RAG cascade (AC-FEAT-003-009)
   - **Components:** Specialist agent, Archon MCP, WebSearch fallback
   - **Setup:** Archon available with Supabase documentation
   - **Scenario:** Specialist invoked → queries Archon → finds answer → returns with source
   - **Assertions:** Archon queried first, answer contains source citation "via Archon RAG"

3. **Test:** Specialist falls back to WebSearch when Archon insufficient (AC-FEAT-003-009)
   - **Components:** Specialist agent, Archon MCP, WebSearch
   - **Setup:** Archon available but lacks specific Supabase content
   - **Scenario:** Specialist invoked → queries Archon (insufficient) → queries WebSearch → finds answer
   - **Assertions:** Archon attempted first, WebSearch used second, answer contains source "via WebSearch"

4. **Test:** Specialist handles missing file gracefully (AC-FEAT-003-104)
   - **Components:** Researcher agent, Task tool, error handling
   - **Setup:** Researcher tries to invoke non-existent specialist
   - **Scenario:** Invocation fails → error caught → Researcher falls back to standard research
   - **Assertions:** Error handled gracefully, research.md created successfully without specialist input

---

#### `tests/integration/specialist-creation/documenter-indexing.integration.test.md`

**Purpose:** Test Documenter agent integration for specialist indexing in docs/README.md

**Test Stubs:**
1. **Test:** Documenter detects and indexes specialists (AC-FEAT-003-303)
   - **Components:** Documenter agent, file system scan, docs/README.md writer
   - **Setup:** Two specialists exist: supabase-specialist.md, fastapi-specialist.md
   - **Scenario:** /update-docs runs → Documenter scans .claude/subagents/ → finds specialists → updates index
   - **Assertions:** docs/README.md contains "Active Specialists" section listing both specialists with names and dates

2. **Test:** Documenter handles no specialists gracefully (AC-FEAT-003-303)
   - **Components:** Documenter agent, file system scan
   - **Setup:** No specialist files in .claude/subagents/ directory
   - **Scenario:** /update-docs runs → Documenter scans → finds no specialists → skips section
   - **Assertions:** docs/README.md does not include "Active Specialists" section or shows "None"

---

## Manual Testing

### Manual Test Scenarios

*See `manual-test.md` for detailed step-by-step instructions.*

**Quick Reference:**
1. **Full Specialist Creation Flow:** Verify /explore detects library, prompts creation, generates specialist in <2 minutes
2. **Specialist Invocation Flow:** Verify Researcher invokes specialist, receives answer with sources
3. **Documentation Indexing:** Verify /update-docs lists specialists in docs/README.md
4. **Archon Graceful Degradation:** Verify specialists work with Archon disabled (WebSearch fallback)
5. **Edge Cases:** Verify multiple libraries, existing specialists, failed research scenarios

**Manual Test Focus:**
- **Workflow Integration:** End-to-end /explore with specialist creation and invocation
- **Performance Validation:** Time full creation workflow (target: <120 seconds)
- **Quality Verification:** Review auto-populated specialist content for accuracy and completeness
- **Error Handling:** Test edge cases like missing files, failed research, duplicate libraries

## Test Data Requirements

### Fixtures & Seed Data

**Unit Test Fixtures:**
```markdown
# TEMPLATE.md mock
---
name: {{NAME}}
description: {{DESCRIPTION}}
tools: Read, WebSearch, Grep
---

# {{LIBRARY}} Specialist

## Primary Objective
{{OBJECTIVE}}

## Core Responsibilities
{{RESPONSIBILITIES}}
```

**Integration Test Data:**
- Mock Archon RAG responses for Supabase, FastAPI, PydanticAI queries
- Mock WebSearch responses with documentation URLs
- Sample library detection phrases for testing regex patterns
- Existing specialist files for reuse scenarios

**Manual Test Data:**
- Clean .claude/subagents/ directory for fresh creation tests
- Pre-created specialists for invocation tests
- Sample PRDs mentioning Supabase, FastAPI for research tests

## Mocking Strategy

### What to Mock

**Always Mock:**
- Archon MCP API calls (to avoid dependency and ensure consistent test data)
- WebSearch API calls (to avoid external network dependency and rate limits)
- File system I/O for unit tests (for speed and isolation)
- Task tool invocation for agent communication (to test agent logic independently)

**Sometimes Mock:**
- File system for integration tests (real for workflow tests, mock for error scenarios)
- TEMPLATE.md loading (real template for integration, mock for unit)

**Never Mock:**
- Template substitution logic (core functionality to test)
- Library detection regex (core functionality to test)
- Specialist content validation (must test real parsing)

### Mocking Approach

**Framework:** Markdown-based test stubs with TODO comments (no executable tests in Phase 1)

**Mock Examples:**
```markdown
# Mock Archon RAG response
{
  "success": true,
  "results": [
    {
      "content": "Supabase provides Row Level Security (RLS) for fine-grained access control...",
      "metadata": {"source": "Supabase Documentation", "url": "https://supabase.com/docs/guides/auth/row-level-security"}
    }
  ]
}

# Mock WebSearch response
{
  "results": [
    {"title": "Supabase Auth Guide", "url": "https://supabase.com/docs/guides/auth", "snippet": "Supabase Auth provides..."}
  ]
}
```

## Test Execution

### Running Tests Locally

**Unit Tests:**
```bash
# Phase 2 implementation (Phase 1: stubs only)
# Placeholder for future test execution
# TODO: Implement test runner for template processing, library detection, research population tests
```

**Integration Tests:**
```bash
# Phase 2 implementation (Phase 1: stubs only)
# Placeholder for future test execution
# TODO: Implement workflow integration tests for Explorer, Researcher, Documenter
```

**Manual Tests:**
```bash
# Run during manual testing phase
# See manual-test.md for step-by-step instructions
# Execute full /explore workflow with library mentions
# Verify specialist creation, invocation, indexing manually
```

### CI/CD Integration (Phase 2)

**Pipeline Stages:**
1. Unit tests (run on every commit) - template, regex, file operations
2. Integration tests (run on every commit) - workflow integration, agent invocation
3. Manual tests (run on PR, pre-merge) - full workflow validation
4. Performance benchmarks - creation time <2 minutes validation
5. Test result artifacts - logs, created specialist files for review

**Failure Handling:**
- Failing unit tests block commit
- Failing integration tests block merge
- Performance regression (>2 minutes) blocks merge
- Manual test failures require retesting after fix

## Coverage Goals

### Coverage Targets

| Test Level | Target Coverage | Minimum Acceptable |
|------------|----------------|-------------------|
| Unit | 90% | 80% |
| Integration | 75% | 65% |
| Manual | N/A (workflow-based) | All 5 scenarios pass |

### Critical Paths

**Must Have 100% Coverage:**
- Template loading and placeholder substitution
- Library detection regex patterns
- Specialist file writing and validation
- Archon → WebSearch fallback cascade
- Error handling for missing files, failed research

**Can Have Lower Coverage:**
- Edge case combinations (multiple libraries + existing specialists + Archon unavailable)
- UI/UX of prompts (covered by manual testing)
- Logging and telemetry

## Performance Testing

**Requirement:** From AC-FEAT-003-201

**Test Approach:**
- **Tool:** Manual timing with stopwatch or automated test harness (Phase 2)
- **Scenarios:**
  1. **Full Creation Workflow:** User confirmation → specialist file written (target: <120 seconds)
  2. **Static Scaffolding Only:** Template load → substitution → write (target: <30 seconds)
  3. **Archon RAG Query:** Query → results → extraction (target: <30 seconds)
  4. **WebSearch Query:** Query → results → extraction (target: <40 seconds)

**Acceptance:**
- <120 seconds total creation time for 95% of specialist creations
- <30 seconds static scaffolding for all cases
- <30 seconds Archon query for 90% of queries (network dependent)
- <40 seconds WebSearch query for 85% of queries (network dependent)

## Security Testing

*No specific security requirements for this feature (documentation-focused).*

**Basic Security Checks:**
1. **File Path Validation:** Verify specialist files only written to .claude/subagents/ (no directory traversal)
2. **Input Sanitization:** Verify library names sanitized before filename generation (no special characters that break file system)
3. **YAML Injection:** Verify YAML frontmatter generated safely (no injection via library names)

**Tools:**
- Manual code review for file path handling
- Unit tests for filename sanitization

## Test Stub Generation (Phase 1)

*These test files will be created with TODO stubs during planning:*

```
tests/
├── features/
│   └── specialist-creation/
│       ├── template-processing.test.md (5 unit test stubs)
│       ├── library-detection.test.md (4 unit test stubs)
│       ├── research-population.test.md (5 unit test stubs)
│       └── performance-timing.test.md (4 unit test stubs)
└── integration/
    └── specialist-creation/
        ├── explorer-workflow.integration.test.md (4 integration test stubs)
        ├── researcher-workflow.integration.test.md (4 integration test stubs)
        └── documenter-indexing.integration.test.md (2 integration test stubs)
```

**Total Test Stubs:** 28 test functions with TODO comments

## Out of Scope

*What we're explicitly NOT testing in this phase:*

- **Specialist-to-specialist invocation:** Future enhancement, not implemented in Phase 1
- **Specialist self-updating (stateful knowledge):** Future enhancement, Phase 3
- **Cross-project specialist sharing:** Out of scope for single-project template
- **Automated specialist deprecation:** Manual management in Phase 1
- **NLP-based library detection:** Using regex only, NLP is future enhancement
- **Specialist performance metrics:** No usage tracking in Phase 1

---

**Next Steps:**
1. Planner will generate test stub files based on this strategy
2. Phase 2: Implementer will make stubs functional
3. Phase 2: Tester will execute and validate
