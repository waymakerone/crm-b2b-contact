---
sync:
  type: doc
  layer: CRM B2B Contact
build:
  status: todo
  phase: 2
  priority: P0
  depends_on: ["phase-1-pipeline-and-contacts"]
  started_at: null
  completed_at: null
---

# Phase 2: Activities & Detail Pages

**Goal:** Rich detail pages for contacts, companies, and deals. Full activity timeline with Commander integration — log calls, create tasks, schedule meetings, add notes. Everything flows through Commander's tools.

**PRD Reference:** `docs/01-planning/product-requirements/crm-b2b-contact-prd.md` — Phase 2

---

## What to Build

### 1. Activity System

The activity system is the heart of the CRM. Every interaction with a contact or deal is logged as an activity.

**Activity types and Commander integration:**

| Type | What Happens | Commander API |
|------|-------------|---------------|
| `note` | Freeform text saved to activity timeline | None — stays in CRM |
| `call` | Log outcome + notes + duration | None — stays in CRM |
| `task` | Creates a Commander Task, stores `commander_task_id` | `commander-task-operations` → `create_task` |
| `meeting` | Creates a Commander Calendar event, stores `commander_event_id` | `commander-calendar-operations` → `create_event` |
| `email` | Logs an email sent (manual for now) | None — future Journeys integration |

**Creating a Task from CRM:**
```
POST /functions/v1/commander-task-operations
Body: {
  "action": "create_task",
  "data": {
    "title": "Follow up with Jane at Acme Corp",
    "description": "Re: Q2 expansion deal — send pricing proposal",
    "due_date": "2026-03-05",
    "priority": "high",
    "tags": ["crm", "deal:acme-q2-expansion"]
  }
}
```
The returned `task_id` is stored in `crm_activities.commander_task_id`. The task now appears on the user's Commander Taskboard alongside all their other work.

**Creating a Meeting from CRM:**
```
POST /functions/v1/commander-calendar-operations
Body: {
  "action": "create_event",
  "data": {
    "title": "Discovery call — Acme Corp",
    "start_time": "2026-03-03T14:00:00Z",
    "end_time": "2026-03-03T14:30:00Z",
    "description": "Initial discovery with Jane Smith, VP Sales"
  }
}
```
The returned `event_id` is stored in `crm_activities.commander_event_id`.

### 2. Contact Detail Page (/contacts/:id)

**Layout:**
```
┌─────────────────────────────────────────────────┐
│ [Back to Contacts]                              │
│                                                 │
│ ┌──────────────────┐  ┌──────────────────────┐  │
│ │ Contact Info      │  │ Activity Timeline    │  │
│ │ Name, email, phone│  │ [Log Call] [Task]    │  │
│ │ Company (link)    │  │ [Meeting] [Note]     │  │
│ │ Job title         │  │ [Email]              │  │
│ │ Lifecycle stage   │  │                      │  │
│ │ Lead source       │  │ ── Today ──          │  │
│ │ Owner             │  │ 🔔 Call logged (2m)  │  │
│ │ Last contacted    │  │ 📝 Note: "Interested │  │
│ │                   │  │    in Q2 expansion"  │  │
│ │ [Edit Contact]    │  │                      │  │
│ ├──────────────────┤  │ ── Yesterday ──      │  │
│ │ Deals             │  │ ✅ Task: Send deck   │  │
│ │ • Acme Q2 ($50K)  │  │ 📅 Meeting: Intro   │  │
│ │   Proposal stage   │  │                      │  │
│ │ • Acme Pilot ($5K)│  │ ── Mar 1 ──         │  │
│ │   Closed Won       │  │ 📧 Email: Welcome   │  │
│ │                   │  │                      │  │
│ │ [+ New Deal]      │  │                      │  │
│ └──────────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────┘
```

**Left column (1/3 width):**
- Contact info card with edit button
- Lifecycle stage — editable dropdown, change logs as an activity
- Company link — click to go to company detail
- Deals list — all deals for this contact, with stage badge and amount
- "New Deal" shortcut

**Right column (2/3 width):**
- Quick action buttons: Log Call, Create Task, Schedule Meeting, Add Note, Log Email
- Activity timeline: reverse chronological, grouped by date
- Each activity shows: icon, type, subject, body preview, timestamp, created by
- Tasks show completion status (synced from Commander)
- Meetings show date/time

### 3. Company Detail Page (/companies/:id)

**Layout:**
- Company info card (name, domain, industry, size, revenue band, notes)
- Contacts at this company — table with lifecycle stage badges
- Deals for this company — table with stage, amount, close date, owner
- Total revenue: sum of Closed Won deals
- "Add Contact" and "New Deal" buttons

### 4. Deal Detail Page (/deals/:id)

**Layout:**
- **Stage progression bar** at top — visual indicator of current stage through pipeline
  - Stages as connected dots/steps, current stage highlighted in gold
  - Click a stage to move the deal (with confirmation for Closed)
- Deal info: name, amount, probability, expected close, owner
- Company and primary contact (links)
- Activity timeline (same component as Contact Detail, filtered to this deal)
- Quick actions: Log Call, Task, Meeting, Note
- Edit deal button

### 5. Activity Form Components

**Log Call Dialog:**
- Fields: outcome (dropdown: connected, voicemail, no answer, wrong number), duration (minutes), notes
- Auto-sets `last_contacted_at` on the contact

**Create Task Dialog:**
- Fields: title (pre-filled with context like "Follow up with {contact} re: {deal}"), description, due date, priority
- Creates Commander Task via API
- Shows confirmation with link to Commander Taskboard

**Schedule Meeting Dialog:**
- Fields: title, date/time, duration, description
- Creates Commander Calendar event via API
- Shows confirmation with link to Calendar

**Add Note Dialog:**
- Fields: subject (optional), body (textarea)
- Simplest activity — just saves to timeline

**Log Email Dialog:**
- Fields: subject, body, direction (sent/received)
- Manual logging for now — placeholder for future Journeys integration

### 6. Activity Service

```
src/services/activities.ts
├── listForContact(contactId)     — all activities for a contact
├── listForDeal(dealId)           — all activities for a deal
├── createNote(contactId, dealId?, data)
├── logCall(contactId, dealId?, data)
├── createTask(contactId, dealId?, data)  — also calls Commander Tasks API
├── scheduleMeeting(contactId, dealId?, data)  — also calls Commander Calendar API
├── logEmail(contactId, dealId?, data)
└── getTaskStatus(commanderTaskId)  — check if task is completed in Commander
```

---

## Acceptance Criteria

- [ ] Contact detail page shows full profile with activity timeline
- [ ] Company detail page shows contacts, deals, and total revenue
- [ ] Deal detail page shows stage progression bar and activities
- [ ] "Log Call" creates an activity and updates `last_contacted_at`
- [ ] "Create Task" creates a Commander Task AND a CRM activity linked to it
- [ ] "Schedule Meeting" creates a Commander Calendar event AND a CRM activity
- [ ] "Add Note" creates a note activity on the timeline
- [ ] Activities are sorted reverse chronological, grouped by date
- [ ] Lifecycle stage changes are logged as activities
- [ ] Deal stage changes are logged as activities
- [ ] Navigation between contact → company → deal works via links
- [ ] Empty states: "No activities yet — log your first call or note"
