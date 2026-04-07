# Training Automation — Context & Architecture

Athlete: Max (i210583) | Last updated: April 6, 2026 (evening)

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

5-week progressive build targeting CTL 55–60 by May 12, followed by a taper into:
- **Bikepacking trip**: May 12–23 (900km, ~10k elevation)
- **Gravel race**: June 6 (140km, 2500m — Gravelooza)

Weekly structure: Mon strength · Tue indoor hard · Wed run + bikepacking strength · Thu indoor Z2 · Sat long ride · Sun back-to-back

Periodization: 3 build weeks → 1 recovery week → 1 final build/taper. All events loaded in intervals.icu calendar through May 10.

---

## Key Settings

| Parameter | Value |
|-----------|-------|
| Athlete ID | i210583 |
| LTHR | 176 bpm |
| FTP (set) | 265W (updated Apr 2026) |
| FTP (estimated) | 273–276W (icu_eftp) |
| HR Z2 | 119–145 bpm |
| HR Z3 | 145–165 bpm |
| HR Z4/Threshold | 165–184 bpm |
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
- **Outdoor rides**: structured steps with `%lthr` HR ranges + text labels ✅
- **Runs**: structured steps with `%lthr` HR ranges only — **no text labels** (Wahoo rejects the combination)
- **Strength/WeightTraining**: description text only — no structured steps
- Nested `reps` blocks inside other `reps` blocks are not supported — one level of reps only

### Wellness Color System
Tasks use emoji circles in calendar note titles:
- 🟢 Wellness — all clear
- 🟠 Wellness — one flag (HRV or RHR out of range)
- 🔴 Wellness — multiple flags or TSB < –20

### HR Source Note
Polar H10 chest strap is the reliable HR source. Watch-based HR broadcast is acceptable for easy/outdoor rides but unreliable in cold conditions (< 5°C) due to peripheral vasoconstriction. Don't trust wrist HR for intensity analysis in cold.

---

## Current Fitness Snapshot (Apr 6, 2026)

| Metric | Value |
|--------|-------|
| CTL | 36.4 |
| ATL | 32.6 |
| TSB | +3.8 |
| eFTP | 273W |
| Peak CTL ever | 78.5 (Aug 2025) |
| Peak eFTP ever | 306W (Sep 2025) |

CTL target is 55–60 by May 12. At current plan volume (~350 TSS/week if executed consistently), realistic peak is ~43–46. Hitting 55 would require ~490 TSS/week — not feasible with weather-dependent long rides. Focus is on consistent execution over volume chasing. Missing weekday sessions (Mon/Tue/Wed/Thu) is the main CTL drag.

---

## Completed Steps

- ✅ FTP updated to 265W in intervals.icu
- ✅ MCP source patched to add `category` and `color` to `add_or_update_event`
- ✅ Wellness 🟢/🟠/🔴 color system working
- ✅ Thursday indoor Z2 rides added to all build weeks
- ✅ All run workouts have HR interval structure (HR ranges only, no text labels — Wahoo compatible)
- ✅ All outdoor ride workouts have HR range steps with text labels
- ✅ Nutrition guidance added to all ride events (60g carbs/hour target)
- ✅ Wednesday bikepacking strength circuit added to all run days (Apr 8 → May 6)
  - Focus: knee stability (step-ups, clamshells) + upper body/handlebar endurance (push-ups, supermans, planks)
  - Progressive: incline push-ups Week 1 → floor push-ups Week 3

## Immediate Next Steps

1. **Execute weekday sessions consistently** — Saturday long rides are landing, but Mon/Tue/Wed/Thu are the CTL gap. Missing these is why CTL is flat
2. **Consider FTP test** — eFTP has been stable at 275–276W for 2+ months. A proper 20-min test would confirm and let you set it officially
3. **Verify Wahoo sync** after run workout updates — runs use HR range steps with no text labels (text+HR mix was rejected by Wahoo)
