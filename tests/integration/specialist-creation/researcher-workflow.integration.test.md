# Integration Tests: Researcher Workflow

**Feature ID:** FEAT-003
**Test File:** researcher-workflow.integration.test.md
**Purpose:** Test Researcher agent integration for specialist invocation during research phase

## Test 1: Researcher invokes specialist for domain-specific question

**Acceptance Criteria:** AC-FEAT-003-008

**Components:** Researcher agent, Task tool, specialist agent
**Setup:** Specialist exists, PRD contains Supabase questions
**Scenario:** Researcher identifies Supabase question → invokes specialist via Task tool → receives answer
**Assertions:** Specialist invoked with correct question, answer returned, findings incorporated into research.md

**TODO:** Implement test with pre-existing Supabase specialist, simulate PRD with Supabase-specific questions (e.g., "How to implement Row Level Security?"), verify Researcher identifies domain match, check Task tool invocation with @.claude/subagents/supabase-specialist.md reference, validate answer returned with specifics, ensure research.md includes specialist findings

---

## Test 2: Specialist uses Archon RAG cascade

**Acceptance Criteria:** AC-FEAT-003-009

**Components:** Specialist agent, Archon MCP, WebSearch fallback
**Setup:** Archon available with Supabase documentation
**Scenario:** Specialist invoked → queries Archon → finds answer → returns with source
**Assertions:** Archon queried first, answer contains source citation "via Archon RAG"

**TODO:** Implement test with Archon MCP mocked to return relevant results, trigger specialist invocation with question, verify Archon query sent before WebSearch, check answer includes Archon citation, validate source attribution format correct

---

## Test 3: Specialist falls back to WebSearch when Archon insufficient

**Acceptance Criteria:** AC-FEAT-003-009

**Components:** Specialist agent, Archon MCP, WebSearch
**Setup:** Archon available but lacks specific Supabase content
**Scenario:** Specialist invoked → queries Archon (insufficient) → queries WebSearch → finds answer
**Assertions:** Archon attempted first, WebSearch used second, answer contains source "via WebSearch"

**TODO:** Implement test with Archon mocked to return insufficient results, WebSearch mocked to return relevant content, verify Archon queried first, check fallback to WebSearch triggered, validate answer includes WebSearch URL citation, ensure cascade order correct

---

## Test 4: Specialist handles missing file gracefully

**Acceptance Criteria:** AC-FEAT-003-104

**Components:** Researcher agent, Task tool, error handling
**Setup:** Researcher tries to invoke non-existent specialist
**Scenario:** Invocation fails → error caught → Researcher falls back to standard research
**Assertions:** Error handled gracefully, research.md created successfully without specialist input

**TODO:** Implement test with specialist file missing (deleted or never created), trigger Researcher invocation attempt, verify error caught and logged, check Researcher continues with standard research (no specialist), ensure research.md created with direct research findings, validate no workflow failure

---

**Total Test Stubs:** 4
**Status:** TODO - Awaiting Phase 2 implementation
