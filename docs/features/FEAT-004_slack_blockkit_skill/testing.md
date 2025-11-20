# Testing Strategy: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Created:** 2025-11-20
**Status:** Planning
**Related:** `architecture.md`, `acceptance.md`, `manual-test.md`

## Testing Philosophy

This is a **Claude Skill**, not a code implementation, so testing focuses on:
1. **Skill Quality:** Does the Skill produce valid, correct JSON?
2. **Multi-Model Consistency:** Does the Skill work across Haiku, Sonnet, Opus?
3. **External Validation:** Does generated JSON pass Block Kit Builder validation?
4. **Real-World Integration:** Does JSON work in actual N8N workflows and Python code?

Testing is validation-centric rather than unit/integration in the traditional sense.

## Test Levels

### Level 1: Skill Quality Testing (Unit-equivalent)

**Objective:** Verify Skill produces syntactically valid, semantically correct Block Kit JSON

**Approach:**
- Test with 3 Claude models: Haiku (speed), Sonnet (balance), Opus (complexity)
- Generate JSON for 20 test scenarios covering all 13 official block types
- Validate JSON syntax with JSON parser
- Check structural requirements (required fields, proper nesting)

**Test Scenarios:**
1. Basic section block message
2. Message with buttons (using actions block with button elements)
3. Message with image and context blocks
4. Simple modal with text input (input block)
5. Complex modal with multiple input types (text, select, datepicker elements in input blocks)
6. Modal with validation requirements
7. Home tab dashboard with sections
8. Home tab with dividers and context blocks
9. Rich text block formatting
10. Header block usage
11. Video block embedding
12. File block display
13. Table block with structured data
14. Markdown block formatting (messages only)
15. Context_actions block (messages only)
16. Message approaching 50-block limit
17. Modal approaching 100-block limit
18. N8N Jinja2 variable placeholders
19. Python f-string variable guidance
20. Invalid request handling (input blocks in messages - should be modals only)

**Pass Criteria:**
- 100% valid JSON (passes JSON.parse)
- 100% structural correctness (all required fields present)
- Consistent output across all 3 models
- No hallucinated block types or fields

**Tools:**
- JSON parser (Python `json.loads()` or Node.js `JSON.parse()`)
- Custom validation script to check Block Kit schema
- Multi-model testing harness

### Level 2: External Validation (Integration-equivalent)

**Objective:** Verify generated JSON passes official Block Kit Builder validation

**Approach:**
- Copy generated JSON into Block Kit Builder (https://api.slack.com/tools/block-kit-builder)
- Verify visual rendering in preview pane
- Check for validation errors (red X) or warnings (yellow triangle)
- Document any Block Kit Builder failures for debugging

**Test Scenarios:**
- All 20 scenarios from Level 1
- Additional edge cases specific to Block Kit Builder quirks
- Cross-surface validation (message JSON fails in modal context, etc.)

**Pass Criteria:**
- 100% validation pass rate (green checkmark in Block Kit Builder)
- Visual rendering matches intent
- No warnings for best practices violations

**Tools:**
- Block Kit Builder web interface
- Screenshot/recording tool for visual validation
- Validation checklist spreadsheet

### Level 3: Real-World Integration (E2E-equivalent)

**Objective:** Verify JSON works in actual N8N workflows and Python code

**Approach:**

**N8N Testing:**
1. Create test N8N workflow with Slack node
2. Use Skill-generated JSON with Jinja2 variables
3. Run workflow with test data
4. Verify message/modal/home tab posts to Slack correctly
5. Verify variables substitute correctly
6. Test at least 5 scenarios: message, modal, home tab, complex message, error handling

**Python Testing:**
1. Create Python script using `slack_sdk` library
2. Follow Skill-generated Python examples
3. Generate Block Kit JSON with Python variables
4. Post to Slack test workspace
5. Verify rendering and interactivity
6. Test at least 5 scenarios matching N8N tests

**Pass Criteria:**
- N8N workflows execute without JSON parsing errors
- Variables substitute correctly (no raw `{{...}}` in Slack)
- Messages/modals/home tabs render correctly in Slack
- Python code runs without syntax errors
- Python-generated JSON validates in Block Kit Builder
- Interactive elements (buttons, inputs) function correctly in Slack

**Tools:**
- N8N instance (local or cloud)
- Python 3.9+ with slack_sdk library
- Slack test workspace with app configured
- Test data fixtures for variable substitution

### Level 4: Manual Testing (User Acceptance)

**Objective:** Non-technical users validate Skill usability and output quality

**Approach:**
- Follow step-by-step manual testing guide (see `manual-test.md`)
- Product managers and testers paste JSON into Block Kit Builder
- Visual validation of rendering
- Usability assessment: Is Skill easy to use? Are examples clear?

**Test Scenarios:**
- 6 comprehensive scenarios covering messages, modals, home tabs, variables, edge cases
- Each tester completes all 6 scenarios independently
- Testers record pass/fail and notes for each scenario

**Pass Criteria:**
- 90%+ tester success rate (testers can complete scenarios without help)
- 100% JSON validity (all outputs pass Block Kit Builder)
- Positive usability feedback (Skill is helpful, examples are clear)

**Tools:**
- Manual test guide (see `manual-test.md`)
- Block Kit Builder
- Test result tracking spreadsheet

## Coverage Goals

### Block Type Coverage
**Target:** 100% of 13 official Slack Block Kit block types
- section
- header
- image
- video
- markdown
- rich_text
- divider
- context
- file
- table
- actions
- input
- context_actions

**Validation:** Each block type tested in at least 2 scenarios

> **Note:** Do not confuse with "elements" (buttons, select menus, checkboxes, date pickers, etc.) which are interactive components placed *inside* blocks. Elements are not separate block types.

### Surface Coverage
**Target:** 100% of 3 surfaces
- Messages (50-block limit)
- Modals (100-block limit)
- Home Tabs (100-block limit)

**Validation:** Each surface tested with basic, complex, and edge-case scenarios

### Variable Templating Coverage
**Target:** Both N8N and Python approaches
- N8N Jinja2: `{{$node.data.field}}`
- Python f-strings: `f"text {variable}"`
- Python .format(): `"text {}".format(variable)`
- Python Jinja2 library

**Validation:** Real N8N workflow and Python code tests

### Multi-Model Coverage
**Target:** 3 Claude models
- Haiku (speed, basic scenarios)
- Sonnet (balance, most scenarios)
- Opus (complexity, edge cases)

**Validation:** All 20 Skill quality scenarios run on all 3 models

## Test Environment Setup

### Prerequisites
1. **Slack Workspace:**
   - Create test workspace or use sandbox
   - Install Slack app with permissions: `chat:write`, `views.open`, `views.publish`
   - Obtain bot token for testing

2. **N8N Instance:**
   - Local N8N (docker) or cloud instance
   - Slack node configured with bot token
   - Test workflow templates ready

3. **Python Environment:**
   - Python 3.9+
   - Install: `pip install slack_sdk`
   - Configure bot token as environment variable

4. **Block Kit Builder Access:**
   - https://api.slack.com/tools/block-kit-builder
   - No auth required for validation testing

5. **Claude Code:**
   - Skill installed in `/skills/blockkit/`
   - Supporting docs in `/skills/blockkit/docs/`

### Test Data Fixtures

**N8N Variables:**
```json
{
  "userName": "Alice Smith",
  "requestTitle": "Budget Approval",
  "amount": "$5,000",
  "urgency": "High",
  "approvalUrl": "https://example.com/approve/123"
}
```

**Python Variables:**
```python
user_name = "Bob Johnson"
task_title = "Complete Q4 Report"
due_date = "2025-11-30"
priority = "Medium"
task_url = "https://example.com/tasks/456"
```

## Test Automation

### Automated Tests (Level 1 & 2)

**Script:** `tests/skills/blockkit/skill_quality_test.py`

```python
# Pseudo-code structure
def test_skill_quality():
    for model in ['haiku', 'sonnet', 'opus']:
        for scenario in TEST_SCENARIOS:
            json_output = invoke_skill(model, scenario)
            assert is_valid_json(json_output)
            assert has_required_fields(json_output)
            assert validate_block_kit_builder(json_output)
```

**Coverage:**
- All 20 Skill quality scenarios
- All 3 models
- Automated JSON validation
- Block Kit Builder API validation (if available)

**Note:** Block Kit Builder validation may require manual step if no API available

### Manual Tests (Level 3 & 4)

**N8N Integration:** Manual execution with test workflows (documented in `manual-test.md`)

**Python Integration:** Manual execution with test scripts (documented in `manual-test.md`)

**User Acceptance:** Manual testing by product team (documented in `manual-test.md`)

## Regression Testing

**Trigger:** Any change to SKILL.md or supporting docs

**Scope:**
- Re-run all 20 Skill quality scenarios
- Re-validate 5 critical scenarios in Block Kit Builder
- Spot-check N8N and Python integration (1 scenario each)

**Frequency:**
- Before any Skill update commit
- Weekly scheduled run for drift detection

## Performance Benchmarks

**Skill Response Time:**
- Haiku: <2 seconds for basic scenarios
- Sonnet: <5 seconds for complex scenarios
- Opus: <10 seconds for edge cases

**Validation Time:**
- JSON parsing: <100ms
- Block Kit Builder validation: Manual (30 seconds per scenario)

## Known Limitations & Edge Cases

### Limitation 1: Multi-Step Modals (v1)
**Description:** v1 does not support multi-step modal workflows with `previous_view_id`
**Test Coverage:** AC-FEAT-004-006 validates Skill warns users appropriately
**Workaround:** Single-step modals with clear instructions

### Limitation 2: Variable Substitution
**Description:** Skill shows template placeholders only; substitution happens externally
**Test Coverage:** AC-FEAT-004-011 validates clarity of guidance
**Workaround:** Clear documentation in variables-and-templating.md

### Limitation 3: Block Kit Builder API Access
**Description:** No official API for automated validation
**Test Coverage:** Manual validation required for Level 2 tests
**Workaround:** Screenshot-based regression testing

## Defect Management

**Severity Levels:**
1. **Critical:** Invalid JSON, 100% failure rate, Skill unusable
2. **High:** Block Kit Builder validation failure, integration broken
3. **Medium:** Multi-model inconsistency, missing best practices
4. **Low:** Cosmetic issues, minor documentation gaps

**Acceptance Criteria:**
- Zero Critical or High severity defects at release
- <3 Medium severity defects
- Low severity defects documented as known issues

## Test Deliverables

1. **Test Results Spreadsheet:** All 20 scenarios x 3 models with pass/fail status
2. **Block Kit Builder Validation Report:** Screenshots of validation for 20 scenarios
3. **N8N Integration Report:** 5 workflow test results with screenshots
4. **Python Integration Report:** 5 script test results with output samples
5. **Manual Test Results:** User acceptance testing summary from product team
6. **Defect Log:** All issues found during testing with severity and resolution

## Next Steps

1. **Set Up Test Environment:** Slack workspace, N8N, Python, test data
2. **Create Test Automation Script:** `skill_quality_test.py` for Level 1 & 2
3. **Prepare Test Fixtures:** N8N workflows and Python scripts for Level 3
4. **Execute Skill Quality Tests:** Run 20 scenarios x 3 models
5. **Execute Block Kit Builder Validation:** Manual validation of all scenarios
6. **Execute N8N Integration Tests:** 5 workflow scenarios
7. **Execute Python Integration Tests:** 5 script scenarios
8. **Execute Manual Testing:** Product team validation (see `manual-test.md`)
9. **Compile Test Report:** All results and defects documented
10. **Sign-Off:** Product owner and tech lead approve for release

## References

- Block Kit Builder: https://api.slack.com/tools/block-kit-builder
- Slack Block Kit Docs: https://api.slack.com/block-kit
- Claude Skills Best Practices: Multi-model testing guidance
- N8N Documentation: https://docs.n8n.io/
- slack_sdk Documentation: https://slack.dev/python-slack-sdk/
- Related: `architecture.md`, `acceptance.md`, `manual-test.md`
