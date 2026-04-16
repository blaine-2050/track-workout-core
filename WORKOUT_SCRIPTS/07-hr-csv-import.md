# 07 — HR CSV Import and Alignment

## Goal
Import a second-by-second heart-rate CSV (Polar/Garmin/COROS export), align the samples to a logged workout by time window, and render HR summary on the workout in history.

## Preconditions
- App open.
- A prior workout exists in history with a known `startedAt` / `endedAt` window covered by the CSV.
- The CSV follows the schema in [`DATA_MODEL.md § CSV Import Schema`](../DATA_MODEL.md). Minimum columns: `timestamp`, `heart_rate_bpm`. Optional: `elevation_m`, `speed_kmh`, `distance_km`.

## Steps

1. **Open Settings** → tap **Import HR data**. A file picker appears.
2. **Select the CSV file.** A preview sheet shows: filename, sample count, time range, and which optional columns were detected.
3. **Confirm import.** The app parses, creates a fresh `importBatchId`, writes `HeartRateSample` rows, and runs alignment: each sample is tagged with the Workout whose `[startedAt, endedAt]` window contains its `timestamp`.
4. **Return to history.** The aligned Workout now shows a small HR badge: avg BPM, max BPM, sample count.
5. **(Optional) Undo import** — from Settings, tap the batch in the import history to delete its samples. Other batches are unaffected.

## Pass criteria

- Import of a 600-sample (10-minute) CSV completes in under 2 seconds on test hardware.
- Aligned samples have `workoutId` set to the matching Workout.
- Samples whose `timestamp` falls outside any Workout window keep `workoutId = NULL` but are still persisted and become alignable if a matching Workout is later created.
- Malformed rows (missing required column, non-numeric `heart_rate_bpm`) are skipped with a visible count; the import does not fail entirely.
- Sentinel `heart_rate_bpm = 0` is treated as NULL (not a real 0 bpm reading).
- Re-importing the same file creates a new batch (no dedupe) — the user undoes via batch-delete if they want to avoid duplicates.
- After import, per-entry timing in history is unaffected — structured entries still show weight/reps/duration/notes as before.

## Not in scope for this script

- Live BLE streaming of HR samples during a workout (separate path).
- FIT-file import (binary; deferred per `DECISIONS.md`).
- HR-zone analytics or effort scoring.
- Sync of HR samples to the server (HR data stays local in v2.1).
