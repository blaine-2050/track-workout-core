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

## Related repos
- `blaine-2050/track-workout-swift` — iOS Swift implementation (primary)
- Future: ios-web, android-react, android-other, macos-swift
- Legacy reference (not consumed by platforms): `apps/macos-electron-legacy` inside the legacy `track-workout` monorepo
