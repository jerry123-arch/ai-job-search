# Marathon Dashboard — Design and Build Plan

Runs on the Mac. `localhost:8000`. Desktop only. Manual refresh. Coach inside.

## Stack (ponytail: minimal, boring)

- **Python 3.11+**, standard library plus `requests`. No framework. `http.server` serves one page and three JSON routes.
- **SQLite**, one file `data.db`. Three tables: `runs`, `whoop`, `coach_notes`. Plus a `settings` key/value table.
- **One HTML file**, vanilla JS, one small chart library (uPlot, ~40 KB, vendored). No build step, no npm, no React.
- **`plan.json`** is the source of truth for the plan. Read at startup. Never duplicated into the DB.
- **Coach** = `claude -p` subprocess on your subscription. Prompt in one text file, `coach_prompt.md`, so you can edit the voice without touching code.

Why this stack: you asked for local, minimal, reusable. Every piece is something a future session can read in one sitting.

## Files

```
marathon-dashboard/
  app.py            server + routes + refresh (~250 lines)
  whoop.py          OAuth + pull (~120 lines)
  strava_import.py  reads downloader output folder → runs table (~80 lines)
  coach.py          builds prompt, calls `claude -p`, stores note (~60 lines)
  plan.py           week lookup, compliance, cMP ratchet, zone calc (~120 lines)
  coach_prompt.md   the coach's standing instructions
  plan.json         copied from this repo
  static/index.html one page, inline CSS + JS
  static/uplot.min.js, uplot.min.css
  .env              WHOOP_CLIENT_ID, WHOOP_CLIENT_SECRET, WHOOP_REDIRECT_URI, STRAVA_EXPORT_DIR
  data.db           created on first run, gitignored
```

## Data flow: the Refresh button

1. **Whoop.** `GET /v1/recovery`, `/v1/cycle`, `/v1/activity/sleep` since last pull. Token refresh handled, tokens stored in `settings`, never logged. First run opens the OAuth URL in the browser once.
2. **Strava.** The browser downloader (rafaeldrrmachado/strava-activity-downloader) writes files to a folder you choose. `strava_import.py` scans that folder for files newer than the last import, parses them (format confirmed at build: CSV first, GPX/FIT fallback), writes one row per run: date, distance, duration, avg HR, max HR, avg pace, splits if present, treadmill flag.
3. **Coach.** For every new run, build a prompt: plan week + planned session + the run + today's Whoop + last 7 days of runs + last 3 coach notes + open flags. Call `claude -p --output-format json`. Store the note. Note has a fixed shape: `verdict` (one line), `body` (2–4 sentences), `tomorrow` (one line), `flags` (list).
4. Page reloads. Done.

**Open risk, checked at build:** how long the Strava downloader's login lasts and what it emits. If the downloader needs a click per refresh, the Refresh button opens it in a tab and you click. Still one tap plus one click.

## Coach logic (deterministic part, in `plan.py`, so Claude isn't guessing)

Computed before the prompt, passed in as facts:
- Zone of the run vs the planned zone (easy ceiling / cMP / threshold / TT).
- Flags: easy run over HR ceiling; non-TT Thursday over 160; weekly miles over plan by >10%; long run over 40% of week; two hard days back to back; recovery red + sleep < 6 h + HRV below 30-day baseline.
- Compliance: planned vs done this week and cumulative. Workouts nailed = run happened, within ±15% distance, in the right zone.
- cMP ratchet: after a TT, cMP = predicted marathon pace from that TT (Riegel, exponent 1.06). Only moves on a TT.
- Prediction: Riegel from the best TT in the last 8 weeks, adjusted by long-run completion. Stored, shown low on the page.
- Gate status: for weeks 15 and 23, compare TT to target, write the honest sentence.

Claude gets these facts and writes the words. It does not compute them.

## Page layout (desktop, one screen, scroll for detail)

```
┌──────────────────────────────────────────────────────────────────────┐
│ LONDON 2027 · 2:45:00              Week 12 of 29 · Build 1  [Refresh]│
├───────────────────────────┬──────────────────────────────────────────┤
│ COMPLIANCE                │ FITNESS                                  │
│ this week  23 / 30 mi     │ line: predicted marathon pace per week   │
│ workouts   3 / 5 nailed   │ dashed line: 6:17 goal                   │
│ season     312 / 1,190 mi │ dots: time trials                        │
│ long runs  11 / 11 done   │ shaded: down weeks, taper                │
├───────────────────────────┴──────────────────────────────────────────┤
│ COACH · Wed 23 Dec                                                   │
│ Verdict line.                                                        │
│ Two to four sentences. Direct. No fluff.                             │
│ Tomorrow: Thu club, easy ceiling, HR under 150.          flags: ⚑ ⚑ │
├───────────────────────────┬──────────────────────────────────────────┤
│ LATEST RUN                │ RECOVERY                                 │
│ 5.0 mi · 10:52 /mi        │ recovery 67%  sleep 6h40  HRV 58  strain│
│ avg HR 148 · zone Easy ✓  │ 7-day sparklines under each             │
│ planned: 5 mi easy        │ red when below your 30-day baseline      │
├───────────────────────────┴──────────────────────────────────────────┤
│ THIS WEEK   Mon 12 ✓  Tue 5 ✓  Wed 5 ✓  Thu TT 5K  Fri —  Sat —  Sun 4│
├──────────────────────────────────────────────────────────────────────┤
│ prediction 3:48 (from 5K TT 22:10, 23 Dec)  ·  gate 1 in 3 weeks     │
│ calf raises: Wed ☑ Sun ☐   ·   settings: test numbers, zones, cMP    │
└──────────────────────────────────────────────────────────────────────┘
```

Look and feel decided on the Mac with impeccable: design context → direction → audit → polish. Not decided here. Only constraints: desktop width, readable from a chair, red means red.

## Build steps (subagent-driven on the Mac, each with a review)

1. **Scaffold.** `app.py` serving `index.html` and `GET /api/state` returning plan + empty data. Open it. Done when the page loads with the plan's week number correct.
2. **Strava import.** Install the downloader, log in, export last 90 days. Write the parser against the real files. Pull your real 5K time. Done when `runs` has every run since July.
3. **Whoop.** OAuth once, pull 30 days. Done when `whoop` has recovery, sleep, HRV, strain per day and tokens refresh without you.
4. **Plan logic.** `plan.py`: zones, flags, compliance, cMP ratchet, prediction, gates. Unit tests against hand-worked cases. Done when tests pass.
5. **Coach.** `coach_prompt.md` + `coach.py`. Run it on the last 5 real runs, read the notes, tune the prompt until they sound like a coach and not a bot. Done when you'd act on them.
6. **Page.** Layout above, uPlot chart, real data. Then impeccable passes. Done when it looks good at your screen width and you say so.
7. **Refresh wiring.** Button → Whoop → Strava → coach → reload. Done when a fresh run shows up with a note in one tap.
8. **Settings.** Enter 12 Oct test numbers, zones recalc, cMP editable. Done when placeholder paces are gone.
9. **Review.** Code review for correctness, then simplify pass. Delete anything the page doesn't use.

## What to carry to the Mac

- This folder: `INTERVIEW.md`, `PLAN.md`, `plan.json`, `DASHBOARD.md`.
- Install on Mac first: superpowers, impeccable, ponytail, caveman (`npx skills add JuliusBrussee/caveman -g`).
- Open Claude Code in a new `marathon-dashboard/` folder, paste: "Read DASHBOARD.md and PLAN.md. Build step 1."
