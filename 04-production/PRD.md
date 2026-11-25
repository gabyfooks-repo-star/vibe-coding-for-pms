# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

The people who should make channel decisions can't get one answer out of the current dashboard: what share of referrals to the purchase flow, and of purchases, come from the native mobile app versus the web, by region. The data exists, but the dashboard shows it as twelve equal-weight charts with no takeaway, so most visitors leave without acting. 

Bounce rate: 60%
Charts on the default landing view: 12
Avg. weekly sessions per active user: 1.3
Users who export to a spreadsheet: 74%

These baselines come from internal research supplied with the brief, not from the prototype. Users describe the same problem:
• “I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it.” Marketing manager
• “It tells me what happened but never what to do about it. I still export to a spreadsheet to think.” Product lead
• “The number I need is in there, but it takes six clicks and three filters to find it.” Growth analyst

## Users & jobs

- **Primary user:** Primary user: the mobile app product manager. They own the native app channel and decide where to invest: pushing web visitors into the app, fixing app checkout, or leaving a region alone. They open the dashboard to make that call, not to explore.
- **Job to be done:** Job to be done: When I open the channel dashboard, I want to see what share of referrals to the purchase flow and of purchases come from the app versus the web, for my region and period, and what to do about it, so I can decide where to invest without exporting to a spreadsheet.

## Scope

- **In:** Version 1 is the decision-first Channel share page plus the measurement to test the hypothesis; everything else in the prototype supports it.
In scope
• Channel share page: headline metric, plain-language takeaway, one recommended action, app vs. web split, region table
• Segmentation by region (All, North America, EMEA, APAC, LATAM) and by date range, with a calendar picker and presets
• Recommended action that opens a prefilled brief and creates a record
• Revenue page: gross ticket fees by purchase flow
• Retention page: share of buyers who purchase again within 6 months, by first-purchase channel
• Chart detail pages: daily trend, insights and region table for each chart
• Navigation between pages with no dead ends
• Loading, empty and error states on Channel share
• Event tracking to measure bounce rate and action clicks
• CSV export of the current view
- **Out (explicitly):** Out of scope for version 1
• Redesigning or retiring the current 12-chart dashboard
• An A/B test harness inside the product (the hypothesis is measured with events instead; see Open questions)
• Channels other than the native app and web (box office, partners, resellers)
• Attribution beyond the channel where the session started, including cross-device stitching
• Drill-down below region (country, city, venue, event)
• Machine-learned recommendations; version 1 uses fixed rules
• Alerts, scheduled reports and notifications
• Sending briefs to an external tool such as a ticketing or experiment system, until the destination is agreed

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | _____ | _____ | _____ |
| 2 | _____ | _____ | _____ |

## Data & events

_What gets stored, what gets tracked._

Referrals to the purchase flow
- date, region, channel (app or web), count
- Daily
- Mocked. Hard-coded 30-day totals per region for Oct 6 – Nov 4, 2026, spread into made-up daily values for Jan 1 – Nov 4 so the date filter works

Purchases
- date, region, channel, count
- Daily
- Mocked, same method as referrals

Gross ticket fees
- date, region, channel, fee amount per purchase
- Per purchase, rolled up daily
- Mocked. A fixed average fee per region and channel, multiplied by purchases

Retention cohort
- buyer, first purchase date, first-purchase channel, region, repeat purchase within 6 months
- Per buyer
- Mocked. Fixed counts for one cohort (first purchases Apr 6 – May 5, 2026)

Data-through date
- latest complete day
- Daily
- Mocked. Fixed at Nov 4, 2026

## Open questions

[ ] How will the hypothesis be tested before full build? Moderated sessions with the prototype, a live pilot with the events above, or a live A/B against the current dashboard? Who signs off on the result?
[ ] What are the pass lines? The prototype proposed bounce rate at or below 40% and action click rate at or above 30%. Neither is agreed.
[ ] Is the mobile app PM the right primary user? The user quotes come from a marketing manager, a product lead and a growth analyst, not a PM.
[ ] How is a referral defined and attributed? Channel where the session started, or where the purchase flow began? How are sessions that move from the app to the web, or across devices, counted?
[ ] Where does the real data come from, and how fresh is it? Which tables or services hold daily referrals, purchases and fees by channel and region, and how many days does the data lag?
[ ] What exactly counts as gross ticket fees? Do delivery, facility or insurance fees count? Are refunds and chargebacks ever netted out?
[ ] How is retention measured? Is 6 months 180 days, 183 days or calendar months? Is the first purchase the buyer's first ever, or the first in the cohort window? Which cohort should the page show by default?
[ ] Who owns the recommendation rules and estimates? Fixed rules (as prototyped) or analyst-curated actions? The 5% shift assumption needs an owner.
[ ] Where does a confirmed brief go? An experiment platform, a ticketing tool, or a shared list? Until that's agreed, briefs are out of scope for sending.
[ ] Should loading, empty and error states cover Revenue, Retention and detail pages too? The prototype has them on Channel share only.
[ ] Are four regions enough? Or is country drill-down needed for version 1?
