# Iteration: Analytics, Sprint, Final Recommendation

> Module 6 · Evals & Iteration. Read the analytics, run an iteration sprint, present with evidence.

## What the evidence says

_What real usage showed: numbers if your tool has analytics, counted behaviour if it does not. Put the signal that matters on screen._

- **Primary signal:** Bheavior, not analytics: most testers went straight to the "Start this test" call to action and spent little time on other views or filters.
- **What moved:** The recommended action now appears on all three pages, so Revenue and Retention are no longer dead ends. Every recommendation is computed from that page's own data: fees per referral on Revenue, repeat rates on Retention. LATAM gets a review ticket wherever the app trails the web. Everything is stored in Supabase, and the brief returns to the page it came from.
- **What didn't:** Bounce rate is still unmeasured, so half the hypothesis can't be confirmed. The new Revenue and Retention actions haven't been tested with users yet. Sign-up and email confirmation still sit in front of the first screen, which adds friction before the hypothesis is even tested.

_Observed behaviour: reach I had peers outside the class check my link as I was not present for the live session. 2 people clicked around on my live website.; core action Both individuals clicked every surface they could and were very thorough with it. All of them were click to go to the 'start this test' CTA and follow through with that action, but they weren't entirely sure what doing so would accomplish -- unclear outcome/goal.; stall point 'start this test' button -- it's the most prominent CTA on the page.; explained away biggest friction point was sign-up, having to explain that you need to create an account to access the dashboard, can use any emaik.; own first run pauses before reaching the dashboard, all before the core flow: (1) the site opened on Log in, and "Log in" failed with no account, so I had to spot "Create an account"; (2) I had to leave for my inbox to confirm my email; (3) I came back and logged in again. I haven't confirmed a brief on the live site yet (0 in the database), so the core action is still untested even by me.._

## Iteration sprint

| Change | Hypothesis | Result |
|---|---|---|
| Give Revenue and Retention their own one recommended action | If Revenue and Retention each lead with one recommended action, like Channel share, users will act on those pages instead of treating them as dead ends. | Shipped and live. Each page now has one action per region (8 new rules, North America the default), and the brief opens from and returns to any page. Not yet re-tested with peers, so there's no behaviour data on the new cards yet. |

## Peer feedback

Peer feedback is positive, but there's still the lingering question of: how much SHOULD be done based on this dashboard? Do we need 3 CTAs that are worded slightly differently? Does that not compete for interest?

## The recommendation

**Decision:** ☐ Go  ☑ Iterate  ☐ Kill

_The evidence that justifies the call:_

Peer testing: most people went straight to "Start this test" on the first screen, which supports half the hypothesis (the recommended action gets clicked). But they rarely explored filters or other views, and peers called Revenue and Retention dead ends with nothing to act on. This sprint fixed that gap. What's still unproven: bounce rate wasn't measured against the 60% baseline, and nobody has used the new Revenue and Retention actions yet.

## Final showcase

- **Demo link:** https://houselights-ticketing-dashboard.netlify.app/
- **The one-sentence story:** A ticketing analytics dashboard that leads every page with one number, one takeaway and one recommended action, so decision-makers act instead of bouncing or exporting to spreadsheets.
- **Where it landed on the Confidence Line (M2 → now):** Value risk partly closed (people act on the recommendation when they see it), still iterating on activation: sign-up sits in front of the first screen, and bounce isn't measured yet.
