# London Marathon 2027 — Coaching Interview Notes

Living record of the coaching interview. Read this first in any new session.
Date started: 2026-10-06. Race: Sunday 2027-04-25, London Marathon. Goal: 2:45:00 (hard number, athlete's call, coach flagged as far beyond current fitness).

## Athlete
- Age 21, student in the Chicago Loop Mon–Thu. Runs in the Loop.
- Units: miles.
- Chicago Marathon 4:26:06 on 9 weeks of training. Cramped at mile 20. Suspects short build + fueling.
- Half marathon 2:14 (ran 13.1 two Sundays before 2026-10-06).
- 5K: under 25:00 recently. Pull exact time from Strava at build time. Gap between 5K and half says endurance is the limiter, not speed.
- Current volume ~20 mi/week, mostly club runs. Longest recent run 13.1.
- Competitive. Bored by Zone-2-only weeks.
- Metabolic assessment + VO2 test booked Monday 2026-10-12. Plan zones should come from that; dashboard must let athlete enter test numbers and recalc paces.

## Injury
- Left calf. Comes and goes. Quiet for a while now.
- Triggers: mileage jumps, running faster than fitness allows. No hills in Chicago.
- Plan rule: cap weekly jumps, cap pace hard. Dashboard warns when caps broken.

## Fueling
- Long runs: half-planned, gel sometimes. Make it fully planned from month one (gel every 30–45 min, electrolytes). Train gut early.

## Weekly schedule (5 run days, locked)
- Mon: long run. Free 12pm–7pm. Locked to Monday (already downtown). Time budget unlimited.
- Tue: club track workout, morning. Coach sets workout. Three groups: sub-20 5K, sub-24, slow. Optional easy PM shakeout.
- Wed: easy run.
- Thu: club 5K + 1 mile to get there (~4 mi). Social pace officially but group is fast, athlete runs it at high HR. Needs taming on non-test weeks.
- Fri: pickleball + night out (only night out of the week). No run.
- Sat: no run (athlete's choice, hungover day).
- Sun: medium run, fresh.

## Testing (no races)
- Thursday club 5K: all-out time trial every 4–6 weeks. Hold back other weeks.
- Monday long run: half-marathon time trial twice — mid-January and early March.

## Whoop
- Trusts all of it: recovery, sleep, HRV, strain.
- Friday nights out will show as red recovery Sat/Sun. Dashboard shows it, coach doesn't nag.

## Treadmill
- Only when Chicago weather is brutal (ice, wind, dark). Logged to Strava like any run.

## Dashboard decisions
- Runs on Mac, localhost only. Desktop screen. NOT viewed on phone.
- Refresh is manual: tap a button, it pulls Whoop + Strava.
- Top of page: progress to 2:45 shown as plan compliance (weeks done, miles hit vs planned, workouts nailed) plus a fitness curve with race-pace line. Then coach one-liner. Then the run. Recovery lower. Predicted finish time computed but placed low on the page.
- Coach voice in dashboard: normal coach. Few sentences, direct, no fluff. (Caveman stays in chat.)
- Look: athlete said "use impeccable and make it nice and cool". No fixed style chosen.

## Data access
- Whoop: developer API, OAuth. Credentials in `.env`. Never print, commit, or paste.
- Strava: no API. Browser downloader https://github.com/rafaeldrrmachado/strava-activity-downloader. Athlete logs in.
- Coach inside dashboard: local `claude` CLI on athlete's subscription.

## Process decisions
- Interview + plan happen in this cloud session. Build happens on the Mac (needs .env, claude CLI, browser preview).
- Old project in "CLAUD DO NOT DELETE" folder: start fresh, ignore it.
- Tools wanted on Mac: superpowers (brainstorm → writing-plans → subagent-driven-development), impeccable, ponytail, caveman (`npx skills add JuliusBrussee/caveman -g`, then `/ultracave`).
- Chat style: caveman. Short.

## Late answers (Q28–Q32)
- Long run habit: 6 mi at avg HR 162, 10:00–10:45/mi, "felt easy". Coach note: ~82% of likely max, that is moderate, not easy. Zones from Oct 12 test.
- Strength: none now. Will do calf raises 2x/week (after Wed and Sun runs, 10 min). Dashboard tracks.
- Sleep: bad now, working toward 7–8h. Dashboard shows sleep next to load; coach backs off when both bad.
- Plan vs body: dashboard coach decides each day from plan + Whoop. Athlete follows it.
- Plan B: none wanted. Build for 2:45. Checkpoints show truth; athlete decides at gates.

## Files in this folder
- `INTERVIEW.md` this file. `PLAN.md` the 29-week plan. `plan.json` same plan for the dashboard. `DASHBOARD.md` dashboard design and build steps.
