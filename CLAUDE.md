# B2B CRM (Contact-Centric)

Contact-centric B2B CRM for WaymakerOS. Contacts + Companies + Deals + Pipeline + Activities — all integrated with Commander.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React + TypeScript + Vite + Tailwind CSS |
| Auth | Clerk (via WaymakerOS) |
| Data | Commander Tables (Supabase PostgreSQL) |
| Drag & Drop | @dnd-kit/core + @dnd-kit/sortable |
| Charts | recharts |
| Hosting | Waymaker Host (EX app) |

## Documentation

| Folder | Contents |
|--------|----------|
| `docs/01-planning/product-requirements/` | PRD — data model, views, integration map |
| `docs/02-working/prompts/active/` | Build prompts — 4 phases with YAML front matter |
| `docs/02-working/sessions/` | Session briefs as you build |
| `docs/03-knowledge/` | Patterns discovered during the build |

## Build Phases

| Phase | Prompt | Status |
|-------|--------|--------|
| 1 — Pipeline & Contacts | `docs/02-working/prompts/active/phase-1-pipeline-and-contacts.md` | todo |
| 2 — Activities & Detail Pages | `docs/02-working/prompts/active/phase-2-activities-and-detail-pages.md` | todo |
| 3 — Dashboard & Journeys | `docs/02-working/prompts/active/phase-3-dashboard-and-journeys.md` | todo |
| 4 — Reporting & Polish | `docs/02-working/prompts/active/phase-4-reporting-and-polish.md` | todo |

## Data Model (Quick Reference)

```
crm_companies ──< crm_contacts ──< crm_activities
                       │                  │
                       └──< crm_deals ──<─┘
                                │
                       crm_pipeline_stages
```

- **Company** has many Contacts and Deals
- **Contact** belongs to a Company, has a lifecycle stage, links to Commander Contacts
- **Deal** belongs to a Company + primary Contact, moves through pipeline stages
- **Activity** belongs to a Contact and optionally a Deal. Types: note, call, task, meeting, email
- Tasks link to Commander Tasks (shows on Taskboard)
- Meetings link to Commander Calendar events

## Commander Integration

| CRM Action | Commander Tool | API |
|------------|---------------|-----|
| Create follow-up task | Tasks | `commander-task-operations` → `create_task` |
| Schedule meeting | Calendar | `commander-calendar-operations` → `create_event` |
| Email sequence | Journeys | `commander-journey-operations` → `enroll_contact` |
| Attach document | Docs | Link via activity note |
| Pipeline metrics | Metrics | Dashboard queries |
| Contact data | Contacts | `commander-contacts` → base contact record |
| All CRM tables | Tables | `commander-table-operations` → CRUD |

## Critical Rules

- All data lives in Commander Tables — the app is the view + logic layer
- Auth via Clerk: `useAuth().getToken()` → Bearer token on all API calls
- API calls POST to `${SUPABASE_URL}/functions/v1/{function-name}` with `{ action, data }` body
- Creating a CRM contact also creates a Commander Contact (linked via `commander_contact_id`)
- Creating a task also creates a Commander Task (linked via `commander_task_id`)
- Creating a meeting also creates a Commander Calendar event (linked via `commander_event_id`)
- Design tokens: Gold `#A49886`, Navy `#001126`, Blue `#3D5B6C`, Sand `#F5F5F0`
- Use Geist font family
- All tables scoped by `organization_id` with RLS policies

## How to Build

1. Read the PRD: `docs/01-planning/product-requirements/crm-b2b-contact-prd.md`
2. Work through each phase prompt in order
3. Update the YAML `status` field as you go: `todo` → `in-progress` → `review` → `done`
4. Write session briefs in `docs/02-working/sessions/completed/` between sessions

## Quick Reference

| What | How |
|------|-----|
| Dev server | `npm run dev` |
| Build | `npm run build` |
| Tables API | `commander-table-operations` → `query`, `insert`, `update`, `delete` |
| Contacts API | `commander-contacts` → `list_contacts`, `create_contact` |
| Tasks API | `commander-task-operations` → `create_task`, `update_task` |
| Calendar API | `commander-calendar-operations` → `create_event` |
| Journeys API | `commander-journey-operations` → `enroll_contact` |
