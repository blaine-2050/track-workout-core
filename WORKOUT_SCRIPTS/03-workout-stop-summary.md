# 03 — Workout Stop Summary

## Goal
Verify Stop produces a correct workout summary: end time, total duration, total sets, total weight moved.

## Preconditions
- App open, no active workout.

## Steps

1. **Select "Deadlift".** Workout starts at T0.
2. **Log `315 × 3`.**
3. **Wait ≥ 30 seconds.**
4. **Log `315 × 3`.**
5. **Log `335 × 2`.**
6. **Tap Stop.** A summary screen or modal appears.

## Pass criteria
- Summary shows: `3 sets`, `total weight moved = 315×3 + 315×3 + 335×2 = 2560`, duration ≈ (now − T0).
- Summary formats duration human-readably (e.g. `1:15 min`, not `75`).
- Workout's `endedAt` is set and equals the Stop tap time (within 1s).
- History still shows all three entries grouped under this workout.
