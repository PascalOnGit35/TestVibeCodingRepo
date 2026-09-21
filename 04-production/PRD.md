# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

The recurring dashboard failure this prototype answers: reporting screens say what happened but not what to do, and the number people want is buried. Tied directly to the hypothesis under test — a first screen with two headline figures, a plain-language recommendation and one suggested next step makes people go deeper and record a decision instead of leaving. Includes the baseline it must beat (60% bounce under 15 seconds with no interaction, 6 clicks to the most-requested figure, 1.3 weekly sessions per user) and the kill switch: if a guided headline still bounces, the metric itself may be wrong. Honest note that the hypothesis is stated and instrumented here, not yet validated by real users.

## Users & jobs

- **Primary user:** the network operations person starting a shift.
- **Job to be done:** come in, learn what needs attention, decide one thing, and leave a record the next shift can read. Secondary users named honestly: new hires (the one-time guided overview) and the researcher running the test session.

## Scope

- **In:** shift start with simulated user selection, one-time guided overview per simulated user, site breakdown as the operational hub with loading / empty / error states, action acknowledgement with owner, review date and note, shifts journal grouped by day and person.
- **Out (explicitly):** real data feeds, real authentication, persistence across refresh, pricing or billing enforcement, engineer dispatch, charts and filter walls on the first screen, alerting, permissions.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | navigation rules | Must | shift start opens the guided overview for every explicit user change; the guided overview never reopens by itself; site-level screens return to site breakdown |
| 2 | evidence-before-action ordering on the priority card | Should | _____ |

## Data & events

_What gets stored, what gets tracked._

The data objects the prototype uses (site record fields, operator list, acknowledgement record, shift record) and the events it records (shift started, overview completed, drill-in, action acknowledged, interaction click, session seconds). Each marked mocked or real: all data is hard-coded sample data held in memory, all events are counted in the browser only and lost on refresh; nothing is stored or sent anywhere.

## Open questions

The decisions this prototype cannot answer: where real occupancy and energy data comes from and how fresh it is, who owns and approves an idle fee, what a review date triggers, whether the journal needs to survive a refresh and for how long, whether the guided overview should be dismissible permanently, and which single metric replaces the headline if the kill switch fires.
