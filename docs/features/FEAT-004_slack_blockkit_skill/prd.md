# Product Requirements Document: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Status:** Exploration Phase Complete
**Created:** 2025-11-20
**Last Updated:** 2025-11-20

## Problem Statement

Creating Slack messages and modals requires precise JSON formatting following the Slack Block Kit specification. Developers must manually construct deeply nested JSON objects, reference 15+ block types, handle composition objects, and manage interactive elements. This is error-prone and time-consuming, especially when building complex workflows with dynamic content.

Current workflow pain points:
- Manual JSON construction is verbose and error-prone
- Block Kit Builder is great for prototyping but requires manual JSON extraction
- Template variables for N8N (Jinja2) and Python (f-strings/format) need guidance
- Each message/modal type (approval workflow, notifications, task management) has unique patterns
- Developers must switch between multiple documentation sources and tools

## Goals & Success Criteria

### Primary Goals
- Create a Claude Skill that converts conversational descriptions → valid Slack Block Kit JSON
- Support both N8N workflows (Jinja2 variables) and Claude Code + Python (string templates)
- Provide production-ready examples for common use cases (approval workflows, notifications, task management, modals, surveys)
- Generate copyable JSON suitable for Block Kit Builder testing, N8N workflows, and production code

### Success Metrics
- Skill generates JSON that passes Block Kit Builder validation (100%)
- Skill covers all 15 block types with equal depth
- Variable templating guidance works for both N8N and Python contexts
- Examples are real, tested, and copy-pasted from Block Kit Builder
- User can go from description → working JSON → deployed message in <5 minutes

## User Stories

### Story 1: N8N Workflow Builder Creates Slack Notification
**As a** workflow builder using N8N
**I want** to create a Slack notification message with dynamic content from my workflow
**So that** I can notify team members with structured, visually appealing messages

**Acceptance Criteria:**
- [ ] Skill accepts conversational request ("Create a notification with title, description, and dismiss button")
- [ ] Outputs valid Block Kit JSON
- [ ] Includes Jinja2 template variables (e.g., `{{$node.xyz.data.field}}`)
- [ ] JSON is copyable and works directly in N8N Slack integration
- [ ] Message renders correctly in Slack when variables substituted

### Story 2: Claude Code Developer Builds Approval Workflow
**As a** Claude Code developer building a workflow with Python
**I want** to generate Block Kit JSON for approval modals with dynamic content
**So that** I can integrate Slack approvals into my Python application

**Acceptance Criteria:**
- [ ] Skill generates single-step modal JSON for approval requests
- [ ] Includes Python template guidance (f-strings, .format(), or Jinja2)
- [ ] JSON includes callback_id and action_id for form handling
- [ ] Includes validation error response examples
- [ ] User can copy JSON directly into Python code or N8N

### Story 3: Team Uses Skill for Common Message Patterns
**As a** developer on a team
**I want** access to proven examples of common Slack patterns
**So that** I can reuse tested structures instead of building from scratch

**Acceptance Criteria:**
- [ ] Skill includes real examples for: approval workflows, notifications, task management, polls, settings forms
- [ ] Examples are copied from Block Kit Builder (100% verified)
- [ ] Each example includes surface type (message/modal/home tab)
- [ ] Each example includes variable substitution guidance
- [ ] Examples cover all 15 block types across different patterns

### Story 4: User Tests Output in Block Kit Builder
**As a** developer validating Skill output
**I want** JSON that I can directly paste into Block Kit Builder
**So that** I can verify the rendering matches my intent before deployment

**Acceptance Criteria:**
- [ ] Skill output JSON is syntactically valid (no errors)
- [ ] JSON can be pasted directly into Block Kit Builder
- [ ] Block Kit Builder link is provided in Skill for easy testing
- [ ] User needs zero modifications to test the JSON

## Scope & Non-Goals

### In Scope
- All 15 Slack Block Kit block types covered equally
- Three surfaces: Messages (50 block limit), Modals (100 block limit, form patterns), Home Tabs (100 block limit, dashboard patterns)
- Interactive elements: buttons, select menus, date/time pickers, text inputs
- Composition objects: text formatting, option lists, confirm dialogs
- Variable templating for N8N (Jinja2: `{{$node.data.field}}`) and Python (f-strings, .format())
- Real, tested examples from Block Kit Builder for common use cases
- Block Kit Builder integration link for testing
- Single-step modal workflows (no multi-step flows)

### Out of Scope (v1)
- Multi-step modal flows (previous_view_id navigation)
- Deno Slack SDK-specific features
- Real-time message updates via Socket Mode
- File attachments or legacy message attachments
- Slack app configuration or manifest files
- Action/button handling code (only JSON structure)
- Slack App approval workflows
- Advanced accessibility customizations beyond Block Kit recommendations

## Constraints & Assumptions

### Technical Constraints
- JSON output must be valid and Block Kit compliant
- Variable substitution happens outside the Skill (N8N templating engine or Python code)
- Single-step modals only (multi-step deferred to v2)
- Must work with Slack's official Block Kit specification (no proprietary extensions)
- Skill must remain under 500 lines (Claude Skills best practice)

### Variable & Integration Constraints
- N8N variables use Jinja2 syntax: `{{$node.triggering_node.data.output_field}}`
- N8N templating is handled by N8N workflow engine (not Skill responsibility)
- Python variables can use f-strings, .format(), or Jinja2 library (Skill shows examples, user chooses)
- Variables are substituted at deployment time, not by Claude Skill

### Business Constraints
- Skill must support freelance developers and enterprise teams
- Output must be production-ready (no prototype-quality examples)
- All examples must be real and tested (copy-pasted from Block Kit Builder)
- Examples must NOT include variable substitution (keep examples clean/testable)
- Documentation of variable handling must be clear but separate from examples

### Assumptions
- Users understand JSON structure at a basic level
- Users have access to Block Kit Builder (free tool)
- Users can use either N8N or Python/Claude Code as deployment context
- Users can copy/paste JSON into their respective platforms
- Variable substitution is handled by their platform (N8N engine, Python string methods, or template library)

## Open Questions

1. **Variable Examples in Skill**
   - Should Skill show side-by-side comparisons of N8N vs Python variable syntax?
   - Or separate sections for each use case?
   - **Why it matters:** Affects how users find relevant guidance
   - **Who can answer:** User testing after Skill creation

2. **Home Tab Dashboard Complexity**
   - Should v1 focus on simple dashboards or support complex nested layouts?
   - **Why it matters:** Scope of home-tab examples
   - **Who can answer:** User feedback on typical use cases

3. **Real Example Population Strategy**
   - Should all 15 block types be covered in initial example set, or phased?
   - **Why it matters:** MVP vs comprehensive launch
   - **Who can answer:** User priority preferences

4. **Error Handling Documentation**
   - How detailed should validation error response examples be?
   - Should we show error patterns for every input type?
   - **Why it matters:** Affects helper documentation size
   - **Who can answer:** User experience after first implementation

## Related Context

### Why This Matters Now
- V2 improvement roadmap for AI Workflow Starter
- Increases utility of Claude Code for Slack integrations
- Bridges gap between N8N workflows and Claude Code automation
- Solves common developer frustration with Block Kit JSON manually

### Dependencies
- Slack Block Kit Documentation (in Archon MCP: source_id `bb572b8bdfc4d40c`)
- Claude Skills Best Practices documentation
- Block Kit Builder tool (free, no setup required)
- N8N Slack integration (for N8N use cases)
- Python 3.8+ with string formatting capabilities

### Future Enhancements (v2+)
- Multi-step modal workflows with view stacking
- Home tab interactive dashboards
- Rich text editor integration
- Accessibility audit helpers
- Slack Bolt SDK integration for button/action handling
- Workflow template library

## Implementation Approach

### Phase 1: Planning (This Document)
- ✅ Deep research on Slack Block Kit specification
- ✅ Research Claude Skills best practices
- ✅ Clarify variable handling (N8N vs Python)
- ✅ Define scope (15 blocks, 3 surfaces, single-step modals)
- → Next: Create planning documentation

### Phase 2: Planning Documentation
- Architecture & reference structure
- Acceptance criteria for Skill
- Testing strategy
- Manual testing procedures

### Phase 3: Skill Development & Example Collection
- Create main SKILL.md (conversational interface)
- Create supporting reference docs (blocks, elements, composition, variables)
- Manually copy real examples from Block Kit Builder
- User collects examples; Claude Code integrates them

### Phase 4: Testing & Validation
- Test with Claude Haiku, Sonnet, Opus (per best practices)
- Validate all examples in Block Kit Builder
- Test N8N variable substitution with real workflow
- Test Python variable substitution with real code
- Gather user feedback

---

## Next Steps

1. **User Review** - Review this PRD for accuracy and completeness
2. **Proceed to Research** - Research deep dives into specific Block Kit patterns if needed
3. **Proceed to Planning** - Run `/plan FEAT-004` to create comprehensive planning documentation
   - Architecture docs
   - Acceptance criteria
   - Testing strategy
   - Manual test procedures
4. **Skill Development** - Create Skill artifact with reference documentation
5. **Example Population** - User collects real examples from Block Kit Builder, we integrate them

---

**Feature Owner:** User
**Last Reviewed:** 2025-11-20
**Status:** Ready for Planning Phase
