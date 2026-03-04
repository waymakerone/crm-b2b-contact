---
sync:
  type: doc
  layer: CRM B2B Contact
build:
  status: todo
  phase: 3
  priority: P1
  depends_on: ["phase-2-activities-and-detail-pages"]
  started_at: null
  completed_at: null
---

# Phase 3: Dashboard & Journeys

**Goal:** CRM dashboard with pipeline metrics, win rates, and upcoming tasks. Journeys integration for automated email sequences. Bulk contact import and global search.

**PRD Reference:** `docs/01-planning/product-requirements/crm-b2b-contact-prd.md` — Phase 3

---

## What to Build

### 1. CRM Dashboard (/ or /dashboard)

The dashboard is the first thing a sales rep sees. Fast overview of what matters today.

**Layout:**
```
┌──────────────────────────────────────────────────┐
│ CRM Dashboard                          [Today ▼] │
│                                                  │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────┐ │
│ │ Pipeline  │ │ Deals    │ │ Win Rate │ │ Avg  │ │
│ │ Value     │ │ Open     │ │          │ │ Deal │ │
│ │ $245,000  │ │ 12       │ │ 38%      │ │$20.4K│ │
│ │ +12% ↑    │ │ -2 ↓     │ │ +5% ↑   │ │      │ │
│ └──────────┘ └──────────┘ └──────────┘ └──────┘ │
│                                                  │
│ ┌─────────────────────┐ ┌──────────────────────┐ │
│ │ Pipeline by Stage    │ │ My Tasks (Due Soon)  │ │
│ │ ████████ Discovery   │ │ ☐ Follow up — Acme   │ │
│ │ ██████ Qualification │ │ ☐ Send proposal — BW  │ │
│ │ ████ Proposal        │ │ ☑ Call recap — Delta  │ │
│ │ ██ Negotiation       │ │                      │ │
│ └─────────────────────┘ └──────────────────────┘ │
│                                                  │
│ ┌─────────────────────┐ ┌──────────────────────┐ │
│ │ Recent Activity      │ │ Deals Closing Soon   │ │
│ │ You logged a call... │ │ Acme Q2 — $50K — 3d │ │
│ │ New deal created...  │ │ BW Pilot — $8K — 7d  │ │
│ │ Stage changed...     │ │                      │ │
│ └─────────────────────┘ └──────────────────────┘ │
└──────────────────────────────────────────────────┘
```

**Summary cards (top row):**
- Total pipeline value (sum of open deal amounts)
- Open deals count
- Win rate (closed_won / total closed, last 90 days)
- Average deal size (closed_won, last 90 days)
- Each card shows period-over-period change if enough data

**Pipeline by Stage:**
- Horizontal bar chart showing deal count or value per stage
- Click a bar to filter pipeline board to that stage

**My Tasks (Due Soon):**
- Pulls from `crm_activities` where `activity_type = 'task'` and `completed_at IS NULL`
- Sorted by due date ascending
- Click to open in Commander Taskboard
- Mark complete inline (updates Commander Task via API)

**Recent Activity:**
- Last 10 activities across all contacts/deals for the current user
- Quick scan of what happened today

**Deals Closing Soon:**
- Deals with `expected_close_date` within next 14 days
- Sorted by close date ascending
- Red highlight if overdue (close date passed, still open)

### 2. Journeys Integration

When a contact's lifecycle stage changes, the CRM can trigger a Commander Journey (email sequence).

**Trigger points:**
- Contact moves to `lead` → trigger "New Lead Welcome" journey
- Contact moves to `qualified` → trigger "Qualified Lead Nurture" journey
- Contact moves to `customer` → trigger "Customer Onboarding" journey
- Deal moves to `closed_won` → trigger "Welcome New Customer" journey
- Deal moves to `closed_lost` → trigger "Win-Back" journey

**Implementation:**
```
POST /functions/v1/commander-journey-operations
Body: {
  "action": "enroll_contact",
  "data": {
    "journey_slug": "new-lead-welcome",
    "contact_id": "{commander_contact_id}",
    "trigger_source": "crm",
    "metadata": {
      "lifecycle_stage": "lead",
      "lead_source": "website"
    }
  }
}
```

**Settings page** (`/settings`):
- Map lifecycle stage transitions to Journey slugs
- Toggle automations on/off per trigger
- Shows which Journeys are available in the org
- If Journeys is not configured: friendly message explaining the integration

### 3. Contact Import

**CSV Import** (`/contacts/import`):
- Upload CSV file
- Column mapping UI: map CSV headers to CRM fields (first_name, last_name, email, company_name, phone, job_title, lifecycle_stage, lead_source)
- Preview first 5 rows with mapped data
- "Company_name" auto-creates companies if they don't exist (match by name)
- Import creates Commander Contacts AND CRM contacts
- Summary: X contacts imported, Y companies created, Z errors (with downloadable error log)

### 4. Global Search

**Search bar** in the top navigation:
- Searches across contacts (name, email), companies (name, domain), and deals (name)
- Results grouped by type with icons
- Click to navigate to the detail page
- Keyboard shortcut: `Cmd+K` or `/`

### 5. Filters on Pipeline Board

Extend the pipeline board from Phase 1:
- Filter by: owner, expected close date range, amount range
- Filters persist in URL params
- "Clear filters" button

---

## Acceptance Criteria

- [ ] Dashboard shows 4 summary cards with real data
- [ ] Pipeline by Stage chart renders correctly
- [ ] My Tasks section shows upcoming CRM tasks
- [ ] Recent Activity shows last 10 activities
- [ ] Deals Closing Soon highlights overdue deals in red
- [ ] Lifecycle stage change triggers Journeys enrollment (if configured)
- [ ] Settings page allows mapping stage transitions to Journeys
- [ ] CSV import works: upload → map columns → preview → import
- [ ] Import creates companies that don't exist
- [ ] Global search returns results across contacts, companies, deals
- [ ] Pipeline board filters by owner, date range, amount range
- [ ] Dashboard metrics update in real time as deals are created/moved
