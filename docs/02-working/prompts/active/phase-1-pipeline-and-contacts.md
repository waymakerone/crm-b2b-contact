---
sync:
  type: doc
  layer: CRM B2B Contact
build:
  status: todo
  phase: 1
  priority: P0
  depends_on: []
  started_at: null
  completed_at: null
---

# Phase 1: Pipeline & Contacts Foundation

**Goal:** App scaffold with auth, data schema, and the two core views — a Kanban pipeline board for deals and a searchable contact list. Companies, contacts, and deals all working with CRUD.

**PRD Reference:** `docs/01-planning/product-requirements/crm-b2b-contact-prd.md` — Phase 1

---

## What to Build

### 1. Project Setup

Create a React + Vite + TypeScript + Tailwind app.

**Files:**
- `src/main.tsx` — Clerk provider wrapper
- `src/App.tsx` — Router with routes: `/`, `/contacts`, `/companies`, `/deals/:id`
- `src/lib/api.ts` — Authenticated fetch helper for Commander Tables
- `src/lib/types.ts` — TypeScript interfaces for Company, Contact, Deal, Activity
- `src/index.css` — Tailwind base + Waymaker design tokens

**Auth pattern:**
```typescript
import { useAuth } from '@clerk/clerk-react'

const { getToken } = useAuth()
const token = await getToken()

const res = await fetch(`${SUPABASE_URL}/functions/v1/commander-table-operations`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
  body: JSON.stringify({ action: 'query', data: { table: 'crm_deals', filters: {} } }),
})
```

### 2. Database Schema

Create these tables in Commander Tables. Use the API or run migrations directly.

**crm_pipeline_stages** — Seed with defaults:

| display_order | name | probability_default | is_won | is_lost |
|---|---|---|---|---|
| 1 | Discovery | 10 | false | false |
| 2 | Qualification | 25 | false | false |
| 3 | Proposal | 50 | false | false |
| 4 | Negotiation | 75 | false | false |
| 5 | Closed Won | 100 | true | false |
| 6 | Closed Lost | 0 | false | true |

**crm_companies** — See PRD for full schema.

**crm_contacts** — See PRD. Link to Commander Contacts via `commander_contact_id`. Lifecycle stages: subscriber, lead, qualified, opportunity, customer, evangelist.

**crm_deals** — See PRD. Links to company and primary contact. Stage references `crm_pipeline_stages.name`.

**crm_activities** — See PRD. This table is created now but populated in Phase 2.

### 3. API Layer

Create a service module for each entity:

```
src/services/
├── companies.ts    — list, get, create, update, delete
├── contacts.ts     — list, get, create, update, delete, updateLifecycleStage
├── deals.ts        — list, get, create, update, delete, moveStage
└── pipeline.ts     — listStages
```

All operations go through `commander-table-operations` edge function with the appropriate table name and action.

### 4. Company Management

**Company List** (`/companies`):
- Table view: name, domain, industry, size band, contact count, deal count
- "New Company" button → modal form
- Click row → Company detail (Phase 2)
- Search by name or domain

**Company Form** (modal):
- Fields: name (required), domain, industry (dropdown), size band (dropdown), notes
- Create and edit modes

### 5. Contact Management

**Contact List** (`/contacts`):
- Table view: name, email, company, job title, lifecycle stage, owner, last contacted
- Lifecycle stage shown as colored badge:
  - subscriber (gray), lead (blue), qualified (yellow), opportunity (orange), customer (green), evangelist (purple)
- "New Contact" button → modal form
- Search by name, email, or company
- Filter by lifecycle stage

**Contact Form** (modal):
- Fields: first name, last name, email (required), phone, company (dropdown), job title, lifecycle stage (dropdown), lead source (dropdown), owner (dropdown of org users)
- On create: also creates a Commander Contact via `commander-contacts` API, stores the `commander_contact_id`

### 6. Pipeline Board (/)

The home page. Kanban board showing deals by stage.

**Layout:**
- Columns for each pipeline stage (from `crm_pipeline_stages`, ordered by `display_order`)
- Each column header shows: stage name + deal count + total value
- "Closed Won" and "Closed Lost" are collapsed columns on the right (show totals, expandable)

**Deal Cards:**
- Company name (bold)
- Deal name (subtitle)
- Amount (formatted currency)
- Expected close date
- Owner avatar/initials
- Color-coded by: overdue (red border), closing this week (gold border), normal (default)

**Drag and Drop:**
- Use `@dnd-kit/core` and `@dnd-kit/sortable`
- Dragging a deal to a new column calls `deals.moveStage(dealId, newStage)`
- If moved to "Closed Won" or "Closed Lost": prompt for reason (modal)
- Probability auto-updates based on stage's `probability_default`

**"New Deal" button:**
- Quick-create: name, company (dropdown), amount, expected close date, stage (defaults to Discovery)
- Must have at least one Company to create a deal

### 7. Design Tokens

Use Waymaker brand:
- Font: `Geist` (load from Google Fonts)
- Gold: `#A49886` — accents, active states, deal amount highlights
- Navy: `#001126` — primary text
- Blue: `#3D5B6C` — headers, pipeline column headers
- Sand: `#F5F5F0` — page backgrounds, pipeline board background
- Cards: white, `border-radius: 12px`, subtle shadow
- Pipeline columns: `border-radius: 8px`, slightly darker sand background
- Spacing: 8px base grid

---

## Acceptance Criteria

- [ ] App loads with Clerk auth (sign-in required)
- [ ] Pipeline board shows all stages as columns
- [ ] Deals render as cards in the correct stage column
- [ ] Drag-and-drop moves deals between stages and persists to DB
- [ ] Moving to Closed Won/Lost prompts for reason
- [ ] Company list with CRUD (create, read, update)
- [ ] Contact list with CRUD, lifecycle stages as colored badges
- [ ] Creating a contact also creates a Commander Contact
- [ ] Search works on contacts (name, email, company) and companies (name, domain)
- [ ] "New Deal" creates a deal linked to a company
- [ ] Responsive: pipeline scrolls horizontally on mobile, tables stack
- [ ] Empty states for all views (no companies → "Add your first company", etc.)
