# B2B CRM (Contact-Centric) — Product Requirements

**Status:** Approved
**Author:** Waymaker
**Date:** 2026-02-26
**Last Updated:** 2026-02-26

## Problem Statement

Every B2B company needs to track who they're talking to, what deals are in play, and what happens next — but most SMBs are stuck between two bad options: pay $1,200+/month for HubSpot or Salesforce and use 15% of it, or cobble together spreadsheets and lose track of deals.

The contact-centric CRM model (popularized by HubSpot) is the right fit for most SMBs: everyone is a Contact from day one, linked to a Company, with Deals created when there's a real opportunity. Lifecycle stages track where each contact is in the journey. No confusing Lead conversion, no separate objects for the same person.

The key difference: this CRM doesn't live in a silo. Tasks show up on the user's Commander Taskboard. Emails go through Journeys. Meetings appear in Calendar. Documents live in Docs. The CRM is part of the operating system, not a walled garden.

## Goals

1. Single pipeline view — every deal, every stage, at a glance
2. Contact + Company as the core objects — no Lead conversion complexity
3. Lifecycle stages — know where every contact is (subscriber → customer → evangelist)
4. Activity timeline — every email, call, meeting, and note on the contact record
5. Commander integration — tasks, email, calendar, and docs flow through existing tools

## Non-Goals

- This is NOT a marketing automation platform — no landing pages, forms, or A/B testing
- This is NOT a customer support tool — no ticketing or SLA tracking
- This is NOT a Lead-centric CRM — no separate Lead object or conversion workflow
- This does NOT replace Commander's Contacts — it extends them with CRM context

## Data Model

### Core Objects

**Contact** — A person you're doing business with.
- Extends Commander Contacts (name, email, phone, company)
- Adds CRM fields: lifecycle_stage, lead_source, owner_id, last_contacted_at
- Lifecycle stages: `subscriber` → `lead` → `qualified` → `opportunity` → `customer` → `evangelist`

**Company** — An organization (account).
- Fields: name, domain, industry, size_band, annual_revenue_band, billing_address
- Linked to Contacts (one-to-many)
- Linked to Deals (one-to-many)

**Deal** — A sales opportunity with a value and a stage.
- Fields: name, amount, currency, stage, expected_close_date, owner_id, probability
- Linked to a Company and a primary Contact
- Pipeline stages: `discovery` → `qualification` → `proposal` → `negotiation` → `closed_won` | `closed_lost`
- Closed deals track won/lost reason

**Activity** — Anything that happened on a contact or deal.
- Types: `email`, `call`, `meeting`, `note`, `task`
- For tasks: creates a Commander Task (shows on user's Taskboard)
- For meetings: creates a Commander Calendar event
- For emails: triggers a Journeys sequence or logs a sent email
- All activities have: type, subject, body, created_by, created_at, linked_contact_id, linked_deal_id

### Tables Schema (Commander Tables)

```
crm_companies
├── id (uuid, PK)
├── organization_id (text, FK → Clerk org)
├── name (text, required)
├── domain (text, unique per org)
├── industry (text)
├── size_band (text: 1-10, 11-50, 51-200, 201-1000, 1000+)
├── annual_revenue_band (text)
├── billing_address (jsonb)
├── notes (text)
├── created_at / updated_at
└── created_by (text, Clerk user ID)

crm_contacts
├── id (uuid, PK)
├── organization_id (text)
├── commander_contact_id (uuid, FK → commander contacts)
├── company_id (uuid, FK → crm_companies)
├── lifecycle_stage (text: subscriber, lead, qualified, opportunity, customer, evangelist)
├── lead_source (text: website, referral, partner, event, outbound, other)
├── owner_id (text, Clerk user ID — the salesperson)
├── job_title (text)
├── last_contacted_at (timestamptz)
├── created_at / updated_at
└── created_by (text)

crm_deals
├── id (uuid, PK)
├── organization_id (text)
├── company_id (uuid, FK → crm_companies)
├── primary_contact_id (uuid, FK → crm_contacts)
├── name (text, required)
├── amount (numeric)
├── currency (text, default 'USD')
├── stage (text: discovery, qualification, proposal, negotiation, closed_won, closed_lost)
├── probability (integer, 0-100)
├── expected_close_date (date)
├── actual_close_date (date)
├── won_lost_reason (text)
├── owner_id (text, Clerk user ID)
├── created_at / updated_at
└── created_by (text)

crm_activities
├── id (uuid, PK)
├── organization_id (text)
├── contact_id (uuid, FK → crm_contacts)
├── deal_id (uuid, FK → crm_deals, nullable)
├── activity_type (text: email, call, meeting, note, task)
├── subject (text)
├── body (text)
├── commander_task_id (uuid, nullable — links to Commander Task)
├── commander_event_id (uuid, nullable — links to Commander Calendar event)
├── due_date (timestamptz, nullable)
├── completed_at (timestamptz, nullable)
├── created_at
└── created_by (text)

crm_pipeline_stages
├── id (uuid, PK)
├── organization_id (text)
├── name (text)
├── display_order (integer)
├── probability_default (integer)
├── is_won (boolean, default false)
├── is_lost (boolean, default false)
└── created_at
```

### Commander Integration Map

| CRM Action | Commander Tool | How |
|------------|---------------|-----|
| Create a follow-up task | Tasks | Activity with type `task` → creates Commander Task via API, stores `commander_task_id` |
| Schedule a meeting | Calendar | Activity with type `meeting` → creates Calendar event, stores `commander_event_id` |
| Send an email sequence | Journeys | Contact enters a Journey based on lifecycle stage change or manual trigger |
| Attach a proposal | Docs | Document created in Docs, linked to deal via activity note |
| Track a metric | Metrics | Pipeline value, win rate, conversion rates shown in Metrics dashboard |
| Look up a contact | Contacts | `crm_contacts.commander_contact_id` links back to the base contact record |

## Proposed Solution

### Overview

An internal (EX) app deployed to Waymaker Host that provides a full CRM experience: pipeline board, contact/company management, activity timeline, and reporting. All data lives in Commander Tables. All actions flow through Commander's existing tools — Tasks, Calendar, Journeys, Docs.

The app is the view layer and the CRM logic. Commander is the data layer and the action layer.

### Key Views

1. **Pipeline Board** — Kanban board of deals by stage. Drag to move between stages. Deal cards show: name, company, amount, expected close, owner avatar.

2. **Contact List** — Searchable table of all contacts. Columns: name, company, lifecycle stage, owner, last contacted. Click to open contact detail.

3. **Contact Detail** — Full contact profile: info, company, deals, and activity timeline. Actions: log a call, send email, create task, schedule meeting, add note.

4. **Company Detail** — Company profile: info, all contacts at this company, all deals, total revenue.

5. **Deal Detail** — Deal profile: stage, amount, contacts, activities. Stage progression bar at top.

6. **Dashboard** — Summary: total pipeline value, deals by stage, win rate, average deal size, upcoming tasks.

### User Flow

1. Sales rep opens CRM from Host dashboard
2. Sees Pipeline Board — deals in columns by stage
3. Clicks a deal → sees detail with activity timeline
4. Logs a call (note + outcome), creates a follow-up task
5. The task appears on their Commander Taskboard
6. Drags a deal to "Proposal" stage → probability auto-updates
7. Goes to Contact List, searches for a specific contact
8. Opens contact → sees all deals, all activities, lifecycle stage
9. Triggers a Journeys sequence for nurturing
10. Checks Dashboard for weekly pipeline summary

## Scope

### Phase 1 (MVP) — Pipeline & Contacts Foundation

- [ ] App scaffold: React + Vite + Tailwind + Clerk auth
- [ ] Tables schema: crm_companies, crm_contacts, crm_deals, crm_activities, crm_pipeline_stages
- [ ] API layer: CRUD operations for all CRM tables via authenticated edge function
- [ ] Company list + create/edit
- [ ] Contact list + create/edit (linked to Commander Contacts)
- [ ] Lifecycle stage management on contacts
- [ ] Basic deal CRUD (list, create, edit, delete)
- [ ] Pipeline board: Kanban view with drag-and-drop between stages
- [ ] Deal cards with key info (name, company, amount, close date)

### Phase 2 — Activities & Commander Integration

- [ ] Activity timeline on Contact and Deal detail pages
- [ ] Log a call: type, notes, outcome
- [ ] Log a note: freeform text on contact or deal
- [ ] Create a task: creates Commander Task via API, links back
- [ ] Schedule a meeting: creates Commander Calendar event, links back
- [ ] Activity feed: chronological list of all activities per contact/deal
- [ ] Contact detail page: profile + company + deals + activities
- [ ] Company detail page: info + contacts + deals
- [ ] Deal detail page: stage bar + info + activities

### Phase 3 — Dashboard & Journeys

- [ ] Dashboard: total pipeline value, deal count by stage
- [ ] Win rate calculation (closed_won / total closed)
- [ ] Average deal size, average time to close
- [ ] Upcoming tasks and overdue tasks
- [ ] Journeys integration: trigger email sequence on lifecycle stage change
- [ ] Contact import: bulk CSV import into crm_contacts + crm_companies
- [ ] Search: global search across contacts, companies, deals
- [ ] Filters: filter pipeline by owner, date range, amount range

### Phase 4 — Reporting & Polish

- [ ] Pipeline value over time (chart)
- [ ] Conversion funnel: discovery → qualification → proposal → negotiation → won
- [ ] Activity leaderboard: calls, emails, meetings per rep
- [ ] Deal aging: flag stale deals with no activity in 14+ days
- [ ] Customizable pipeline stages (add/remove/reorder)
- [ ] waymaker.config.ts manifest for Host Schema
- [ ] Deploy-ready configuration
- [ ] Mobile-responsive layouts

### Out of Scope

- Lead object or lead conversion workflow (that's the `crm-b2b-lead` blueprint)
- Marketing automation (landing pages, forms, ad tracking)
- Customer support / ticketing
- Invoicing or billing integration
- Email inbox sync (manual logging for now; inbox sync is a future platform feature)

## Success Criteria

| Metric | Target |
|--------|--------|
| Deal visibility | 100% of deals visible on pipeline board |
| Activity logging | < 30 seconds to log a call or create a follow-up |
| Commander integration | Tasks and meetings appear in Commander within 2 seconds |
| Contact lookup | < 3 seconds via search |
| Build time (with AI) | < 6 hours for all 4 phases |

## Dependencies

- WaymakerOS organization with Commander access
- Commander Tables for data storage
- Commander Tasks API for task creation
- Commander Calendar API for meeting scheduling
- (Optional) Commander Journeys for email sequences
- (Optional) Commander Docs for proposal/contract attachment

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| No contacts yet | Medium | Low | Seed data + CSV import in Phase 3 |
| Complex drag-and-drop | Medium | Medium | Use proven library (dnd-kit or react-beautiful-dnd) |
| Journeys API not ready | Medium | Low | Phase 3 — degrade gracefully, log emails manually |
| Large dataset (1000+ contacts) | Low | Medium | Server-side pagination + search |

## Open Questions

- [x] Contact-centric or Lead-centric? **Contact-centric — simpler, better for SMBs**
- [x] EX (internal) or CX (public)? **EX — sales team only**
- [x] Custom pipeline stages? **Phase 4 — seed defaults in Phase 1**
- [ ] Email sync? **Out of scope — manual logging for now, platform feature later**
