# Product Requirement Document (v2) — "The Dashboard Nobody Reads"

Source: the working clickable prototype (six screens, feature-grouped code, a live database behind the operational data, in-memory experiment instrumentation). This version replaces v1, which described the earlier routes and a memory-only prototype with no backend.

**What changed since v1**

1. Screen routes and source locations were renamed and regrouped by feature: `/shiftstart`, `/`, `/evchargersites`, `/experimentlog`, `/shiftsjournal`, `/evidence`.
2. A database backend is live. Charging sites, operator names, shifts and action acknowledgements are read from and written to it; shift and action records now survive a refresh.
3. The model is hybrid and this document is explicit about the split: operational data is persisted, experiment instrumentation is still in-memory and still resets on refresh.

---

## Problem

The operations dashboard this project reacts to reports what happened but never what to do about it, and the number people actually need is buried:

> "It tells me what happened but never what to do about it. I still export to a spreadsheet to think." — Product lead
>
> "The number I need is in there, but it takes six clicks and three filters to find it." — Growth analyst

Baseline behaviour being challenged:

| Baseline metric | Value |
|---|---|
| Bounce rate (sessions under 15 seconds, no interaction) | 60% |
| Clicks to reach the most-requested metric (occupancy duration per site) | 6 |
| Average weekly sessions per active user | 1.3 |

**The hypothesis this PRD is tied to:** a first screen that leads with two headline KPIs, a plain-language recommendation, and one suggested next step will make the people who set priorities explore further and acknowledge actions, instead of bouncing. We will know we are right when bounce rate drops and the most important acknowledgement gets clicked.

**Honesty note:** the hypothesis is *stated and instrumented* here — it is not yet validated by real users. The prototype exists to test it. The kill switch is explicit: if a guided headline plus direction to priority actions still bounces, the metric itself may be wrong — pivot.

Operational scenario: seven public EV charging sites in Scotland near motorway/expressway junctions (Gretna M74 J22, Abington M74 J13, Stirling M9 J9, Perth Broxden M90 J11, Livingston M8 J3, Bothwell M74 J5, Dundee Kingsway A90). High occupancy can signal real demand or idle parking; the Stirling reading (high occupancy, ~5h average session, ~1h of it charging) is idle blocking, not demand.

---

## Users & jobs

**Primary user:** the network operations person on shift (e.g. Fiona Ross, Network Ops). Simulated by a picker of 10 named operators, now loaded from the `operators` table.

*Job to be done:* come on shift, quickly learn what needs attention and why, decide one thing, and leave a record — what was decided, on which site, who owns it, when it gets reviewed — that the next shift can read.

**Secondary users:**

- **New hires** — the Guided overview is a one-time onboarding step per simulated user journey, not part of the recurring workflow.
- **The researcher / product team running the test** — uses the experiment instrumentation (live strip, evidence screen, reset) to observe whether the hypothesis holds per session.

---

## Scope (in / out)

**In scope (built):**

- Shift start: pick the simulated operator, start a shift (written to the database), open a journal day.
- Guided overview: two headline KPIs, evidence-backed recommendation, one primary action; shown once per simulated user journey.
- Site breakdown: ranked per-site table from the database, with verdicts, readings, per-row actions; the operational hub every site-level screen returns to; loading / empty / error states.
- Actions log: acknowledge a decision with owner, review date, optional note; the record is stored and linked to its site and shift.
- Shifts journal: stored actions grouped by day and operator, with timestamps; survives a refresh.
- Experiment instrumentation (see "Product vs. Experiment" below).

**Out of scope (explicitly not built):**

- Real telemetry feeds — site figures are seeded sample rows in the database, not live readings from chargers.
- Accounts, sign-in, permissions, multi-tenancy. There is no authentication; the shift and action tables currently accept anonymous writes, which is acceptable for a closed test and must be locked to real logins before any external use.
- Enforcement of actions: no idle-fee engine, no engineer dispatch, no billing.
- Persistence of the experiment layer: session timer, click counts, drill-in flag and per-journey overview completion are in-memory and reset on refresh, by design.
- The dashboard anti-patterns the hypothesis argues against: chart walls and filter bars on the first screen, alerting, multi-step wizards.

---

## Requirements

Priorities: **Must** = the prototype breaks without it; **Should** = needed for a credible test; **Could** = present but not essential.

Each table is titled with the front-end screen name (as it appears in the left navigation) and its route code name (URL path and source location).

### Screen: Shift start — route `/shiftstart` (`src/routes/shiftstart.tsx` → `src/features/shift-start/ShiftStartScreen.tsx`)

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Operator picker loaded from the database | Must | The 10 names come from the `operators` table in `display_order` (Fiona Ross, Callum MacLeod, Eilidh Fraser, Struan Gillespie, Isla Kinnaird, Hamish Buchanan, Morag Strachan, Rory Dalgleish, Ailsa Cruickshank, Lachlan Tavendale); the selection is highlighted; the "Welcome back" heading personalises with the first name. |
| Start shift always opens the Guided overview | Must | "Start shift and continue" is disabled until an operator is selected; on click it inserts a `shifts` row (operator, day key, day label, start time), opens a new journal day, and navigates to `/` — for a new user, a changed user, or the same user again. |
| Optimistic shift creation | Should | The shift appears in the journal immediately with a pending id, then is replaced by the stored row; a failed write is logged and does not block the flow. |
| KPI cards mirror the overview style | Should | Two cards: "Shifts logged today" (count of stored shifts whose day key is today) and "Actions on current shift" (count, or "No shift running yet" with on-duty operator and start time when active). |
| Journal link without starting a shift | Should | A quiet secondary link opens `/shiftsjournal` directly. |
| Network map image below the content | Could | A framed image (same frame style and width as the "Welcome back" panel, centred) shows the Scotland network map. |

### Screen: Guided overview — route `/` (`src/routes/index.tsx` → `src/features/guided-overview/GuidedOverviewScreen.tsx`)

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Two headline KPIs only | Must | Occupancy rate (78%, network-wide, this week, +9 pts vs last week) and average occupancy duration (3h 42m, of which 1h 12m charging). No charts, no filter bars. Both KPI cards are clickable and lead to Site breakdown. |
| Evidence before action on the priority card | Must | The "Highest Priority Action Required" card shows, in order: recommendation headline and body → "Why this action is recommended" (4 bullets: occupancy >5h; charging ~1h; >80% idle; pattern is blocking, not demand) → "Expected outcome" (3 bullets: availability without new infrastructure; shorter occupancy; better throughput at Stirling M9 J9) → then the action. |
| One primary action | Must | "Apply idle fee at Stirling M9 J9 (4 stalls)" navigates to `/experimentlog?site=stirling-m9-j9`; once acknowledged, the button is replaced by a positive "Idle fee acknowledged for Stirling M9 J9" badge. |
| Completion recorded per simulated user journey | Must | Reaching the overview with no started shift redirects to `/shiftstart`; completing it (primary action, drill-down link, or a KPI card) marks the active journey complete, stamps `guided_overview_completed_at` on the stored shift, and later visits of `/` redirect to `/evchargersites`. |
| Completion is per user, not global | Must | Starting a shift resets journey completion, so an explicit user change — including switching back to a user who already completed it — reopens the Guided overview. |
| Never reopens automatically | Must | No product path returns to the Guided overview from Site breakdown or any site-level screen; it reopens only when a shift is started. |
| Quiet drill-down link | Should | "Why this location was flagged and which other locations require your attention" navigates to Site breakdown and counts as a drill-down. |
| Two supporting cards | Should | "Sites flagged for maintenance" (3, warning tone) and "Sites at real capacity" (2, accent tone), each linking to Site breakdown. |

### Screen: Site breakdown — route `/evchargersites` (`src/routes/evchargersites.tsx` → `src/features/ev-charger-sites/EvChargerSitesScreen.tsx`)

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Ranked 7-site table from the database | Must | Rows come from `charging_sites` ordered by `display_rank`; columns: site/junction with stall count, occupancy %, average duration, charge-vs-idle bar with energy per stall (highest marked), fault count (warn-coloured when > 0). |
| Per-row verdict, reading and action | Must | Each row shows a colour-coded verdict pill (Idle blocking / Real capacity / Maintenance / Healthy), a one-line plain-language reading, and — for non-healthy sites — its own action button into `/experimentlog?site=<id>` (or an "Acknowledged" badge once recorded). |
| Operational hub | Must | "Back to shift start" in the header; Actions log and Shifts journal return here; site rows are the way into site-level actions. No path from here routes back to the Guided overview. |
| Simulated 3-second loading state | Must | On entry, on demo-state change, and on retry, a skeleton shows for 3 seconds: flipping hourglass, progress bar, column placeholders and 7 shimmering rows. The 3 seconds are deliberate, independent of the real query latency. |
| Empty state | Must | Shown when the demo switcher forces it or the query returns no rows. Exact message: "No charging data for this period — try a different time range" (Inbox icon, time-range hint). |
| Error state | Must | Shown when the demo switcher forces it or the query fails. Exact message: "Something went wrong loading the breakdown. Our team has been notified." with a Retry button that re-runs the query and the skeleton. |
| Demo state switcher | Should | Data / Empty / Error buttons (experiment-styled) let the tester force each state regardless of what the database returns. |

### Screen: Actions log — route `/experimentlog` (`src/routes/experimentlog.tsx` → `src/features/experiment-log/ExperimentLogScreen.tsx`)

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Confirm the decision with context | Must | Reached with `?site=<id>`: shows the action, site, plain-language reading, stalls affected, current occupancy and average duration from the stored site row. Without a site param, it renders the log only. |
| Record owner, review date, note | Must | Editable owner (default "Fiona Ross, Network Ops"), date-typed review date, optional note; "Acknowledge and record" inserts an `action_acknowledgements` row and clears the site param. |
| Log accumulates and persists | Must | The list shows every acknowledgement with decision, site, timestamp, owner, review date and note; empty state reads "Nothing acknowledged yet…". Records reload after a refresh and keep acknowledged badges on the overview and Site breakdown. |
| Entries stamped to the shift | Must | Each record carries the active shift's operator, day key and day label (or "Unassigned" plus today when no shift is running) and links to the stored shift when one exists. |
| Return paths | Must | "Back to site breakdown" (top and bottom) returns to `/evchargersites`; a "View in shifts journal" link opens `/shiftsjournal` once any acknowledgement exists. |

### Screen: Shifts journal — route `/shiftsjournal` (`src/routes/shiftsjournal.tsx` → `src/features/shifts-journal/ShiftsJournalScreen.tsx`)

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Grouped by day, then operator | Must | Day headings (weekday, date), then operator blocks with shift start time and action count, then that operator's actions in order — each with timestamp, decision, site, owner, review date, note. |
| Stored history, not just this session | Must | Shifts and actions recorded earlier reappear after a refresh, loaded from the database and merged with anything created in the current session. |
| Active-shift marker | Should | "On shift now: \<operator\> since \<time\> · \<day\>" above the list, or "No shift running" when none. |
| Empty states | Should | Empty journal: "The journal is empty…"; an operator with a shift but no actions reads "Shift started, no actions yet." |
| Return path | Must | "Back to shift start" at the top; "Back to site breakdown" at the bottom. |

### Screen: Experiment evidence — route `/evidence` (`src/routes/evidence.tsx` → `src/features/experiment-evidence/ExperimentEvidenceScreen.tsx`)

*Experiment-only; not part of the product story.*

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Hypothesis on screen | Must | The full hypothesis and its success condition are written out. |
| User quotes and baseline metrics | Must | The two attributed quotes; the three baseline cards (60% bounce, 6 clicks, 1.3 sessions). |
| Kill switch with live readout | Must | The kill-switch statement plus the current run's seconds, interactions, drill-down status, acknowledgement count and whether the kill switch is armed. |
| Reset | Must | "Reset run for next participant" clears the in-memory session state (timer, clicks, drill-in, journey, active shift). It does not delete stored shifts or actions. |

### Cross-cutting

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Left navigation | Must | Title "EV Charge Network Ops" (Scotland · motorway corridor); items in order: Shift start (`/shiftstart`), Guided overview (`/`), Site breakdown (`/evchargersites`), Shifts journal (`/shiftsjournal`), Actions log (`/experimentlog`), App experiment (`/evidence`). All destinations work; the last two are experiment-flagged and styled fluorescent yellow. |
| Live experiment strip on every screen | Must | Session seconds, interactions, clicks-to-metric (baseline was 6), acknowledgement count, and a verdict badge that reads "Run in progress" → "Engaged, no acknowledgement yet" → "Hypothesis supported this run", or "Bounced — kill switch armed" after 15 seconds with no interaction. |
| Hybrid state model | Must | Sites, operators, shifts and acknowledgements come from the database through one shared state layer; session timer, click counts, drill-in flag and per-journey overview completion are in-memory and reset on refresh. |
| Code organisation | Should | Each screen lives in its own feature folder with its data access in a separate file beside it; route files are thin wrappers. |
| Sample-data honesty | Must | The footer states: "Prototype with sample data — figures are illustrative, not live readings." |

---

## Data & events

The operational entities are now real database records. The experiment layer is still entirely client-side. Nothing here comes from real chargers.

### Stored data (database tables, explicit column names)

| Table | Columns | Rows | Mocked / Real |
|---|---|---|---|
| `charging_sites` | site_id, site_name, junction_label, stall_count, occupancy_percent, average_duration_label, average_charge_hours, average_idle_hours, energy_kwh_per_stall_week, fault_count, plain_language_reading, verdict, recommended_action, display_rank | 7 seeded | **Real storage, mocked figures** — illustrative readings, no telemetry feed |
| `operators` | full_name, display_order | 10 seeded | **Real storage, simulated users** — no accounts, no sign-in |
| `shifts` | shift_id, operator_name, shift_day_key, shift_day_label, started_at, ended_at, guided_overview_completed_at | created by use | **Real** — genuinely written and read back |
| `action_acknowledgements` | acknowledgement_id, site_id, site_label, decision_text, owner_name, review_date, note_text, operator_name, shift_day_key, shift_day_label, shift_id, acknowledged_at | created by use | **Real** — genuinely written and read back |

Access control: no authentication exists, so reads and inserts are open to anonymous callers. That is a deliberate test-time shortcut, not a production posture.

### Client-side data (not stored)

| Object | Fields | Source | Mocked / Real |
|---|---|---|---|
| Headline KPIs | occupancy (78%), duration (3h 42m) + notes/deltas | `src/features/guided-overview/data/overview.ts` | **Mocked** — fixed copy, not computed from the site rows |
| Recommendation and evidence bullets | headline, body, why-recommended, expected outcome, action, siteId | same file | **Mocked** |
| Supporting cards | maintenance count/detail, capacity count/detail | same file | **Mocked** |
| Quote / baseline / kill-switch copy | strings | `src/features/experiment-evidence/data/experiment.ts` | **Mocked** (research framing, not measured data) |

### Events

| Event | Where recorded | Persisted? |
|---|---|---|
| Shift started (operator, day, time) | `shifts` row → Shifts journal | **Yes** |
| Guided overview completed | `guided_overview_completed_at` on the shift; journey flag in memory drives the `/` → `/evchargersites` redirect | Stamp yes; redirect state no |
| Action acknowledged | `action_acknowledgements` row → Actions log, journal, acknowledged badges | **Yes** |
| Drill-in to Site breakdown | In-memory (`drilled`); sets clicks-to-metric to 1 vs baseline 6 | No |
| Interaction click | In-memory (`clicks`); avoids the bounce | No |
| Session seconds | 1-second timer in memory; drives the 15s bounce rule | No |
| Site list "load" delay | 3-second simulated delay in the Site breakdown screen, on top of the real query | Simulated |
| Empty / error list states | Forced by the Demo state switcher, or genuinely surfaced when the query returns nothing or fails | Both paths exist |

Still decorative: the header time-range switcher (24h / 7d / 30d / 90d) and the site filter pills ("All sites", "Idle blocking", …) change nothing.

---

## Open questions

1. **Data source:** where would real occupancy, energy and fault data come from, at what freshness, and how is idle-vs-charging time measured per session? The site table currently holds fixed rows.
2. **Derived KPIs:** the headline occupancy and duration figures are fixed copy — should they be computed from the stored site rows, and over which window?
3. **Identity and access:** the prototype has no sign-in and allows anonymous writes. What identity model do real operators use, and who may read another operator's journal?
4. **Retention:** how long must shift and acknowledgement records be kept, and who can delete or amend them?
5. **Action ownership:** who owns and approves an idle fee in reality — network ops, pricing, or legal? The prototype defaults the owner to one person.
6. **Review dates:** what does the review date trigger in a real system — a reminder, a re-opened decision, an escalation?
7. **Guided overview lifecycle:** completion is currently per journey and resets whenever a shift is started. Should it be once ever per real user, or re-shown after a major change?
8. **If the kill switch fires:** which single metric replaces the guided headline — and does the "what to do about it" framing survive?
9. **Scope of acknowledgement:** should acknowledgement require confirmation of the fee amount and signage, as the note placeholder hints, or is decision + owner + review date enough?
10. **Verdict thresholds:** what occupancy/idle ratio boundaries define "idle blocking" vs "real capacity" in production, and should the verdict be computed rather than stored?

---

## Product vs. Experiment

**Product features (what an operator would use):** Shift start, Guided overview, Site breakdown (rows, verdicts, readings, per-row actions, loading/empty/error states), the Actions log acknowledgement flow (owner, review date, note), Shifts journal, the navigation flow that ties them together, and the stored records behind shifts, actions, sites and operators.

**Experiment features (instrumentation for the test, not part of the product story):** the live experiment strip on every screen (timer, interactions, clicks-to-metric, acknowledgement count, verdict badge), the 15-second bounce / kill-switch rule, the Demo state switcher (Data / Empty / Error), the deliberate 3-second load delay, the Experiment evidence screen and its reset, the fluorescent yellow/orange styling of everything experiment-related, and the reset-on-refresh behaviour of the session counters.
