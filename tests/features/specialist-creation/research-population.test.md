# Unit Tests: Research Auto-Population

**Feature ID:** FEAT-003
**Test File:** research-population.test.md
**Purpose:** Test research auto-population logic for Archon RAG and WebSearch

## Test 1: Extract content from Archon RAG results

**Acceptance Criteria:** AC-FEAT-003-004

**Given:** Mock Archon RAG response with Supabase authentication patterns
**When:** Content extraction function called
**Then:** Relevant patterns extracted and formatted for specialist "Core Responsibilities" section

**TODO:** Implement test with mock Archon RAG JSON response, extract relevant content (tools, patterns, best practices), format into markdown sections, verify extracted content is actionable and specific

---

## Test 2: Extract content from WebSearch results

**Acceptance Criteria:** AC-FEAT-003-005

**Given:** Mock WebSearch response with Supabase documentation links
**When:** Content extraction function called
**Then:** Documentation content extracted and formatted with source URLs

**TODO:** Implement test with mock WebSearch JSON response, extract titles, URLs, snippets from results, format with citations (URL + retrieval date), verify official documentation prioritized over blogs

---

## Test 3: Fall back to WebSearch when Archon unavailable

**Acceptance Criteria:** AC-FEAT-003-203

**Given:** Archon MCP not available (tool detection returns false)
**When:** Research auto-population executes
**Then:** WebSearch used directly, no Archon query attempted

**TODO:** Implement test with Archon availability check mocked to return false, verify WebSearch called immediately, check Archon query not attempted, ensure graceful handling without errors

---

## Test 4: Handle research failure gracefully

**Acceptance Criteria:** AC-FEAT-003-103

**Given:** Archon returns no results and WebSearch returns empty
**When:** Research auto-population executes
**Then:** Minimal static template returned with TODO markers for user population

**TODO:** Implement test with both Archon and WebSearch mocked to return empty results, verify fallback to minimal template, check TODO markers added for manual population, ensure error message shown to user

---

## Test 5: Document knowledge sources correctly

**Acceptance Criteria:** AC-FEAT-003-010

**Given:** Content extracted from Archon RAG
**When:** Source attribution added
**Then:** Citation format: "Retrieved via Archon RAG (Supabase Documentation, 2025-10-25)"

**TODO:** Implement test for source citation formatting, verify correct format with retrieval method (Archon RAG / WebSearch), check source name and date included, ensure URLs included for WebSearch sources

---

**Total Test Stubs:** 5
**Status:** TODO - Awaiting Phase 2 implementation
