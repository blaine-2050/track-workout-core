# 04 — Cardio Segment (tap-start / tap-stop, with optional note)

## Goal
Verify the **first-class cardio entry** path: tap to start a segment, tap to stop, optionally attach a note. Also verify the older direct-duration entry still works.

## Preconditions
- App open, no active workout.
- Seed moves include `Treadmill` (measurementType = duration).

## Path A — tap-start / tap-stop (the primary cardio flow)

1. **Select "Treadmill".** Workout starts. The entry UI shows a large **Start segment** button (no keypad required).
2. **Tap Start segment.** A live timer begins counting. Entry UI shows the elapsed time and a **Stop segment** button.
3. **Wait ~10 seconds** (representing real time on the machine).
4. **Tap Stop segment.** A pending entry appears with the captured duration (~10 sec).
5. **Tap the entry's note affordance** and type `wind in face`. Save.
6. **Confirm** the entry shows in history with: move = Treadmill, duration ≈ 10 sec, note = "wind in face".

## Path B — direct duration entry (still supported, useful when you know the duration)

1. **Select "Treadmill"** in a fresh workout.
2. **Tap "Enter duration"** (alternative entry mode toggle).
3. **Enter `00:25:00`** via the HH:MM:SS keypad.
4. **Tap Log.**
5. **Confirm** entry shows duration = 25:00 min.

## Pass criteria

- Path A produces a `LogEntry` with `measurementType = duration`, `durationSeconds` ≈ wall-clock elapsed, `notes = "wind in face"`, `startedAt` and `endedAt` populated.
- Path B produces a `LogEntry` with `measurementType = duration`, `durationSeconds = 1500`, no note, `startedAt` ≈ user's tap-start time and `endedAt` set to log time (or `endedAt = startedAt + durationSeconds`).
- UI clearly differentiates duration entries from strength entries in history (no `weight × reps` rendering for cardio).
- Switching between strength and cardio moves mid-workout does not crash and preserves both kinds of entries.
- Stop summary correctly reports a "duration entries" count and total cardio time alongside any strength counts.
