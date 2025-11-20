# Manual Testing Guide: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Created:** 2025-11-20
**Intended Audience:** Product managers, QA testers, developers validating Skill output

## Overview

This guide provides step-by-step instructions for manually testing the Slack Block Kit Claude Skill. Since this is a Claude Skill (not a traditional application), testing focuses on:
1. **Skill Quality:** Does the Skill produce valid Block Kit JSON?
2. **External Validation:** Does the JSON work in Block Kit Builder?
3. **Real-World Integration:** Does the JSON work in N8N workflows and Python code?
4. **Usability:** Can users complete common tasks with the Skill?

**Prerequisites:**
- Access to Claude (web interface or Claude Code)
- Block Kit Builder (free web tool at https://api.slack.com/tools/block-kit-builder)
- N8N instance (local or cloud) for integration testing
- Python 3.9+ with slack_sdk library for Python integration testing
- Slack test workspace with app configured

**Estimated Time:** 45-60 minutes for all scenarios

---

## Test Setup

### Before You Begin

1. **Claude Access:** Ensure you can access Claude and load the Skill
2. **Block Kit Builder:** Have Block Kit Builder open in a browser tab
3. **Test Environment:**
   - N8N: Workflow with Slack node configured
   - Python: Script template with slack_sdk imported
   - Slack test workspace: Bot token configured

### Test Data

**For N8N Variable Substitution:**
```json
{
  "userName": "Alice Smith",
  "requestTitle": "Budget Approval",
  "amount": "$5,000",
  "urgency": "High",
  "approvalUrl": "https://example.com/approve/123"
}
```

**For Python Variable Substitution:**
```python
user_name = "Bob Johnson"
task_title = "Complete Q4 Report"
due_date = "2025-11-30"
priority = "Medium"
task_url = "https://example.com/tasks/456"
```

---

## Test Scenarios

### Test Scenario 1: Generate Simple Message

**Acceptance Criteria:** AC-FEAT-004-001, AC-FEAT-004-013

**Purpose:** Verify Skill generates syntactically valid Block Kit JSON for a simple message

**Steps:**

1. **Open Claude with the Skill loaded**
   - Expected: Skill is available and ready to use

2. **Request a simple notification message:**
   - **Prompt:** "Create a Slack notification message with a title 'New Order', description 'Order #12345 from Alice Smith', and a button to view the order"
   - Expected: Skill responds with Block Kit JSON

3. **Copy the generated JSON**
   - Expected: JSON is properly formatted and copyable

4. **Paste into Block Kit Builder**
   - Navigate to: https://api.slack.com/tools/block-kit-builder
   - Paste the JSON into the payload editor
   - Expected: No syntax errors, green checkmark in Block Kit Builder

5. **Verify visual rendering**
   - Expected: Preview pane shows:
     - Title text "New Order"
     - Description text visible
     - Button labeled "View Order" or similar

**✅ Pass Criteria:**
- JSON is syntactically valid (no errors in parser)
- Block Kit Builder shows green checkmark (validation passed)
- Visual preview matches the requested content
- JSON contains only message-valid block types (not input blocks)

**❌ Fail Scenarios:**
- JSON has syntax errors (red X in Block Kit Builder)
- Preview shows unexpected layout or missing elements
- Input blocks appear in message (invalid for messages)

---

### Test Scenario 2: Generate Modal with Form

**Acceptance Criteria:** AC-FEAT-004-004, AC-FEAT-004-005

**Purpose:** Verify Skill generates valid single-step modals with input fields

**Steps:**

1. **Request a modal with form inputs:**
   - **Prompt:** "Create a Slack approval modal with fields for: requester name (text input), amount (number), urgency (select: Low/Medium/High), and two buttons: Approve and Deny"
   - Expected: Skill responds with modal JSON

2. **Examine the JSON structure**
   - Expected: JSON includes:
     - `"type": "modal"`
     - `"title"` field with plain text
     - `"blocks"` array with input blocks
     - `"submit"` button with label
     - `"close"` button or cancel option
     - Each input has unique `block_id` and `action_id`

3. **Paste into Block Kit Builder**
   - Expected: Validation passes (green checkmark)
   - Expected: No errors about modal structure

4. **Verify form fields in preview**
   - Expected: Preview shows:
     - Title at top
     - Text input for requester
     - Number input or text input for amount
     - Dropdown/select for urgency
     - Approve and Deny buttons at bottom

5. **Check block count warning (if applicable)**
   - If Skill mentions approaching 100-block limit: Expected: Warning is clear and helpful
   - If under limit: Expected: No unnecessary warnings

**✅ Pass Criteria:**
- Modal JSON validates in Block Kit Builder
- All input fields are present with correct types
- Buttons are labeled correctly
- block_id and action_id values are present and unique
- Modal structure is valid (not a message structure)

**❌ Fail Scenarios:**
- Invalid modal structure (missing required fields)
- Duplicate block_id or action_id values
- Input blocks missing labels or action_id
- Block count exceeds 100 without warning

---

### Test Scenario 3: Generate Home Tab Dashboard

**Acceptance Criteria:** AC-FEAT-004-007, AC-FEAT-004-008

**Purpose:** Verify Skill generates valid home tab JSON with dashboard layout

**Steps:**

1. **Request a home tab dashboard:**
   - **Prompt:** "Create a Slack home tab dashboard showing: a welcome header, user's recent tasks (use task list with 3 items), and a settings button"
   - Expected: Skill responds with home tab JSON

2. **Examine the JSON structure**
   - Expected: JSON includes:
     - `"type": "home"`
     - `"blocks"` array with appropriate blocks
     - Header block for welcome message
     - Multiple section or context blocks for tasks
     - Optional button for settings

3. **Paste into Block Kit Builder**
   - Expected: Validation passes
   - Expected: Preview shows home tab layout (not message or modal)

4. **Verify visual hierarchy**
   - Expected: Preview shows:
     - Clear header section
     - Task items with proper visual separation
     - Settings button accessible

5. **Check if example referenced**
   - Expected: Skill references supporting documentation for home tab examples (if applicable)

**✅ Pass Criteria:**
- Home tab JSON validates
- Structure is appropriate for home tabs (not message or modal)
- Visual hierarchy is clear
- All blocks render correctly in preview

**❌ Fail Scenarios:**
- Invalid home tab structure
- Wrong block types for home context
- Missing required fields

---

### Test Scenario 4: Generate JSON with N8N Variables

**Acceptance Criteria:** AC-FEAT-004-009, AC-FEAT-004-019

**Purpose:** Verify Skill provides N8N Jinja2 variable syntax guidance and generates testable JSON

**Steps:**

1. **Request JSON with N8N variable placeholders:**
   - **Prompt:** "Create a Slack message for N8N that shows a budget approval request. Use variables for: {{$node.requestData.requesterName}}, {{$node.requestData.amount}}, and {{$node.requestData.urgencyLevel}}"
   - Expected: Skill responds with JSON and variable guidance

2. **Examine the variable syntax**
   - Expected: Jinja2 syntax is correct: `{{$node.xxx}}`
   - Expected: Skill explains that N8N substitutes variables at runtime
   - Expected: Skill shows what the rendered message looks like after substitution

3. **Copy the JSON**
   - Expected: JSON includes variable placeholders, not actual values

4. **Paste into N8N test workflow**
   - In N8N, configure a Slack node with the provided JSON
   - Expected: N8N doesn't report JSON errors
   - Expected: Variables appear as `{{...}}` in the node

5. **Test with sample data**
   - Use test data from setup section
   - Run the N8N workflow
   - Expected: Message posts to Slack test channel
   - Expected: Variables are substituted with actual values
   - Expected: Final message in Slack matches requested content

6. **Verify in Slack**
   - Check the test channel
   - Expected: Message displays with substituted values (not raw `{{...}}`)
   - Expected: Layout matches intended design

**✅ Pass Criteria:**
- Skill provides correct Jinja2 syntax
- JSON validates in Block Kit Builder
- N8N workflow executes without errors
- Variables substitute correctly
- Final Slack message renders correctly

**❌ Fail Scenarios:**
- Syntax errors in Jinja2 placeholders
- N8N workflow fails with JSON error
- Variables don't substitute (show raw `{{...}}`)
- Message doesn't appear in Slack

---

### Test Scenario 5: Generate Python Code with Variables

**Acceptance Criteria:** AC-FEAT-004-010, AC-FEAT-004-020

**Purpose:** Verify Skill provides Python variable guidance and generates runnable code

**Steps:**

1. **Request Python-ready JSON with variable guidance:**
   - **Prompt:** "Create Block Kit JSON for a task notification. Show me how to use Python variables (f-strings) to substitute: task_title, due_date, and priority. Also show .format() alternative."
   - Expected: Skill responds with JSON examples and Python code

2. **Examine the Python examples**
   - Expected: Shows f-string syntax: `f"Task: {task_title}"`
   - Expected: Shows .format() syntax: `"Task: {}".format(task_title)`
   - Expected: Clear explanation of when to use each

3. **Copy the Python code example**
   - Expected: Code is complete and runnable
   - Expected: Uses test data variables from setup

4. **Run the Python code**
   - Execute the provided code with test data
   - Expected: Code runs without syntax errors
   - Expected: Generates valid Block Kit JSON

5. **Validate generated JSON**
   - Pass the output JSON to Block Kit Builder
   - Expected: JSON validates (green checkmark)
   - Expected: Visual preview shows task notification

6. **Test Slack posting (optional)**
   - If slack_sdk is installed: Post the message to Slack test channel
   - Expected: Message appears in Slack
   - Expected: All variables are substituted
   - Expected: Visual appearance matches Python code intent

**✅ Pass Criteria:**
- Skill provides both f-string and .format() examples
- Python code is complete and executable
- Generated JSON is valid Block Kit JSON
- Block Kit Builder validates the output
- Variables are properly formatted

**❌ Fail Scenarios:**
- Python code has syntax errors
- Generated JSON is invalid
- Variables aren't properly substituted in examples
- Missing .format() or f-string alternatives

---

### Test Scenario 6: Coverage of All 13 Block Types

**Acceptance Criteria:** AC-FEAT-004-003, AC-FEAT-004-022

**Purpose:** Verify Skill can generate all 13 official Slack Block Kit block types

**Steps:**

1. **Request examples for each block type**
   - **Prompt:** "Show me quick examples of the following block types: section, header, image, video, markdown, rich_text, divider, context, file, table, actions, input, context_actions"
   - Expected: Skill responds with examples or references supporting docs

2. **For 3-5 selected block types, generate full JSON**
   - **Example prompts:**
     - "Create JSON with a table block showing sales data"
     - "Create JSON with a rich_text block with bold, italic, and code formatting"
     - "Create JSON with a context_actions block (for messages)"
   - Expected: Skill generates valid JSON for each

3. **Paste each into Block Kit Builder**
   - Expected: All validate successfully (green checkmarks)
   - Expected: Block type is correctly rendered in preview

4. **Check documentation references**
   - Expected: Skill mentions or references supporting docs for block types
   - Expected: Examples are provided or referenced from docs

**✅ Pass Criteria:**
- Skill can generate examples for all 13 block types
- Generated JSON validates in Block Kit Builder
- Block types render correctly in preview
- Skill documentation supports all 13 types

**❌ Fail Scenarios:**
- Skill doesn't know how to generate certain block types
- Generated JSON for a block type is invalid
- Block type isn't rendered correctly (or renders wrong type)
- Supporting documentation is missing for any block type

---

### Test Scenario 7: Error Handling - Invalid Requests

**Acceptance Criteria:** AC-FEAT-004-012

**Purpose:** Verify Skill handles invalid requests gracefully

**Steps:**

1. **Request an invalid combination (input blocks in message):**
   - **Prompt:** "Create a message with input blocks for collecting user feedback"
   - Expected: Skill warns that input blocks are only for modals
   - Expected: Skill suggests using a modal instead or alternative approach

2. **Request multi-step modal (v1 limitation):**
   - **Prompt:** "Create a multi-step modal workflow with view stacking"
   - Expected: Skill indicates this is v2 feature (not supported in v1)
   - Expected: Skill suggests single-step modal alternative

3. **Request invalid block type:**
   - **Prompt:** "Create JSON using the call block type"
   - Expected: Skill either generates valid call block or indicates it's not standard
   - Expected: Helpful guidance provided

**✅ Pass Criteria:**
- Skill provides helpful warnings for invalid combinations
- Skill doesn't generate invalid JSON
- Suggestions for alternatives are clear
- v1/v2 limitations are clearly documented

**❌ Fail Scenarios:**
- Invalid JSON is generated without warning
- Error message is unclear or unhelpful
- Skill ignores limitations and generates unsupported structures

---

### Test Scenario 8: Usability - Quick Example Request

**Acceptance Criteria:** AC-FEAT-004-023

**Purpose:** Verify Skill provides clear, actionable guidance for common use cases

**Steps:**

1. **Request a common use case:**
   - **Prompt:** "Show me an approval workflow notification for N8N"
   - Expected: Skill provides quick, relevant example
   - Expected: Response is concise and actionable

2. **Examine the example**
   - Expected: Example is real (copied from Block Kit Builder, not synthesized)
   - Expected: Variables are shown for N8N substitution
   - Expected: Instructions are clear

3. **Follow the example**
   - Copy the provided JSON
   - Paste into Block Kit Builder
   - Expected: Example validates and renders correctly

4. **Time the interaction**
   - Expected: User can go from "I need an approval workflow" to "working JSON in Block Kit Builder" in < 5 minutes

**✅ Pass Criteria:**
- Examples are real and verified
- Response is concise and actionable
- Example validates in Block Kit Builder
- User can complete task quickly

**❌ Fail Scenarios:**
- Example is synthesized or incorrect
- Instructions are unclear
- Example doesn't validate
- Takes >5 minutes to get working result

---

## Visual & UX Validation

### Skill Output Quality
- [ ] JSON is consistently formatted (proper indentation)
- [ ] JSON is copyable without formatting issues
- [ ] Explanation text is clear and concise
- [ ] Examples are relevant to the request
- [ ] Supporting documentation is referenced when appropriate

### Documentation Quality
- [ ] Supporting docs are clear and well-organized
- [ ] All 13 block types documented
- [ ] N8N and Python variable guidance is clear
- [ ] Examples are real and verified
- [ ] Documentation is easy to find and navigate

---

## Test Completion Checklist

### All Scenarios Complete
- [ ] Test Scenario 1: Simple Message - PASS / FAIL
- [ ] Test Scenario 2: Modal with Form - PASS / FAIL
- [ ] Test Scenario 3: Home Tab Dashboard - PASS / FAIL
- [ ] Test Scenario 4: N8N Variables - PASS / FAIL
- [ ] Test Scenario 5: Python Code - PASS / FAIL
- [ ] Test Scenario 6: All 13 Block Types - PASS / FAIL
- [ ] Test Scenario 7: Error Handling - PASS / FAIL
- [ ] Test Scenario 8: Usability - PASS / FAIL

### Additional Checks
- [ ] Visual & UX validation complete
- [ ] All JSON outputs validated in Block Kit Builder
- [ ] N8N integration tested (if applicable)
- [ ] Python integration tested (if applicable)

### Summary
- **Total Scenarios:** 8
- **Passed:** ___
- **Failed:** ___
- **Issues Found:** ___

**Overall Assessment:**
- [ ] ✅ Skill is ready for release
- [ ] ⚠️ Skill has minor issues (specify below)
- [ ] ❌ Skill has blocking issues (specify below)

**Issues Found (if any):**
1. [Issue 1]
2. [Issue 2]
3. [Issue 3]

**Tester Sign-off:**
- **Name:** [Tester name]
- **Date:** [Testing completion date]
- **Notes:** [Any additional observations]

---

**Next Steps:**
- If all tests pass: Skill approved for release
- If issues found: Development team will address and retest
- Update this document if Skill changes significantly

