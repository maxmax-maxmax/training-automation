# Training Automation — Context & Architecture

Athlete: Max (i210583) | Last updated: April 16, 2026

---

## What's Been Built

A fully automated training monitoring system on top of intervals.icu, replacing JOIN Cycling's coaching layer with three scheduled tasks that run autonomously.

### Scheduled Tasks

| Task | Schedule | Output |
|------|----------|--------|
| `wellness-monitoring` | 7:00 AM daily | Calendar note with HRV/RHR/TSB snapshot |
| `post-workout-analysis` | 11:00 PM daily | Comment on most recent activity |
| `weekly-training-review` | 7:00 PM Sundays | Calendar note with week recap + next week tweaks |

### Training Plan

5-week progressive build targeting CTL 55–60 by May 12 (realistic ceiling ~43–46), followed by a taper into:
- **Bikepacking trip**: May 12–23 (900km, ~10k elevation)
- **Gravel race**: June 6 (140km, 2500m — Gravelooza)

Weekly structure: Mon strength · Tue indoor hard · Wed run + bikepacking strength · Thu indoor Z2 · Sat long ride · Sun back-to-back

Periodization: 3 build weeks → 1 recovery week (Apr 27–May 3) → 1 taper week (May 4–11). All events loaded in intervals.icu calendar through May 10.

Recovery week is intentional — bikepacking trip is 12 days/900km and requires arriving fresh. The taper week should be genuinely low volume to ensure positive TSB at trip start.

---

## Key Settings

| Parameter | Value |
|-----------|-------|
| Athlete ID | i210583 |
| Max HR | ~201 bpm (confirmed Apr 16) |
| LTHR | 176 bpm |
| FTP (set) | 265W (updated Apr 2026) |
| FTP (estimated) | 273–276W (icu_eftp) |
| Cycling HR Z2 | 119–145 bpm (59–72% max HR) |
| Cycling HR Z3 | 145–165 bpm (72–82% max HR) |
| Cycling HR Z4/Threshold | 165–184 bpm (82–92% max HR) |
| Running HR Z1 | 0–148 bpm |
| Running HR Z2 | 149–157 bpm (74–78% max HR) |
| Running HR Z3 Tempo | 158–166 bpm (79–83% max HR) |
| Running HR Z4 SubThreshold | 167–175 bpm (83–87% max HR) |
| Running HR Z5 Threshold | 176–180 bpm (88–90% max HR) |
| Weight | 72 kg |

---

## Architecture Decisions

### intervals.icu MCP
- Server: `github.com/mvilanova/intervals-mcp-server` (community, Python)
- Installed locally on Mac — exact path TBD (not a pip install, `pip show intervals-mcp-server` returns nothing)
- **Patch applied**: `add_or_update_event` now exposes `category` and `color` fields. Source file patched at `~/intervals-mcp-server/src/intervals_mcp_server/tools/events.py`. Backup at `workout/events.py` in this repo.
  - `category` controls event type (use `"NOTE"` for calendar notes, `"WORKOUT"` for workouts)
  - `color` sets the dot color on the calendar (e.g. `"green"`, `"orange"`, `"red"`)
  - Wellness task uses `workout_type: "Note"` to set the intervals.icu Notes category correctly

### Workout Compatibility Rules (Wahoo)
Hard-won rules to avoid Wahoo sync errors:
- **Indoor rides** (`VirtualRide`): structured steps with `%ftp` power targets + text labels ✅
- **Outdoor rides**: plain text description using `- Xm XX-XX% HR (label)` format ✅
- **Runs**: plain text description using `- Xm XX-XX% HR (label)` format ✅ — no structured JSON steps
- **Strength/WeightTraining**: description text only — no structured steps
- Nested `reps` blocks inside other `reps` blocks are not supported — one level of reps only

### HR Target Format — Critical
**Never use `%lthr` in workout steps.** It is not recognized by intervals.icu and silently defaults to Z2 for all targets.

Correct format — pass as `{"description": "..."}` with no `steps` array:
```
Warmup
- 15m 60-70% HR (easy spin build gradually)

- 25m 72-79% HR (upper Z2 cruise)

3x
- 10m 83-88% HR (tempo surge)
- 25m 72-79% HR (recover in Z2)

Cooldown
- 20m 60-70% HR (easy spin home)
```

%HR reference (max HR ~201 bpm):
- Easy warmup/cooldown: 60–70% HR (~121–141 bpm)
- Z2 cruise (long rides): 72–79% HR (~145–159 bpm)
- Tempo/Z3: 83–88% HR (~167–177 bpm)
- Threshold/Z4: 88–93% HR (~177–187 bpm)

### Wellness Color System
Tasks use emoji circles in calendar note titles:
- 🟢 Wellness — all clear
- 🟠 Wellness — one flag (HRV or RHR out of range)
- 🔴 Wellness — multiple flags or TSB < –20

### HR Source Note
Polar H10 chest strap is the reliable HR source. Watch-based HR broadcast is acceptable for easy/outdoor rides but unreliable in cold conditions (< 5°C) due to peripheral vasoconstriction. Don't trust wrist HR for intensity analysis in cold.

---

## Current Fitness Snapshot (Apr 16, 2026)

| Metric | Value |
|--------|-------|
| CTL | ~40.4 |
| ATL | ~50.3 |
| TSB | ~–9.9 |
| HRV | 49 (low) |
| eFTP | 273W |
| Peak CTL ever | 78.5 (Aug 2025) |
| Peak eFTP ever | 306W (Sep 2025) |

CTL ceiling with current plan is ~43–46 by May 12 — not 55–60. That's fine. The bikepacking trip is the A event; arriving fresh matters more than hitting a number. HRV at 49 confirms fatigue is accumulating — recovery week is warranted.

---

## Completed Steps

- ✅ FTP updated to 265W in intervals.icu
- ✅ MCP source patched to add `category` and `color` to `add_or_update_event`
- ✅ Wellness 🟢/🟠/🔴 color system working
- ✅ Thursday indoor Z2 rides added to all build weeks
- ✅ Wednesday bikepacking strength circuit added to all run days (Apr 8 → May 6)
  - Focus: knee stability (step-ups, clamshells) + upper body/handlebar endurance (push-ups, supermans, planks)
  - Progressive: incline push-ups Week 1 → floor push-ups Week 3
- ✅ Nutrition guidance added to all ride events (60g carbs/hour target)
- ✅ All workout HR targets migrated from `%LTHR` to `%HR` plain text format (Apr 16)
- ✅ Saturday Apr 18 long ride corrected — ~2h45m, Z2 cruise at 72–79% HR
- ✅ May 9 Pre-Trip Ride updated to %HR format (65–72% HR main block)
- ✅ Recovery week Thu Apr 30 strength session added (event 104815174)

## Immediate Next Steps

1. **Execute weekday sessions consistently** — Saturday long rides are landing, but Mon/Tue/Wed/Thu are the CTL gap
2. **Consider FTP test** — eFTP has been stable at 275–276W for 2+ months
3. **Keep taper week (May 4–11) genuinely easy** — low volume, not just lighter, to arrive at May 12 with positive TSB
