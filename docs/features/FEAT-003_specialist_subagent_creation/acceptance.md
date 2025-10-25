# Acceptance Criteria: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Created:** 2025-10-25
**Status:** Draft

## Overview

This feature is complete when:
- Specialists can be created in <2 minutes during /explore with comprehensive auto-populated instructions
- Specialists successfully answer domain-specific questions using Archon RAG → WebSearch → User cascade
- Explorer and Researcher agents can invoke specialists as subordinates via Task tool
- Documentation index automatically lists all active specialists in project
- Test case validated: Database Specialist created and invoked for Postgres/Supabase questions

## Functional Acceptance Criteria

### AC-FEAT-003-001: Library Detection During Explorer Q&A

**Given** user is in /explore conversation answering Explorer's discovery questions
**When** user mentions a library or framework in their response (e.g., "We'll use Supabase for database")
**Then** Explorer detects the library mention via regex pattern matching

**Validation:**
- [ ] Explorer recognizes well-known libraries: Supabase, FastAPI, PydanticAI, Django, React, PostgreSQL, Prisma, TypeScript
- [ ] Detection works case-insensitively ("supabase", "Supabase", "SUPABASE")
- [ ] Detection doesn't trigger on partial matches (e.g., "super" doesn't match "Supabase")

**Priority:** Must Have

---

### AC-FEAT-003-002: Specialist Creation Suggestion After PRD

**Given** Explorer has detected library mention during Q&A and completed PRD creation
**When** Explorer is about to invoke Researcher agent
**Then** Explorer prompts user with specialist creation choice: "Create [Library] Specialist? (narrow/broad/skip)"

**Validation:**
- [ ] Prompt appears after PRD written, before Researcher invoked
- [ ] Prompt includes scope choice: narrow (library-specific) vs. broad (category-wide)
- [ ] User can choose: confirm narrow, confirm broad, or skip
- [ ] Skipping specialist creation allows /explore to continue normally

**Priority:** Must Have

---

### AC-FEAT-003-003: Hybrid Template-Based Specialist Creation in <2 Minutes

**Given** user confirms specialist creation with scope choice
**When** creation workflow executes
**Then** specialist file created at .claude/subagents/[library-name]-specialist.md with comprehensive instructions in <2 minutes

**Validation:**
- [ ] Total creation time <120 seconds from user confirmation to file written
- [ ] File naming uses kebab-case (e.g., "supabase-specialist.md", "pydantic-ai-specialist.md")
- [ ] File contains valid YAML frontmatter (name, description, tools, phase, status, color)
- [ ] File follows TEMPLATE.md structure with all required sections
- [ ] Auto-populated sections include: name, description, primary objective, core responsibilities, tools access
- [ ] Sources documented for auto-populated content (Archon RAG / WebSearch / User input)

**Priority:** Must Have

---

### AC-FEAT-003-004: Archon RAG Auto-Population (If Available)

**Given** Archon MCP is available and configured with library documentation
**When** specialist auto-population research phase executes
**Then** specialist instructions populated with Archon RAG query results

**Validation:**
- [ ] Archon availability checked via MCP tool detection
- [ ] Relevant source identified (e.g., "Supabase Documentation" source_id)
- [ ] Query sent: "[Library] tools patterns best practices authentication"
- [ ] Results extracted and formatted into specialist sections
- [ ] Knowledge source documented: "Retrieved via Archon RAG ([Source Name], 2025-10-25)"
- [ ] Query completes in <60 seconds

**Priority:** Should Have

---

### AC-FEAT-003-005: WebSearch Fallback Auto-Population

**Given** Archon MCP is unavailable or lacks library documentation coverage
**When** specialist auto-population research phase executes
**Then** specialist instructions populated with WebSearch query results

**Validation:**
- [ ] WebSearch query sent: "[Library] documentation getting started best practices 2025"
- [ ] Results extracted from official documentation sources when possible
- [ ] Unofficial sources (blogs, Medium) used only if official docs unavailable
- [ ] Knowledge source documented: "Retrieved via WebSearch ([URL], 2025-10-25)"
- [ ] Query completes in <60 seconds

**Priority:** Must Have

---

### AC-FEAT-003-006: Specialist Review and Edit Before Finalization

**Given** specialist auto-population completes
**When** specialist file is written
**Then** user is prompted to review and can edit specialist instructions before finalizing

**Validation:**
- [ ] User sees summary of populated sections
- [ ] User can approve as-is or request to edit
- [ ] If edit requested, file opened for manual review
- [ ] User can modify any section before confirming
- [ ] After user confirmation, specialist is considered "active"

**Priority:** Should Have

---

### AC-FEAT-003-007: Specialist Invocation by Explorer Agent

**Given** specialist exists at .claude/subagents/[library-name]-specialist.md and user is in /explore workflow
**When** Explorer needs domain-specific clarification during discovery phase
**Then** Explorer invokes specialist via Task tool: @.claude/subagents/[library-name]-specialist.md

**Validation:**
- [ ] Explorer references specialist in invocation: "@.claude/subagents/supabase-specialist.md"
- [ ] Question passed to specialist is specific and domain-focused
- [ ] Specialist returns findings to Explorer (not directly to user)
- [ ] Explorer incorporates specialist findings into PRD

**Priority:** Should Have

---

### AC-FEAT-003-008: Specialist Invocation by Researcher Agent

**Given** specialist exists and user is in research phase after /explore
**When** Researcher encounters domain-specific questions from PRD
**Then** Researcher invokes specialist via Task tool for deep-dive investigation

**Validation:**
- [ ] Researcher identifies which specialist to invoke based on library/framework in PRD
- [ ] Question passed to specialist is research-focused
- [ ] Specialist returns findings with sources cited
- [ ] Researcher incorporates specialist findings into research.md with attribution

**Priority:** Must Have

---

### AC-FEAT-003-009: Specialist Knowledge Cascade (Archon → Web → User)

**Given** specialist is invoked with a domain-specific question
**When** specialist gathers knowledge to answer
**Then** specialist tries Archon RAG first, falls back to WebSearch, then asks user as last resort

**Validation:**
- [ ] Step 1: Specialist checks Archon MCP availability
- [ ] Step 2: If available, queries Archon with targeted search
- [ ] Step 3: If Archon lacks answer, queries WebSearch with refined terms
- [ ] Step 4: If both sources insufficient, formulates specific question for user
- [ ] Knowledge source documented in response: "(Found via Archon RAG)" or "(Found via WebSearch)" or "(User provided)"
- [ ] Specialist prioritizes Archon over Web for speed and reliability

**Priority:** Must Have

---

### AC-FEAT-003-010: Specialist Findings Documentation

**Given** specialist completes knowledge gathering and answers question
**When** specialist returns findings to calling agent
**Then** findings include answer, source attribution, and confidence level

**Validation:**
- [ ] Answer is domain-specific and actionable
- [ ] Source cited with URL or Archon source name
- [ ] Retrieval method documented (RAG/Web/User)
- [ ] Confidence level indicated if answer is partial or uncertain
- [ ] Calling agent receives structured response, not raw data dump

**Priority:** Must Have

---

## Edge Cases & Error Handling

### AC-FEAT-003-101: Multiple Libraries Mentioned in Single Response

**Scenario:** User mentions multiple libraries in one answer during Explorer Q&A

**Given** user says "We'll use Supabase for database and FastAPI for backend"
**When** Explorer detects multiple library mentions
**Then** Explorer prompts for each specialist separately or offers batch creation

**Validation:**
- [ ] Explorer detects both "Supabase" and "FastAPI"
- [ ] Prompt: "Create specialists for: Supabase, FastAPI? (narrow/broad/skip each)"
- [ ] User can choose scope individually per library or skip individually
- [ ] Creation workflows execute sequentially (not in parallel to avoid conflicts)

**Priority:** Should Have

---

### AC-FEAT-003-102: Specialist Already Exists for Library

**Scenario:** User mentions library that already has specialist from previous feature

**Given** specialist file exists at .claude/subagents/supabase-specialist.md
**When** user mentions "Supabase" in new /explore session
**Then** Explorer skips creation prompt and uses existing specialist

**Validation:**
- [ ] Explorer checks for existing specialist file before prompting
- [ ] No duplicate specialist created
- [ ] Explorer notifies user: "Using existing Supabase Specialist"
- [ ] Existing specialist invoked normally during workflow

**Priority:** Must Have

---

### AC-FEAT-003-103: Auto-Population Research Fails (No Results)

**Scenario:** Archon and WebSearch both fail to return useful content

**Given** specialist creation for obscure or custom library
**When** Archon query returns no results and WebSearch finds no documentation
**Then** specialist created with minimal static template, user prompted to populate manually

**Validation:**
- [ ] Error message: "Could not auto-populate [Library] Specialist. Minimal template created."
- [ ] Specialist file contains TODO sections for user to complete
- [ ] User can choose to populate now, populate later, or cancel creation
- [ ] If canceled, specialist file deleted (cleanup)

**Priority:** Must Have

---

### AC-FEAT-003-104: Specialist Invocation When File Missing

**Scenario:** Agent tries to invoke specialist but file doesn't exist (deleted or moved)

**Given** Researcher attempts to invoke @.claude/subagents/supabase-specialist.md
**When** file does not exist at that path
**Then** Researcher gracefully handles error and falls back to direct research

**Validation:**
- [ ] Error caught: "Supabase Specialist not found"
- [ ] Researcher continues with standard research workflow (no specialist)
- [ ] User notified: "Specialist missing, using standard research"
- [ ] Research completes successfully without specialist input

**Priority:** Must Have

---

### AC-FEAT-003-105: User Skips Specialist Creation, Changes Mind Later

**Scenario:** User skips specialist creation initially but wants to create later

**Given** user skipped specialist creation during /explore
**When** user is in later workflow phase (research, planning)
**Then** user can manually trigger specialist creation (future enhancement documented)

**Validation:**
- [ ] Skip decision recorded for current session
- [ ] Specialist can be created manually by directly editing .claude/subagents/ directory
- [ ] TEMPLATE.md available for manual specialist creation reference
- [ ] Documentation notes: "Future: /create-specialist command for post-explore creation"

**Priority:** Nice to Have

---

## Non-Functional Requirements

### AC-FEAT-003-201: Creation Performance

**Requirement:** Specialist creation completes in <2 minutes from user confirmation

**Criteria:**
- **Static Scaffolding:** <30 seconds (load TEMPLATE.md, substitute placeholders, write file structure)
- **Research Auto-Population:** <90 seconds total (Archon query <30s, WebSearch query <40s, content extraction <20s)
- **Total Creation Time:** <120 seconds measured from user "Yes" to file write complete
- **Resource Usage:** Minimal memory/CPU, no blocking operations

**Validation:**
- [ ] Timed test of full creation workflow <120 seconds
- [ ] Archon query optimization (targeted searches, not broad)
- [ ] WebSearch query optimization (specific URLs when possible)
- [ ] No user waiting indicators needed (completes before impatience threshold)

---

### AC-FEAT-003-202: Specialist File Quality

**Requirement:** Auto-populated specialists contain comprehensive, accurate instructions

**Criteria:**
- **Accuracy:** Auto-populated content factually correct (verified against sources)
- **Completeness:** All required sections populated (Primary Objective, Core Responsibilities, Tools Access, Workflow, Quality Criteria)
- **Relevance:** Content specific to library/framework, not generic boilerplate
- **Citations:** All auto-populated content has source attribution

**Validation:**
- [ ] Manual review of 3 test specialists (Supabase, FastAPI, PydanticAI)
- [ ] All sections contain library-specific content
- [ ] No placeholder text or TODO markers in auto-populated sections
- [ ] Sources cited for every auto-populated fact

---

### AC-FEAT-003-203: Graceful Archon Degradation

**Requirement:** System works with or without Archon MCP available

**Criteria:**
- **Archon Available:** Specialists use RAG for faster, more reliable research
- **Archon Unavailable:** Specialists fall back to WebSearch seamlessly
- **No Breaking:** Archon unavailability does not block specialist creation or invocation
- **User Awareness:** User notified of knowledge sources used

**Validation:**
- [ ] Test with Archon MCP enabled: specialists use RAG
- [ ] Test with Archon MCP disabled: specialists use WebSearch
- [ ] Both scenarios complete successfully
- [ ] User sees knowledge source in specialist responses

---

### AC-FEAT-003-204: Specialist File Naming and Storage

**Requirement:** Specialists stored consistently in .claude/subagents/ with unique names

**Criteria:**
- **Naming Convention:** kebab-case derived from library name (e.g., "pydantic-ai-specialist.md" for "PydanticAI")
- **Storage Location:** .claude/subagents/ directory (created if missing)
- **Uniqueness:** No duplicate specialist files for same library
- **Discoverability:** All specialists visible via file system and @import references

**Validation:**
- [ ] Specialist files follow kebab-case naming
- [ ] Files written to correct directory
- [ ] Duplicate detection prevents overwrites
- [ ] Specialists can be referenced via @.claude/subagents/[name]-specialist.md

---

## Integration Requirements

### AC-FEAT-003-301: Explorer Agent Integration

**Requirement:** Explorer agent extended to detect libraries and suggest specialist creation

**Given** /explore workflow is running
**When** library mentioned during discovery Q&A
**Then** Explorer integration activates without breaking existing functionality

**Validation:**
- [ ] Library detection logic added to Explorer agent instructions
- [ ] Specialist creation prompt inserted after PRD, before Researcher invocation
- [ ] Existing /explore workflows unaffected if no libraries mentioned
- [ ] Explorer can invoke specialists during discovery if they exist

---

### AC-FEAT-003-302: Researcher Agent Integration

**Requirement:** Researcher agent can invoke specialists for domain-specific research

**Given** research phase after /explore completion
**When** Researcher identifies domain-specific questions from PRD
**Then** Researcher invokes relevant specialists via Task tool

**Validation:**
- [ ] Researcher checks for available specialists in .claude/subagents/ directory
- [ ] Researcher matches questions to specialist domains (e.g., Supabase questions → Supabase Specialist)
- [ ] Researcher incorporates specialist findings into research.md with attribution
- [ ] Research workflow completes successfully with or without specialist input

---

### AC-FEAT-003-303: Documenter Agent Integration

**Requirement:** Documenter agent indexes active specialists in docs/README.md

**Given** specialists exist in .claude/subagents/ directory
**When** /update-docs runs
**Then** Documenter lists all specialists in "Active Specialists" section

**Validation:**
- [ ] Documenter detects specialist files (suffix: "-specialist.md")
- [ ] docs/README.md includes "Active Specialists" section
- [ ] Each specialist listed with name, scope, and creation date
- [ ] Specialist count shown in documentation index header

---

## Data Requirements

### AC-FEAT-003-401: TEMPLATE.md Requirements

**Requirement:** TEMPLATE.md provides comprehensive base structure for specialists

**Criteria:**
- **Location:** .claude/subagents/TEMPLATE.md (must exist)
- **Placeholders:** {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}}, {{TOOLS}}, {{OBJECTIVE}}
- **Structure:** All required sections for agent instructions (Primary Objective, Core Responsibilities, Tools Access, Workflow, Quality Criteria, Integration Points, Guardrails)
- **Completeness:** No missing sections that would break specialist functionality

**Validation:**
- [ ] TEMPLATE.md exists and is readable
- [ ] All placeholders defined and documented
- [ ] Template follows same structure as existing agents (Explorer, Researcher, etc.)
- [ ] Template can be loaded and substituted successfully

---

### AC-FEAT-003-402: Library Detection Patterns

**Requirement:** Maintain curated list of well-known library/framework patterns for regex detection

**Criteria:**
- **Coverage:** Includes most common frameworks: Supabase, FastAPI, PydanticAI, Django, React, PostgreSQL, Prisma, Next.js, TypeScript, Tailwind
- **Accuracy:** Patterns avoid false positives (e.g., "super" doesn't match "Supabase")
- **Extensibility:** Easy to add new patterns as frameworks emerge
- **Documentation:** Pattern list documented in Explorer agent instructions

**Validation:**
- [ ] Regex patterns defined for 10+ common libraries
- [ ] Patterns tested against sample text for precision/recall
- [ ] False positive rate <5% on test cases
- [ ] New patterns can be added by editing Explorer instructions

---

## Acceptance Checklist

### Development Complete
- [ ] All functional criteria met (AC-FEAT-003-001 through AC-FEAT-003-010)
- [ ] All edge cases handled (AC-FEAT-003-101 through AC-FEAT-003-105)
- [ ] Non-functional requirements met (AC-FEAT-003-201 through AC-FEAT-003-204)
- [ ] Integration requirements met (AC-FEAT-003-301 through AC-FEAT-003-303)
- [ ] Data requirements met (AC-FEAT-003-401 through AC-FEAT-003-402)

### Testing Complete
- [ ] Unit tests written and passing (template loading, substitution, file writing)
- [ ] Integration tests written and passing (Explorer workflow, Researcher workflow, Documenter indexing)
- [ ] Manual testing completed per manual-test.md
- [ ] Performance testing: creation time <2 minutes validated
- [ ] Graceful degradation tested: Archon enabled and disabled scenarios

### Documentation Complete
- [ ] Specialist creation workflow documented in CLAUDE.md or SOP
- [ ] TEMPLATE.md created and documented
- [ ] Explorer agent instructions updated with library detection logic
- [ ] Researcher agent instructions updated with specialist invocation guidance
- [ ] Documenter agent instructions updated with specialist indexing logic
- [ ] docs/README.md includes "Active Specialists" section template

### Deployment Ready
- [ ] Code reviewed and approved
- [ ] All tests passing
- [ ] No breaking changes to existing /explore, /plan workflows
- [ ] Test case validated: Database Specialist created for Supabase and invoked successfully
- [ ] Rollback plan: remove specialist files, revert agent instruction changes

---

**Appended to `/AC.md`:** No (will be appended after validation)
**Next Steps:** Proceed to testing strategy and test stub generation
