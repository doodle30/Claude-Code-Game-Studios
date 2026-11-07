---
name: analytics-plan
description: "Telemetry event spec + KPI definitions for this game — design questions, events with code hooks, stage-gated KPIs, first dashboard, privacy. Run before installing an analytics SDK."
argument-hint: "[--update] [backend: firebase|gameanalytics|unity|custom]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Agent, AskUserQuestion, Bash(bash "*/.claude/skills/analytics-plan/../../hooks/yaml-helper.sh" resolve_config *)
model: sonnet
---

!`bash "${CLAUDE_SKILL_DIR}/../../hooks/yaml-helper.sh" resolve_config --keys automation,rigor,stage`



# Analytics Plan

Produces one game's telemetry spec from the studio's game-agnostic KPI catalog.
The catalog says **what every game can measure**; this skill decides **what this
game measures, which events carry it, and where in the code they fire**.

**Output:** `analytics.spec` from `project.yaml`, default `docs/analytics/telemetry-spec.md`
**Template:** `.claude/docs/templates/telemetry-spec.md`
**Reference:** `.claude/docs/analytics-kpi-catalog.md` (read by section)
**When to run:** after the core loop is playable and before the analytics SDK is
installed. Re-run with `--update` when monetization or paid UA turns on, or when
the core loop changes.

Every `AskUserQuestion` call follows `.claude/docs/automation-modes.md`.

## 1. Read the config

Read the `analytics:` block of `project.yaml` (Grep `^analytics:` -A 12). Keys:

| Key | Values | Default when absent |
|-----|--------|---------------------|
| `analytics.backend` | `firebase`, `gameanalytics`, `unity`, `custom`, `none` | ask; recommend `firebase` for mobile |
| `analytics.monetization` | list of `iap`, `iaa`; `[]` = none planned | ask |
| `analytics.paid_ua` | `true` / `false` | ask |
| `analytics.timezone` | IANA name or `UTC` | `UTC` |
| `analytics.spec` | path | `docs/analytics/telemetry-spec.md` |

A backend named in the argument overrides the config for this run. Nothing but
`/analytics-plan` writes this block, so an absent gate key is the common case and
must not silently drop KPI rows: ask for every absent key of `backend`,
`monetization` and `paid_ua` in one question, then offer to write the block (show
it first). If the user skips the question, treat each unanswered gate as **on**
(over-listing KPIs costs a few rows; under-listing ships a game without its
revenue metrics) and say so in the spec's header.

## 2. Gather the game

Read only what the spec needs:

1. The design record: `design/game-brief.md` (or `design/gdd/` sections on the
   core loop, progression and win/lose conditions).
2. The epic and stories for the playable loop (`production/epics/*/EPIC.md`).
3. The code root for the engine in `project.yaml` (see
   `.claude/docs/directory-structure.md`): find the hooks for content start,
   content end with outcome, choice offered / picked, retry, app pause or
   background, and the IDs that name content and choices. Grep first, then read
   the matching methods.

If `--update` and the spec exists, read it and change only what the new gate or
loop change touches.

## 3. Delegate the draft

Spawn `analytics-engineer`. **Objective:** a complete draft of this game's
telemetry spec that a programmer can instrument without asking questions.
**Output:** the filled template as Markdown in the final report, followed by an
assumptions list and an open-questions list. Pass it: the config values, the paths read in §2, the
catalog sections it needs (`Backend naming constraints`, the gate-relevant KPI
sections, `Data sources and time zones`, `Privacy and store compliance checklist`),
and the template path. Instruct it to:

- fill every template section in the user's language (identifiers stay English);
- name the exact class/method (or node/function) for every event, and list code
  changes the hooks need instead of assuming they exist;
- copy only KPI rows whose gate is on, listing the rest under "Later";
- map the studio naming convention for the backend and prefer reserved events;
- return the draft as text and **not write files**;
- list assumptions and open questions.

## 4. Review and present

Check the draft before showing it. Each check reports `PASS`, `FAIL` or
`NOT ASSESSED` with the reason:

| Check | PASS when | NOT ASSESSED when |
|-------|-----------|-------------------|
| Hooks exist | every Phase 1 hook name is found by Grep in the code root | no code root yet, or a hook is marked **new** (to be written); list each by name |
| Backend limits | every event / param name passes the catalog row for the backend | backend is `custom` or `none`, or has no catalog row |
| No PII | no param carries user text, names, emails, device names | — |
| Dashboard actionable | each chart names the config or code it would change | — |

Fix each `FAIL` with a follow-up to the same agent. Then show a summary: question
count, events with hooks, KPI rows by gate (and which gates are off), the check
table, open questions (top three first), and code issues found. Write the check
table into the spec's header too, so a later reader sees what was not verified.

## 5. Write

Ask "May I write this to [analytics.spec]?" On yes, write it. Then offer, in order:

1. hand §4 to the engine's code specialist (`specialists.code`) as the
   integration spec;
2. answer the open questions;
3. before release, run `/launch-checklist`: its Analytics section checks the
   spec, time zones and the catalog's privacy and store items.

Recommended next step: `/analytics-plan --update` when `analytics.monetization`
or `analytics.paid_ua` changes.

Do not install SDKs or edit game code in this skill.
