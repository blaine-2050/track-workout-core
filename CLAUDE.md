# CLAUDE.md — track-workout-core

## What this repo is
Authoritative cross-platform specification for Track Workout. **Prose only.** No code, no configs.

## Role
- Single source of truth for product requirements, data model, sync contract, computer-use protocol, and workout acceptance scripts.
- Referenced by platform repos **by URL**, not by submodule. Platform repos cite specific files here.

## When editing this repo
- Resist adding code, schemas-as-code, or build config. If a concept needs to be executable, it belongs in a platform repo, not here.
- When a change breaks platforms, note the migration path in `DECISIONS.md`.
- Use absolute GitHub URLs (`https://github.com/blaine-2050/track-workout-core/blob/main/…`) when citing files from platform repos, not relative paths.

## When working from a platform repo
- Fetch the relevant file from this repo (WebFetch) instead of assuming you remember it.
- If a platform's behavior diverges from a spec here, the spec wins unless there's an explicit decision recorded in the platform's `CLAUDE.md`.

## Delegation graph

This repo is the **prototype object** (in Self terms). It doesn't delegate upward — it IS the spec. Other projects delegate TO it.

### Delegated by (objects that reference this spec)

| Repo | Tool | Platforms | What it uses from here |
|------|------|-----------|----------------------|
| [track-workout-swift](https://github.com/blaine-2050/track-workout-swift) | SwiftUI + Core Data | iOS, macOS | PRD, DATA_MODEL, WORKOUT_SCRIPTS, COMPUTER_USE_PROTOCOL |
| [track-workout-expo](https://github.com/blaine-2050/track-workout-expo) | Expo + React Native | iOS, Android | PRD, DATA_MODEL, WORKOUT_SCRIPTS |
| [track-workout-api](https://github.com/blaine-2050/track-workout-api) | Express + Drizzle | Railway | DATA_MODEL (wire format, schema), DECISIONS |

### Peer services (not delegated, but referenced)

| Repo | What it provides |
|------|-----------------|
| [body-metrics](../body-metrics/) | FIT→CSV conversion (`fit_to_csv.py`); HR integration hub |
| [ble](../ble/) | Polar H10 BLE research (future live HR streaming) |

### Legacy
- `track-workout/apps/macos-electron-legacy` — feature-complete UX reference. Not actively maintained.
