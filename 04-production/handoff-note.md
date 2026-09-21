# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

This is a clickable, high-fidelity prototype of an EV charging network operations console built to test one hypothesis: if the first screen leads with two headline KPIs, a plain-language recommendation, evidence for it, and one suggested next step, operators will explore further and acknowledge actions instead of bouncing (baseline to beat: 60% of sessions under 15s with no interaction, 6 clicks to the most-requested metric, 1.3 weekly sessions per user). It is a pure front end — TanStack Start (React 19, Vite 7, Tailwind v4) with hard-coded Scottish demo data and a single in-memory React context holding all state; there is no database, no auth, no API, and a page refresh wipes everything. Roughly half the codebase is "product" (the six operational screens an operator would use) and half is "experiment" (instrumentation that measures and simulates the study itself); the two layers are visually separated — experiment UI is always fluorescent yellow/orange, product UI never is. If you only do one thing after reading this: open `/shiftstart`, pick a name, and click through the journey described under "How to run it".

## Architecture (plain language)

- **Frontend:** - **Stack:** TanStack Start v1 (SSR-capable React 19 on Vite 7), TanStack Router with file-based routes, Tailwind CSS v4 with semantic design tokens in `src/styles.css`, shadcn-style components, lucide-react icons. Dark navy "analytics console" look. - **Routing:** every route under `src/routes/` is a thin wrapper (head metadata + re-export). The real screen lives in `src/features/<screen-name>/<ScreenName>Screen.tsx`. Screen names, routes, and folder names match one-to-one:  | Screen (front-end name) | Route | Screen code | Feature folder | |---|---|---|---| | Shift start | `/shiftstart` | `src/routes/shiftstart.tsx` | `src/features/shift-start/` | | Guided overview | `/` | `src/routes/index.tsx` | `src/features/guided-overview/` | | Site breakdown | `/evchargersites` | `src/routes/evchargersites.tsx` | `src/features/ev-charger-sites/` | | Actions log | `/experimentlog` | `src/routes/experimentlog.tsx` | `src/features/experiment-log/` | | Shifts journal | `/shiftsjournal` | `src/routes/shiftsjournal.tsx` | `src/features/shifts-journal/` | | App experiment | `/evidence` | `src/routes/evidence.tsx` | `src/features/experiment-evidence/` |  - **Shell:** `src/features/shared/components/app-shell.tsx` renders the left icon rail + labelled nav ("EV Charge Network Ops"), header breadcrumb, time-range controls, and the experiment strip (see below). Note: despite the name "Actions log", that screen and the App experiment screen are experiment-layer; their nav labels are styled with the fluorescent tokens via an `experiment: true` flag on the nav item.
- **Backend / data:** - **There is none.** No Lovable Cloud, no Supabase, no server functions, no persistence of any kind. Every piece of content is a hard-coded sample-data file sitting next to its screen, deliberately separated from display code:   - `src/features/ev-charger-sites/data/sites.ts` — 7 charging sites (Gretna M74 J22, Abington M74 J13, Stirling M9 J9, Perth Broxden M90 J11, Livingston M8 J3, Bothwell M74 J5, Dundee Kingsway A90) with occupancy/queue/verdict data.   - `src/features/shift-start/data/operators.ts` — 10 Scottish operator names used to simulate different users.   - `src/features/guided-overview/data/overview.ts` — the headline KPIs, recommendation text, evidence bullets, expected outcomes.   - `src/features/experiment-evidence/data/experiment.ts` — user quotes, baseline metrics, kill-switch copy. - The header time-range and filter controls are decorative; nothing fetches anything.
- **Key flows:** 1. **Product flow (the thing being tested):** Shift start (pick a simulated user) → Guided overview (two headline KPIs, "Highest priority action required" card: recommendation → evidence → expected outcome → one primary action) → Site breakdown (operational hub; skeleton → 3-second simulated load → ranked site table) → a site's action opens the Actions log (acknowledge with owner, review date, note) → back to Site breakdown. Shifts journal records actions per user per day. No path returns to the Guided overview except starting/changing a user in Shift start.
2. **Experiment flow (instrumentation, not product):** a live strip on every screen counts session seconds, interactions, clicks-to-metric, and acknowledgements, and shows a verdict badge — after 15 seconds with no click it flips to "Bounced — kill switch armed". The App experiment screen (`/evidence`) shows the hypothesis, quotes, baseline, kill-switch status, and a reset button. Site breakdown's loading/empty/error states are driven by a manual "Demo state" switcher (data / empty / error) plus a simulated 3-second load; the empty and error messages are prescribed copy, not real fetch failures.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| **Solid (keep as-is when productising):** - Feature-folder structure with screen/data separation — adding a real data source means swapping a `data/*.ts` file for a fetch layer, nothing else moves. - Route names that match screen names; thin route files with proper head metadata per screen. - The Site breakdown state machine (skeleton with hourglass progress → empty → error → data, re-triggerable via `runId`). - The semantic token system: product colours in `--color-*` theme tokens, experiment colours isolated in `--experiment-text` (fluorescent yellow) and `--experiment-emphasis` (fluorescent orange), applied only to experiment components. No white text in the experiment layer; product colours (warning/fault/capacity/verdicts) untouched. - The no-loop navigation rules, including per-user onboarding completion — the state model is small and explicit. | solid | _____ |
| **Duct tape (known shortcuts — do not copy into production):** - **Zero persistence.** Refresh wipes shifts, acknowledgements, journey state, and experiment metrics. There is no audit trail of any kind. - **Simulated everything.** The 3-second load, the empty state, and the error state are triggered by a hand-operated "Demo state" switcher, not by network conditions. The bounce verdict is a 15-second timer heuristic, not real telemetry. - **Hard-coded sample data**, including the recommendation ("Apply idle fee at Stirling M9 J9") and its evidence bullets. The header time-range/filters change nothing. - **`globalThis` context singleton** — a workaround for dev-mode duplicate modules; it's load-bearing, but a real app should replace it with a proper provider tree and verified single-module builds. - Default owner ("Fiona Ross, Network Ops") and review date in the Actions log are hard-coded stubs. - Playwright quirk: the Demo state switcher buttons don't match `get_by_role(..., exact=True)`; use `locator("button", has_text=...)`, and wait ~2s for hydration before clicking. | rough | _____ |

## Risks & assumptions for the team

- **The hypothesis is instrumented, not validated.** No real users have run through this yet; the baseline (60% bounce / 6 clicks / 1.3 sessions) is the target to beat, and the kill-switch rule applies: if a guided headline still bounces, the metric itself may be wrong — pivot.
- **Experiment metrics are simulated, so any "result" from a demo session proves nothing on its own.** A real study needs actual session telemetry replacing the in-context counters.
- **Single-operator-per-session model.** The 10 operators are personas one tester switches between; there is no real multi-user or auth concept, and roles/permissions are out of scope entirely.
- **No backend means no data integrity story**: acknowledgements can't collide, be lost, or be audited because they only exist in one browser tab.
- **TanStack Start + Worker runtime assumptions:** if this ships beyond a prototype, server code must respect the edge runtime constraints (no `child_process`, no native addons); currently moot since there are no server functions.
- **The globalThis singleton is a race-prone pattern** if multiple providers ever mount (e.g. SSR + client hydration ordering); it works today, but it is the first thing to suspect if "useRun must be used inside RunProvider" reappears.

## How to run it

```
```bash
# install and run
bun install        # or pnpm/npm install
bun run dev        # dev server on http://localhost:8080

# other scripts (package.json)
bun run build      # production build (vite build)
bun run lint       # eslint
bun run format     # prettier
```

Then click through the intended journey:

1. Open `http://localhost:8080/shiftstart` — pick a name (e.g. Fiona Ross), "Start shift and continue".
2. Land on the Guided overview (`/`) — read the priority card, click "Acknowledge and record action".
3. On the Actions log, record the acknowledgement, then "Back to site breakdown".
4. On Site breakdown (`/evchargersites`) — watch the 3-second skeleton, then try the Demo state switcher (empty / error / data) and a per-row action.
5. Shifts journal (`/shiftsjournal`) shows the day's actions grouped per user.
6. App experiment (`/evidence`) shows hypothesis, baseline, kill switch, and Reset. Let a session sit idle 15s on any screen to see the "Bounced — kill switch armed" verdict in the strip.
7. Back to Shift start and pick a different name — the Guided overview reopens for that user (per-user onboarding), then continues to Site breakdown.

Further reading, both already delivered as documents alongside this file:

- **PRD:** `dashboard-nobody-reads-prd.md` — per-screen requirements, what's mocked vs real, Product vs Experiment split.
- **README (v2):** `dashboard-nobody-reads-prototype-readme_v2.md` — hypothesis, screens, user flow, build decisions, tester notes.
```
