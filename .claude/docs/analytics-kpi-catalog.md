# Analytics KPI Catalog

> Game-agnostic reference for `analytics-engineer` and `/analytics-plan`.
> Read by section (`Grep "^## " -A 60`). Per-game events live in each game's
> own spec (default `docs/analytics/telemetry-spec.md`), never here.

Every KPI carries a **stage gate** so a game only tracks what it can act on:

| Gate | Turns on when | `project.yaml` signal |
|------|---------------|----------------------|
| `launch` | First build reaches players (closed test included) | always |
| `iaa` | Ads ship | `analytics.monetization` contains `iaa` |
| `iap` | In-app purchases ship | `analytics.monetization` contains `iap` |
| `ua` | Paid user acquisition starts (an MMP is integrated) | `analytics.paid_ua: true` |

## Users, retention, stability

| KPI | Formula | Source of truth | Gate |
|-----|---------|-----------------|------|
| NRU | Users with `first_open` in the period | Analytics backend (store installs are a different number) | launch |
| DAU / MAU | Daily / monthly engaged users | Analytics backend | launch |
| Stickiness | DAU ÷ MAU | Analytics backend / warehouse | launch |
| D1 / D7 / D30 retention | Share of an install-day cohort active on day N | Analytics backend / warehouse | launch |
| Cohort retention table | Install week × day-N matrix | Warehouse | launch |
| Sessions per DAU, session length | session starts ÷ DAU, engaged time ÷ sessions | Analytics backend | launch |
| Crash-free users | 1 − users with a crash ÷ active users | Crash reporter | launch |
| Store vitals (crash / ANR rate) | Store's own definition and bad-behaviour thresholds (check the store's current numbers) | Store console | launch |
| Platform / country split | Any KPI above split by platform and country | Analytics backend | launch |
| Store listing conversion | Store page visitors → installs | Store console | launch |

## In-game funnel (shape per game)

These are **patterns**. `/analytics-plan` turns each into concrete events for the
game's own loop. Every game should have at least the first three.

| Pattern | Question it answers | Typical events |
|---------|--------------------|----------------|
| Onboarding funnel | Do new players start and finish the first session / run / level? | `first_open` → first content start → first content end → second start |
| Difficulty curve | Where do players fail or quit? | content end with outcome + time/progress at failure, mid-content checkpoints |
| Choice balance | Which options (upgrades, units, cards, routes) get picked, and do they win? | offer + pick in one event, per-content pick counts on the end event |
| "One more" loop | Do players start again after finishing? | retry / next tapped ÷ content end |
| Interruption | When do players leave mid-content? | app backgrounded with current state |
| Performance by device | Does low-end hardware hurt play? | per-content perf summary × device tier |

## Revenue, ads, P&L

| KPI | Formula | Source of truth | Gate |
|-----|---------|-----------------|------|
| IAP gross by store | Gross sales per store | Store reports (Google Play reports, App Store Connect, web shop API) | iap |
| Store fee, fee rate | Gross − settled; fee ÷ gross | Store settlement reports | iap |
| IAA revenue by platform / network | Ad revenue net of network share | Ad network / mediation reports (cross-check with MMP once `ua`) | iaa |
| Gross revenue | IAP gross + IAA | Sum of the above | iap or iaa |
| **Net revenue** | IAP settled (after store fee) + IAA | Store settlement + ad network | iap or iaa |
| PU, PU rate | Paying users; PU ÷ DAU (state the denominator) | Analytics backend / warehouse | iap |
| ARPPU | IAP revenue ÷ PU | Warehouse + store | iap |
| ARPDAU (IAP / total) | IAP ÷ DAU; gross ÷ DAU | Warehouse | iap or iaa |
| Ad watch rate | Users with an ad impression ÷ DAU | Analytics backend | iaa |
| Avg ads watched | Impressions ÷ ad-watching users (or ÷ DAU; state which) | Analytics backend | iaa |
| eCPM | IAA ÷ impressions × 1000 | Ad network | iaa |
| Placement metrics | Offered → started → completed per placement, eCPM and fill rate per placement | Analytics backend + ad network | iaa |
| User type split | e.g. normal vs no-ads purchasers, by DAU | Analytics backend | iap |
| UA spend | Spend per channel | MMP API / channel consoles | ua |
| **Marketing profit** | Net revenue − UA spend | Sum | ua |
| Attribution cost | MMP contract fee | MMP invoice | ua |
| Server cost | Backend + warehouse + infra bills | Cloud billing | launch |
| Operating profit | Marketing profit − attribution − server | Sum | ua |
| CPI | UA spend ÷ paid installs | MMP | ua |
| **Cohort ROAS D7 / D30 / D90** | Install cohort's cumulative net revenue by day N ÷ that cohort's spend | MMP (+ warehouse revenue) | ua |
| LTV D7 / D30 / D90 | Cohort cumulative net revenue ÷ cohort installs | Warehouse / MMP | iap or iaa |
| Payback days | Days until cumulative ROAS reaches 100% | MMP | ua |
| Country ARPDAU / LTV / CPI | Any of the above by country | Warehouse / MMP | iap, iaa or ua |

**Why cohort ROAS, not "revenue − spend":** a month's revenue comes mostly from
users acquired in earlier months, and a month's spend buys future revenue. Daily
or monthly net-minus-spend says whether the business is cash-positive, not whether
this month's spend paid back. Always pair the P&L view with cohort ROAS.

## Data sources and time zones

| Number | Authoritative source | Time zone | Note |
|--------|---------------------|-----------|------|
| Google Play IAP | Play reports (Cloud Storage): estimated sales before settlement, earnings report once settled | Settled earnings are dated in US Pacific time | Daily values drift from other sources; close months on the earnings report only |
| Apple IAP | App Store Connect API (sales → settlement) | Apple reporting time | |
| Web shop (e.g. Xsolla) | Vendor API | Usually UTC | Check store payment policy before adding one |
| IAA | Ad network / mediation | Set the report time zone to the project's (recommended UTC) | Same-day values are estimates, final a few days later |
| Installs | Store console (store installs), MMP (attributed installs), analytics backend (`first_open`) | Each differs | Three different numbers. Never mix them in one chart |
| UA spend | MMP API | UTC | |
| DAU, retention, in-game | Analytics backend / warehouse | The analytics property's time zone (set it to the project's) | Warehouse event dates follow the property time zone |

Rules: one project time zone (`analytics.timezone`, default UTC). Daily finance
numbers are labelled estimates; only monthly settled numbers are final. Client-sent
purchase events are for funnels and per-user analysis, never the revenue figure.

## Store fees (verify before relying on them)

- **Google Play:** 15% on the first US$1M of earnings each calendar year, 30% above
  it; the 15% tier requires **enrolling** in Play Console. Subscriptions are 15%.
- **Apple:** 30% standard; 15% through the Small Business Program (enrolment) and
  for subscriptions after year one.
- Effective rates in reports differ from headline rates because of tax, refunds
  and currency, so compute fees from settlement reports instead of assuming them.

## Backend naming constraints

The studio convention is `category.action.detail`. Map it per backend:

| `analytics.backend` | Event name rule | Map the convention as | Prefer reserved events |
|---------------------|-----------------|-----------------------|------------------------|
| `firebase` | ≤40 chars, letters/digits/`_`, starts with a letter; no `firebase_`, `google_`, `ga_` prefixes; ≤25 params/event, string values ≤100 chars; ≤25 user properties (name ≤24, value ≤36); ~500 event types | `.` → `_` (`game_run_checkpoint`) | `level_start`, `level_end`, `level_up`, `tutorial_begin`, `tutorial_complete`, `ad_impression`, `purchase`, `earn_virtual_currency`, `spend_virtual_currency` |
| `gameanalytics` | Design events are colon-separated, up to 5 parts | `.` → `:` | Progression, Resource, Business, Ad event types |
| `unity` | Custom events registered in the dashboard schema | keep snake_case | Standard events where they exist |
| `custom` | Whatever the pipeline defines | state the rule in the spec | none |

Check the backend's current docs before trusting a limit; they change.

## Privacy and store compliance checklist

- [ ] Store data-safety form lists what the analytics, crash and ad SDKs collect
- [ ] Privacy policy URL exists and names the SDKs
- [ ] No PII: no user-entered text, names, emails or device names in events
- [ ] Advertising ID collection off until ads ship; Android 13+ `AD_ID` permission declared when it is on
- [ ] Consent defaults set if shipping to EEA / UK; certified CMP once ads ship
- [ ] Analytics opt-out reachable in the game
- [ ] Target age decided first: including children under 13 restricts SDKs, identifiers and ads (Google Play Families, COPPA)
- [ ] Data retention period chosen in the backend; warehouse retention policy written
- [ ] Local privacy law notices (e.g. Korea PIPA, GDPR) reflected in the policy

## Pre-launch store accounts

- [ ] Package / bundle ID final before registering the analytics app (analytics apps bind to it)
- [ ] Google Play: new personal developer accounts must run a closed test with enough opted-in testers for a continuous period before production access (check the current tester count and days in Play Console)
- [ ] Google Play 15% fee tier enrolled
- [ ] Analytics property and ad-network report time zones set to `analytics.timezone`
