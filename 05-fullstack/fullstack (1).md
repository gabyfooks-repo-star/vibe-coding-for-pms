# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

https://houselights-ticketing-dashboard.netlify.app/

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| daily_channel_metrics | region_id (FK → regions), day, channel (enum: app / web), referrals, purchases, gross_fees | One row per region, day and channel, with 2,632 rows covering Jan 1 – Nov 25, 2026. Every metric and chart on Channel share and Revenue comes from it. Shared data, read-only through the API. |
| action_briefs | id, user_id (FK → auth.users), owner_team_id (FK → teams), recommendation_id (FK → recommendations), status (enum), updated_at | Created when a user confirms the recommended action; user_id is filled from the session. Status and owner team are editable with last-write-wins (the database stamps updated_at). Paginated 10 per page on My briefs. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Row level security is on for every table, and sign-in (email and password) is required for everything; signed-out requests get 401. Users only see and edit records where user_id matches their session: they can create, read, and update the status or owner team of their own briefs, and nothing else. Creating a brief under someone else's user_id is rejected, and briefs can't be deleted. Shared data (metrics, regions, cohorts, recommendations, teams) is read-only for any signed-in user, and nobody can write it through the API.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | A new user's My briefs just said "You haven't confirmed a brief yet". Filters with no data showed "No data" with only a Refresh button, and Revenue and Retention showed blank numbers. | Every empty screen explains what's missing and offers a Get started button. It resets filters to the last 30 days in all regions, or opens the recommended action, ready to confirm. |
| Bad / malicious input | No sign-in. Anyone with the public key could read all data and insert briefs, and the brief owner was free text. | Sign-in is required, and RLS limits each user to their own rows; a brief created under another user's ID is rejected. Only status and owner team are editable. Enums and check constraints reject invalid values, the password must be at least 6 characters, and displayed text is escaped. |
| Failure / offline | A failed load reloaded the whole page, and a request with no answer left a skeleton or "Saving…" on screen forever. A failed save had no Retry, and a retry could create a duplicate. | Every request has a 10-second limit and then shows "Connection lost. Please refresh the dashboard." inline with Retry. Retry re-runs only that request, keeping filters and the signed-in session. Retried saves can't duplicate, and failed edits go back to their saved value. |

## Stress test results

_What you threw at it, and what held / broke._

a request that failed already showed "Connection lost. Please refresh the dashboard." with Retry. But a request that never got an answer, for example on a dead network, never failed either. The screen would sit on its skeleton, "Saving…" or "Checking your sign-in…" forever. 

Now, every Supabase request has a 10-second limit, both data and sign-in (fetchWithTimeout.ts). After 10 seconds the request is cancelled and the screen moves to its inline error with Retry. The sign-in check gives up after 12 seconds and shows Retry instead of "Checking your sign-in…". The saved session is kept, so Retry picks it up. Log-in network failures now say "Connection lost. Check your connection, then try again." instead of a technical error.
