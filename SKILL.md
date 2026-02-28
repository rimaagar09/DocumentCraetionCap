---
name: jira-brd
description: >
  Use this skill whenever the user wants to: (1) CREATE a BRD (Business Requirements Document)
  from scratch or from a project description, (2) create Jira tickets, epics, or user stories
  from a BRD, requirements document, or any structured requirements input — AND push them
  directly to a Jira board via the Atlassian API or export a Jira-ready CSV/XLSX.
  Trigger for any of: "create a BRD", "write requirements", "create Jira tickets",
  "push to Jira", "upload to board", "create epics and stories", "generate user stories from BRD",
  "create sprint tickets", "set epic links", "add story points", "import to Jira",
  "create Jira backlog", "make Jira issues", or any mention of Jira combined with BRD,
  requirements, stories, or sprint. Also trigger when the user says "add to sprint",
  "upload to [BoardName]", "push tickets to [ProjectKey]", or asks to convert a
  description/document into Jira format. Always use this skill — not a generic file creator —
  when BRD creation, Jira ticket creation, Jira import, or Atlassian board management is
  involved. Full pipeline: User Input → BRD → Epics → User Stories → Jira.
  Outputs: (1) A complete BRD document (.docx), THEN (2) direct API push to Jira with Epic
  Links, Story Points, and Sprint assignment, OR (3) Jira-standard CSV + styled XLSX.
---

# BRD Creator + Jira Ticket Generator

**End-to-End Pipeline:** User Input / Project Description → BRD → Jira Epics & User Stories

This skill handles the complete product requirements lifecycle:
1. Generate a professional Business Requirements Document (BRD) from any input
2. Parse the BRD into structured Epics and User Stories
3. Push directly to Jira via API — or export as Jira-ready CSV + styled XLSX

**Every run produces:**
- A complete BRD document (Word .docx)
- Epics with correct Epic Names
- Stories with Epic Links, Story Points, Priority, Labels, Component, Fix Version/Sprint
- Either a direct API push OR a Jira-ready CSV + styled 3-sheet XLSX

---

## PHASE 0 — Understand the Request

**First, determine what the user already has:**

| User Has | Entry Point |
|----------|------------|
| Only a project idea / description | → Start at Phase 1 (Create BRD) |
| A partial feature list or PRD | → Start at Phase 1 (Enhance into BRD) |
| An existing BRD (uploaded file or pasted) | → Skip to Phase 2 (Parse BRD) |
| Existing BRD already created in this conversation | → Skip to Phase 2 (Parse BRD) |

**Ask the user (if not already clear):**
1. Project name / product name
2. What the system does (1–3 sentences)
3. Who the users are (roles/personas)
4. Key features or modules needed
5. Jira project key (e.g. `MYAPP`, `AUTH`, `CRM`)
6. Sprint name (e.g. `Sprint 1`, `Alpha Sprint`)
7. Delivery method: push to Jira via API, or export CSV/XLSX?

If the user says "just start" or provides a description, proceed with what you have — ask clarifying questions inline as needed.

---

## PHASE 1 — Create the BRD

### 1.1 BRD Document Structure

Generate a complete BRD with ALL of the following sections. Do not skip sections.

```
1. Document Control
   - Project Name, Version, Date, Author, Status, Reviewers

2. Executive Summary
   - Project overview (3–5 sentences)
   - Business problem being solved
   - Expected outcomes / success metrics

3. Project Scope
   - In-scope features and modules
   - Out-of-scope (explicitly state what is NOT included)
   - Assumptions and constraints

4. Stakeholders & User Personas
   - Stakeholder list (role, department, responsibility)
   - User personas (name, role, goals, pain points)

5. Business Requirements (BR-XXX)
   - High-level business needs
   - Each requirement: ID, description, priority, rationale

6. Functional Requirements (FR-XXX)
   - Detailed feature requirements
   - Each requirement: ID, description, priority, acceptance criteria
   - Group by module/epic

7. Non-Functional Requirements (NFR-XXX)
   - Performance, security, scalability, reliability
   - Each NFR: ID, type, description, measurable target

8. System Architecture Overview
   - High-level component diagram (described in text)
   - Integration points and external systems

9. Data Requirements
   - Key data entities
   - Data flows
   - Data retention and privacy requirements

10. User Journey / Process Flows
    - Step-by-step flows for each major use case
    - Happy path + error paths

11. Acceptance Criteria Summary
    - High-level sign-off criteria for each module

12. Risk Register
    - Risk, likelihood, impact, mitigation strategy

13. Dependencies & Timeline
    - Dependencies on other systems/teams
    - Suggested phasing (Phase 1 / Phase 2 / Phase 3)

14. Glossary
    - Key terms and abbreviations
```

### 1.2 Requirement ID Convention

Use consistent IDs throughout:
- Business Requirements: `BR-001`, `BR-002`, ...
- Functional Requirements: `FR-001`, `FR-002`, ...
- Non-Functional Requirements: `NFR-001`, `NFR-002`, ...

### 1.3 BRD Quality Rules

- Every FR must have at least 1 acceptance criterion
- Every module/feature must have at least 1 FR
- NFRs must have measurable targets (e.g., "< 3 sec response at p95", "≥ 99.5% uptime")
- Priority must be set on every requirement: **Must Have / Should Have / Nice to Have**
- Phase tagging: every requirement tagged Phase 1 / Phase 2 / Phase 3

### 1.4 Save the BRD

After generating the BRD content:
1. Read `/mnt/skills/public/docx/SKILL.md` to generate a properly formatted `.docx` file
2. Save as `[ProjectName]_BRD_v1.0.docx`
3. Present it to the user using `present_files`
4. Ask: **"BRD is ready! Shall I now generate the Jira Epics and User Stories from this BRD?"**

If the user says yes (or already requested both), continue to Phase 2.

---

## PHASE 2 — Parse BRD into Ticket Structure

### 2.1 Gather Jira Project Info

| Field | Required? | Default |
|-------|-----------|---------|
| Jira project key | Yes | Ask user |
| Sprint name | Yes | e.g. `Sprint 1` |
| Delivery method | Yes | API push OR CSV/XLSX |
| Author / reporter name | Optional | `Business Analysis Team` |

### 2.2 Extract Ticket Hierarchy

Parse the BRD and structure tickets as:

```
Epic (one per functional module/domain)
 └── Story (one per FR/NFR/UAT requirement)
      ├── Summary          → [REQ-ID] Short descriptive title
      ├── Description      → Full user story + business rules + edge cases
      ├── Acceptance Criteria
      ├── Priority         → High / Medium / Low
      ├── Story Points     → Fibonacci: 1/2/3/5/8/13
      ├── Labels           → phase-X, module-tag
      ├── Component        → module name
      └── Fix Version/Sprint
```

### 2.3 Epic Naming Rules

Group FRs into 4–8 meaningful Epics by functional domain:

✅ **Good Epic names:**
- `User Registration & Onboarding`
- `Core Transaction Processing`
- `Admin Configuration & Reporting`
- `API Integrations`
- `Security & Compliance`
- `Non-Functional Requirements`
- `UAT & Go-Live`

❌ **Avoid:**
- `FR-001 to FR-010` (not descriptive)
- `Phase 1` (too broad — use as Fix Version)
- `All Features` (meaningless)

### 2.4 User Story Writing Format

Every Story description must follow:

```
As a [role],
I want [capability],
so that [benefit].

Business Rules:
- [Rule 1]
- [Rule 2]

Edge Cases:
- [Edge case → outcome]
- [Invalid input → error message]
```

**Roles to use:**
| Role | Use When |
|------|----------|
| `end user / customer` | Core user-facing features |
| `admin / system admin` | Configuration, management panels |
| `backend system` | API, validation, processing logic |
| `business owner / manager` | Reporting, analytics, dashboards |
| `security officer` | Security, compliance, encryption |
| `QA tester` | UAT scenarios |

### 2.5 Story Point Guidelines

| Points | Complexity | Typical |
|--------|-----------|---------|
| 1 | Trivial | Config toggle, text update |
| 2 | Simple | Single API call, minor UI change |
| 3 | Standard | One integration + validation |
| 5 | Medium | Multi-step flow, conditional logic |
| 8 | Complex | Multi-system, async, retry logic |
| 13 | Very complex | Full subsystem, high uncertainty |

Rules:
- No story should exceed 13 points — split into sub-stories if larger
- UAT stories: always 1–2 points
- NFR stories: 3–5 points
- Security/fraud: 5–8 points
- Notification stories: 3 points (5 if multi-channel)

### 2.6 Priority Rules

- `High` → Must Have / blocking / Phase 1 core flow
- `Medium` → Should Have / Phase 2 / enhancement
- `Low` → Nice to Have / Phase 3 / non-blocking

---

## PHASE 3 — Choose Delivery Method

### Option A — Direct Jira API Push

Check if Atlassian MCP is connected:
- Call `atlassianUserInfo` to verify connection
- Call `getAccessibleAtlassianResources` to get `cloudId`
- Call `getVisibleJiraProjects` to find the project key
- Look up sprint ID using `searchJiraIssuesUsingJql`

If tools load → proceed to Phase 4 (API Push).
If tools fail → fall back to Option B, explain to user.

Read `references/jira-api.md` for full API call patterns.

### Option B — CSV + XLSX Export

Generate Jira-standard CSV and styled Excel workbook.
Proceed to Phase 5 (File Generation).

Read `references/jira-csv-format.md` for CSV column spec.

---

## PHASE 4 — Direct API Push to Jira

**Always create Epics FIRST, then Stories.**

Epics must exist before stories so Epic Link IDs are available.

```
For each Epic:
  1. POST /rest/api/3/issue  (issuetype: Epic)
  2. Set customfield_10011 = Epic Name (display name)
  3. Capture returned issue key (e.g. PROJ-1)

For each Story (under each Epic):
  1. POST /rest/api/3/issue  (issuetype: Story)
  2. Set customfield_10014 = Epic key captured above
  3. Set story_points / customfield_10016 = story points value
  4. Set Sprint via customfield_10020 = {id: sprintId}
  5. Set priority, labels, components, description
```

Read `references/jira-api.md` for:
- Full payload schemas
- Sprint assignment field names (vary by Jira version)
- Custom field IDs
- Error handling and retry logic

**Progress reporting:** After every 5 tickets created, report:
`"✓ Created X/Y tickets — [last ticket key] [summary]"`

---

## PHASE 5 — CSV + XLSX Export

### CSV Format (Jira Bulk Import Standard)

Standard columns — use EXACTLY these header names:

```
Issue Type | Summary | Description | Epic Name | Epic Link |
Priority | Story Points | Labels | Fix Version/s |
Acceptance Criteria | Component/s | Reporter
```

Rules:
- **Epics:** populate `Epic Name`; leave `Epic Link` empty
- **Stories:** populate `Epic Link` with the Epic Name (not key); leave `Epic Name` empty
- Jira matches `Epic Link` → `Epic Name` of the epic created in the same import batch

### XLSX Format (3 Sheets)

**Sheet 1 — Jira Import** (upload-ready)
- Exact CSV columns
- Colour-coded by Epic (each epic gets unique background colour)
- Priority tinted: High=red-tint, Medium=yellow-tint, Low=blue-tint
- Auto-filter on row 1, freeze pane at A2

**Sheet 2 — Backlog View** (human-readable)
- Columns: Epic | Type | Ticket ID | User Story | Priority | Points | Phase | Component | AC
- Epic header rows in epic colour, story rows in light gray

**Sheet 3 — Summary Dashboard**
- Title bar: navy
- Epic breakdown table: name, story count, total points, phase, component
- Phase breakdown: stories and points per phase
- Totals row in navy

Read `references/jira-xlsx.md` for full openpyxl code patterns and colour schemes.

---

## PHASE 6 — Quality Checklist

### BRD Quality
- [ ] All 14 sections present and populated
- [ ] Every FR has at least 1 acceptance criterion
- [ ] Every NFR has a measurable target
- [ ] Requirements have unique IDs (BR-XXX, FR-XXX, NFR-XXX)
- [ ] Priority set on every requirement
- [ ] Phase tagging applied

### Jira Tickets Quality
- [ ] Every Epic has a unique Epic Name
- [ ] Every Story has a non-empty Epic Link matching an Epic Name
- [ ] Story Points set on all Stories (no blanks)
- [ ] Priority set on all tickets
- [ ] Acceptance Criteria present on all Stories
- [ ] Labels applied (phase + module tags)
- [ ] Component/s set per ticket
- [ ] Fix Version/s = Sprint name on all Stories
- [ ] Reporter = correct author name
- [ ] At least 1 Story per Epic
- [ ] User story format: "As a / I want / so that"
- [ ] Summary starts with [REQ-ID], under 100 characters

---

## PHASE 7 — Final Output

### If BRD Only:
```
✅ BRD created: [ProjectName]_BRD_v1.0.docx
📄 [N] Business Requirements
📄 [N] Functional Requirements
📄 [N] Non-Functional Requirements
→ Say: "Shall I generate the Jira Epics and User Stories from this BRD?"
```

### If API Push:
```
✅ BRD created: [ProjectName]_BRD_v1.0.docx
✅ [N] Epics created in Jira
✅ [N] Stories created with Epic Links + Story Points
✅ All tickets assigned to [Sprint Name]
🔗 Board: [Board URL]
```

### If CSV/XLSX Export:
Save both files and call `present_files`:
```
[ProjectKey]_BRD_v1.0.docx
[ProjectKey]_Jira_Import.csv
[ProjectKey]_Jira_Import.xlsx
```

Then give import instructions:
1. Jira → Project Settings → Import Issues → CSV
2. Upload the CSV → map columns (auto-match headers)
3. Map `Fix Version/s` → Sprint field → select Sprint
4. Run import → verify ticket count matches

---

## Reference Files

| File | When to Read |
|------|-------------|
| `/mnt/skills/public/docx/SKILL.md` | Always — to generate the BRD .docx file |
| `references/jira-api.md` | API push — payloads, field IDs, sprint assignment, error handling |
| `references/jira-csv-format.md` | CSV export — column spec, Epic/Story linking rules, encoding |
| `references/jira-xlsx.md` | XLSX export — openpyxl patterns, colour scheme, 3-sheet structure |
| `references/story-writing.md` | User story format, AC patterns, story point estimation guide |

**Always read the relevant reference file BEFORE writing any code.**

---

## Quick Reference: Trigger Phrases

This skill should activate for ANY of these phrases:

| Phrase Type | Examples |
|-------------|---------|
| BRD creation | "create a BRD", "write requirements", "document this project", "make a requirements doc" |
| Jira from scratch | "create Jira tickets for my app", "make epics and stories for [project]" |
| Jira from BRD | "generate stories from this BRD", "convert BRD to Jira", "create tickets from requirements" |
| Push to Jira | "push to Jira", "upload to board", "create in Jira", "add to sprint" |
| Export | "export Jira CSV", "create Jira XLSX", "download tickets", "import-ready file" |
| Combined | "create BRD and Jira tickets", "end-to-end requirements", "full backlog from requirements" |
