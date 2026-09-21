# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

The recurring dashboard failure this prototype answers: reporting screens say what happened but not what to do, and the number people want is buried. Tied directly to the hypothesis under test — a first screen with two headline figures, a plain-language recommendation and one suggested next step makes people go deeper and record a decision instead of leaving. Includes the baseline it must beat (60% bounce under 15 seconds with no interaction, 6 clicks to the most-requested figure, 1.3 weekly sessions per user) and the kill switch: if a guided headline still bounces, the metric itself may be wrong. Honest note that the hypothesis is stated and instrumented here, not yet validated by real users.

## Users & jobs

- **Primary user:** the network operations person on shift (e.g. Fiona Ross, Network Ops). Simulated in the prototype by a picker of 10 named operators.
- **Job to be done:** come on shift, quickly learn what needs attention and why, decide one thing, and leave a record — what was decided, on which site, who owns it, when it gets reviewed — that the next shift can read.

## Scope

- **In:** - Shift start: pick the simulated operator, start a shift, open a journal day.
- Guided overview: two headline KPIs, evidence-backed recommendation, one primary action; shown once per simulated user journey.
- Site breakdown: ranked per-site table with verdicts, readings, per-row actions; the operational hub every site-level screen returns to; loading / empty / error states.
- Actions log: acknowledge a decision with owner, review date, optional note; running log.
- Shifts journal: actions grouped by day and operator with timestamps.
- Experiment instrumentation (see "Product vs. Experiment" below).
- **Out (explicitly):** - Real data feeds — all site figures are hard-coded sample data.
- Persistence — everything lives in browser memory; a refresh resets the run. No backend, no accounts, no authentication, no permissions.
- Enforcement of actions: no actual idle-fee engine, no engineer dispatch, no billing.
- The dashboard anti-patterns the hypothesis argues against: chart walls and filter bars on the first screen, alerting, multi-step wizards.
- Multi-tenant or role-based access.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | navigation rules | Must | shift start opens the guided overview for every explicit user change; the guided overview never reopens by itself; site-level screens return to site breakdown |
| 2 | evidence-before-action ordering on the priority card | Should | _____ |

## Data & events

_What gets stored, what gets tracked._

## Data & events

**Everything below is mocked. There is no backend, no database, and no network call anywhere in the prototype; nothing is stored or sent anywhere.**

### Data objects (all hard-coded sample data)

| Object | Fields | Source | Mocked / Real |
|---|---|---|---|
| Site record | id, name, junction, stalls, occupancy %, avgDuration, chargeHours, idleHours, energy (kWh/stall/week), faults, reading, verdict (idle-blocking / capacity / maintenance / healthy), action | `src/lib/prototype-data.ts` — 7 records | **Mocked** — illustrative figures, not live readings |
| Operator list | 10 names | same file | **Mocked** — simulated users, no accounts |
| Headline KPIs | occupancy (78%), duration (3h 42m) + notes/deltas | same file | **Mocked** |
| Recommendation | headline, body, action, siteId (Stirling M9 J9) | same file | **Mocked** |
| Supporting cards | maintenance count/detail, capacity count/detail | same file | **Mocked** |
| Quote / baseline / kill-switch copy | strings | same file | **Mocked** (research framing, not measured data) |
| Acknowledgement record | id, siteId, siteLabel, decision, owner, reviewDate, note, timestamp, operator, dayKey, dayLabel | `src/components/prototype/run-state.tsx` | **Real interaction, mocked storage** — genuinely created in-session, held in memory only |
| Shift record | id, operator, dayKey, dayLabel, startedAt | same file | **Real interaction, mocked storage** |

### Events (all counted in the browser only, lost on refresh)

| Event | Where recorded | Mocked / Real |
|---|---|---|
| Shift started (operator, day, time) | Run state → Shifts journal | Real interaction, in-memory |
| Guided overview completed (per operator journey) | Run state; drives the `/` → `/why` redirect | Real interaction, in-memory |
| Drill-in to Site breakdown | Run state (`drilled`), sets clicks-to-metric to 1 vs baseline 6 | Real interaction, in-memory |
| Interaction click | Run state (`clicks`); avoids the bounce | Real interaction, in-memory |
| Action acknowledged | Run state → Actions log + journal + acknowledged badges | Real interaction, in-memory |
| Session seconds | 1-second timer in Run state; drives the 15s bounce rule | Real timing, in-memory |
| Site data "load" | 3-second simulated delay in `why.tsx`; no fetch occurs | **Fully mocked** |
| Empty / error list states | Triggered only by the Demo state switcher, not by real failures | **Fully mocked** |

Also mocked but worth naming: the header time-range switcher (24h / 7d / 30d / 90d) and the site filter pills ("All sites", "Idle blocking", …) are static — they do not filter or change anything.

## Open questions

1. **Data source:** where would real occupancy, energy and fault data come from, at what freshness, and how is idle-vs-charging time actually measured per session?
2. **Action ownership:** who owns and approves an idle fee in reality — network ops, pricing, or legal? The prototype defaults the owner to one person.
3. **Review dates:** what does the review date trigger in a real system — a reminder, a re-opened decision, an escalation?
4. **Journal persistence:** how long must shift records survive (across refresh, days, handovers), and who can read them?
5. **Guided overview lifecycle:** should completion persist per real user (once ever), or per period (e.g. re-shown after a major change)?
6. **If the kill switch fires:** which single metric replaces the guided headline — and does the "what to do about it" framing survive?
7. **Scope of acknowledgement:** should acknowledgement require confirmation of the fee amount and signage, as the note placeholder hints, or is decision + owner + review date enough?
8. **Verdict thresholds:** what occupancy/idle ratio boundaries define "idle blocking" vs "real capacity" in production?
