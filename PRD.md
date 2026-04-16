# Track Workout — Product Requirements

> **Spec version:** 2 (2026-04-15) — major revision: cardio is first-class, free-form moves and notes are first-class, sync is opt-in, HR data is a planned dimension. See `DECISIONS.md`.

## Product Vision
Track Workout helps athletes and everyday users log workouts quickly across iOS, Android, macOS, and web — even when offline.

A "workout" is whatever the user says it is. A real session might include a bike commute to the gym, fifteen minutes on a treadmill, three sets of bench press, two sets of lat pull-down, ten minutes on the elliptical, a bike ride home, and fifteen minutes of yoga in the back yard. The app accommodates that without making the user fight the categories.

Core principle: **capture enough temporal signal to understand recovery at every scale**, from short heavy sets (e.g. 30-second leg press work) to multi-modal full-day sessions.

## Users
- **Primary:** individual users who want low-friction logging that doesn't impose a workout philosophy.
- **Secondary:** developer/tester running frequent usability experiments across platforms.

## Core Jobs To Be Done
1. **Log a structured set in under 5 seconds.** Strength: pick move, type weight + reps, tap Log.
2. **Log a cardio segment with a tap.** "I'm on the treadmill now" → tap-start, tap-stop later. Optional duration / distance / note.
3. **Log a free-form moment without fighting the catalog.** Type "Yoga in the park" or "Stretching" — the app accepts whatever the user calls it.
4. **Attach a quick note** to any entry: "felt heavy", "wind in face", "right knee twinge".
5. **Continue logging without internet connectivity.** Default state: no network required.
6. **Optionally sync** to a central backend when the user enables it. Sync is opt-in, not the price of admission.
7. **Align heart-rate data** to workout timing — either from a live BLE source (e.g. Polar H10) or from an imported CSV (e.g. Polar Flow / Garmin Connect export).
8. **Review history hierarchically:** workout → modality block → entry, with timing and notes.

## Functional Requirements

### Logging — structured (strength)
1. Users can log strength work as `move + weight + reps + unit`, with field timestamps for time-under-load and inter-set recovery.
2. Sticky inputs: after Log, weight and reps persist; the next keypress in either field replaces independently (per-field stickiness).
3. Edit/delete recent entries locally.

### Logging — cardio / interval
4. Users can log a cardio segment via **tap-to-start / tap-to-stop**. Duration is computed from the timestamps. Optional distance and intensity fields.
5. Users can log a cardio segment via **direct duration entry** (HH:MM:SS) when they know it ahead of time (e.g. "ran 25:00 on the treadmill").
6. Users can mark `work` / `rest` intervals within a cardio segment with quick taps.

### Logging — free-form
7. Users can log against any move name, whether or not it's in the seeded catalog. The catalog provides suggestions; it does not constrain entry.
8. Users can attach a free-text note (up to ~1000 chars) to any entry, regardless of measurement type.

### Workout lifecycle
9. A workout is a container for one or more entries spanning any mix of modalities and any number of minutes/hours.
10. Workout starts on first entry; user manually Stops. Auto-stop after a long quiet period is **not** required.
11. Stop shows a summary: end time, total duration, count of entries (broken out by modality if mixed), total weight moved (strength), total cardio duration.

### Heart-rate data
12. The app can ingest HR samples from two sources, both optional and pluggable:
    - **CSV import** (Polar Flow / Garmin Connect or compatible export).
    - **Direct BLE** (Polar H10 over GATT Heart Rate Service 0x180D, or compatible).
13. HR samples align to entries by time-window overlap. The app shows HR alongside entries when present; absence of HR data does not degrade the structured logging UX.

### Persistence and sync
14. App keeps working offline. Every write hits local storage immediately.
15. Sync is **opt-in**. The default state requires no remote configuration; the app is fully usable without enabling sync.
16. When enabled, sync resolves duplicates deterministically (latest-`updatedAt` wins, UTC). Each event has a stable UUID.
17. Sync conflicts are surfaced (count and list); UX for resolving conflicts is deferred but the data is preserved.

## Non-Functional Requirements
- Startup under 2 seconds on test hardware.
- Event write path should feel instantaneous (<100 ms perceived on local write).
- Data model versioned and migration-friendly.
- Platform codebases can be developed independently against this spec.
- App must remain fully usable with sync disabled and no network.

## UX Invariants (all platforms)
- **Numeric keypad first** for strength entry. Text-heavy forms are rejected for weight/reps.
- **Sticky inputs (per field).** Weight and reps persist after Log; first keypress in each field clears independently.
- **Set timer** visible during a workout; reflects time since last log or move selection.
- **Workout lifecycle:** auto-start on first entry; manual Stop; Stop shows a summary.
- **History shape:** hierarchical (Workout → consecutive-modality block → entry) with per-entry timing.
- **Unit toggle (kg/lbs):** per-entry, not global. Mixed-unit workouts are valid.
- **Free-form move entry no slower than catalog selection.** A "+ new move" path is on the main screen, not buried.
- **Notes are optional and second-tier.** Never block a Log on a note. Tap-to-add, tap-to-skip.
- **Cardio entry is one-tap to start, one-tap to end.** No required keypad interaction for the simple case.

## Scope — Current
- **Strength logging (working on iOS):** move select, weight/reps keypad, sticky inputs, per-entry timing, set/workout boundaries, workout summaries, unit tracking, CSV export.
- **Cardio logging (in progress):** tap-start/tap-stop sessions, optional duration entry, free-form notes.
- **Free-form moves (in progress):** "+ new move" entry path; logged entries against ad-hoc names.
- **Local-first persistence** (Core Data on iOS).
- **Optional remote sync** (Railway-hosted API). Disabled by default.
- **HR ingest path #1 (CSV import):** parse Polar / Garmin export, align by time, store HR samples.

## Scope — Deferred
- HR ingest path #2 (direct BLE Polar H10) — prototype exists in the legacy monorepo's `services/ble/`, awaits Swift port.
- Apple Watch companion.
- Conflict-resolution UX.
- Endurance/aerobic intensity heuristics (HR-zone aware).
- Social/community features.
- Complex coaching plans.
- macOS Swift, Android (RN), iOS Web — clients exist as plans but are not active.

## Success Metrics (Revised Milestone)
- iOS app can create, view, and export a multi-modal workout (strength + cardio + free-form moves + notes) reliably.
- Stop summary correctly counts entries across modalities; per-entry timing is preserved.
- Sync can be enabled and used end-to-end against a Railway server, but disabling sync leaves the app fully functional.
- HR samples from a Polar/Garmin CSV align correctly to a logged workout's time window.

## Milestone Plan (revised 2026-04-15)
1. **Spec stabilization (this revision)** — cardio first-class, free-form, notes, opt-in sync. Done in this commit.
2. **iOS: data model migration** — add `notes` to `LogEntry`; relax `moveId` to optional; add `moveName` (always populated). Lightweight Core Data migration.
3. **iOS: free-form move entry path** — make the "+ new move" button real on the main screen; new moves are logged against an ad-hoc name with no catalog id.
4. **iOS: cardio session UX** — tap-start/tap-stop with optional note. Distinct from existing duration-entry flow.
5. **iOS: notes field on entries** — add to entry UI and history rendering.
6. **HR CSV import (iOS-local first)** — parse Polar/Garmin CSV, store HR samples, align to workout window, render on history.
7. **Real gym test** — physical-device session covering #2–#6.
8. **API server bring-up** — only after iOS + spec are stable. Add `notes` and free-form `moveName` to schema; Railway deploy.
9. **HR direct BLE (Polar H10) on iOS** — port `services/ble/` parser logic to Swift CoreBluetooth.
10. **Other platforms (macOS Swift, Android, web)** — when prioritized.

## Risks
- Divergence between platforms if this spec is not honored. Mitigation: spec lives in this repo; every platform repo references it by URL.
- Offline merge complexity introduces data integrity bugs. Mitigation: latest-edit-wins policy is simple; conflicts are logged not silently dropped.
- Free Apple provisioning requires weekly re-signing during pre-paid-developer phase.
- HR alignment depends on accurate timestamps in CSV exports — formats vary by vendor; validation is required at import time.
