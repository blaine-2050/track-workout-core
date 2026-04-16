# Decisions

Architectural and product decisions, newest first.

## 2026-04-15 — Spec v2: cardio first-class, free-form moves, notes, opt-in sync, HR planned
**Decision:** Fundamental scope revision. The spec now treats:
1. **Cardio as first-class.** A real session mixes a bike commute, machine cardio, free weights, more cardio, and yoga. The data model and UX must accommodate this without making cardio a second-class citizen.
2. **Free-form move names as legitimate.** The seeded catalog is *suggestions*, not constraints. Users can log "Yoga in the back yard" without first registering it.
3. **Notes as a first-class field on every entry.** Free-text up to ~1000 chars, optional, never gating Log.
4. **Sync as opt-in.** Default state requires no remote configuration; the app is fully usable without enabling sync. Implemented in iOS settings sheet.
5. **HR data as a planned dimension** with two ingest paths: (a) CSV import from Polar/Garmin exports, (b) direct BLE from Polar H10. Both target the same `HeartRateSample` entity.

**Why:** The original spec optimized for structured strength training and assumed a mature sync backend. Real workouts are messier — multi-modal, often offline, often containing moments that don't fit a clean schema. The user wants the app to fit their life, not vice versa. Sync is valuable but should not gate first use.

**Consequence:**
- `LogEntry.moveId` becomes optional; `moveName` is always populated.
- `LogEntry.notes` is added (nullable).
- `LogEntry.measurementType` gains `note_only`; `interval` renamed to `duration` (with read compatibility for old values).
- `Move.isCustom` is added.
- `HeartRateSample` gains `rrIntervalMs`, widened `source`, `importBatchId`.
- iOS app must implement: `notes` field, free-form move entry path, cardio tap-start/tap-stop UX, HR CSV import.
- API server (`track-workout-api`) needs schema migration to add `notes`, accept null `moveId`, expand `measurementType` enum.
- Milestone plan revised: HR CSV import and free-form moves precede Railway deploy.

## 2026-04-14 — Restructure to per-platform repos + prose-only core
**Decision:** Split the legacy `track-workout` monorepo into per-platform repos. Create `track-workout-core` as the single source of truth, containing prose specs only. Platform repos reference core by URL.

**Why:** Simpler AI context per repo. Independent deploys. Delegation (URL reference) beats inheritance (submodule/vendoring) for mobile/remote work via Claude apps.

**Consequence:** Changes to the contract (data model, sync, UX invariants) happen here first, then propagate.

## 2026-04-14 — Defer paid Apple Developer account
**Decision:** No TestFlight for now. Physical-device iOS testing uses free provisioning with weekly cert re-signs.

**Why:** Avoid $99 until feature stabilization makes TestFlight worth it.

**Consequence:** Gym testing requires a cabled install and a weekly re-sign ritual. No over-the-air beta distribution.

## 2026-04-14 — Keep `macos-electron-legacy` as reference
**Decision:** Do not migrate Electron legacy app to a standalone repo. Leave in legacy `track-workout` monorepo.

**Why:** It's feature-complete and serves as a visible UX reference. No future iteration planned.

## Pre-restructure decisions (inherited)
- Android stack: React Native for current delivery.
- Offline conflict resolution: latest-edit-wins with UTC timestamps.
- Weight unit stored per `LogEntry`, not globally, to support mixed-unit workouts.
- Auth: simple low-friction login with JWT + basic abuse protection.
