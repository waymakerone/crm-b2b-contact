---
sync:
  type: doc
  layer: CRM B2B Contact
build:
  status: todo
  phase: 4
  priority: P2
  depends_on: ["phase-3-dashboard-and-journeys"]
  started_at: null
  completed_at: null
---

# Phase 4: Reporting & Polish

**Goal:** Visual reporting (pipeline over time, conversion funnel, activity leaderboard), customizable pipeline stages, deal aging alerts, and mobile responsiveness. Production-ready polish.

**PRD Reference:** `docs/01-planning/product-requirements/crm-b2b-contact-prd.md` — Phase 4

---

## What to Build

### 1. Pipeline Value Over Time

**Chart:** Line chart showing total pipeline value at weekly snapshots.
- X-axis: weeks (last 12 weeks)
- Y-axis: total pipeline value ($)
- Implementation: query deals with `created_at` and `stage` changes to reconstruct weekly totals
- Optional: overlay Closed Won revenue as a second line

**Library:** Use `recharts` (lightweight, React-native) or `chart.js` with `react-chartjs-2`.

### 2. Conversion Funnel

**Chart:** Horizontal funnel showing deal progression through stages.
- Each stage shows: count of deals that entered that stage, conversion % to next stage
- Example: Discovery (50) → 60% → Qualification (30) → 50% → Proposal (15) → 67% → Negotiation (10) → 40% → Won (4)
- Color-coded: wider bars = more deals, percentage labels between stages
- Time range selector: last 30/60/90 days

### 3. Activity Leaderboard

**Table:** Activity counts per team member for the current period.
- Columns: name, calls, emails, meetings, tasks, notes, total
- Sortable by any column
- Helps sales managers see team activity levels
- Time range: this week, this month, this quarter

### 4. Deal Aging Alerts

**Logic:** Flag deals with no activity in the last 14 days (configurable).
- Add a "Stale" badge to deal cards on the pipeline board
- Dashboard widget: "Stale Deals" count with list
- Optional: auto-create a reminder task when a deal goes stale

**Implementation:**
- Compare `MAX(crm_activities.created_at)` for each deal against current date
- Deals in closed stages are excluded
- Default threshold: 14 days, configurable in settings

### 5. Customizable Pipeline Stages

**Settings → Pipeline Stages:**
- List all stages with drag-to-reorder
- Add a new stage: name, default probability, position
- Edit existing: rename, change probability
- Delete: only if no deals are in that stage (show warning)
- Cannot delete the "Closed Won" or "Closed Lost" system stages

**Implementation:**
- `crm_pipeline_stages` table already supports this
- UI: sortable list with inline edit
- Changes take effect immediately on the pipeline board

### 6. Mobile Responsiveness

All views need to work on tablet (768px) and phone (375px):

**Pipeline Board:**
- Horizontal scroll with snap points on mobile
- Stage column headers sticky at top
- Deal cards full-width within column

**Contact/Company/Deal Lists:**
- Switch from table to card layout below 768px
- Key info visible, secondary info hidden behind expand

**Detail Pages:**
- Stack to single column below 768px
- Activity timeline below the info card

**Dashboard:**
- Summary cards 2x2 grid on tablet, stacked on mobile
- Charts full-width and scrollable

### 7. Host Schema Manifest

Create `waymaker.config.ts` for Host Schema integration:

```typescript
export default {
  name: 'CRM B2B Contact',
  slug: 'crm-b2b-contact',
  tables: [
    { name: 'crm_companies', access: 'read-write' },
    { name: 'crm_contacts', access: 'read-write' },
    { name: 'crm_deals', access: 'read-write' },
    { name: 'crm_activities', access: 'read-write' },
    { name: 'crm_pipeline_stages', access: 'read-write' },
  ],
  ambassadors: [],
  commander_apis: [
    'commander-table-operations',
    'commander-contacts',
    'commander-task-operations',
    'commander-calendar-operations',
    'commander-journey-operations',
  ],
}
```

### 8. Deploy Configuration

- Verify all env vars use `${VITE_*}` placeholders
- Test with production Clerk keys
- Confirm RLS policies on all CRM tables (org-scoped: `organization_id = auth.org_id()`)
- Test with 2+ users to verify owner-based filtering
- Load test with 100 contacts, 50 companies, 25 deals

---

## Acceptance Criteria

- [ ] Pipeline value over time chart renders with real data
- [ ] Conversion funnel shows stage-to-stage percentages
- [ ] Activity leaderboard ranks team members by activity count
- [ ] Stale deals (14+ days no activity) show "Stale" badge on pipeline board
- [ ] Pipeline stages are editable in settings (add, rename, reorder, delete)
- [ ] Mobile: pipeline board scrolls horizontally with snap points
- [ ] Mobile: lists switch to card layout below 768px
- [ ] Mobile: detail pages stack to single column
- [ ] waymaker.config.ts manifest is complete and valid
- [ ] All tables have RLS policies scoped to organization_id
- [ ] No hardcoded secrets or internal references (passes blueprints:validate)
- [ ] App deploys successfully to Waymaker Host
