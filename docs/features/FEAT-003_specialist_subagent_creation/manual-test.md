# Manual Testing Guide: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Created:** 2025-10-25
**Intended Audience:** Non-technical testers, QA, product managers

## Overview

This guide provides step-by-step instructions for manually testing the Specialist Sub-Agent Creation System. Each test scenario validates that specialists can be created dynamically during /explore workflows, invoked by agents for domain-specific research, and indexed in documentation automatically.

**Prerequisites:**
- Claude Code CLI installed and configured
- Access to this AI workflow starter repository
- Archon MCP installed and configured (for some scenarios) - optional
- Basic understanding of /explore and /plan workflow commands

**Estimated Time:** 45 minutes for all scenarios

## Test Setup

### Before You Begin

1. **Environment:** Local development environment with repository cloned
2. **Data Reset:** Clear .claude/subagents/ directory except TEMPLATE.md (preserve existing agents: explorer.md, researcher.md, etc.)
3. **Browser:** N/A (CLI-based testing)
4. **Working Directory:** Repository root (/Users/builder/dev/ai-workflow-starter or your clone path)

### Test Account Setup

**Not applicable** (local CLI testing, no user accounts needed)

### Sample Data Needed

- **TEMPLATE.md:** Ensure .claude/subagents/TEMPLATE.md exists with placeholders {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}}
- **Test Library Names:** Use "Supabase" for database specialist, "FastAPI" for backend specialist in test scenarios
- **Clean Slate:** Remove any existing specialist files from previous tests (keep core agents)

## Test Scenarios

### Test Scenario 1: Full Specialist Creation During /explore (Happy Path)

**Acceptance Criteria:** AC-FEAT-003-001, AC-FEAT-003-002, AC-FEAT-003-003

**Purpose:** Validate end-to-end specialist creation workflow from library detection through file generation

**Steps:**
1. **Run /explore command** in Claude Code CLI
   - Command: `/explore database layer for todo app`
   - **Expected:** Explorer agent starts discovery conversation with questions

2. **Answer Explorer questions, mentioning "Supabase"**
   - Example response: "We'll use Supabase for database with PostgreSQL and authentication"
   - **Expected:** Explorer detects "Supabase" library mention via regex

3. **Wait for PRD creation to complete**
   - **Expected:** Explorer creates docs/features/FEAT-XXX/prd.md successfully

4. **Observe specialist creation prompt**
   - **Expected:** Explorer prompts: "I noticed you mentioned Supabase. Create Supabase Specialist? (narrow/broad/skip)"
   - **Expected:** Prompt appears AFTER PRD creation, BEFORE Researcher invocation

5. **Choose "narrow scope" specialist**
   - Response: "Yes, narrow scope"
   - **Expected:** Creation workflow begins with status message

6. **Wait for specialist creation to complete** (measure time)
   - **Expected:** Completion message within 120 seconds: "Supabase Specialist created at .claude/subagents/supabase-specialist.md"
   - **Screenshot/Note:** Record actual creation time in seconds

7. **Verify specialist file exists and contains content**
   - Navigate to .claude/subagents/supabase-specialist.md
   - **Expected:** File exists with valid YAML frontmatter (name, description, tools)
   - **Expected:** File contains sections: Primary Objective, Core Responsibilities, Tools Access, Workflow
   - **Expected:** Content is Supabase-specific (not generic template boilerplate)
   - **Expected:** Sources cited for auto-populated content (Archon RAG or WebSearch URLs)

8. **Verify Explorer continues to Researcher invocation**
   - **Expected:** Explorer invokes Researcher agent automatically after specialist creation
   - **Expected:** Research phase completes normally

**✅ Pass Criteria:**
- Supabase library detected during Q&A
- Specialist creation prompt appeared after PRD, before Researcher
- Specialist file created in <120 seconds
- File contains comprehensive auto-populated instructions
- Workflow continued normally to research phase

**❌ Fail Scenarios:**
- If library not detected, report: "Library detection failed for 'Supabase' mention"
- If creation takes >120 seconds, report: "Creation time exceeded target: [X] seconds"
- If file missing sections, report: "Incomplete specialist: missing [section name]"
- If content is generic template, report: "Auto-population failed: no Supabase-specific content"

---

### Test Scenario 2: Specialist Invocation by Researcher Agent

**Acceptance Criteria:** AC-FEAT-003-008, AC-FEAT-003-009, AC-FEAT-003-010

**Purpose:** Validate Researcher can invoke specialist for domain-specific questions and receive answers with source attribution

**Steps:**
1. **Ensure specialist exists from Scenario 1** (or create manually)
   - File: .claude/subagents/supabase-specialist.md
   - **Expected:** Specialist file present and valid

2. **Run /explore with Supabase-related feature idea**
   - Command: `/explore user authentication with social login`
   - During Q&A, mention: "Using Supabase Auth for authentication"
   - **Expected:** PRD created mentioning Supabase

3. **Observe Researcher phase** (after specialist creation prompt, if any)
   - **Expected:** Researcher identifies Supabase-specific questions from PRD (e.g., "How to implement Row Level Security?")

4. **Verify specialist invocation occurs**
   - **Expected:** Researcher invokes Supabase Specialist via Task tool: "@.claude/subagents/supabase-specialist.md"
   - **Expected:** Specialist receives domain-specific question

5. **Verify specialist uses knowledge cascade**
   - If Archon available: **Expected:** Specialist queries Archon RAG first
   - If Archon unavailable or insufficient: **Expected:** Specialist queries WebSearch
   - **Expected:** Specialist documents which source was used

6. **Verify specialist returns findings to Researcher**
   - **Expected:** Specialist answer includes specific guidance (e.g., RLS policies, authentication flows)
   - **Expected:** Answer cites source: "(Found via Archon RAG)" or "(Found via WebSearch with URL)"
   - **Expected:** Researcher incorporates findings into research.md with attribution

7. **Review research.md for specialist findings**
   - Open docs/features/FEAT-XXX/research.md
   - **Expected:** Research contains Supabase-specific findings
   - **Expected:** Attribution notes specialist was consulted: "Per Supabase Specialist..." or similar

**✅ Pass Criteria:**
- Researcher invoked specialist for Supabase questions
- Specialist used Archon RAG → WebSearch cascade appropriately
- Findings returned with clear source attribution
- Research.md incorporates specialist findings with citation

**❌ Fail Scenarios:**
- Specialist not invoked despite Supabase questions in PRD
- No source attribution in specialist response
- Research.md missing specialist findings
- Specialist returned generic answer (not Supabase-specific)

---

### Test Scenario 3: Multiple Libraries Detected and Created

**Acceptance Criteria:** AC-FEAT-003-101

**Purpose:** Validate system handles multiple library mentions in single response

**Steps:**
1. **Run /explore command**
   - Command: `/explore full-stack todo application`

2. **Mention multiple libraries in one response**
   - Example: "We'll use FastAPI for backend and Supabase for database"
   - **Expected:** Explorer detects both "FastAPI" and "Supabase"

3. **Observe specialist creation prompts**
   - **Expected:** Explorer prompts for each library separately or offers batch creation
   - Example: "Create specialists for: FastAPI, Supabase? (narrow/broad/skip each)"

4. **Choose creation for both**
   - Response: "Yes for both, narrow scope"
   - **Expected:** Two specialist creation workflows execute sequentially

5. **Verify both specialist files created**
   - Files: .claude/subagents/fast-api-specialist.md, .claude/subagents/supabase-specialist.md
   - **Expected:** Both files exist with unique content (not duplicates)
   - **Expected:** FastAPI specialist has FastAPI-specific content
   - **Expected:** Supabase specialist has Supabase-specific content

6. **Verify no naming conflicts or overwrites**
   - **Expected:** Filenames are unique kebab-case
   - **Expected:** Neither file overwrote existing core agents

**✅ Pass Criteria:**
- Both libraries detected from single user response
- Specialist creation prompted for each library
- Both specialists created with unique, domain-specific content
- No file conflicts or overwrites

**❌ Fail Scenarios:**
- Only one library detected when both mentioned
- Duplicate filenames or content
- One specialist overwrote the other
- Generic content in both specialists (not differentiated)

---

### Test Scenario 4: Existing Specialist Reuse (No Duplicate Creation)

**Acceptance Criteria:** AC-FEAT-003-102

**Purpose:** Validate system reuses existing specialists instead of creating duplicates

**Steps:**
1. **Ensure Supabase specialist exists** from previous scenario
   - File: .claude/subagents/supabase-specialist.md
   - **Expected:** File present with valid content

2. **Run new /explore mentioning Supabase again**
   - Command: `/explore real-time notifications feature`
   - During Q&A: "Using Supabase Realtime for notifications"

3. **Observe specialist handling**
   - **Expected:** Explorer detects "Supabase" but checks for existing specialist
   - **Expected:** NO creation prompt shown
   - **Expected:** Message: "Using existing Supabase Specialist" or similar

4. **Verify no duplicate file created**
   - Check .claude/subagents/ directory
   - **Expected:** Only one supabase-specialist.md file (not supabase-specialist-1.md, etc.)

5. **Verify existing specialist is used during workflow**
   - **Expected:** Researcher invokes existing Supabase Specialist if needed
   - **Expected:** Specialist content unchanged (not overwritten)

**✅ Pass Criteria:**
- Existing specialist detected automatically
- No duplicate creation prompt
- No duplicate file created
- Existing specialist used successfully

**❌ Fail Scenarios:**
- Duplicate creation prompt shown despite existing specialist
- Duplicate file created (supabase-specialist-1.md)
- Existing specialist file overwritten or modified
- Specialist not used despite existing

---

### Test Scenario 5: Graceful Degradation Without Archon MCP

**Acceptance Criteria:** AC-FEAT-003-203

**Purpose:** Validate specialists work correctly when Archon MCP is unavailable (WebSearch fallback)

**Steps:**
1. **Disable or ensure Archon MCP is not configured**
   - Check Claude Code settings: Archon MCP should be disabled or not installed
   - **Expected:** Archon RAG tools unavailable

2. **Run /explore with library mention**
   - Command: `/explore API documentation generation`
   - During Q&A: "Using FastAPI for REST API"

3. **Observe specialist creation**
   - Choose "Yes, narrow" for FastAPI Specialist
   - **Expected:** Creation proceeds without Archon queries

4. **Verify WebSearch used for auto-population**
   - After creation, open .claude/subagents/fast-api-specialist.md
   - **Expected:** Content populated from WebSearch (not Archon RAG)
   - **Expected:** Sources cited with WebSearch URLs (e.g., "Retrieved via WebSearch: https://fastapi.tiangolo.com/")

5. **Verify specialist invocation uses WebSearch cascade**
   - During research phase, specialist invoked with FastAPI question
   - **Expected:** Specialist skips Archon check (unavailable)
   - **Expected:** Specialist queries WebSearch directly
   - **Expected:** Answer returned with WebSearch URL citation

6. **Verify no errors or failures due to Archon absence**
   - **Expected:** No error messages about missing Archon
   - **Expected:** Workflow completes successfully
   - **Expected:** Research.md contains FastAPI findings from WebSearch

**✅ Pass Criteria:**
- Specialist creation completes without Archon
- Auto-population uses WebSearch successfully
- Specialist invocation uses WebSearch cascade
- No errors or workflow failures due to Archon unavailability

**❌ Fail Scenarios:**
- Specialist creation fails or errors when Archon unavailable
- Auto-population incomplete without Archon
- Specialist invocation fails without Archon
- Error messages about missing Archon block workflow

---

### Test Scenario 6: Documentation Indexing of Active Specialists

**Acceptance Criteria:** AC-FEAT-003-303

**Purpose:** Validate Documenter agent indexes specialists in docs/README.md

**Steps:**
1. **Ensure multiple specialists exist** (from previous scenarios)
   - Files: supabase-specialist.md, fast-api-specialist.md
   - **Expected:** Both files present in .claude/subagents/

2. **Run /update-docs command**
   - Command: `/update-docs`
   - **Expected:** Documenter agent scans documentation and updates index

3. **Wait for documentation update to complete**
   - **Expected:** Completion message confirming docs/README.md updated

4. **Review docs/README.md**
   - Open docs/README.md
   - **Expected:** "Active Specialists" section exists
   - **Expected:** Section lists "Supabase Specialist" with creation date
   - **Expected:** Section lists "FastAPI Specialist" with creation date
   - **Expected:** Specialist count shown (e.g., "2 active specialists")

5. **Verify specialist details are accurate**
   - **Expected:** Each specialist entry shows name, scope (narrow/broad), and creation date
   - **Expected:** Entries link to specialist files or show file paths

6. **Test with no specialists**
   - Delete both specialist files (keep backup)
   - Run `/update-docs` again
   - **Expected:** "Active Specialists" section shows "None" or is omitted
   - **Expected:** No errors when no specialists present

**✅ Pass Criteria:**
- Documenter detects specialists automatically
- docs/README.md includes "Active Specialists" section
- All specialists listed with accurate details
- Section gracefully handles zero specialists

**❌ Fail Scenarios:**
- "Active Specialists" section missing from docs/README.md
- Specialists not listed or incomplete information
- Incorrect specialist count
- Errors when no specialists present

---

## Visual & UX Validation

*Specialist creation is CLI-based, minimal visual elements to validate.*

### Overall Workflow Check
- [ ] Specialist creation prompts are clear and actionable
- [ ] Timing messages show progress (e.g., "Creating specialist... 30% complete")
- [ ] Completion messages confirm file location
- [ ] Error messages (if any) are specific and helpful
- [ ] Workflow doesn't feel interrupted or jarring when specialist creation occurs

### User Experience Check
- [ ] Scope choice (narrow/broad/skip) is intuitive
- [ ] User can skip specialist creation without confusion
- [ ] Reusing existing specialists is transparent (user knows what's happening)
- [ ] Specialist invocation is invisible to user (happens in background during research)

### Performance Check
- [ ] Specialist creation completes in <2 minutes (feel "fast enough")
- [ ] No noticeable delay when checking for existing specialists
- [ ] Workflow continues smoothly after specialist creation

## Accessibility Testing

*Not applicable for CLI-based specialist system.*

## Cross-Browser Testing

*Not applicable for CLI-based specialist system.*

## Device Testing

*Not applicable for CLI-based specialist system.*

## Bug Reporting

**If You Find a Bug, Report:**
1. **Title:** [Brief description of issue]
2. **Scenario:** [Which test scenario: e.g., "Test Scenario 2"]
3. **Steps to Reproduce:**
   - [Exact commands run]
   - [User responses given]
   - [Library names mentioned]
4. **Expected Result:** [What should have happened]
5. **Actual Result:** [What actually happened]
6. **Files Affected:** [Specialist files created/modified, logs]
7. **Environment:**
   - OS: [macOS, Linux, Windows]
   - Claude Code Version: [Version number]
   - Archon MCP: [Enabled/Disabled]
   - Repository Branch: [Branch name]

## Test Completion Checklist

### All Scenarios Complete
- [ ] Test Scenario 1: Full Specialist Creation - PASS / FAIL
- [ ] Test Scenario 2: Specialist Invocation - PASS / FAIL
- [ ] Test Scenario 3: Multiple Libraries - PASS / FAIL
- [ ] Test Scenario 4: Existing Specialist Reuse - PASS / FAIL
- [ ] Test Scenario 5: Graceful Degradation Without Archon - PASS / FAIL
- [ ] Test Scenario 6: Documentation Indexing - PASS / FAIL

### Additional Checks
- [ ] Visual & UX validation complete
- [ ] Performance: Creation time <2 minutes validated
- [ ] No errors or warnings in CLI output

### Summary
- **Total Scenarios:** 6
- **Passed:** [Y]
- **Failed:** [Z]
- **Bugs Filed:** [Number and descriptions]

**Overall Assessment:**
- [ ] ✅ Feature is ready for release (all scenarios pass)
- [ ] ⚠️ Feature has minor issues: [specify which scenarios failed]
- [ ] ❌ Feature has blocking issues: [specify critical failures]

**Tester Sign-off:**
- **Name:** [Tester name]
- **Date:** [Testing completion date]
- **Notes:** [Observations on creation quality, workflow smoothness, any edge cases discovered]

**Key Observations:**
- Average creation time observed: [X] seconds
- Auto-population quality: [Excellent / Good / Needs improvement]
- Most common issue encountered: [Description]
- Recommended improvements: [List any suggestions]

---

**Next Steps:**
- If all tests pass: Feature approved for deployment
- If bugs found: Development team will fix and retest affected scenarios
- Update this document if feature changes significantly during implementation
