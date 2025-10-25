# Unit Tests: Performance and Timing

**Feature ID:** FEAT-003
**Test File:** performance-timing.test.md
**Purpose:** Validate specialist creation completes in <2 minutes (AC-FEAT-003-201)

## Test 1: Static scaffolding completes in <30 seconds

**Acceptance Criteria:** AC-FEAT-003-201

**Given:** TEMPLATE.md and library name "Supabase"
**When:** Template loading and substitution executes
**Then:** Completes in <30 seconds, file structure ready

**TODO:** Implement test with timer, load TEMPLATE.md and perform substitution, measure elapsed time from start to file ready, verify time <30 seconds, ensure no blocking operations

---

## Test 2: Archon RAG query completes in <30 seconds

**Acceptance Criteria:** AC-FEAT-003-201

**Given:** Archon MCP available and query "Supabase tools patterns"
**When:** RAG query executes
**Then:** Results returned in <30 seconds

**TODO:** Implement test with Archon RAG mock (simulated latency), measure query execution time, verify <30 seconds timeout, check results are non-empty and relevant

---

## Test 3: WebSearch query completes in <40 seconds

**Acceptance Criteria:** AC-FEAT-003-201

**Given:** WebSearch query "Supabase documentation best practices 2025"
**When:** Search executes
**Then:** Results returned in <40 seconds

**TODO:** Implement test with WebSearch mock (simulated latency), measure query execution time, verify <40 seconds timeout, check results include URLs and snippets

---

## Test 4: Full creation workflow completes in <120 seconds

**Acceptance Criteria:** AC-FEAT-003-201

**Given:** Full specialist creation workflow for "FastAPI"
**When:** User confirms creation → workflow executes → file written
**Then:** Total elapsed time <120 seconds measured

**TODO:** Implement end-to-end test with all components (template load, Archon query, WebSearch fallback, file write), measure total time from user confirmation to completion message, verify <120 seconds, validate file created successfully

---

**Total Test Stubs:** 4
**Status:** TODO - Awaiting Phase 2 implementation
