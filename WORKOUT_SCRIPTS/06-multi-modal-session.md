# 06 — Multi-Modal Session (commute + cardio + strength + free-form)

## Goal
Verify a **realistic full-day workout** that mixes modalities: bike commute → treadmill warmup → free weights → elliptical cooldown → bike commute home → yoga in the back yard. This is the use case the v2 spec was rewritten for.

## Preconditions
- App open, no active workout.
- Free-form move entry available ("+ new move").
- Notes available on every entry.
- Sync may be on or off — either way, the workout must succeed.

## Steps

### Outbound commute
1. **Tap "+ new move"**, type `Bike commute`, select **duration**.
2. **Tap Start segment.** (Or if the user prefers Path B, enter the known duration.)
3. **Wait ~5 sec** (representing the ride). Tap Stop segment.
4. **Add note:** `light traffic, cool morning`.

### Treadmill warmup
5. **Select "Treadmill"** from the catalog.
6. **Tap Start segment.** Wait ~5 sec. **Tap Stop.**

### Strength block — bench press
7. **Select "Bench Press"** from the catalog. Log `135 × 10`. Log `145 × 8`. Log `145 × 8`.
8. **Add note** to the third set: `right shoulder twinge — back off next time`.

### Strength block — lat pull-down
9. **Select "Lat Pull Down"** from the catalog. Log `100 × 12`. Log `110 × 10`.

### Elliptical cooldown
10. **Select "Elliptical"** from the catalog. Tap Start segment. Wait ~5 sec. Tap Stop.

### Return commute
11. **Select "Bike commute"** (now in catalog as a custom move). Tap Start segment. Wait ~5 sec. Tap Stop.

### Yoga at home
12. **Tap "+ new move"**, type `Yoga in the back yard`, select **note_only** (or duration if user wants to time it).
13. **Add note:** `15 min, mostly stretching, hamstrings tight`.
14. **Tap Log** (or Stop segment if duration mode).

### Wrap up
15. **Tap Stop** (workout-level). Summary appears.

## Pass criteria

- Workout summary shows: **10 entries** across 5 distinct move names. Total cardio duration > 0. Total weight moved = `135*10 + 145*8 + 145*8 + 100*12 + 110*10 = 5,970 lbs`.
- All entries are present in history under one Workout, with their notes attached and visible.
- Free-form move "Bike commute" was logged once as new, then reusable on the second commute (catalog autocomplete).
- Free-form move "Yoga in the back yard" was logged with a note and no duration/weight.
- No crash when switching between strength, duration, and note_only entries within the same workout.
- If sync is enabled, all entries POST successfully (or queue if offline) — none fail validation due to a free-form `moveName` or null `moveId`.
- Per-entry timestamps are preserved — history shows the chronological order of the day.
