# [Game Name] — Telemetry Spec + KPI Definitions (v[0.1])

> Status: [Draft / Approved] · Platforms: [Android, iOS, ...] · Backend: [analytics.backend] ([installed / not installed])
> Data informs, designers decide. Every threshold below is a **hypothesis** until `game-designer` confirms it.
> KPI formulas and sources come from `.claude/docs/analytics-kpi-catalog.md`; this file lists only what this game uses.

## 1. Design questions

| # | Question | Answered by |
|---|----------|-------------|
| Q1 | [e.g. What share of runs reach the win condition? Does it change with experience?] | [event.param × property] |

At least: onboarding, difficulty curve, choice balance, "one more", stability.

## 2. Naming + backend constraints

- Convention mapping for `[analytics.backend]`: [e.g. `.` → `_`]
- Reserved events used as-is (marked ★): [list]
- Stable IDs: [which enum / data IDs identify content; never display strings]
- Run / session key: [how one play unit is identified]

## 3. User properties

Auto-collected by the backend (do not duplicate): [list]

| Name | Values | Set when | Purpose |
|------|--------|----------|---------|
| `balance_ver` | [e.g. "b0.3"] | App start | Compare before/after tuning changes |
| `device_tier` | low / mid / high | App start | Split performance and difficulty by hardware |

## 4. Phase 1 events (implementable now)

| # | Event | Fires when | Params (type: example) | Hook (class / method or node / function) | Questions |
|---|-------|-----------|------------------------|------------------------------------------|-----------|
| 1 | [★level_start] | [...] | [...] | [...] | [Q1] |

**Implementation notes (for the programmer)**
- Analytics goes behind an interface injected into gameplay code (DI, per coding standards); a log-only implementation runs until the SDK is installed.
- [Code changes the hooks need: missing callbacks, values not exposed yet, off-by-one traps]
- Verification: [backend debug view command]

## 5. Phase 2 reserved events

| Area | Event | Key params | Gate |
|------|-------|-----------|------|
| Tutorial | [★tutorial_begin / ★tutorial_complete] | [...] | launch |
| Ads | [★ad_impression, rewarded offered/started/completed/failed] | placement, ... | iaa |
| Purchases | [★purchase, store opened, purchase failed] | product_id, ... | iap |

## 6. KPIs this game tracks

Copy rows from the catalog whose gate is on for this game (`analytics.monetization`, `analytics.paid_ua`); list gated-off rows by name only under "Later".

### 6-A. Users, retention, in-game, stability
| KPI | Formula (this game's events) | Source | Gate |
|-----|-----------------------------|--------|------|

### 6-B. Revenue, ads, P&L
| KPI | Formula | Source | Gate |
|-----|---------|--------|------|

**Later (gate off):** [names]

### 6-C. Data sources and time zone
- Project time zone: [analytics.timezone]
- [Any game-specific deviation from the catalog's source table]

## 7. First dashboard (launch)

| # | Chart | Data | Decision it drives (threshold = hypothesis) |
|---|-------|------|---------------------------------------------|
| 1 | Onboarding funnel | [...] | [...] |

5–7 charts. Each names the config or code it would change.

## 8. Privacy and compliance

Run the catalog's privacy checklist; note only this game's answers and risks here (target age, countries, ad ID, opt-out location).

## 9. Open questions

1. [Decision only the user can make]

**Assumptions:** [everything inferred rather than read]
