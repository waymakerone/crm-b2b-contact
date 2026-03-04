# B2B CRM (Contact-Centric)

A full-featured contact-centric B2B CRM built on WaymakerOS. Manage contacts, companies, deals, and your sales pipeline — all integrated with Commander's tools.

## What You Get

- **Pipeline Board** — Kanban view of deals by stage, drag-and-drop between stages
- **Contact Management** — Contacts linked to companies with lifecycle stages
- **Activity Timeline** — Log calls, create tasks, schedule meetings, add notes
- **Commander Integration** — Tasks on your Taskboard, meetings in Calendar, sequences in Journeys
- **Dashboard** — Pipeline value, win rate, upcoming tasks, deals closing soon
- **Reporting** — Conversion funnel, activity leaderboard, deal aging alerts

## CRM Model

This blueprint uses the **contact-centric** model (like HubSpot):

- Everyone is a **Contact** from day one, linked to a **Company**
- **Lifecycle stages** track progression: subscriber → lead → qualified → opportunity → customer → evangelist
- **Deals** are created when there's a real sales opportunity
- No separate "Lead" object, no conversion workflow — simple and clean

Looking for lead-centric (Salesforce-style)? See the `crm-b2b-lead` blueprint.
Looking for B2C? See the `crm-b2c` blueprint.

## Commander Tools Used

| Tool | How the CRM Uses It |
|------|---------------------|
| **Tables** | All CRM data (companies, contacts, deals, activities, pipeline stages) |
| **Contacts** | Base contact records — CRM contacts extend them with sales context |
| **Tasks** | Follow-up tasks created from the CRM appear on your Taskboard |
| **Calendar** | Meetings scheduled from the CRM appear in your Calendar |
| **Journeys** | Email sequences triggered by lifecycle stage changes |
| **Docs** | Proposals and contracts attached to deals |
| **Metrics** | Pipeline metrics on your Commander dashboard |

## How to Build

1. Clone this blueprint into your project
2. Open `CLAUDE.md` — it's the router file for your AI coding tool
3. Read the PRD in `docs/01-planning/product-requirements/`
4. Work through the 4 phase prompts in `docs/02-working/prompts/active/`
5. Point Claude Code, Cursor, or Codex at each phase and build

**Estimated build time:** 4-6 hours across all 4 phases.

## Build Phases

| Phase | What You Get |
|-------|-------------|
| 1 — Pipeline & Contacts | App scaffold, schema, company/contact CRUD, Kanban pipeline board |
| 2 — Activities & Detail Pages | Activity timeline, Commander task/calendar integration, detail pages |
| 3 — Dashboard & Journeys | Metrics dashboard, Journeys email automation, CSV import, global search |
| 4 — Reporting & Polish | Charts, funnel analysis, custom stages, mobile responsive, deploy-ready |

## Prerequisites

- WaymakerOS organization with Commander access
- Commander Tables, Tasks, Calendar, and Contacts enabled
- (Optional) Commander Journeys for email automation
