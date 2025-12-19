# FEAT-006: Meeting Task Review Before Creation

## Problem Statement

The current Make.com meeting analysis workflow automatically creates tasks in Notion without user review. This causes:

1. **Duplicate tasks** - Weak AI-based duplicate detection creates redundant entries
2. **Incorrect assignees** - AI guesses assignees based on transcript context, often wrong
3. **Missing project context** - Tasks created without project relations, requiring manual cleanup
4. **No opportunity for refinement** - Users discover issues only after tasks exist in Notion

## Desired Outcome

Insert a human-in-the-loop confirmation step where users can:
- Review all extracted tasks before creation
- Delete irrelevant or duplicate tasks
- Optionally edit task properties (assignee, project, description)
- Confirm creation only for approved tasks

## Current Workflow Analysis

### Existing Make.com Flow (from `make-meeting-actions-extractor.json`)

```
1. TRIGGER: Slack slash command → Modal upload transcript
2. PROCESS: OpenAI extracts/summarizes transcript (module 1)
3. VARIABLES: Store meeting_date, notion_meeting_id, slack_thread_ts (module 30)
4. NOTION SEARCH: Query existing tasks for duplicate check (module 919)
5. AGGREGATE: Build task string (module 2370)
6. PARSE JSON: Extract tasks array (module 2428)
7. ROUTER: Branch to task creation (module 3363)
   └─ FEEDER: Iterate through tasks (module 27)
      └─ SWITCH: Map owner name → Notion UUID (module 28)
         - Dan → 1b5d872b594c8131a5ef000271c071f9
         - Remko → 15ed872b594c81caa59e00022496a23b
         - Jack → 1b5d872b594c817283e300023b28eb5c
         - Default → Jack's UUID
      └─ CREATE PAGE: Add task to Notion (module 26)
         - Database: 218edda2a9a081ddb9fff4038c47a396
         - Fields: Name, Meeting relation, Due date, Assignee
         - NO PROJECT RELATION
8. AGGREGATE: Build completed tasks string (module 4858)
9. SUMMARIZE: OpenAI creates summary (module 5098)
10. SLACK: Send Block Kit message to thread (module 37)
```

### Key Observations

- **Assignees are hardcoded** in Switch module (3 people)
- **No project field** in Notion task creation
- **Tasks go directly to Notion** - no staging area
- **Final Slack message is informational only** - no interactivity

---

## Option Comparison

### Option 1: Approve/Delete Only (Simple)

**User Experience:**
1. Tasks extracted → stored temporarily
2. Slack message shows task list with Approve ✓ / Delete ✗ buttons per task
3. User clicks buttons to approve or delete each task
4. "Confirm All Approved" button creates remaining tasks
5. Confirmation message sent to thread

**Pros:**
- Minimal complexity
- Single webhook for button handling
- No modal complexity
- Fast to implement (~1 day)

**Cons:**
- No inline editing
- Assignee changes require editing in Notion after
- No project assignment capability

**Architecture:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        MODIFIED WORKFLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│ [Existing modules 1-2428] → Parse Tasks                             │
│                              ↓                                      │
│ [NEW] Data Store: Save tasks with session_id key                    │
│                              ↓                                      │
│ [NEW] Build Block Kit message with task sections + buttons          │
│                              ↓                                      │
│ [MODIFIED] Slack: Send interactive message to thread                │
│                              ↓                                      │
│                    ════════ PAUSE ════════                          │
│                              ↓                                      │
│ [NEW SCENARIO] Webhook: Receive button clicks                       │
│                              ↓                                      │
│ [NEW] Route by action_id:                                           │
│       ├─ delete_task_X → Remove from Data Store, update message     │
│       ├─ approve_task_X → Mark approved in Data Store, update msg   │
│       └─ confirm_all → Create approved tasks in Notion              │
│                              ↓                                      │
│ [Existing] Slack: Send confirmation to thread                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Option 2: Full Edit with Modal (Complex)

**User Experience:**
1. Tasks extracted → stored temporarily
2. Slack message: "8 tasks extracted from meeting - Click to Review"
3. User clicks "Review Tasks" button
4. Modal opens with all tasks listed:
   - Task description (editable text input)
   - Assignee (static dropdown: Dan, Remko, Jack)
   - Project (external_select dropdown - fetched from Notion)
   - Delete checkbox
5. User edits as needed, clicks "Create Tasks"
6. Tasks created in Notion with full properties
7. Confirmation message sent to thread

**Pros:**
- Full editing capability
- Project assignment before creation
- Single review action for all tasks
- Matches user's mental model

**Cons:**
- Two webhooks required (button + external_select)
- 3-second timeout constraint for Notion queries
- Modal block limit (100 blocks for ~8 tasks = tight)
- More failure points
- ~3 days to implement

**Architecture:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        MODIFIED WORKFLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│ [Existing modules 1-2428] → Parse Tasks                             │
│                              ↓                                      │
│ [NEW] Data Store: Save tasks with session_id key                    │
│                              ↓                                      │
│ [NEW] Build simple Block Kit message with "Review Tasks" button     │
│                              ↓                                      │
│ [MODIFIED] Slack: Send message to thread                            │
│                              ↓                                      │
│                    ════════ PAUSE ════════                          │
│                              ↓                                      │
│ [NEW SCENARIO 1] Webhook: Button click handler                      │
│                              ↓                                      │
│ [NEW] Webhook Response: Immediate 200 OK                            │
│                              ↓                                      │
│ [NEW] Data Store: Retrieve tasks by session_id                      │
│                              ↓                                      │
│ [NEW] Build modal view JSON with task form fields                   │
│                              ↓                                      │
│ [NEW] HTTP: POST to views.open API                                  │
│                              ↓                                      │
│                    ════════ PAUSE ════════                          │
│                              ↓                                      │
│ [NEW SCENARIO 2] Webhook: External select options handler           │
│       (Called when user clicks project dropdown)                    │
│                              ↓                                      │
│ [NEW] HTTP: Query Notion for active projects                        │
│                              ↓                                      │
│ [NEW] Webhook Response: Return options JSON                         │
│                              ↓                                      │
│                    ════════ PAUSE ════════                          │
│                              ↓                                      │
│ [NEW SCENARIO 3] Webhook: Modal submission handler                  │
│                              ↓                                      │
│ [NEW] Webhook Response: Immediate 200 OK                            │
│                              ↓                                      │
│ [NEW] Parse view_submission payload                                 │
│                              ↓                                      │
│ [NEW] Filter deleted tasks                                          │
│                              ↓                                      │
│ [NEW] Feeder: Iterate through approved tasks                        │
│       └─ Create Page: Add to Notion with all properties             │
│                              ↓                                      │
│ [NEW] HTTP: POST to response_url to update original message         │
│                              ↓                                      │
│ [Existing] Slack: Send confirmation to thread                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Technical Specifications

### Slack App Configuration Required (Both Options)

1. **Enable Interactivity** in Slack App settings
2. **Request URL**: Point to Make.com webhook for button handling
3. **Options Load URL** (Option 2 only): Point to Make.com webhook for external_select
4. **OAuth Scopes Required**:
   - `commands` (already have for slash command)
   - `chat:write` (already have)
   - `chat:write.public` (already have)

### Make.com Data Store Structure

**Store Name**: `meeting_tasks_staging`
**Size**: 1 MB minimum

**Record Structure**:
```json
{
  "key": "session_12345",  // Unique per meeting analysis run
  "tasks": [
    {
      "id": "task_1",
      "taskName": "Follow up with client about proposal",
      "owner": "Dan",
      "dueDate": "2024-01-15",
      "status": "pending",  // pending | approved | deleted
      "project": null       // Option 2 only
    }
  ],
  "meeting_date": "2024-01-08",
  "notion_meeting_id": "abc123",
  "slack_thread_ts": "1704715200.000100",
  "slack_channel_id": "C085F7EQJTZ",
  "response_url": "https://hooks.slack.com/actions/...",
  "created_at": "2024-01-08T10:00:00Z",
  "expires_at": "2024-01-08T10:30:00Z"  // 30 min TTL
}
```

---

## Option 1: Detailed Implementation

### Block Kit Message Structure

```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "Review Meeting Tasks",
        "emoji": true
      }
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": "8 tasks extracted. Approve or delete each task before creating in Notion."
        }
      ]
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Task 1:* Follow up with client about proposal\n:bust_in_silhouette: Dan  :calendar: Jan 15"
      },
      "accessory": {
        "type": "button",
        "text": {
          "type": "plain_text",
          "text": "Delete",
          "emoji": true
        },
        "style": "danger",
        "value": "task_1|session_12345",
        "action_id": "delete_task"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Task 2:* Schedule demo for next week\n:bust_in_silhouette: Remko  :calendar: Jan 12"
      },
      "accessory": {
        "type": "button",
        "text": {
          "type": "plain_text",
          "text": "Delete",
          "emoji": true
        },
        "style": "danger",
        "value": "task_2|session_12345",
        "action_id": "delete_task"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Create All Tasks",
            "emoji": true
          },
          "style": "primary",
          "value": "session_12345",
          "action_id": "confirm_all_tasks"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Cancel All",
            "emoji": true
          },
          "value": "session_12345",
          "action_id": "cancel_all_tasks"
        }
      ]
    }
  ]
}
```

### Button Click Webhook Payload (from Slack)

When user clicks a button, Make.com receives:

```json
{
  "type": "block_actions",
  "user": {
    "id": "UA8RXUSPL",
    "username": "dan"
  },
  "trigger_id": "12321423423.333649436676.d8c1bb837935619ccad0f624c448ffb3",
  "channel": {
    "id": "C085F7EQJTZ",
    "name": "bot-updates"
  },
  "message": {
    "ts": "1704715200.000100"
  },
  "response_url": "https://hooks.slack.com/actions/T123/456/abc",
  "actions": [
    {
      "action_id": "delete_task",
      "value": "task_1|session_12345",
      "type": "button",
      "action_ts": "1704715300.000000"
    }
  ]
}
```

### Make.com Webhook Response (within 3 seconds)

Must respond with HTTP 200 and empty body or acknowledgment:

```json
{
  "response_action": "clear"
}
```

### Message Update via response_url

After processing, POST to `response_url` with updated blocks:

```json
{
  "replace_original": true,
  "blocks": [
    // Updated block list with deleted task removed
  ]
}
```

### New Make.com Scenario: Button Handler

**Modules:**

1. **Webhooks > Custom webhook** (Trigger)
   - Name: `slack_task_button_handler`
   - URL: `https://hook.us1.make.com/xxx`

2. **Webhooks > Webhook response** (Immediate)
   - Status: 200
   - Body: `{}`

3. **JSON > Parse JSON**
   - Parse the `payload` field (Slack sends URL-encoded)

4. **Tools > Set variable**
   - Extract: `action_id`, `value`, `response_url`, `message.ts`

5. **Router** (by action_id)

   **Route A: delete_task**
   - **Data store > Update record**
     - Key: session_id from value
     - Update task status to "deleted"
   - **JSON > Create JSON**
     - Build updated blocks without deleted task
   - **HTTP > Make request**
     - URL: `{{response_url}}`
     - Method: POST
     - Body: `{"replace_original": true, "blocks": {{updated_blocks}}}`

   **Route B: confirm_all_tasks**
   - **Data store > Get record**
     - Key: session_id from value
   - **Tools > Set variable**
     - Filter tasks where status != "deleted"
   - **Flow control > Iterator**
     - Array: approved tasks
   - **Tools > Switch**
     - Map owner name to Notion UUID
   - **Notion > Create a page**
     - Database: Tasks
     - Properties: Name, Meeting, Due date, Assignee
   - **HTTP > Make request**
     - URL: `{{response_url}}`
     - Method: POST
     - Body: `{"replace_original": true, "text": "✅ {{count}} tasks created in Notion"}`
   - **Data store > Delete record**
     - Key: session_id (cleanup)

   **Route C: cancel_all_tasks**
   - **Data store > Delete record**
     - Key: session_id
   - **HTTP > Make request**
     - URL: `{{response_url}}`
     - Method: POST
     - Body: `{"replace_original": true, "text": "❌ Task creation cancelled"}`

---

## Option 2: Detailed Implementation

### Initial Message (Trigger for Modal)

```json
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*8 tasks extracted from meeting*\nReview and edit tasks before creating in Notion."
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Review Tasks",
            "emoji": true
          },
          "style": "primary",
          "value": "session_12345",
          "action_id": "open_review_modal"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Skip Review - Create All",
            "emoji": true
          },
          "value": "session_12345",
          "action_id": "create_all_immediately"
        }
      ]
    }
  ]
}
```

### Modal View Structure

**Important Constraints:**
- Max 100 blocks per modal
- For 8 tasks × ~12 blocks each = 96 blocks (at limit)
- Must include submit/cancel buttons

```json
{
  "type": "modal",
  "callback_id": "task_review_modal",
  "private_metadata": "{\"session_id\":\"session_12345\",\"response_url\":\"https://hooks.slack.com/...\"}",
  "title": {
    "type": "plain_text",
    "text": "Review Tasks"
  },
  "submit": {
    "type": "plain_text",
    "text": "Create Tasks"
  },
  "close": {
    "type": "plain_text",
    "text": "Cancel"
  },
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Task 1*"
      }
    },
    {
      "type": "input",
      "block_id": "task_1_description",
      "element": {
        "type": "plain_text_input",
        "action_id": "description_input",
        "initial_value": "Follow up with client about proposal",
        "multiline": false
      },
      "label": {
        "type": "plain_text",
        "text": "Description"
      }
    },
    {
      "type": "input",
      "block_id": "task_1_assignee",
      "element": {
        "type": "static_select",
        "action_id": "assignee_select",
        "initial_option": {
          "text": {
            "type": "plain_text",
            "text": "Dan"
          },
          "value": "1b5d872b594c8131a5ef000271c071f9"
        },
        "options": [
          {
            "text": {
              "type": "plain_text",
              "text": "Dan"
            },
            "value": "1b5d872b594c8131a5ef000271c071f9"
          },
          {
            "text": {
              "type": "plain_text",
              "text": "Remko"
            },
            "value": "15ed872b594c81caa59e00022496a23b"
          },
          {
            "text": {
              "type": "plain_text",
              "text": "Jack"
            },
            "value": "1b5d872b594c817283e300023b28eb5c"
          }
        ]
      },
      "label": {
        "type": "plain_text",
        "text": "Assignee"
      }
    },
    {
      "type": "input",
      "block_id": "task_1_project",
      "element": {
        "type": "external_select",
        "action_id": "project_select",
        "min_query_length": 0,
        "placeholder": {
          "type": "plain_text",
          "text": "Select a project"
        }
      },
      "label": {
        "type": "plain_text",
        "text": "Project"
      },
      "optional": true
    },
    {
      "type": "input",
      "block_id": "task_1_delete",
      "element": {
        "type": "checkboxes",
        "action_id": "delete_checkbox",
        "options": [
          {
            "text": {
              "type": "plain_text",
              "text": "Delete this task"
            },
            "value": "delete"
          }
        ]
      },
      "label": {
        "type": "plain_text",
        "text": "Actions"
      },
      "optional": true
    },
    {
      "type": "divider"
    }
    // Repeat for tasks 2-8...
  ]
}
```

### External Select Options Request (from Slack)

When user clicks project dropdown, Slack calls Options Load URL:

```json
{
  "type": "block_suggestion",
  "user": {
    "id": "UA8RXUSPL"
  },
  "action_id": "project_select",
  "block_id": "task_1_project",
  "value": "",
  "view": {
    "id": "V123456",
    "private_metadata": "{\"session_id\":\"session_12345\"}"
  }
}
```

### External Select Options Response (from Make.com)

Must respond within 3 seconds with:

```json
{
  "options": [
    {
      "text": {
        "type": "plain_text",
        "text": "Project Alpha"
      },
      "value": "notion_page_id_1"
    },
    {
      "text": {
        "type": "plain_text",
        "text": "Project Beta"
      },
      "value": "notion_page_id_2"
    },
    {
      "text": {
        "type": "plain_text",
        "text": "Project Gamma"
      },
      "value": "notion_page_id_3"
    }
  ]
}
```

### Modal Submission Payload (from Slack)

```json
{
  "type": "view_submission",
  "user": {
    "id": "UA8RXUSPL"
  },
  "view": {
    "id": "V123456",
    "callback_id": "task_review_modal",
    "private_metadata": "{\"session_id\":\"session_12345\",\"response_url\":\"https://hooks.slack.com/...\"}",
    "state": {
      "values": {
        "task_1_description": {
          "description_input": {
            "type": "plain_text_input",
            "value": "Follow up with client about proposal"
          }
        },
        "task_1_assignee": {
          "assignee_select": {
            "type": "static_select",
            "selected_option": {
              "value": "1b5d872b594c8131a5ef000271c071f9"
            }
          }
        },
        "task_1_project": {
          "project_select": {
            "type": "external_select",
            "selected_option": {
              "value": "notion_project_page_id"
            }
          }
        },
        "task_1_delete": {
          "delete_checkbox": {
            "type": "checkboxes",
            "selected_options": []
          }
        }
      }
    }
  }
}
```

### New Make.com Scenarios for Option 2

**Scenario 1: Button Click → Open Modal**

1. **Webhooks > Custom webhook** (Trigger)
2. **Webhooks > Webhook response** (Status: 200, Body: `{}`)
3. **JSON > Parse JSON** (Parse payload)
4. **Data store > Get record** (Get tasks by session_id)
5. **JSON > Create JSON** (Build modal view with tasks)
6. **HTTP > Make request**
   - URL: `https://slack.com/api/views.open`
   - Method: POST
   - Headers: `Authorization: Bearer {{slack_bot_token}}`
   - Body: `{"trigger_id": "{{trigger_id}}", "view": {{modal_json}}}`

**Scenario 2: External Select → Return Projects**

1. **Webhooks > Custom webhook** (Trigger)
   - Name: `slack_options_handler`
2. **HTTP > Make request**
   - URL: `https://api.notion.com/v1/databases/{{projects_db_id}}/query`
   - Method: POST
   - Headers: `Authorization: Bearer {{notion_token}}`
   - Body: `{"filter": {"property": "Status", "select": {"equals": "Active"}}}`
3. **JSON > Transform**
   - Map Notion results to Slack options format
4. **Webhooks > Webhook response**
   - Status: 200
   - Body: `{"options": {{transformed_options}}}`

**Scenario 3: Modal Submit → Create Tasks**

1. **Webhooks > Custom webhook** (Trigger)
2. **Webhooks > Webhook response** (Status: 200, Body: `{}`)
3. **JSON > Parse JSON** (Parse payload)
4. **JSON > Parse JSON** (Parse private_metadata)
5. **Tools > Set variable** (Extract response_url)
6. **Flow control > Iterator** (Loop through state.values)
7. **Router** (Check if delete checkbox selected)
   - **Skip if deleted**
   - **Otherwise:**
     - **Notion > Create a page**
       - Database: Tasks
       - Properties: Name, Meeting, Due date, Assignee, Project
8. **HTTP > Make request**
   - URL: `{{response_url}}`
   - Method: POST
   - Body: `{"replace_original": true, "text": "✅ {{count}} tasks created"}`
9. **Data store > Delete record** (Cleanup)

---

## Risk Analysis

### Option 1 Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Webhook response timeout | Low | High | Use Webhook Response module first |
| Data Store cleanup failure | Low | Low | Set TTL, run cleanup job daily |
| Message update fails | Medium | Medium | Store response_url, retry logic |

### Option 2 Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| External select timeout (3s) | **High** | High | Cache projects, use static if >3s |
| Modal block limit exceeded | Medium | High | Limit to 6-7 tasks per modal |
| Multi-scenario coordination | Medium | Medium | Use shared Data Store for state |
| Webhook response timing | Medium | High | Immediate 200 OK before processing |
| Notion API latency | Medium | High | Pre-fetch projects on modal open |

---

## Recommendation

**Start with Option 1 (Approve/Delete)** because:

1. **Solves the core problem** - prevents bad tasks from auto-creating
2. **Lower risk** - single webhook, no external_select complexity
3. **Faster implementation** - ~1 day vs ~3 days
4. **Foundation for Option 2** - Data Store pattern reusable

**Then iterate to Option 2** if you find:
- Frequent need to change assignees before creation
- Strong desire for project assignment in Slack
- Tolerance for additional maintenance complexity

### Phased Approach

**Phase 1 (Week 1):** Implement Option 1
- Data Store setup
- Interactive message with delete buttons
- Button webhook handler
- Task creation flow

**Phase 2 (Week 2+, if needed):** Upgrade to Option 2
- Add "Review Tasks" button that opens modal
- Implement modal with static assignee dropdown
- Add external_select for projects (with caching fallback)

---

## Implementation Checklist

### Prerequisites (Both Options)

- [ ] Slack App: Enable Interactivity
- [ ] Slack App: Set Request URL to Make webhook
- [ ] Make.com: Create Data Store `meeting_tasks_staging`
- [ ] Make.com: Define Data Store structure (see spec above)

### Option 1 Implementation

- [ ] Modify existing scenario: Add Data Store save after task parsing
- [ ] Create new variable: Generate unique session_id
- [ ] Build Block Kit JSON with task sections and buttons
- [ ] Update Slack message module to send interactive blocks
- [ ] Create new scenario: Button click webhook handler
- [ ] Implement delete_task route
- [ ] Implement confirm_all_tasks route
- [ ] Implement cancel_all_tasks route
- [ ] Test: Delete single task, confirm remaining
- [ ] Test: Delete all tasks
- [ ] Test: Confirm without deleting
- [ ] Test: Timeout handling

### Option 2 Additional Implementation

- [ ] Slack App: Set Options Load URL to Make webhook
- [ ] Create Scenario 2: Modal opener
- [ ] Build modal view JSON generator
- [ ] Implement views.open API call
- [ ] Create Scenario 3: External select handler
- [ ] Cache Notion projects (or use static fallback)
- [ ] Create Scenario 4: Modal submission handler
- [ ] Parse view_submission state values
- [ ] Filter deleted tasks
- [ ] Create tasks with all properties
- [ ] Test: Full flow with edits
- [ ] Test: Project dropdown performance
- [ ] Test: 8 tasks modal rendering

---

## References

- [Slack Block Kit Building Guide](https://api.slack.com/block-kit/building)
- [Slack Interactive Messages](https://api.slack.com/messaging/interactivity)
- [Slack block_actions Payload](https://docs.slack.dev/reference/interaction-payloads/block_actions-payload/)
- [Slack Button Element](https://docs.slack.dev/reference/block-kit/block-elements/button-element)
- [Slack External Select](https://docs.slack.dev/tools/node-slack-sdk/reference/types/interfaces/ExternalSelect/)
- [Make.com Data Stores](https://help.make.com/l6du-data-stores)
- [Make.com Webhook Response](https://community.make.com/t/create-a-plain-webhook-response/67640)
- [Make.com + Slack Buttons](https://community.make.com/t/slack-button-with-interaction-functionality/27631)
