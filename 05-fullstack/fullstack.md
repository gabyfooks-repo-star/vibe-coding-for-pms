# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

_____

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| daily_channel_metrics | region_id (FK → regions), day, channel (enum app/web), referrals, purchases, gross_fees | One row per region, day and channel; 2,632 rows, Jan 1 – Nov 25, 2026. Every metric and chart on Channel share and Revenue is calculated from it. Read-only through the API; RLS on. |
| action_briefs | id, owner_team_id (FK → teams), recommendation_id (FK → recommendations), region_id, kind (enum), status (enum), range_start, range_end | Written when a user confirms the recommended action. The only table the app writes to. Inserts allowed from the app (prototype only); reading requires sign-in. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

row level security is on for every table. The metrics and reference tables are read-only with the public key, action_briefs accepts inserts but can't be read without signing in, and nothing can be updated or deleted through the API.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | A new user's My briefs just said "You haven't confirmed a brief yet". If a filter returned no rows, Channel share showed the "No data" message with only a Refresh button. Revenue and Retention showed blank or broken numbers. | Every empty screen says what's missing and has a Get started button that does something useful. On Channel share and Revenue it resets to the last 30 days in all regions. On Retention it goes to Channel share. On My briefs it opens the recommended action, filled in and ready to confirm. |
| Bad / malicious input | No sign-in. Anyone with the public key could read all the data and insert briefs, and the brief owner was free text. | Sign-in is required, and row level security limits each user to their own briefs. A brief created under another user's ID is rejected, and only status and owner team can be edited. Enum types and check constraints reject invalid values (e.g. purchases above referrals, unknown status), and React escapes all displayed text. |
| Failure / offline | A failed startup load showed an error whose button reloaded the whole page. A failed brief save showed a line of text with no way to retry, and a retry after a lost reply could create a duplicate. | Every failure shows "Connection lost. Please refresh the dashboard." inline with a Retry that re-runs only that request. The page, the filters and the sign-in session all stay put. Retried saves reuse the same brief ID, so they can't duplicate, and a failed edit goes back to its saved value with a Retry option. |

## Stress test results

_What you threw at it, and what held / broke._

_____
