# Decisions

Architectural and product decisions, newest first.

## 2026-04-16 — Defer searchable-sheet move picker
**Decision:** Keep the existing SwiftUI `Menu` for move selection for now. Do not replace it with a sheet-style searchable picker until a concrete need for search (catalog > ~25 moves, user complaint about scrolling) arises.

**Why:** Attempted rewrite to `.sheet(isPresented:) { MovePickerSheet }` produced a reproducible SwiftUI quirk on iOS 26.2: tapping a row inside `List` → sheet toggles `isPresented = false` via @Binding AND via `DispatchQueue.main.async` still failed to dismiss the presenting sheet, even though the selected-move binding updated correctly (checkmark rendered). The issue appears specific to `List` rows inside a plain `View` sheet; moving to `NavigationStack + .searchable` made dismissal work when tapping the toolbar Cancel button but still broke row-tap dismissal. Fighting this isn't worth it against the current value: the default 13-move catalog fits mostly on-screen, real users can scroll a Menu with their finger, and the remaining gap ("Treadmill falls below fold") was a Maestro-test-design constraint rather than a user-facing bug.

**How to apply:** When a future user really does have ~30+ moves and asks for search, revisit with one of:
- `NavigationStack + Picker(selection:) { ForEach(...) } .pickerStyle(.navigationLink)` — standard SwiftUI idiom for large selections.
- A separate screen (push, not sheet) via NavigationLink.
- `sheet(item:)` with an Identifiable wrapper — the selection itself drives presentation, avoiding the `isPresented` dismissal dance.

The legacy `services/ble/` doc in the old monorepo has a note about SwiftUI sheet/List pitfalls; worth referencing when this is re-attempted.

## 2026-04-16 — HR ingest: CSV first, FIT deferred, BLE later
**Decision:** For v2.1, HR data arrives as CSV (Polar Flow / Garmin Connect / COROS export). The `HeartRateSample` entity gains `elevationMeters`, `speedKmh`, `distanceKm` as optional fields so a single CSV import can carry HR + co-metrics typical of sports-watch exports. FIT (binary) import and direct BLE streaming are both deferred behind the CSV path.

**Why:** A CSV importer is self-contained (no device drivers, no binary parser, no BLE permissions). Vendor apps all export CSV. The user's reference files in `~/Athenia/projects/body-metrics/data` are FIT, but that confirms we also need FIT eventually — just not first. Starting with CSV unblocks actual use immediately.

**Consequence:**
- `HeartRateSample` in iOS gets three additional optional attributes; lightweight Core Data migration.
- New `WORKOUT_SCRIPTS/07-hr-csv-import.md` spec.
- iOS importer UI lives inside Settings (file picker → preview row count → commit or cancel).
- When FIT is prioritized later: either pull a Swift FIT library (e.g. FitFileParser) into the iOS repo, or do the FIT→CSV conversion inside `body-metrics` and import the CSV here.
- HR samples stay local (no sync) — high-volume time-series sync is a later spec version.

## 2026-04-16 — "Elipitical" seed typo known, fix queued
**Decision:** The seed catalog in iOS contains the typo "Elipitical" (should be "Elliptical"). Spec (`DATA_MODEL.md`) has been correct since v2. Fix in iOS at the next seed change; no migration story is strictly needed since the typo is user-visible only as a display label.

**Why:** Low-priority cosmetic bug. Flagging here so it's not forgotten but deferring the fix behind higher-value work.

**How to apply:** When touching `PersistenceController.seedMoves`, update the string and any fixture tests/flows that reference it.

## 2026-04-16 — note_only entry UX deferred
**Decision:** The `note_only` `measurementType` is defined in the spec and honored by the Add Move sheet, but the **entry path** (a Log button that writes a LogEntry with just `moveName` + `notes` + no weight/reps/duration) is not implemented yet. In the meantime, users pick "Cardio" for moves like "Yoga at home" and log them as tap-start/tap-stop segments.

**Why:** Unblocks script 06 (multi-modal) without adding UX complexity during rapid iteration on the core flows. The deferral is visible in `WORKOUT_SCRIPTS/06-multi-modal-session.md` comments.

**How to apply:** When implementing: for a `note_only` move, replace the keypad + Log with a compact "Log with note" affordance that requires at least an attached `pendingNote` or an empty-log confirmation. Entry stores `measurementType="note_only"`, `weight=NULL`, `reps=NULL`, `durationSeconds=NULL`.

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
