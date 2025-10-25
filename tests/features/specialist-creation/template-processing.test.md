# Unit Tests: Template Processing

**Feature ID:** FEAT-003
**Test File:** template-processing.test.md
**Purpose:** Test template loading, placeholder substitution, and specialist file generation logic

## Test 1: Load TEMPLATE.md successfully

**Acceptance Criteria:** AC-FEAT-003-401

**Given:** TEMPLATE.md exists at .claude/subagents/TEMPLATE.md
**When:** Template loading function called
**Then:** Template content returned with all required sections intact

**TODO:** Implement test to load TEMPLATE.md, verify all sections present (Primary Objective, Core Responsibilities, Tools Access, Workflow, Quality Criteria, Integration Points, Guardrails), validate YAML frontmatter structure

---

## Test 2: Substitute placeholders correctly

**Acceptance Criteria:** AC-FEAT-003-003

**Given:** Template content with {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}} placeholders
**When:** Substitution function called with values: name="supabase", description="Supabase database specialist", library="Supabase"
**Then:** All placeholders replaced with correct values, no remaining {{ }} markers

**TODO:** Implement test to perform string substitution, verify all placeholders replaced, check no leftover {{ }} in output, validate substituted values match input

---

## Test 3: Generate kebab-case filename from library name

**Acceptance Criteria:** AC-FEAT-003-204

**Given:** Library names with various cases: "Supabase", "PydanticAI", "FastAPI", "Next.js"
**When:** Filename generation function called
**Then:** Correct kebab-case filenames: "supabase-specialist.md", "pydantic-ai-specialist.md", "fast-api-specialist.md", "next-js-specialist.md"

**TODO:** Implement test with various library name formats, verify kebab-case conversion logic, check special character handling (dots, hyphens), ensure suffix "-specialist.md" added correctly

---

## Test 4: Write specialist file to correct location

**Acceptance Criteria:** AC-FEAT-003-003

**Given:** Specialist content and filename
**When:** File write function called
**Then:** File written to .claude/subagents/[filename] with correct content

**TODO:** Implement test with mock file system, verify file path constructed correctly (.claude/subagents/ prefix), check content written matches input, validate directory created if not exists

---

## Test 5: Validate YAML frontmatter structure

**Acceptance Criteria:** AC-FEAT-003-003

**Given:** Generated specialist file content
**When:** YAML parser validates frontmatter
**Then:** Frontmatter contains required fields: name, description, tools, phase, status, color

**TODO:** Implement test to parse YAML frontmatter from specialist content, verify all required fields present, check field values are valid types (strings, arrays), ensure frontmatter is valid YAML syntax

---

**Total Test Stubs:** 5
**Status:** TODO - Awaiting Phase 2 implementation
