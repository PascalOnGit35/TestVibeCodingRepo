# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

_____

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| charging_sites | • site_id (PK, TEXT, e.g. 'stirling-m9-j9')<br>• site_name (TEXT)<br>• junction_label (TEXT)<br>• stall_count (INTEGER)<br>• verdict (ENUM: 'idle-blocking', 'capacity', 'maintenance', 'healthy')<br>• recommended_action (TEXT)<br>• display_rank (INTEGER) | Static seed in migration 0000: Pre-populated with the 7 key Scottish highway junction sites (Stirling M9 J9, Perth Broxden M90 J11, Abington M74 J13, Gretna M74 J22, Bothwell M74 J5, Livingston M8 J3, Dundee Kingsway A90). |
| operators | • operator_id (PK, UUID)<br>• full_name (TEXT, UNIQUE)<br>• display_order (INTEGER) | Static seed in migration 0000: Pre-populated with 10 Scottish operational personas (Fiona Ross, Callum MacLeod, Eilidh Fraser, etc.) for operator assignment on shift start. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Product Access Rules (Operational Core)
Unauthenticated (Anon): Blocked completely; all tables reject anonymous access with zero read/write permissions.
User Profiles (profiles): Operators read and edit their own profile; Supervisors and Admins can view all profiles.
Role Assignments (user_roles): Users view their own role; only Admins can grant or modify role assignments.
Ops Reference Data (charging_sites, operators): Read-only for provisioned users (operator, supervisor, admin); writes restricted to Admins.
Operational Guidance (network_headlines, network_recommendations, reasons): Read-only for provisioned users; only Admins can create or update recommendations.
Work Shifts (shifts): Operators create, read, and update only their own shifts; Supervisors and Admins can audit all operator shifts.
Decision Logs (action_acknowledgements): Operators submit and read their own logged site actions; Supervisors and Admins can inspect all team actions.
Data Deletion: Regular operators have zero delete privileges across operational data; records are append-only.
Experiment Access Rules (Telemetry & Hypothesis)
Experiment Baselines & Quotes (experiment_baselines, experiment_quotes): Readable by provisioned ops team members; editable only by Admins.
Experiment Configuration (experiment_config): Thresholds (15s bounce, 6-click target) are readable by provisioned users; only Admins can adjust criteria.
Telemetry Sessions (experiment_sessions): Operators write and read metrics for their own running sessions; Admins can review all session analytics.
Telemetry Events (experiment_events): Fine-grained UI click and route events are insert-only by the acting user; cross-user reading is blocked except for Admins.
Audit Isolation: Experiment telemetry cannot modify product operational state (shifts, sites), keeping experiment tracking strictly segregated.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | A newly registered user has no linked_operator_name yet, or their linked name was renamed/deleted from the operators table. | Add a fallback state on /shiftstart: if linked_operator_name is null or invalid, force an explicit selection from active operators before enabling "Start shift". Store a foreign key reference or validated match against operators.operator_id rather than a loose string. |
| Bad / malicious input | If an RLS policy rejects an insert or the network drops, the optimistic row remains visible on screen while only throwing a silent console error. | Add transactional error rollback with user-facing toast alerts (e.g., via sonner): if Supabase returns a 403 Forbidden or network failure, roll back the UI list and notify the operator immediately. |
| Failure / offline | _____ | Toaster pop up to ask to retry.  Then restore and toaster "Recorded". |

## Stress test results

_What you threw at it, and what held / broke._

_____
