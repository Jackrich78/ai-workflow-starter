# Integration Tests: Documenter Indexing

**Feature ID:** FEAT-003
**Test File:** documenter-indexing.integration.test.md
**Purpose:** Test Documenter agent integration for specialist indexing in docs/README.md

## Test 1: Documenter detects and indexes specialists

**Acceptance Criteria:** AC-FEAT-003-303

**Components:** Documenter agent, file system scan, docs/README.md writer
**Setup:** Two specialists exist: supabase-specialist.md, fastapi-specialist.md
**Scenario:** /update-docs runs → Documenter scans .claude/subagents/ → finds specialists → updates index
**Assertions:** docs/README.md contains "Active Specialists" section listing both specialists with names and dates

**TODO:** Implement test with pre-created specialist files, trigger /update-docs command, verify Documenter scans .claude/subagents/ directory, check specialists detected (suffix: "-specialist.md"), validate docs/README.md updated with "Active Specialists" section, ensure both specialists listed with names and creation dates, verify count shown (e.g., "2 active specialists")

---

## Test 2: Documenter handles no specialists gracefully

**Acceptance Criteria:** AC-FEAT-003-303

**Components:** Documenter agent, file system scan
**Setup:** No specialist files in .claude/subagents/ directory
**Scenario:** /update-docs runs → Documenter scans → finds no specialists → skips section
**Assertions:** docs/README.md does not include "Active Specialists" section or shows "None"

**TODO:** Implement test with clean .claude/subagents/ directory (only core agents, no specialists), trigger /update-docs command, verify Documenter handles empty scan result gracefully, check docs/README.md either omits "Active Specialists" section or shows "None" / "0 active specialists", ensure no errors or warnings

---

**Total Test Stubs:** 2
**Status:** TODO - Awaiting Phase 2 implementation
