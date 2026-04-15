# 04 — Interval Cardio

## Goal
Verify interval (duration-based) entry for a cardio move.

## Preconditions
- App open, no active workout.
- Seed moves include `Treadmill` (measurementType = interval).

## Steps

1. **Select "Treadmill".** Workout starts. The entry UI switches to HH:MM:SS (no weight/reps keypad).
2. **Enter duration `00:25:00`** (25 minutes).
3. **Tap Log.** Entry appears in history with formatted duration (e.g. `25:00 min`).
4. **Tap Stop.** Summary shows 1 interval, total duration = 25 min.

## Pass criteria
- `LogEntry` persisted with `measurementType=interval`, `durationSeconds=1500`, no `weight`/`reps`.
- UI cleanly differentiates interval entries from strength entries in history.
- No crash when switching between strength and interval moves mid-workout.
