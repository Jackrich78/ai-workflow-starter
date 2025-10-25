# Integration Tests: Explorer Workflow

**Feature ID:** FEAT-003
**Test File:** explorer-workflow.integration.test.md
**Purpose:** Test Explorer agent integration for library detection and specialist creation suggestion

## Test 1: Explorer detects library and prompts user

**Acceptance Criteria:** AC-FEAT-003-002

**Components:** Explorer agent, library detection logic, user prompt system
**Setup:** Simulate /explore conversation, user mentions "Supabase" in response
**Scenario:** Explorer Q&A → library detected → PRD created → specialist creation prompt shown
**Assertions:** Prompt appears after PRD, before Researcher invocation; offers narrow/broad/skip choices

**TODO:** Implement integration test simulating full /explore workflow, inject user response with "Supabase", verify library detection triggers, check PRD creation completes first, validate prompt shown with correct options, ensure workflow pauses for user choice

---

## Test 2: User confirms narrow scope specialist creation

**Acceptance Criteria:** AC-FEAT-003-003

**Components:** Explorer agent, specialist creation workflow, file writer
**Setup:** Explorer prompts specialist creation, user chooses "Yes, narrow"
**Scenario:** User confirms → creation workflow executes → specialist file written
**Assertions:** File exists at .claude/subagents/supabase-specialist.md, contains narrow scope content

**TODO:** Implement test with user confirmation mocked, trigger creation workflow, verify all creation phases execute (scaffolding, research, file write), check file exists and is valid, validate content is narrow Supabase-specific (not broad database specialist)

---

## Test 3: User skips specialist creation, workflow continues

**Acceptance Criteria:** AC-FEAT-003-105

**Components:** Explorer agent, Researcher agent invocation
**Setup:** Explorer prompts specialist creation, user chooses "Skip"
**Scenario:** User skips → Explorer immediately invokes Researcher → /explore completes normally
**Assertions:** No specialist file created, Researcher invoked, PRD and research.md created

**TODO:** Implement test with user skip choice mocked, verify no creation workflow triggered, check no specialist file exists after workflow, validate Researcher invoked normally, ensure PRD and research.md created successfully (standard /explore)

---

## Test 4: Explorer reuses existing specialist

**Acceptance Criteria:** AC-FEAT-003-102

**Components:** Explorer agent, file existence check
**Setup:** Specialist file already exists at .claude/subagents/supabase-specialist.md
**Scenario:** User mentions "Supabase" → Explorer checks file → skips creation prompt → uses existing
**Assertions:** No duplicate creation prompt, existing specialist referenced, workflow continues

**TODO:** Implement test with pre-existing specialist file, simulate /explore with Supabase mention, verify Explorer checks for existing specialist before prompting, ensure no creation prompt shown, validate existing specialist used (if invoked), check no duplicate file created

---

**Total Test Stubs:** 4
**Status:** TODO - Awaiting Phase 2 implementation
