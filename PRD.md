---
title: "PRD: Decision-first channel dashboard"
---

# PRD: Decision-first channel dashboard

Nov 12, 2026 · @Gaby Fooks

This PRD is extracted from the [clickable prototype](https://claude.ai/artifact/MnCdza7xz8Gj6f9HLBeJ6G) of a decision-first channel dashboard for Houselights Tickets, a fictional live-events ticketing company. **The hypothesis behind it has not been validated yet:** no user sessions have been run, and every number in the prototype is made up. What the prototype does prove is that the screens, flows and calculations below can work together.


> **Living update, Nov 25, 2026.** This PRD was written on Nov 12 from the clickable prototype. Since then:
>
> - **Built and live.** A React + TypeScript app deployed on Netlify: [houselights-ticketing-dashboard.netlify.app](https://houselights-ticketing-dashboard.netlify.app).
> - **Real backend.** Supabase (Postgres) holds every value the dashboard shows: daily metrics, regions, the retention cohort, recommendations, settings and research. Data runs through Nov 25, 2026 and is still fictional.
> - **Sign-in and per-user records.** Email sign-up and log-in; row level security means each person only sees their own briefs.
> - **Briefs are real records.** Confirming creates a row in `action_briefs`; status and owner team are editable (last write wins).
> - **States everywhere.** Loading skeletons, inline errors with Retry, a 10-second limit on every request, and empty states with a Get started action. This answers the open question about Revenue, Retention and detail pages: yes, they have them now.
> - **One recommended action per page.** After peer testing called Revenue and Retention dead ends, each now has its own action per region.
> - **Still not built:** event tracking (so bounce rate and action click rate are not measured yet) and CSV export.
> - **Validation status: partly supported.** In peer testing most people went straight to the recommended action. Bounce rate has not been measured.

## Problem

The people who should make channel decisions can't get one answer out of the current dashboard: what share of referrals to the purchase flow, and of purchases, come from the native mobile app versus the web, by region. The data exists, but the dashboard shows it as twelve equal-weight charts with no takeaway, so most visitors leave without acting.

| Baseline (current dashboard) | Value | Definition |
| --- | --- | --- |
| Bounce rate | 60% | Sessions under 15s with no interaction |
| Charts on the default landing view | 12 |  |
| Avg. weekly sessions per active user | 1.3 |  |
| Users who export to a spreadsheet | 74% | To actually analyse the data |

These baselines come from internal research supplied with the brief, not from the prototype. Users describe the same problem:

- “I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it.” Marketing manager
- “It tells me *what* happened but never *what to do about it*. I still export to a spreadsheet to think.” Product lead
- “The number I need is in there, but it takes six clicks and three filters to find it.” Growth analyst

**Hypothesis.** We believe a first screen that leads with one headline metric, a plain-language takeaway, and one recommended action will cause users to act on the dashboard instead of bouncing, for the people who should make decisions from it. We'll know we're right when bounce rate drops and the recommended action gets clicked.

**Validation status: not yet validated.** The prototype was built to test this hypothesis, but no sessions have been run. An early version had an A/B harness (old dashboard vs. new, with a 15-second bounce timer); it was removed at the product owner's request, so the current build records nothing. This PRD treats the hypothesis as the bet being made and includes the instrumentation needed to prove or disprove it (see Requirements and Data & events). Proposed pass lines, not yet agreed: bounce rate at or below 40%, and at least 30% of Channel share sessions clicking the recommended action.

## Users & jobs

**Primary user: the mobile app product manager.** They own the native app channel and decide where to invest: pushing web visitors into the app, fixing app checkout, or leaving a region alone. They open the dashboard to make that call, not to explore.

**Job to be done:** When I open the channel dashboard, I want to see what share of referrals to the purchase flow and of purchases come from the app versus the web, for my region and period, and what to do about it, so I can decide where to invest without exporting to a spreadsheet.

Secondary users, who share the same decision and are the source of the user quotes:

| User | What they need from it |
| --- | --- |
| Marketing manager | One clear number and action instead of twelve charts |
| Product lead | A recommendation, not only what happened |
| Growth analyst | The app vs. web split by region in one click, plus the detail behind it |

Follow-on jobs the prototype also covers: see how much gross ticket fee revenue each channel brings in (Revenue), and see whether app buyers come back more often than web buyers (Retention).

## Scope

Version 1 is the decision-first Channel share page plus the measurement to test the hypothesis; everything else in the prototype supports it.

**In scope**

- Channel share page: headline metric, plain-language takeaway, one recommended action, app vs. web split, region table
- Segmentation by region (All, North America, EMEA, APAC, LATAM) and by date range, with a calendar picker and presets
- Recommended action that opens a prefilled brief and creates a record
- Revenue page: gross ticket fees by purchase flow
- Retention page: share of buyers who purchase again within 6 months, by first-purchase channel
- Chart detail pages: daily trend, insights and region table for each chart
- Navigation between pages with no dead ends
- Loading, empty and error states on Channel share
- Event tracking to measure bounce rate and action clicks
- CSV export of the current view

**Out of scope for version 1**

- Redesigning or retiring the current 12-chart dashboard
- An A/B test harness inside the product (the hypothesis is measured with events instead; see Open questions)
- Channels other than the native app and web (box office, partners, resellers)
- Attribution beyond the channel where the session started, including cross-device stitching
- Drill-down below region (country, city, venue, event)
- Machine-learned recommendations; version 1 uses fixed rules
- Alerts, scheduled reports and notifications
- Sending briefs to an external tool such as a ticketing or experiment system, until the destination is agreed

## Requirements

P0 is what the hypothesis needs to be tested: the decision-first first screen and the events that measure it. P1 completes the dashboard; P2 adds depth.

| Requirement | Priority | Acceptance criteria |
| --- | --- | --- |
| Channel share leads with one headline metric: the app's share of purchases | P0 | Visible without scrolling at 1280×800. Equals app purchases ÷ (app + web purchases) for the selected region and range, to 0.1 point. Exactly one headline-size number on the page. |
| A plain-language takeaway under the headline, written from the data | P0 | One or two sentences naming the app's share of referrals, its share of purchases, and app vs. web conversion. Wording changes when the app converts below web (as in LATAM). No fixed copy. |
| One recommended action with its reason, estimated impact and caveat | P0 | Exactly one action per region view. Impact shown per 30 days whatever range is selected. Where the app converts below web, the action is a checkout review, not a push to the app. The caveat states the estimate is an upper bound. |
| The action opens a prefilled brief and creates a record | P0 | Brief is prefilled with region, audience, change, primary metric, guardrail and impact. Owner can be chosen. Confirming creates a record in the agreed system and shows its ID. A back link returns to the dashboard. |
| Event tracking for bounce rate and action clicks | P0 | Every event listed in Data & events fires with its properties. Bounce rate (session under 15s with no interaction) and action click rate can be reported by week, region and user role. |
| App vs. web split of referrals and purchases | P0 | Both shares and raw counts are printed on screen, never only on hover. Mobile app is always blue and web always orange, on every page. |
| Region segmentation | P0 | One click switches between All regions, North America, EMEA, APAC and LATAM. Every figure, the takeaway and the action update. The selection carries across pages. |
| Date range filter with calendar and presets | P1 | Presets: last 7, 30 and 90 days, last full month, year to date. Custom range picked on a calendar, limited to dates with data. Apply, Cancel and Escape work. All date-based figures update; Retention says it uses a fixed cohort instead. |
| Loading, empty and error states on Channel share | P1 | Skeleton in the shape of the cards while loading. Empty: “No data. Check your filters or refresh the screen.” Error: “Connection lost. Please refresh the dashboard.” Each has a Refresh button. A newer request always wins over an older one. |
| Navigation with no dead ends | P1 | Page tabs for Channel share, Revenue and Retention on every product screen, marking the current page. Every screen, including the brief and its confirmation, has a way back to Channel share. |
| Revenue page: gross ticket fees by purchase flow | P1 | Headline is the app's share of gross ticket fees. Shows the definition on screen, fee and purchase splits, average fee per purchase by channel, and fees by region. Linked from the Channel share takeaway. |
| Retention page: repeat buyers within 6 months | P1 | Definition on screen: a buyer is retained if they purchase again within 6 months of their first purchase in the cohort. Shows the cohort dates, the overall repeat rate, app vs. web repeat rates and a region table. |
| Accessible and usable on any screen | P1 | Text contrast at least 4.5:1. Touch targets at least 44px on touch screens. Filters, tabs and charts work by keyboard. Every chart has a text alternative or a table. Works at phone width without sideways scrolling. |
| Chart detail pages | P2 | Every chart opens a detail page with three summary figures, a daily chart readable by hover or arrow keys, three data-driven insights, a definition and a table by region. |
| CSV export of the current view | P2 | Exports the visible table with the current region and date range applied, and logs an export event. |

## Data & events

Every data value in the prototype is mocked; the screens, filters, formulas and navigation are real working logic. Nothing is connected to a data source, and no events are tracked.

### Data the product needs

| Dataset | Fields | Grain | In the prototype |
| --- | --- | --- | --- |
| Referrals to the purchase flow | date, region, channel (app or web), count | Daily | **Mocked.** Hard-coded 30-day totals per region for Oct 6 – Nov 4, 2026, spread into made-up daily values for Jan 1 – Nov 4 so the date filter works |
| Purchases | date, region, channel, count | Daily | **Mocked**, same method as referrals |
| Gross ticket fees | date, region, channel, fee amount per purchase | Per purchase, rolled up daily | **Mocked.** A fixed average fee per region and channel, multiplied by purchases |
| Retention cohort | buyer, first purchase date, first-purchase channel, region, repeat purchase within 6 months | Per buyer | **Mocked.** Fixed counts for one cohort (first purchases Apr 6 – May 5, 2026) |
| Data-through date | latest complete day | Daily | **Mocked.** Fixed at Nov 4, 2026 |

### What is real and what is mocked in the prototype

| Part | Status | Notes |
| --- | --- | --- |
| Screens, layout, navigation, region and date filters | Real | Working logic, kept in one shared state so filters carry across pages |
| Formulas for shares, conversion, fees, repeat rates | Real | Correct as written; they run on mocked inputs |
| Takeaways, recommendations and detail-page insights | Real logic | Generated from the data by fixed rules. The impact estimate assumes 5% of web referrals move to the app and convert at the app's rate, an untested assumption |
| Data fetch | Mocked | A 900 ms timer; a prototype-only Simulate menu forces the empty and error states |
| Brief confirmation | Mocked | No record is created; ticket IDs such as EXP-2148 are fixed text |
| CSV export | Mocked | Shows a note; no file is produced |
| Baseline metrics (60%, 12, 1.3, 74%) | From research | Supplied with the brief; not measured by the prototype |
| Event tracking | Not built | Specified below |

### Events to track

A session counts as **interacted** once any event marked as an interaction fires. **Bounce rate** = Channel share sessions under 15 seconds with no interaction ÷ all Channel share sessions. **Action click rate** = Channel share sessions with `recommended_action_clicked` ÷ all Channel share sessions.

| Event | Fires when | Key properties | Interaction? |
| --- | --- | --- | --- |
| `dashboard_viewed` | A dashboard page loads | page, region, date range, user role, session ID | No |
| `data_state_shown` | Channel share shows loading, empty or error | state, region, date range, load time (ms) | No |
| `region_changed` | A region tab or table row is picked | from, to, page | Yes |
| `date_range_changed` | A preset or custom range is applied | preset or custom, start, end, page | Yes |
| `recommended_action_clicked` | The recommended action button is clicked | action ID, region, date range, estimated impact | Yes |
| `brief_confirmed` | The brief is confirmed | action ID, owner, region | Yes |
| `chart_detail_opened` | A chart or its Details button is clicked | chart ID, page, region | Yes |
| `page_tab_clicked` | A page tab or page link is clicked | from page, to page | Yes |
| `export_clicked` | Export CSV is clicked | page, region, date range | Yes |
| `refresh_clicked` | Refresh is clicked in the empty or error state | state | Yes |
| `session_ended` | The tab closes or is idle for 30 minutes | duration (s), interacted (yes or no) | No |

## Open questions

The first two decide whether this ships; the rest shape how it's built.

- [ ] **How will the hypothesis be tested before full build?** Moderated sessions with the prototype, a live pilot with the events above, or a live A/B against the current dashboard? Who signs off on the result?
- [ ] **What are the pass lines?** The prototype proposed bounce rate at or below 40% and action click rate at or above 30%. Neither is agreed.
- [ ] **Is the mobile app PM the right primary user?** The user quotes come from a marketing manager, a product lead and a growth analyst, not a PM.
- [ ] **How is a referral defined and attributed?** Channel where the session started, or where the purchase flow began? How are sessions that move from the app to the web, or across devices, counted?
- [ ] **Where does the real data come from, and how fresh is it?** Which tables or services hold daily referrals, purchases and fees by channel and region, and how many days does the data lag?
- [ ] **What exactly counts as gross ticket fees?** Do delivery, facility or insurance fees count? Are refunds and chargebacks ever netted out?
- [ ] **How is retention measured?** Is 6 months 180 days, 183 days or calendar months? Is the first purchase the buyer's first ever, or the first in the cohort window? Which cohort should the page show by default?
- [ ] **Who owns the recommendation rules and estimates?** Fixed rules (as prototyped) or analyst-curated actions? The 5% shift assumption needs an owner.
- [ ] **Where does a confirmed brief go?** An experiment platform, a ticketing tool, or a shared list? Until that's agreed, briefs are out of scope for sending.
- [ ] **Should loading, empty and error states cover Revenue, Retention and detail pages too?** The prototype has them on Channel share only.
- [ ] **Are four regions enough?** Or is country drill-down needed for version 1?
