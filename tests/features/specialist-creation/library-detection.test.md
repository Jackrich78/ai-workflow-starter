# Unit Tests: Library Detection

**Feature ID:** FEAT-003
**Test File:** library-detection.test.md
**Purpose:** Test library/framework detection regex patterns during Explorer Q&A

## Test 1: Detect well-known libraries case-insensitively

**Acceptance Criteria:** AC-FEAT-003-001

**Given:** User responses: "We'll use Supabase", "I'm using FASTAPI", "fastapi for backend"
**When:** Library detection regex applied
**Then:** Correctly identifies: "Supabase", "FastAPI", "FastAPI" (normalized)

**TODO:** Implement test with sample user responses, apply regex patterns for well-known libraries, verify case-insensitive matching works, check normalization to standard case (e.g., "FastAPI" not "FASTAPI")

---

## Test 2: Avoid false positives on partial matches

**Acceptance Criteria:** AC-FEAT-003-001

**Given:** User responses: "We'll use super fast databases", "I like Paris", "The base implementation"
**When:** Library detection regex applied
**Then:** No libraries detected (no false positives for "super", "Paris", "base")

**TODO:** Implement test with false positive test cases, verify regex doesn't match substrings of common words, ensure word boundary matching or whitelist approach, validate no spurious detections

---

## Test 3: Detect multiple libraries in single response

**Acceptance Criteria:** AC-FEAT-003-101

**Given:** User response: "We'll use Supabase for database and FastAPI for backend"
**When:** Library detection regex applied
**Then:** Both "Supabase" and "FastAPI" detected as separate libraries

**TODO:** Implement test with multi-library response, verify all libraries detected (not just first), check libraries returned as separate list items, ensure no duplicates if mentioned multiple times

---

## Test 4: Detect libraries with special characters

**Acceptance Criteria:** AC-FEAT-003-001

**Given:** User responses: "Using Next.js", "Tailwind CSS framework", "Vue.js 3"
**When:** Library detection regex applied
**Then:** Correctly identifies: "Next.js", "Tailwind CSS", "Vue.js"

**TODO:** Implement test for libraries with dots, hyphens, spaces in names, verify regex handles special characters correctly, check version numbers stripped or ignored (e.g., "Vue.js 3" → "Vue.js"), validate names match expected patterns

---

**Total Test Stubs:** 4
**Status:** TODO - Awaiting Phase 2 implementation
