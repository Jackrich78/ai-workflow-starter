# Acceptance Criteria (Global)

**Purpose:** Central repository of all acceptance criteria across features

**Format:** AC-FEAT-XXX-### for feature-specific criteria, AC-GLOBAL-### for project-wide standards

---

## Global Standards

### AC-GLOBAL-001: Documentation Completeness
All features must have complete planning documentation before implementation (PRD, research, architecture, acceptance, testing, manual-test).

### AC-GLOBAL-002: Template Compliance
All planning documents must follow templates exactly with all required sections present.

### AC-GLOBAL-003: Word Limits
Planning documents must respect word limits: PRD ≤800 words, research ≤1000 words, others ≤800 words.

### AC-GLOBAL-004: Test Stubs Required
All features must have test stubs generated in tests/ directory with TODO comments and acceptance criteria references.

### AC-GLOBAL-005: Reviewer Validation
All planning workflows must pass Reviewer agent validation before being marked complete.

### AC-GLOBAL-006: No Placeholders
Planning documents must not contain TODO, TBD, or placeholder content (except in test stubs).

### AC-GLOBAL-007: Given/When/Then Format
All acceptance criteria must use Given/When/Then format for testability.

---

## Feature-Specific Criteria

*Feature acceptance criteria will be appended here by the Planner agent during `/plan` command*

### FEAT-001: Example Feature

**AC-FEAT-001-001:** Example Documentation Complete
Given a developer is using the AI workflow template, when they review FEAT-001 example feature, then all 6 planning documents are present and follow templates.

**AC-FEAT-001-002:** Templates Demonstrated
Given an AI agent is generating feature documentation, when the agent references FEAT-001 as an example, then the agent can replicate the structure and quality.

---

*New feature criteria will be appended here automatically during planning*

### FEAT-003: Specialist Sub-Agent Creation System

**AC-FEAT-003-001:** Library Detection During Explorer Q&A
Given user is in /explore conversation answering Explorer's discovery questions, when user mentions a library or framework in their response (e.g., "We'll use Supabase for database"), then Explorer detects the library mention via regex pattern matching.

**AC-FEAT-003-002:** Specialist Creation Suggestion After PRD
Given Explorer has detected library mention during Q&A and completed PRD creation, when Explorer is about to invoke Researcher agent, then Explorer prompts user with specialist creation choice: "Create [Library] Specialist? (narrow/broad/skip)".

**AC-FEAT-003-003:** Hybrid Template-Based Specialist Creation in <2 Minutes
Given user confirms specialist creation with scope choice, when creation workflow executes, then specialist file created at .claude/subagents/[library-name]-specialist.md with comprehensive instructions in <2 minutes.

**AC-FEAT-003-004:** Archon RAG Auto-Population (If Available)
Given Archon MCP is available and configured with library documentation, when specialist auto-population research phase executes, then specialist instructions populated with Archon RAG query results.

**AC-FEAT-003-005:** WebSearch Fallback Auto-Population
Given Archon MCP is unavailable or lacks library documentation coverage, when specialist auto-population research phase executes, then specialist instructions populated with WebSearch query results.

**AC-FEAT-003-006:** Specialist Review and Edit Before Finalization
Given specialist auto-population completes, when specialist file is written, then user is prompted to review and can edit specialist instructions before finalizing.

**AC-FEAT-003-007:** Specialist Invocation by Explorer Agent
Given specialist exists at .claude/subagents/[library-name]-specialist.md and user is in /explore workflow, when Explorer needs domain-specific clarification during discovery phase, then Explorer invokes specialist via Task tool: @.claude/subagents/[library-name]-specialist.md.

**AC-FEAT-003-008:** Specialist Invocation by Researcher Agent
Given specialist exists and user is in research phase after /explore, when Researcher encounters domain-specific questions from PRD, then Researcher invokes specialist via Task tool for deep-dive investigation.

**AC-FEAT-003-009:** Specialist Knowledge Cascade (Archon → Web → User)
Given specialist is invoked with a domain-specific question, when specialist gathers knowledge to answer, then specialist tries Archon RAG first, falls back to WebSearch, then asks user as last resort.

**AC-FEAT-003-010:** Specialist Findings Documentation
Given specialist completes knowledge gathering and answers question, when specialist returns findings to calling agent, then findings include answer, source attribution, and confidence level.

**AC-FEAT-003-101:** Multiple Libraries Mentioned in Single Response
Given user says "We'll use Supabase for database and FastAPI for backend", when Explorer detects multiple library mentions, then Explorer prompts for each specialist separately or offers batch creation.

**AC-FEAT-003-102:** Specialist Already Exists for Library
Given specialist file exists at .claude/subagents/supabase-specialist.md, when user mentions "Supabase" in new /explore session, then Explorer skips creation prompt and uses existing specialist.

**AC-FEAT-003-103:** Auto-Population Research Fails (No Results)
Given specialist creation for obscure or custom library, when Archon query returns no results and WebSearch finds no documentation, then specialist created with minimal static template, user prompted to populate manually.

**AC-FEAT-003-104:** Specialist Invocation When File Missing
Given Researcher attempts to invoke @.claude/subagents/supabase-specialist.md, when file does not exist at that path, then Researcher gracefully handles error and falls back to direct research.

**AC-FEAT-003-105:** User Skips Specialist Creation, Changes Mind Later
Given user skipped specialist creation during /explore, when user is in later workflow phase (research, planning), then user can manually trigger specialist creation (future enhancement documented).

**AC-FEAT-003-201:** Creation Performance
Given specialist creation workflow executes, when measured from user confirmation to file write complete, then total time is less than 120 seconds with static scaffolding <30s and research auto-population <90s.

**AC-FEAT-003-202:** Specialist File Quality
Given specialist auto-population completes, when specialist file content reviewed, then all required sections populated with accurate, library-specific content and source citations.

**AC-FEAT-003-203:** Graceful Archon Degradation
Given Archon MCP is unavailable, when specialist creation or invocation occurs, then system falls back to WebSearch seamlessly without errors or workflow failures.

**AC-FEAT-003-204:** Specialist File Naming and Storage
Given specialist file is created, when filename generated, then file uses kebab-case naming derived from library name and is stored in .claude/subagents/ directory.

**AC-FEAT-003-301:** Explorer Agent Integration
Given /explore workflow is running, when library mentioned during discovery Q&A, then Explorer integration activates without breaking existing functionality.

**AC-FEAT-003-302:** Researcher Agent Integration
Given research phase after /explore completion, when Researcher identifies domain-specific questions from PRD, then Researcher invokes relevant specialists via Task tool.

**AC-FEAT-003-303:** Documenter Agent Integration
Given specialists exist in .claude/subagents/ directory, when /update-docs runs, then Documenter lists all specialists in "Active Specialists" section of docs/README.md.

**AC-FEAT-003-401:** TEMPLATE.md Requirements
Given specialist creation workflow needs base structure, when TEMPLATE.md loaded, then template contains all required placeholders ({{NAME}}, {{DESCRIPTION}}, {{LIBRARY}}, {{TOOLS}}, {{OBJECTIVE}}) and sections.

**AC-FEAT-003-402:** Library Detection Patterns
Given user response contains library name, when library detection regex applied, then well-known libraries (Supabase, FastAPI, PydanticAI, Django, React, PostgreSQL, Prisma, Next.js, TypeScript, Tailwind) detected accurately with <5% false positive rate.

---

### FEAT-004: Slack Block Kit Claude Skill

**AC-FEAT-004-001:** Basic Message Generation
Given I ask Claude to generate a message with sections and buttons, when Claude generates Block Kit JSON using the Skill, then the JSON is syntactically valid (passes JSON parser), validates in Block Kit Builder without errors, and uses only block types valid for message surfaces.

**AC-FEAT-004-002:** Message Block Limit
Given I request a complex message with many blocks, when Claude generates the JSON, then the Skill warns if block count approaches or exceeds 50 blocks (message limit), the JSON structure respects the 50-block maximum, and suggestions are provided for condensing content if over limit.

**AC-FEAT-004-003:** All 13 Block Types Coverage
Given I ask for any of the 13 official Slack Block Kit block types (section, header, image, video, markdown, rich_text, divider, context, file, table, actions, input, context_actions), when Claude generates JSON using the Skill, then the Skill provides correct syntax for the requested block type, the block renders correctly in Block Kit Builder, and examples are available in supporting documentation.

**AC-FEAT-004-004:** Single-Step Modal Generation
Given I request a modal with input blocks and buttons, when Claude generates the modal JSON, then the JSON includes valid `type: modal` structure, includes `title`, `submit`, `close`, and `blocks` fields, input blocks have unique `block_id` and `action_id` values, and the modal validates in Block Kit Builder.

**AC-FEAT-004-005:** Modal Block Limit
Given I request a modal with many input fields, when Claude generates the JSON, then the Skill warns if block count approaches or exceeds 100 blocks (modal limit), and the JSON structure respects the 100-block maximum.

**AC-FEAT-004-006:** Multi-Step Modal Limitation (v1)
Given I request a multi-step modal workflow, when Claude responds using the Skill, then the Skill indicates multi-step modals (previous_view_id) are not supported in v1, suggests single-step modal alternatives, and documents v2 roadmap for multi-step support.

**AC-FEAT-004-007:** Home Tab Generation
Given I request a home tab dashboard with multiple sections, when Claude generates the JSON, then the JSON includes valid `type: home` structure, uses blocks appropriate for home tabs (non-interactive focus), validates in Block Kit Builder, and respects 100-block maximum.

**AC-FEAT-004-008:** Home Tab Use Cases
Given I ask for common home tab patterns (task list, profile, onboarding), when Claude generates the JSON, then the Skill references examples from supporting documentation, generated JSON matches appropriate use case pattern, and examples include real-world context (not abstract placeholders).

**AC-FEAT-004-009:** N8N Jinja2 Variable Syntax
Given I mention N8N or request Jinja2 templating guidance, when Claude responds using the Skill, then the Skill documents N8N Jinja2 syntax: `{{$node.data.field}}`, shows example JSON with Jinja2 placeholders, explains that N8N substitutes variables at runtime, and provides working example that can be tested in N8N.

**AC-FEAT-004-010:** Python Variable Syntax
Given I mention Python or request Python templating guidance, when Claude responds using the Skill, then the Skill documents Python f-string syntax: `f"text {variable}"`, documents .format() method: `"text {}".format(variable)`, documents Jinja2 library option for Python, shows example Python code that generates Block Kit JSON, and examples are runnable Python code (not pseudocode).

**AC-FEAT-004-011:** Variable Substitution Clarity
Given I ask how to use variables in Block Kit, when Claude responds using the Skill, then the Skill clearly states: "This Skill shows template placeholders only", clearly states: "Variable substitution happens in N8N/Python, not by Claude", and provides side-by-side comparison: template → substituted result.

**AC-FEAT-004-012:** Invalid JSON Request Handling
Given I ask for JSON that violates Block Kit rules (e.g., input blocks in messages), when Claude responds using the Skill, then the Skill warns about the violation, explains which block types are valid for which surfaces, and suggests valid alternatives.

**AC-FEAT-004-013:** Block Kit Builder Integration
Given I generate any Block Kit JSON with the Skill, when I copy the JSON into Block Kit Builder (https://api.slack.com/tools/block-kit-builder), then the JSON parses without syntax errors, renders visually in the preview pane, and passes Block Kit Builder validation (green checkmark).

**AC-FEAT-004-014:** Missing Required Fields
Given Claude generates Block Kit JSON, when the JSON includes blocks with required fields (e.g., text in section), then all required fields are present, optional fields are shown with examples but not always included, and the Skill documents which fields are required vs optional.

**AC-FEAT-004-015:** Skill Size & Block Type Coverage Limit
Given the main SKILL.md file, when I count the total lines and review block type coverage, then the file is at or under 500 lines, supporting documentation files supplement without counting toward limit, and the Skill covers all 13 official Slack Block Kit block types (not confused with elements).

**AC-FEAT-004-016:** No External Dependencies
Given the Skill is in use, when Claude generates Block Kit JSON, then no external API calls are made, no web searches are required for basic functionality, and all examples are embedded in Skill or supporting docs.

**AC-FEAT-004-017:** Multi-Model Compatibility
Given the Skill is tested with Haiku, Sonnet, and Opus models, when each model uses the Skill to generate Block Kit JSON, then all models produce valid JSON that passes Block Kit Builder validation, and quality is consistent across models (no model-specific bugs).

**AC-FEAT-004-018:** Example Accuracy
Given the Skill includes Block Kit examples, when examples are traced to their source, then all examples are manually copied from Block Kit Builder, have been validated to render correctly, and no examples are synthesized or assumed (100% real examples).

**AC-FEAT-004-019:** N8N Integration Testing
Given JSON generated by the Skill with N8N Jinja2 variables, when I use the JSON in a real N8N workflow, then N8N parses the JSON without errors, substitutes variables correctly at runtime, and the message/modal/home tab displays correctly in Slack.

**AC-FEAT-004-020:** Python Integration Testing
Given Python code examples from the Skill, when I run the Python code with test data, then the code executes without syntax errors, generates valid Block Kit JSON, and the JSON validates in Block Kit Builder.

**AC-FEAT-004-021:** Block Kit Builder Copyability
Given any JSON output from the Skill, when I copy and paste the JSON into Block Kit Builder, then no manual formatting is required (JSON is properly formatted), the JSON is immediately usable (no syntax cleanup needed), and Block Kit Builder link is provided in Skill output.

**AC-FEAT-004-022:** Supporting Documentation Structure
Given the Skill uses progressive disclosure architecture, when I examine the file structure, then the following supporting docs exist: `docs/blocks-reference.md` (all 13 block types), `docs/elements-guide.md` (interactive elements), `docs/composition-objects.md` (text objects, option groups), `docs/surfaces.md` (messages, modals, home tabs), `docs/variables-and-templating.md` (N8N and Python), `docs/examples.md` (real-world use cases), `docs/best-practices.md` (validation, accessibility).

**AC-FEAT-004-023:** On-Demand Documentation Loading
Given a user asks about a specific block type or pattern, when Claude responds using the Skill, then Claude references the appropriate supporting doc, loads only the relevant section (not entire doc), and response stays focused and concise.

---

**Note:** This file is append-only. Criteria are added by agents during `/plan` command and should not be manually edited or removed.

**Last Updated:** 2025-11-20
