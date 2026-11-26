# Houselights dashboard

A React + TypeScript refactor of the decision-first channel dashboard prototype (originally one file, `Main.dc.html`, in a Claude Design canvas). Screen and feature names follow the PRD, *PRD: Decision-first channel dashboard*.

**Live:** https://houselights-ticketing-dashboard.netlify.app (wired to Supabase, data through Nov 25, 2026)

> **Status: built, tested and deployed.** The app type-checks and builds cleanly (Node.js 26: `tsc` and `vite build` both pass). The production build was tested in a browser against the live Supabase project: it loads regions, daily rows and the retention cohort, calls `channel_totals()`, and saves a confirmed brief to `action_briefs`. It is deployed with Netlify Drop. There are no automated tests yet.

## Run it

Needs Node.js 18 or later.

```bash
npm install
npm run dev        # local dev server
npm run typecheck  # TypeScript only
npm run build      # type-check, then production build into dist/
```

Without a `.env.local` file the app runs on the built-in mock data. To use Supabase, follow the next section.

## Deploy it

The site is a static build, so there's no server to run. The Supabase URL and key are baked in at build time from `.env.local`.

1. Run `npm run build`. This writes the site to `dist/`.
2. In Netlify, open the site and click **Drop new files** (or use app.netlify.com/drop for a new site).
3. Drag the `dist` folder onto it. If a file picker won't take a folder, zip the *contents* of `dist` (so `index.html` sits at the top of the zip) and pick the zip instead.
4. Open the live URL and check the header says the expected "Data through" date. Refresh with Cmd+Shift+R if it shows an old build.

Changing only the data in Supabase needs no redeploy: the app reads it on every load. Changing code or the date constants in `src/data/dates.ts` needs a new build and drop.

## Supabase backend

The schema and data live in `supabase/`. Setup takes about five minutes:

1. Create a project at supabase.com (free tier is fine).
2. In the project, open **SQL Editor** and run, in order: `supabase/migrations/20261125000000_core_schema.sql`, `20261125120000_displayed_values.sql`, then `20261125130000_user_auth.sql`.
3. In new queries, run `supabase/seed.sql` (fictional daily data, re-runnable), then `supabase/seed_reference.sql` (recommendations, settings, research, data-load log).
4. Copy `.env.example` to `.env.local`. From **Project Settings > API**, fill in the project URL and the publishable (anon) key. Never use the service-role key here.
5. In **Authentication → URL Configuration**, set **Site URL** to where the app runs (e.g. your Netlify URL) and add `http://localhost:5173` and `http://localhost:4173` to **Redirect URLs**, so email-confirmation links come back to the app.
6. Run `npm install` and `npm run dev`, then create an account on the sign-in screen.

**Every value the dashboard displays comes from the database.** The only values left in code are the offline mock (used when `.env.local` is missing) and UI wording.

### Types (Postgres enums)

| Type | Values | Used by |
| --- | --- | --- |
| `purchase_channel` | `app`, `web` | `daily_channel_metrics.channel`, `retention_cohorts.channel` |
| `brief_kind` | `experiment_brief`, `review_ticket` | `recommendations.kind`, `action_briefs.kind` |
| `brief_status` | `open`, `in_progress`, `done`, `dropped` | `action_briefs.status` |
| `recommendation_strategy` | `shift_web_to_app`, `fix_app_conversion`, `match_web_fee`, `close_repeat_gap`, `fix_app_repeat` | `recommendations.strategy` (picks the impact formula) |
| `recommendation_page` | `channel_share`, `revenue`, `retention` | `recommendations.page` |
| `impact_metric` | `purchases`, `gross_fees`, `repeat_buyers` | `recommendations.impact_metric` (unit of the estimate) |
| `metric_unit` | `percent`, `count`, `decimal` | `recommendation_settings.unit`, `research_metrics.unit` |
| `load_status` | `running`, `succeeded`, `failed` | `data_loads.status` |

### Tables, views and functions

| Object | What it holds | Displayed as |
| --- | --- | --- |
| `regions` | The four regions and their display order | Region tabs, every table |
| `daily_channel_metrics` | One row per region, day and channel: referrals, purchases, gross ticket fees | Every metric, chart and table on Channel share, Revenue and detail pages |
| `retention_cohorts` | First-time buyers per cohort by first-purchase channel, and how many bought again within `window_months` | Retention figures |
| `latest_complete_cohort` (view) | The newest cohort whose retention window has fully passed | Cohort dates and "6 months" in Retention labels and definitions |
| `data_bounds()` (function) | First and last day with data | "Data through …", the default range, date presets, calendar limits |
| `channel_totals(region, start, end)` (function) | App and web totals for a region and range | Channel share loading, empty and error states |
| `recommendations` | One rule per page (`channel_share`, `revenue`, `retention`) and region: kind, strategy, impact metric, ticket ref, title, sentence templates, brief rows | The recommended action card on Channel share, Revenue and Retention, and the brief |
| `recommendation_settings` | Tunable assumptions: `shift_share` = 0.05, `retention_gap_close` = 0.10 | "if 5% of web referrals move to the app", "if 10% of the gap closes", and every impact estimate |
| `teams` | Teams that can own a brief | Owner picker on the brief |
| `action_briefs` | Confirmed briefs; `user_id` → `auth.users.id` is the person who confirmed it | Saved on confirm; listed on My briefs |
| `data_loads` | Log of data loads | (Not on screen yet; for the pipeline) |
| `research_studies`, `research_metrics`, `research_quotes` | The hypothesis, baseline numbers (60%, 12, 1.3, 74%) and user quotes | Why this design |

### Relationships

- `daily_channel_metrics.region_id` → `regions.id`
- `retention_cohorts.region_id` → `regions.id`
- `recommendations.region_id` → `regions.id` (one per page and region; one per page has `is_default_for_all`)
- `action_briefs.region_id` → `regions.id` (null = All regions)
- `action_briefs.owner_team_id` → `teams.id`
- `action_briefs.recommendation_id` → `recommendations.id`
- `action_briefs.user_id` → `auth.users.id` (filled from the session; the row's owner)
- `research_metrics.study_id` → `research_studies.id`, `research_quotes.study_id` → `research_studies.id` (one study has `is_current`)

**How the app uses it.** At startup, `app/DataGate.tsx` calls `data/supabaseDataset.ts`. It reads `data_bounds()`, `latest_complete_cohort` and the reference tables, then the cohort rows and all daily rows (in pages of 1,000). It swaps them into `data/dates.ts`, `regions.ts`, `dailySeries.ts` and `referenceData.ts`, so `metrics/` and the view models don't change. If any load fails, the app shows the PRD's error message with a Refresh button. Change a value in the database and it shows on the next page load: for example, setting `shift_share` to 0.10 changes the card to "if 10% …" and doubles the estimate.

### Sign-in and per-user records

People sign up and log in with email and password (Supabase Auth). The dashboard only loads after sign-in.

**Users only see records where `user_id` matches their session.** `action_briefs.user_id` defaults to `auth.uid()`, and its row level security policies are:

| Policy | Rule |
| --- | --- |
| Users read their own briefs | `select` for `authenticated` where `user_id = auth.uid()` |
| Users create briefs as themselves | `insert` for `authenticated` with check `user_id = auth.uid()`, status `open`, owner team set |
| (none) | No `update` or `delete`: briefs can't be changed through the API |

Checked on Nov 25, 2026: with two test users in a rolled-back transaction, each saw only their own brief out of four; a brief created with another user's `user_id` was rejected; and with the public key alone, every table and function returned 401.

**Shared data** (metrics, regions, cohorts, recommendations, settings, teams, research) is the same for everyone, so it has no `user_id`: any signed-in user can read it, nobody can write it through the API, and signed-out visitors get nothing.

The two briefs saved before sign-in existed have no `user_id`, so no one can see them. The `action_briefs_user_id_required` constraint is `NOT VALID` so they could stay; delete them and run `alter table action_briefs validate constraint action_briefs_user_id_required` to enforce it on every row.

## One recommended action per page

Peer testing found Revenue and Retention were dead ends: numbers, but nothing to do about them. Each now has its own recommended action beside its headline, like Channel share, from `recommendations` rows with that `page` (`supabase/migrations/20261125150000_page_recommendations.sql`). The definition moved under the takeaway.

| Page | All regions / North America | EMEA | APAC | LATAM | Estimate counts |
| --- | --- | --- | --- | --- | --- |
| Channel share | Open-in-app prompt on web event pages | App-install prompt on order confirmation | Campaign links straight to the app | Review app checkout drop-off | Purchases per 30 days |
| Revenue | Hand web checkouts off to the app | App early access to presales | Seat-upgrade offers in the app | Review why app fees run lower | Gross fees per 30 days |
| Retention | App prompt after a first web purchase | Tickets delivered in the app | App-only presale alerts | Review why app buyers return less | Repeat buyers per cohort |

North America is the All regions default on every page: it has the largest upside on each. LATAM gets a review ticket on every page, because there the app trails the web. The brief opens from any page and returns to it.

## Product states

Every screen that loads or saves data handles all four states the same way, using `shared/components/ProductStates.tsx`.

| State | What the user sees | Where |
| --- | --- | --- |
| Loading | A grey skeleton in the shape of the content, never a blank page | Startup load (whole page), Channel share, My briefs |
| Error | "Connection lost. Please refresh the dashboard." inline, with **Retry**. Retry re-runs only the failed request; nothing reloads, so filters and the signed-in session stay | Sign-in check, startup load, Channel share, brief save (banner), My briefs, each status/owner edit (banner) |
| Empty | What's missing plus a **Get started** button | Channel share and Revenue (reset to last 30 days, all regions), Retention (to Channel share), My briefs (opens the recommended action) |
| Ready | The data | Everywhere |

**Nothing waits forever.** Every Supabase request, data and sign-in, has a 10-second limit (`data/fetchWithTimeout.ts`). A request that gets no answer is aborted and the screen moves to its error state. The sign-in check gives up after 12 seconds and shows Retry without discarding the saved session. Tested against a local server that never answers: cut off at the limit; normal requests and the app's own cancellations are unaffected.

**Pagination.** My briefs loads 10 rows per request (`range()` with an exact count), with Previous / Next and "1–10 of N". A newer page request cancels the older one. The startup load also pages through `daily_channel_metrics` 1,000 rows at a time.

**Last write wins.** Status and owner team are editable on My briefs (`supabase/migrations/20261125140000_brief_edits_lww.sql`). Each edit sends only the changed field; a trigger stamps `updated_at` with the database clock; there's no version check, so simultaneous edits both succeed and the one applied last stands. The row then shows the saved value and time, and returning to a tab re-reads the page so it shows the latest write from anywhere. Retrying a failed brief save reuses the same ID, so it can't create a duplicate.

## How it's organised

Data, calculations and display are separate layers. Each layer only imports from the ones above it.

```
src/
  data/              1. Raw data: mock values, or loaded from Supabase
    regions.ts            region totals, fees, retention cohort
    dailySeries.ts        made-up daily values behind the date filter
    dates.ts              day numbers and formatting; bounds and presets set from data_bounds()
    referenceData.ts      types + offline mock for teams, recommendations, settings, cohort, research
    channelDataSource.ts  Channel share fetch: channel_totals() in Supabase, or a simulated timer
    supabaseClient.ts     the Supabase client, null when .env.local is not set
    fetchWithTimeout.ts   10-second limit on every Supabase request
    supabaseDataset.ts    loads every displayed value from Supabase at startup
  metrics/           2. Pure calculations, no React
    metrics.ts            computeMetrics(): every figure for a region and range
    chartGeometry.ts      scales, ticks and SVG points as plain data
    format.ts             number, percent and money formatting
  features/          3. One folder per PRD screen: a view model (*.model.ts) + components
    channel-share/        ChannelShare, RecommendedActionCard, DataStates, useChannelData
    revenue/              Revenue, revenue.css (Tableau-style polish)
    retention/            Retention
    chart-detail/         ChartDetail, charts/ (LineChart, SmallMultiples, PairedBars, StackedShareBars)
    action-brief/         RecommendedActionBrief, recommendation.ts (rules + estimates), briefRepository.ts (saves to Supabase)
    auth/                 SignIn (email sign-up and log-in)
    my-briefs/            MyBriefs, myBriefs.model.ts (the signed-in user's briefs)
    why-this-design/      WhyThisDesign (research from Supabase)
  shared/            Reused by several features
    types.ts, navigation.ts, split.ts
    date-range/           DateRangeFilter + calendarMonth view model
    components/           RegionTabs, SplitBars, DataTable, PageLinks, ChartBits, Toast, Icons, ProductStates (loading / error / empty)
  app/               App (state + navigation), auth.tsx (session), AuthGate (sign-in first), DataGate (data load), PrototypeBar, ProductHeader
  styles/global.css  tokens and shared styles
supabase/
  migrations/        core_schema, displayed_values (types, tables, relationships, view, functions), user_auth (user_id + per-user RLS)
  seed.sql           the prototype's fictional daily data
  seed_reference.sql recommendations, settings, research, data-load log
```

**The rule:** `*.model.ts` files turn metrics into display-ready strings and lists, and components only render what a model gives them. To change a number or a sentence, look in `data/`, `metrics/` or a `*.model.ts` file, never in a `.tsx` file.

## Screens (PRD names)

| PRD screen | Component | View model |
| --- | --- | --- |
| Channel share | `features/channel-share/ChannelShare.tsx` | `channelShare.model.ts` |
| Revenue | `features/revenue/Revenue.tsx` | `revenue.model.ts` |
| Retention | `features/retention/Retention.tsx` | `retention.model.ts` |
| Chart detail pages | `features/chart-detail/ChartDetail.tsx` | `chartDetail.model.ts` |
| Recommended action brief | `features/action-brief/RecommendedActionBrief.tsx` | `recommendation.ts` |
| Why this design | `features/why-this-design/WhyThisDesign.tsx` | `research_*` tables via `data/referenceData.ts` |

## State and data flow

`App.tsx` holds the only shared state: the current route, the region filter, the date range and the prototype's Simulate setting. Region and range carry across every screen. UI-only state stays inside its component: the calendar's draft range, the line chart's hover, the brief's owner.

```
data/ (Supabase, or mock) → metrics/computeMetrics() → features/*/*.model.ts → components
                                ↑
          App state: region, range (and route, scenario)
```

Only Channel share loads through a fetch (`useChannelData` → `fetchChannelData`). It shows loading, then exactly one of ready, empty or error. A newer request aborts the older one.

## Mocked vs. real

| Part | Status |
| --- | --- |
| Screens, navigation, region and date filters | Real |
| Formulas in `metrics/` and the view models | Real, running on fictional data (from Supabase in the live app) |
| Recommendation rules and impact estimates | Rules and wording in `recommendations`; the 5% shift in `recommendation_settings` is an untested assumption |
| Supabase schema, security rules and `channel_totals()` | Real; applied to a Supabase project and checked (totals, anon reads, anon brief insert, briefs hidden from anon), and used by the live app |
| Numbers in `data/` and `supabase/seed.sql` | Mocked (fictional). The seed matches the mock to within rounding |
| `fetchChannelData` | Real `channel_totals()` call when Supabase is configured, else a 900 ms timer. Simulate still forces empty or error |
| Brief confirmation | Saves a row in `action_briefs` owned by the signed-in user, with its team and recommendation. Ticket refs come from `recommendations`, not a ticketing system |
| Sign-up and log-in | Real (Supabase Auth, email and password) |
| CSV export | Mocked: shows a note, creates no file |
| `PrototypeBar` | Prototype only; remove before shipping |
| Event tracking from the PRD | Not built |

## Where each part of the old file went

| `Main.dc.html` | Now |
| --- | --- |
| `REGIONS` | `data/regions.ts` (fields renamed: `ra/rw` → `referrals.app/web`, `pa/pw` → `purchases`, `fa/fw` → `avgFee`, `ba/bw` → `cohortBuyers`, `ka/kw` → `repeatBuyers`) |
| `series()`, `total()` | `data/dailySeries.ts`, `metrics/metrics.ts` |
| `metric()` | `metrics/computeMetrics()` |
| `load()`, `setScenario` | `data/channelDataSource.ts`, `features/channel-share/useChannelData.ts` |
| `takeawayFor()` | `features/channel-share/channelShare.model.ts` |
| `actionFor()`, `shiftGain()` | `features/action-brief/recommendation.ts` |
| `revenueFor()`, `retentionFor()` | `revenue.model.ts`, `retention.model.ts` |
| `detailFor()`, `lineChart()`, `panels()`, `niceScale()` | `chart-detail/chartDetail.model.ts`, `metrics/chartGeometry.ts` |
| `calVals()` and calendar handlers | `shared/date-range/` |
| `renderVals()` | Split across the view models; each screen builds its own |
| `go()`, `openDetail()`, `state.screen` | `App.tsx` route state, `shared/navigation.ts` |

## Next steps toward production

1. Point the Supabase tables at real data: a pipeline that fills `daily_channel_metrics` and `retention_cohorts`, then restrict reads to signed-in users.
2. Add sign-in, and decide whether Revenue and Retention need their own loading, empty and error states (an open question in the PRD).
3. Move the extra calculations into SQL if the daily table grows, rather than loading every row at startup.
4. Add the event tracking listed in the PRD's Data & events section.
5. Send confirmed briefs on to the agreed system, and make CSV export produce a file.
6. Remove `PrototypeBar`.
