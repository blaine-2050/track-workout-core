# 02 — Multi-Set Sticky Inputs

## Goal
Verify sticky-input behavior: after logging a set, weight/reps remain visible; the next keypress clears and replaces.

## Preconditions
- App open, no active workout.

## Steps

1. **Select "Squat".** Workout starts.
2. **Enter weight `225`, reps `5`.** Tap Log. Entry appears in history.
3. **Observe:** weight display still reads `225`, reps display still reads `5`.
4. **Tap the weight field, then tap `2`.** Weight display now reads `2` (not `2252`). Sticky input has been replaced on first keypress.
5. **Type `35`** (so display reads `235`). Reps still reads `5`.
6. **Tap Log.** Second entry persisted: `235 × 5`.
7. **Tap the reps field, tap `6`.** Reps display reads `6` (not `56`).
8. **Tap Log.** Third entry: `235 × 6` (weight stuck from previous, reps newly entered).

## Pass criteria
- Three `LogEntry` rows: `225×5`, `235×5`, `235×6`.
- Each tap-to-replace happens on the first keypress after a Log, not on field focus or other events.
- Sticky values are visually indistinguishable from freshly entered values (no ghost styling required).
