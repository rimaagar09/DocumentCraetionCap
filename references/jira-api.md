# Jira API Reference

Full patterns for pushing tickets to Jira via the Atlassian MCP and REST API.

---

## Step 1 — Verify Connection & Get Cloud ID

```javascript
// Via MCP tools (preferred)
atlassianUserInfo()                    // confirms auth
getAccessibleAtlassianResources()      // returns cloudId + site URL
```

Response shape:
```json
[{ "id": "abc123-cloud-id", "name": "yourcompany", "url": "https://yourcompany.atlassian.net" }]
```

Store `cloudId` — needed for all subsequent API calls.

---

## Step 2 — Find Project Key

```javascript
searchJiraProjects({ query: "Maruti" })
// or
getJiraProject({ projectKeyOrId: "MS" })
```

Returns: `{ key: "MS", id: "10001", name: "Maruti_S" }`

---

## Step 3 — Get Sprint ID

```javascript
getJiraSprints({ boardId: boardId })
// Returns list of sprints; find by name
```

Or search:
```javascript
searchJiraSprints({ projectKey: "MS", state: "active" })
```

Sprint object: `{ id: 42, name: "Sprint 1", state: "active" }`

Store `sprintId` (the integer) for story creation.

---

## Step 4 — Get Custom Field IDs

Story Points and Epic Link field IDs **vary per Jira instance**. Always look them up:

```javascript
getJiraIssueCreateMeta({ projectKey: "MS", issueType: "Story" })
```

Common field IDs (verify against your instance):

| Field | Common ID | Notes |
|-------|-----------|-------|
| Story Points | `customfield_10016` | Sometimes `customfield_10028` |
| Epic Link | `customfield_10014` | Classic projects |
| Epic Name | `customfield_10011` | On Epic issue type |
| Sprint | `customfield_10020` | Array of sprint objects |
| Story Points (next-gen) | `story_points` | Team-managed projects |

Always verify by calling `getJiraIssueCreateMeta` first and inspecting the response.

---

## Step 5 — Create Epic

```javascript
createJiraIssue({
  cloudId: "abc123",
  projectKey: "MS",
  summary: "Member Registration & NU Bonus",
  issueType: "Epic",
  customFields: {
    "customfield_10011": "Member Registration & NU Bonus"  // Epic Name field
  }
})
```

**Response:** `{ key: "MS-1", id: "10100" }`

Store `epicKey` (`MS-1`) for linking stories.

---

## Step 6 — Create Story with Epic Link + Story Points + Sprint

```javascript
createJiraIssue({
  cloudId: "abc123",
  projectKey: "MS",
  summary: "[FR-001] Wolf-Capillary Registration Integration",
  issueType: "Story",
  description: {
    type: "doc",
    version: 1,
    content: [{
      type: "paragraph",
      content: [{ type: "text", text: "As a new consumer, I want to register via Abbott Family Rewards..." }]
    }]
  },
  priority: "High",
  labels: ["registration", "phase-1"],
  customFields: {
    "customfield_10014": "MS-1",       // Epic Link = epic key
    "customfield_10016": 8,            // Story Points
    "customfield_10020": [{ id: 42 }]  // Sprint (use sprint ID integer)
  }
})
```

---

## Step 7 — Acceptance Criteria

Jira doesn't have a native AC field in all instances. Options:

**Option A — Description field (always works):**
Append AC to description:
```
[User story text]

*Acceptance Criteria:*
- Member created in Capillary within 3 sec
- Duplicate mobile returns existing profile
```

**Option B — Custom AC field (if exists):**
```javascript
customFields: {
  "customfield_10035": "AC text here"  // verify field ID first
}
```

Default to Option A unless user confirms custom AC field exists.

---

## Batch Creation Pattern

Always create in order:
1. All Epics first → collect `{epicName: epicKey}` map
2. All Stories → look up `epicKey` from map by `epicName`

```python
epic_key_map = {}

for epic in epics:
    result = create_epic(epic)
    epic_key_map[epic["name"]] = result["key"]

for story in stories:
    epic_key = epic_key_map[story["epic_name"]]
    create_story(story, epic_key=epic_key, sprint_id=sprint_id)
```

---

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| 400 Bad Request | Invalid field value or missing required field | Check `customfield` IDs; verify issueType spelling |
| 401 Unauthorized | Token expired or wrong cloudId | Re-auth; call `atlassianUserInfo` again |
| 403 Forbidden | User lacks create permission on project | Check project roles |
| 404 Not Found | Wrong project key or sprint ID | Use `searchJiraProjects` to verify |
| Custom field error | Field ID wrong for this instance | Call `getJiraIssueCreateMeta` and inspect |

**Retry logic:** On 429 (rate limit), wait 2 seconds and retry once.

---

## Progress Reporting Template

After every 5 tickets:
```
✓ MS-1  Member Registration & NU Bonus [Epic]
✓ MS-2  [FR-001] Wolf-Capillary Registration Integration [Story] → MS-1 | 8pts
✓ MS-3  [FR-002] New User Bonus Points [Story] → MS-1 | 5pts
...
Progress: 12/39 tickets created
```

Final summary:
```
✅ 6 Epics created
✅ 33 Stories created with Epic Links + Story Points
✅ All tickets assigned to Sprint 1
🔗 https://yourcompany.atlassian.net/jira/software/projects/MS/boards
```
