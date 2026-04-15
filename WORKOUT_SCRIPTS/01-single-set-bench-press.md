# 01 — Single Set Bench Press

## Goal
Log one set of bench press from a clean state. Verifies the shortest happy path.

## Preconditions
- Fresh install, or no active workout in progress.
- Seed moves present.
- Default weight unit: lbs (or whatever the platform defaults to).

## Steps

1. **Open the app.** The main screen is visible. No active workout. Move selector is empty or shows a placeholder.
2. **Select "Bench Press"** from the move selector. The workout auto-starts. A workout status bar shows elapsed = 0s (or similar).
3. **Tap keypad: `1`, `3`, `5`.** The weight display reads `135`.
4. **Switch to reps entry** (however the platform does it — tab, button, or next-field).
5. **Tap keypad: `1`, `0`.** The reps display reads `10`.
6. **Tap Log.** An entry appears in the history. Weight `135`, reps `10`, unit `lbs`. The set timer resets.
7. **Tap Stop.** A workout summary appears: 1 set, 135 × 10 = 1350 total weight moved, duration ≈ the time since move selection.

## Pass criteria
- One `LogEntry` persisted with `weight=135`, `reps=10`, `weightUnit=lbs`, `moveId=<bench-press>`.
- One `Workout` persisted with `endedAt` set, containing that entry.
- No crashes. No console errors. UI never blocks > 100ms on log.
