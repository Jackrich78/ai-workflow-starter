# Research Document: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Researcher:** Claude Code
**Research Date:** 2025-11-20
**Status:** Complete

## Research Summary

This document captures findings from comprehensive research into:
1. Slack Block Kit specification and architecture
2. Claude Skills best practices and design patterns
3. Variable templating patterns for N8N (Jinja2) and Python
4. Common Slack integration use cases
5. Integration patterns with existing platforms

**Key Finding:** Block Kit is a mature, well-documented JSON-based UI framework. Claude Skill can effectively transform conversational descriptions into valid JSON by understanding the 3-layer hierarchy (blocks → elements → composition objects) and providing clear templating guidance.

---

## 1. Slack Block Kit Architecture

### Core Structure (3-Layer Hierarchy)

**Layer 1: Blocks** (visual containers)
- 15 distinct block types
- Max 50 blocks in messages, 100 in modals/home tabs
- Each block has required `type` field + context-specific fields

**Layer 2: Block Elements** (interactive components)
- Buttons, select menus, date/time pickers, text inputs, radio buttons, checkboxes
- Usually placed as `accessory` field in blocks
- Can be grouped in `Actions` block for multiple elements

**Layer 3: Composition Objects** (building blocks)
- Text objects (mrkdwn or plain_text formatting)
- Option lists (for select menus, radio buttons)
- Confirm dialogs (for destructive actions)
- Used within blocks and elements

### Block Types Coverage (15 Total)

**Display Blocks:**
- `section` - Text + optional accessory element (most versatile)
- `header` - Large-sized text header
- `image` - Display image with alt text
- `video` - Embedded video player
- `markdown` - Formatted markdown (messages only)
- `rich_text` - Structured text with rich formatting
- `divider` - Visual separator

**Structure Blocks:**
- `context` - Contextual info (small text + images)
- `file` - Display remote file info
- `table` - Structured tabular data

**Interactive Blocks:**
- `actions` - Container for multiple interactive elements
- `input` - Form input with elements (modals only)

**Specialized:**
- `context_actions` - Actions as contextual info (messages only)

### Message Structure

**Message Payload Fields (Key):**
```json
{
  "text": "Fallback for accessibility/notifications",
  "blocks": [array of blocks, max 50],
  "thread_ts": "Optional: reply in thread",
  "mrkdwn": true (optional: enable markdown)
}
```

**Modal/Home Tab Payload Fields (Key):**
```json
{
  "type": "modal" or "home",
  "callback_id": "form_submission_id",
  "title": {"type": "plain_text", "text": "..."},
  "blocks": [array of blocks, max 100]
}
```

### Text Formatting Options

**Markup Styles:**
- `mrkdwn` - Markdown-like formatting (*bold*, _italic_, `code`)
- `plain_text` - Unformatted text
- `rich_text` - Slack's native rich text format

**Special Syntax:**
- Links: `<https://example.com|Click here>`
- User mentions: `<@U12345678>`
- Channel references: `<#C123456>`
- Timestamps: `<!date^1234567890^{date_pretty}^>`

---

## 2. Claude Skills Best Practices

### Structural Requirements

**File Organization:**
- Main `SKILL.md` under 500 lines (critical constraint)
- Progressive disclosure: main file references supporting docs
- Supporting docs loaded on-demand by Claude
- One level of file references (avoid nested references)

**Documentation Quality:**
- Assume Claude has foundational knowledge
- Challenge each piece of information: "Does Claude really need this?"
- Use consistent terminology throughout
- Avoid vague names ("helper", "utils") - use gerund forms ("processing-pdfs")

### Interaction Patterns

**Skill Interface:**
- User provides conversational input
- Skill provides step-by-step workflow user can follow
- Include checklists user can copy and check off
- Provide feedback loops and validation

**Output Format:**
- Generate copyable code/JSON
- Include validation scripts where applicable
- No user-decision punt—write utilities that solve problems

### Testing & Quality Gates

**Multi-Model Testing:**
- Test with Claude Haiku (budget, fast iteration)
- Test with Claude Sonnet (standard quality)
- Test with Claude Opus (complex reasoning)
- Effectiveness varies by model; design for all three

**Evaluation Criteria:**
- At least 3 real usage scenarios
- Team feedback incorporated
- Quality gates validated

---

## 3. Variable Templating Patterns

### N8N Integration Pattern (Jinja2)

**Variable Syntax:**
```
{{$node.triggering_node.data.output_field}}
{{$node.webhook.data.param}}
{{environment.variable_name}}
```

**Where Templating Happens:**
- N8N workflow engine performs substitution
- Substitution occurs at workflow execution time
- Skill shows template placeholders; N8N replaces them

**Example Usage:**
```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "Request from {{$node.data.requester_name}}"
  }
}
```
N8N substitutes at runtime → `"Request from Alice"`

### Python Integration Pattern

**String Formatting Options:**
1. **f-strings** (Python 3.6+, recommended for clarity):
   ```python
   message = f'''
   {{
     "type": "section",
     "text": {{
       "type": "mrkdwn",
       "text": "Request from {requester_name}"
     }}
   }}
   '''
   ```

2. **.format() method** (Python 3+, compatible):
   ```python
   message = '''
   {{
     "type": "section",
     "text": {{
       "type": "mrkdwn",
       "text": "Request from {requester_name}"
     }}
   }}
   '''.format(requester_name=user_name)
   ```

3. **Jinja2 library** (for consistency with N8N):
   ```python
   from jinja2 import Template
   template = Template(json_template_string)
   result = template.render(requester_name=user_name)
   ```

**Best Practice (per user):**
- Show all three options in documentation
- Let user choose based on project context
- Recommend Jinja2 for consistency across N8N and Python

### Variable Substitution Responsibility

**Critical Design Point:**
- Skill provides templates with placeholders
- Skill shows examples of variable syntax for N8N and Python
- Skill does NOT perform substitution
- Substitution is user's responsibility (N8N engine or Python code)

---

## 4. Common Slack Integration Use Cases

### Use Case 1: Approval Workflows

**Pattern:**
- Modal with request details (sections, fields)
- Input blocks for comments (optional)
- Action buttons (approve, reject)
- Callback handling for form submission

**Block Types Used:**
- `header` - Request title
- `section` with fields - Request details
- `input` - Optional comment field
- `actions` - Approve/reject buttons

**Variables Needed:**
- `request_id`, `requester_name`, `request_details`, `deadline`

### Use Case 2: Notifications

**Pattern:**
- Message with important information
- Action buttons (view, dismiss, acknowledge)
- Optional threading for replies
- Ephemeral variant for individual notifications

**Block Types Used:**
- `header` - Notification title
- `section` - Notification details
- `context` - Metadata (timestamp, source)
- `actions` - Buttons for interaction

**Variables Needed:**
- `title`, `description`, `severity`, `link_url`

### Use Case 3: Task Management

**Pattern:**
- Message or home tab showing task list
- Status indicators (completed, in progress, blocked)
- Task details with metadata
- Optional buttons for quick actions

**Block Types Used:**
- `section` with fields - Task list rows
- `context` - Status metadata
- `divider` - Visual separation
- `image` - Status badges/icons

**Variables Needed:**
- `task_id`, `task_name`, `status`, `assignee`, `due_date`

### Use Case 4: Polls & Surveys

**Pattern:**
- Modal with survey questions
- Input blocks for responses (text, options, radio, checkbox)
- Form submission handling
- Validation for required fields

**Block Types Used:**
- `header` - Survey title
- `section` - Question text
- `input` with various elements - Answer inputs
- `context` - Instructions

**Variables Needed:**
- `question_text`, `options` (for radio/checkbox), `response_id`

### Use Case 5: Settings & Configuration

**Pattern:**
- Modal for user settings
- Input blocks for configuration values
- Validation before submission
- Confirmation dialogs for destructive changes

**Block Types Used:**
- `header` - Settings title
- `input` - Settings fields
- `section` with confirm element - Confirmation dialogs
- `divider` - Section separators

**Variables Needed:**
- `setting_name`, `current_value`, `available_options`

---

## 5. Surface-Specific Patterns

### Messages
- **Limit:** 50 blocks max
- **Best for:** Notifications, announcements, updates, approval requests
- **Interactivity:** Via response_url or button callbacks
- **Threading:** Supported (thread_ts parameter)
- **Ephemeral variant:** Only to one user, session-specific

**Key Fields:**
```json
{
  "channel": "C123456",
  "text": "Fallback text",
  "blocks": [...],
  "thread_ts": "1234567890.123456"  // optional
}
```

### Modals
- **Limit:** 100 blocks max
- **Best for:** Forms, confirmations, multi-step workflows
- **Form Pattern:** Input blocks collect data, callback_id tracks submission
- **Validation:** Response action allows error display per field
- **Not supported:** Threading, ephemeral, rich media

**Key Fields:**
```json
{
  "type": "modal",
  "callback_id": "form_submission_id",
  "title": {"type": "plain_text", "text": "Form Title"},
  "submit": {"type": "plain_text", "text": "Submit"},
  "blocks": [...]
}
```

**Validation Response:**
```json
{
  "response_action": "errors",
  "errors": {
    "email_field": "Invalid email format"
  }
}
```

### Home Tabs
- **Limit:** 100 blocks max
- **Best for:** Personal dashboards, app information, quick actions
- **Interactivity:** Via button callbacks
- **Use:** views.publish API method
- **Personalization:** User-specific content

**Key Fields:**
```json
{
  "type": "home",
  "blocks": [...]
}
```

---

## 6. Interactive Element Patterns

### Button Element
```json
{
  "type": "button",
  "text": {"type": "plain_text", "text": "Click Me"},
  "action_id": "button_click_1",
  "value": "click_me_123",
  "style": "primary"  // or "danger"
}
```
- **Required:** type, text (plain_text), action_id
- **Optional:** value (payload), style, confirm

### Select Menu
```json
{
  "type": "static_select",
  "action_id": "select_action",
  "options": [
    {"text": {"type": "plain_text", "text": "Option 1"}, "value": "option_1"},
    {"text": {"type": "plain_text", "text": "Option 2"}, "value": "option_2"}
  ]
}
```
- **Types:** static_select, external_select, users_select, conversations_select
- **Required:** type, action_id, options (for static)

### Date/Time Picker
```json
{
  "type": "datepicker",
  "action_id": "date_pick",
  "initial_date": "2025-12-25"
}
```
- **Types:** datepicker, timepicker, datetimepicker
- **Optional:** initial_date/time

### Text Input
```json
{
  "type": "plain_text_input",
  "action_id": "text_input_1",
  "placeholder": {"type": "plain_text", "text": "Enter text..."},
  "multiline": false
}
```
- **Required:** type, action_id
- **Optional:** placeholder, multiline, min/max length

---

## 7. Composition Object Patterns

### Text Objects
```json
{
  "type": "mrkdwn",  // or "plain_text"
  "text": "Formatted *text* with _emphasis_"
}
```

### Option Objects (for selects, radio, checkbox)
```json
{
  "text": {"type": "plain_text", "text": "Option Label"},
  "value": "option_value"
}
```

### Confirm Dialog
```json
{
  "title": {"type": "plain_text", "text": "Are you sure?"},
  "text": {"type": "mrkdwn", "text": "This action cannot be undone."},
  "confirm": {"type": "plain_text", "text": "Yes"},
  "deny": {"type": "plain_text", "text": "No"}
}
```

---

## 8. Key Constraints & Gotchas

### JSON Structure Constraints
1. All `type` fields are required (block type, element type, text type)
2. Text in buttons/headers must use `plain_text`; content blocks use `mrkdwn` or `plain_text`
3. Max 10 fields per section block
4. Rich text blocks have different syntax than mrkdwn blocks
5. Emoji support varies by field type

### Variable Constraints
1. Variables must be substituted before sending to Slack API (no runtime evaluation by Slack)
2. Jinja2 syntax in N8N: `{{$node.data.field}}` NOT `{{ variable }}`
3. Python f-strings require careful escaping of JSON braces
4. Long variable values can break visual layouts (designer must handle wrapping)

### Interaction Constraints
1. Button callbacks require webhook endpoint or Socket Mode to receive interactions
2. Modal callbacks require webhook endpoint for view_submission handling
3. Response URLs expire after 3 seconds for initial POST
4. Ephemeral messages cannot be retrieved or updated, only deleted with delete_original

### Surface Constraints
1. Messages: 50 block limit, no input blocks
2. Modals: 100 block limit, form pattern, callback_id required
3. Home tabs: 100 block limit, personal to each user, views.publish method
4. Rich text blocks NOT supported in modals
5. File blocks only in messages, not modals

---

## 9. Block Kit Builder Integration

### What Block Kit Builder Provides
- Visual drag-and-drop interface for building messages
- Real-time JSON output generation
- Preview of rendered message
- Copy-paste friendly JSON format
- Direct link to share/prototype designs

### Workflow (User Perspective)
1. Open Block Kit Builder
2. Drag and drop blocks
3. Configure each block's content and properties
4. See JSON output update in real-time
5. Copy JSON to clipboard
6. Paste into N8N, Python code, or test in Slack app

### URL Format
```
https://api.slack.com/tools/block-kit-builder
```
Can include query parameters to pre-populate JSON:
```
https://api.slack.com/tools/block-kit-builder#%7B%22blocks%22:%5B...%5D%7D
```

---

## 10. Testing & Validation Approach

### Block Kit Builder Validation
- All JSON output must be pasteble into Block Kit Builder
- Zero errors when pasted
- Visual rendering matches intent
- All blocks display correctly

### N8N Workflow Testing
- Variables substitute correctly at N8N execution time
- Message posts to correct channel
- Buttons/actions trigger correctly
- No Slack API errors

### Python Testing
- JSON is syntactically valid
- Variable substitution works with f-strings, .format(), or Jinja2
- Message posts via python-slack-sdk or requests library
- Buttons/actions trigger correctly

### Multi-Model Testing (Claude Skills Best Practice)
- Test Skill with Haiku (budget, speed)
- Test with Sonnet (standard quality)
- Test with Opus (complex scenarios)
- Validate all three generate equivalent quality JSON

---

## 11. Related Documentation & Resources

### Official Documentation
- **Slack Block Kit:** https://docs.slack.dev/block-kit/
- **Block Types Reference:** https://docs.slack.dev/reference/block-kit/blocks
- **Block Elements:** https://docs.slack.dev/reference/block-kit/block-elements
- **Composition Objects:** https://docs.slack.dev/reference/block-kit/composition-objects
- **Surfaces (Messages, Modals, Home Tabs):** https://docs.slack.dev/surfaces

### Tools
- **Block Kit Builder:** https://api.slack.com/tools/block-kit-builder
- **Slack Web API Reference:** https://api.slack.com/methods

### Integration Platforms
- **N8N Slack Integration:** https://docs.n8n.io/nodes/n8n-nodes-base.slack/
- **Python Slack SDK:** https://github.com/slackapi/python-slack-sdk
- **Slack Bolt Framework:** https://github.com/slackapi/bolt-python

---

## Research Conclusions

### Key Takeaways

1. **Block Kit is Mature & Well-Documented**
   - Specification is clear and comprehensive
   - 15 block types cover virtually all UI needs
   - Variable substitution is external (N8N engine, Python code)

2. **Skill Design is Feasible**
   - Claude can understand JSON structure from conversational descriptions
   - 500-line limit is achievable with progressive disclosure
   - Supporting docs can provide detailed references

3. **Variable Handling is Dual-Path**
   - N8N uses Jinja2 (handled by N8N engine)
   - Python uses string formatting (handled by Python code)
   - Skill shows both patterns; user chooses

4. **Testing Strategy is Clear**
   - Block Kit Builder provides validation gate
   - Real examples from Builder ensure 100% accuracy
   - Multi-model testing ensures robustness

5. **Common Patterns are Identifiable**
   - Approval workflows (modal + form)
   - Notifications (message + buttons)
   - Task lists (dashboard + status)
   - Surveys/polls (modal + inputs)
   - Settings (modal + configuration)

### Confidence Level
**High Confidence** - All research questions answered. Skill design is sound. Ready to proceed to planning phase.

---

**Next Steps:** Review this research, then proceed to `/plan FEAT-004` for comprehensive planning documentation.
