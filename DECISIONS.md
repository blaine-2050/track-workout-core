# Decisions

Architectural and product decisions, newest first.

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
