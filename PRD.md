# Track Workout — Product Requirements

## Product Vision
Track Workout helps athletes and everyday users log workouts quickly across iOS, Android, macOS, and web, even when offline, then sync safely to a central cloud repository.

Core principle: capture enough temporal signal to understand recovery at every scale — from short heavy sets (e.g. 30-second leg press work) to mixed sessions (bike, strength, swim, jog).

## Users
- **Primary:** individual users who want low-friction workout logging.
- **Secondary:** developer/tester running frequent usability experiments across platforms.

## Core Jobs To Be Done
1. Capture a workout event in under 5 seconds.
2. Continue logging without internet connectivity.
3. Preserve event history and merge offline edits to a central backend when online.
4. Keep UX behavior consistent enough across clients to compare user feedback.

## Functional Requirements
1. Users can log strength work as `move + weight + reps + unit`, with field timestamps to estimate time-under-load and inter-set recovery.
2. Users can log aerobic work with duration plus intensity heuristics.
3. Users can mark natural aerobic intervals (`work` / `rest`) with quick interval taps.
4. Users can edit/delete recent events locally.
5. App keeps working offline and queues sync operations.
6. Sync resolves duplicates deterministically (latest-edit-wins, UTC).
7. Each event has stable IDs and UTC timestamps.
8. Heart-rate ingestion is pluggable and can fall back to fixture data.

## Non-Functional Requirements
- Startup under 2 seconds on test hardware.
- Event write path should feel instantaneous (<100ms perceived on local write).
- Data model versioned and migration-friendly.
- Platform codebases can be developed independently.

## UX Invariants (all platforms)
- **Numeric keypad first.** Text-heavy forms are rejected.
- **Sticky inputs.** Weight/reps persist after a log; next keypress replaces them.
- **Set timer.** Elapsed time since last log or move selection is visible.
- **Workout lifecycle.** Auto-start on first move select; manual Stop; Stop shows a summary (end time, total duration, total sets, total weight moved).
- **History shape.** Hierarchical display (Workout → Set → Entry) with per-entry timing.
- **Unit toggle.** kg/lbs is per-entry, not global, to support mixed-unit workouts.

## Scope — Current
- **Gym mode (working):** move select, weight/reps keypad, sticky inputs, per-entry timing, set/workout boundaries, workout summaries, unit tracking, sample log export.
- Local-first persistence.
- Sync pipeline to central backend (target: Railway-hosted API).
- Optional BLE heart-rate enrichment when available.

## Scope — Deferred
- Remote API server deployment (Railway).
- Endurance/aerobic mode polish.
- Social/community features.
- Complex coaching plans.
- Wearable integrations beyond Polar H10 proof-of-concept.
- Apple Watch companion.

## Success Metrics (First Milestone)
- iOS app can create, view, and export local workouts reliably.
- Sticky inputs and workout summaries match reference UX.
- Two-way sync works after temporary offline period (requires API server).
- A single data-model spec (this repo) is honored by all clients and the backend.

## Milestone Plan
1. **iOS stabilization** — fix known issues, verify on physical iPhone (gym-tested).
2. **API baseline on Railway** — deploy sync server with health checks and auth.
3. **Offline-first sync path** — retry/backoff, idempotent writes, merge validation.
4. **BLE integration** — Polar H10 live readings with fixture fallback.
5. **macOS Swift** — port iOS patterns to desktop.
6. **Android** — React Native build hardening and sync integration.
7. **Release readiness** — branch gates, runbook, go/no-go checklist.

## Risks
- Divergence between platforms if this spec is not honored.
- Offline merge logic complexity introduces data integrity bugs.
- Free Apple provisioning requires weekly re-signing during pre-paid-developer phase.
