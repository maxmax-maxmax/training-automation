# Training Automation — Context & Architecture

Athlete: Max (i210583) | Last updated: April 5, 2026

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

Weekly structure: Mon strength · Tue indoor hard · Wed easy run · Sat long ride · Sun back-to-back

Periodization: 3 build weeks → 1 recovery week → 1 final build/taper. All events loaded in intervals.icu calendar through May 5.

---

## Key Settings

| Parameter | Value |
|-----------|-------|
| Athlete ID | i210583 |
| LTHR | 176 bpm |
| FTP (set) | 250W ⚠️ outdated |
| FTP (estimated) | 265–275W (icu_eftp consistent at 275–276W since Feb) |
| HR Z2 | 119–145 bpm |
| HR Z3 | 145–165 bpm |
| HR Z4/Threshold | 165–184 bpm |
| Weight | 72 kg |

> **Action needed**: Update FTP in intervals.icu from 250W → 265W. All task prompts already reference 265W, but the platform setting itself hasn't been changed.

---

## Architecture Decisions

### intervals.icu MCP
- Server: `github.com/mvilanova/intervals-mcp-server` (community, Python)
- Installed locally on Mac — exact path TBD (not a pip install, `pip show intervals-mcp-server` returns nothing)
- **Known limitation**: `add_or_update_event` does not expose `category` or `color` fields from the intervals.icu API. Calendar notes land as "Workout" category instead of "Notes", and color can't be set.
- **Fix needed**: Add `category` and `color` parameters to `add_or_update_event` in the MCP source. To find the file:
  ```bash
  find ~ -name "server.py" 2>/dev/null | xargs grep -l "add_or_update_event" 2>/dev/null
  # or
  find ~ -name "*.py" 2>/dev/null | xargs grep -l "intervals.icu" 2>/dev/null
  ```
  Then add `category: Optional[str] = None` and `color: Optional[str] = None` to the function signature and payload.

### Workout Compatibility Rules (Wahoo)
Hard-won rules to avoid Wahoo sync errors:
- **Indoor rides only** get structured `workout_doc` steps with `%ftp` power targets
- **Outdoor rides, runs, strength** use description text only — no structured steps
- `%lthr` HR targets are rejected by Wahoo — power targets only
- Nested `reps` blocks are not supported by intervals.icu API — flatten all intervals into a single reps block

### Wellness Color System
Tasks use emoji circles in calendar note titles:
- 🟢 Wellness — all clear
- 🟠 Wellness — one flag (HRV or RHR out of range)
- 🔴 Wellness — multiple flags or TSB < –20

### HR Source Note
Polar H10 chest strap is the reliable HR source. Watch-based HR broadcast is acceptable for easy/outdoor rides but unreliable in cold conditions (< 5°C) due to peripheral vasoconstriction. Don't trust wrist HR for intensity analysis in cold.

---

## Current Fitness Snapshot (Apr 5, 2026)

| Metric | Value |
|--------|-------|
| CTL | 37.2 |
| ATL | 36.7 |
| TSB | +0.5 |
| eFTP | 275W |
| Peak CTL ever | 78.5 (Aug 2025) |
| Peak eFTP ever | 306W (Sep 2025) |

CTL needs to reach 55–60 by May 12 — requires ~3.5–4.0 CTL/week for 5 weeks. Currently behind due to missed weekday sessions (Mon strength, Tue indoor, Wed run are the most frequently skipped).

---

## Immediate Next Steps

1. **Update FTP to 265W** in intervals.icu settings (Profile → Sport Settings → Cycling FTP)
2. **Find and patch the MCP source** to add `category` and `color` support to `add_or_update_event` (see Architecture section above)
3. **Execute weekday sessions consistently** — Saturday long rides are landing, but Mon/Tue/Wed are the CTL gap. Missing these is why CTL is flat
4. **Run wellness task manually** to verify the 🟢/🟠/🔴 system and Note category are working after MCP patch
5. **Consider FTP test** — eFTP has been stable at 275–276W for 2+ months. A proper 20-min test would confirm and let you set it officially
