# Acceptance Criteria: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Created:** 2025-11-20
**Status:** Planning
**Related:** `architecture.md`, `testing.md`, `prd.md`

## User Story 1: Generate Messages with Block Kit

**As a** workflow builder or Python developer
**I want** Claude to generate valid Block Kit JSON for Slack messages
**So that** I can quickly create rich interactive notifications without memorizing Block Kit syntax

### AC-FEAT-004-001: Basic Message Generation
**Given** I ask Claude to generate a message with sections and buttons
**When** Claude generates Block Kit JSON using the Skill
**Then** the JSON is syntactically valid (passes JSON parser)
**And** the JSON validates in Block Kit Builder without errors
**And** the JSON uses only block types valid for message surfaces

### AC-FEAT-004-002: Message Block Limit
**Given** I request a complex message with many blocks
**When** Claude generates the JSON
**Then** the Skill warns if block count approaches or exceeds 50 blocks (message limit)
**And** the JSON structure respects the 50-block maximum
**And** suggestions provided for condensing content if over limit

### AC-FEAT-004-003: All 13 Block Types Coverage
**Given** I ask for any of the 13 official Slack Block Kit block types (section, header, image, video, markdown, rich_text, divider, context, file, table, actions, input, context_actions)
**When** Claude generates JSON using the Skill
**Then** the Skill provides correct syntax for the requested block type
**And** the block renders correctly in Block Kit Builder
**And** examples are available in supporting documentation

> **Clarification for future agents:** Block Kit has 13 official block types. Do NOT confuse with "elements" (buttons, select menus, date pickers, etc.) which are components that live *inside* blocks like `actions` or `input`. The 13 blocks are the container layer; elements are the interactive components within them.

## User Story 2: Generate Modals with Forms

**As a** workflow builder or Python developer
**I want** Claude to generate Block Kit JSON for modals with input fields
**So that** I can collect user input through Slack interactive forms

### AC-FEAT-004-004: Single-Step Modal Generation
**Given** I request a modal with input blocks and buttons
**When** Claude generates the modal JSON
**Then** the JSON includes valid `type: modal` structure
**And** the JSON includes `title`, `submit`, `close`, and `blocks` fields
**And** input blocks have unique `block_id` and `action_id` values
**And** the modal validates in Block Kit Builder

### AC-FEAT-004-005: Modal Block Limit
**Given** I request a modal with many input fields
**When** Claude generates the JSON
**Then** the Skill warns if block count approaches or exceeds 100 blocks (modal limit)
**And** the JSON structure respects the 100-block maximum

### AC-FEAT-004-006: Multi-Step Modal Limitation (v1)
**Given** I request a multi-step modal workflow
**When** Claude responds using the Skill
**Then** the Skill indicates multi-step modals (previous_view_id) are not supported in v1
**And** the Skill suggests single-step modal alternatives
**And** the Skill documents v2 roadmap for multi-step support

## User Story 3: Generate Home Tabs

**As a** workflow builder or Python developer
**I want** Claude to generate Block Kit JSON for Slack home tabs
**So that** I can create personalized dashboards and persistent interfaces

### AC-FEAT-004-007: Home Tab Generation
**Given** I request a home tab dashboard with multiple sections
**When** Claude generates the JSON
**Then** the JSON includes valid `type: home` structure
**And** the JSON uses blocks appropriate for home tabs (non-interactive focus)
**And** the JSON validates in Block Kit Builder
**And** the home tab respects 100-block maximum

### AC-FEAT-004-008: Home Tab Use Cases
**Given** I ask for common home tab patterns (task list, profile, onboarding)
**When** Claude generates the JSON
**Then** the Skill references examples from supporting documentation
**And** generated JSON matches appropriate use case pattern
**And** examples include real-world context (not abstract placeholders)

## User Story 4: Handle Variable Templating

**As a** N8N workflow builder or Python developer
**I want** Claude to explain and show variable placeholder syntax
**So that** I can integrate dynamic data into my Block Kit JSON

### AC-FEAT-004-009: N8N Jinja2 Variable Syntax
**Given** I mention N8N or request Jinja2 templating guidance
**When** Claude responds using the Skill
**Then** the Skill documents N8N Jinja2 syntax: `{{$node.data.field}}`
**And** the Skill shows example JSON with Jinja2 placeholders
**And** the Skill explains that N8N substitutes variables at runtime
**And** the Skill provides working example that can be tested in N8N

### AC-FEAT-004-010: Python Variable Syntax
**Given** I mention Python or request Python templating guidance
**When** Claude responds using the Skill
**Then** the Skill documents Python f-string syntax: `f"text {variable}"`
**And** the Skill documents .format() method: `"text {}".format(variable)`
**And** the Skill documents Jinja2 library option for Python
**And** the Skill shows example Python code that generates Block Kit JSON
**And** examples are runnable Python code (not pseudocode)

### AC-FEAT-004-011: Variable Substitution Clarity
**Given** I ask how to use variables in Block Kit
**When** Claude responds using the Skill
**Then** the Skill clearly states: "This Skill shows template placeholders only"
**And** the Skill clearly states: "Variable substitution happens in N8N/Python, not by Claude"
**And** the Skill provides side-by-side comparison: template → substituted result

## Edge Cases & Error Scenarios

### AC-FEAT-004-012: Invalid JSON Request Handling
**Given** I ask for JSON that violates Block Kit rules (e.g., input blocks in messages)
**When** Claude responds using the Skill
**Then** the Skill warns about the violation
**And** the Skill explains which block types are valid for which surfaces
**And** the Skill suggests valid alternatives

### AC-FEAT-004-013: Block Kit Builder Integration
**Given** I generate any Block Kit JSON with the Skill
**When** I copy the JSON into Block Kit Builder (https://api.slack.com/tools/block-kit-builder)
**Then** the JSON parses without syntax errors
**And** the JSON renders visually in the preview pane
**And** the JSON passes Block Kit Builder validation (green checkmark)

### AC-FEAT-004-014: Missing Required Fields
**Given** Claude generates Block Kit JSON
**When** the JSON includes blocks with required fields (e.g., text in section)
**Then** all required fields are present
**And** optional fields are shown with examples but not always included
**And** the Skill documents which fields are required vs optional

## Non-Functional Requirements

### AC-FEAT-004-015: Skill Size & Block Type Coverage Limit
**Given** the main SKILL.md file
**When** I count the total lines and review block type coverage
**Then** the file is at or under 500 lines
**And** supporting documentation files supplement without counting toward limit
**And** the Skill covers all 13 official Slack Block Kit block types (not confused with elements)

### AC-FEAT-004-016: No External Dependencies
**Given** the Skill is in use
**When** Claude generates Block Kit JSON
**Then** no external API calls are made
**And** no web searches are required for basic functionality
**And** all examples are embedded in Skill or supporting docs

### AC-FEAT-004-017: Multi-Model Compatibility
**Given** the Skill is tested with Haiku, Sonnet, and Opus models
**When** each model uses the Skill to generate Block Kit JSON
**Then** all models produce valid JSON that passes Block Kit Builder validation
**And** quality is consistent across models (no model-specific bugs)

### AC-FEAT-004-018: Example Accuracy
**Given** the Skill includes Block Kit examples
**When** examples are traced to their source
**Then** all examples are manually copied from Block Kit Builder
**And** all examples have been validated to render correctly
**And** no examples are synthesized or assumed (100% real examples)

## Integration Requirements

### AC-FEAT-004-019: N8N Integration Testing
**Given** JSON generated by the Skill with N8N Jinja2 variables
**When** I use the JSON in a real N8N workflow
**Then** N8N parses the JSON without errors
**And** N8N substitutes variables correctly at runtime
**And** the message/modal/home tab displays correctly in Slack

### AC-FEAT-004-020: Python Integration Testing
**Given** Python code examples from the Skill
**When** I run the Python code with test data
**Then** the code executes without syntax errors
**And** the code generates valid Block Kit JSON
**And** the JSON validates in Block Kit Builder

### AC-FEAT-004-021: Block Kit Builder Copyability
**Given** any JSON output from the Skill
**When** I copy and paste the JSON into Block Kit Builder
**Then** no manual formatting is required (JSON is properly formatted)
**And** the JSON is immediately usable (no syntax cleanup needed)
**And** Block Kit Builder link is provided in Skill output

## Progressive Disclosure Requirements

### AC-FEAT-004-022: Supporting Documentation Structure
**Given** the Skill uses progressive disclosure architecture
**When** I examine the file structure
**Then** the following supporting docs exist:
- `docs/blocks-reference.md` (all 15 block types)
- `docs/elements-guide.md` (interactive elements)
- `docs/composition-objects.md` (text objects, option groups)
- `docs/surfaces.md` (messages, modals, home tabs)
- `docs/variables-and-templating.md` (N8N and Python)
- `docs/examples.md` (real-world use cases)
- `docs/best-practices.md` (validation, accessibility)

### AC-FEAT-004-023: On-Demand Documentation Loading
**Given** a user asks about a specific block type or pattern
**When** Claude responds using the Skill
**Then** Claude references the appropriate supporting doc
**And** Claude loads only the relevant section (not entire doc)
**And** response stays focused and concise

## Summary

**Total Acceptance Criteria:** 23
**Critical Path:** AC-001, AC-004, AC-009, AC-010, AC-013, AC-015, AC-017, AC-019, AC-020
**Edge Cases:** AC-012, AC-014
**Non-Functional:** AC-015, AC-016, AC-017, AC-018
**Integration:** AC-019, AC-020, AC-021

**Pass Criteria:**
- All 23 criteria must pass for feature completion
- Block Kit Builder validation is mandatory for all generated JSON
- Multi-model testing required (Haiku, Sonnet, Opus)
- Real N8N and Python integration testing required

**Next Steps:**
1. Review and approve acceptance criteria
2. Execute architecture spike plan
3. Build Skill and supporting documentation
4. Execute testing strategy (see `testing.md`)
5. Perform manual testing (see `manual-test.md`)
