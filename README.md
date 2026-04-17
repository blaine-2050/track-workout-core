# track-workout-core

Authoritative, cross-platform specification for **Track Workout** — a multi-platform app for logging exercise events during training.

This repo holds **only prose specs**. No executable code. Every platform implementation lives in its own repo and references documents here by URL.

> **Current spec version: 2.1** (2026-04-16). Additive: `HeartRateSample` gains `elevationMeters`, `speedKmh`, `distanceKm`; CSV Import Schema documented; COMPUTER_USE_PROTOCOL gains iOS/Maestro lessons. v2 (2026-04-15): cardio first-class, free-form moves, notes, opt-in sync, HR promoted to first-class. See [`DECISIONS.md`](DECISIONS.md) for full rationale and migration notes.

## What's here

| File | Purpose |
|------|---------|
| [`PRD.md`](PRD.md) | Product Requirements Document — vision, users, jobs-to-be-done, functional/non-functional requirements, scope |
| [`BUSINESS_GOALS.md`](BUSINESS_GOALS.md) | North star, goals, constraints |
| [`DATA_MODEL.md`](DATA_MODEL.md) | Canonical data model (Workout, LogEntry, Move, Sync contract) all platforms must honor |
| [`COMPUTER_USE_PROTOCOL.md`](COMPUTER_USE_PROTOCOL.md) | How AI-driven computer use tests and deploys each platform (screenshots, scripts, feedback loops) |
| [`WORKOUT_SCRIPTS/`](WORKOUT_SCRIPTS/) | Plain-English acceptance scenarios each platform must satisfy |
| [`DECISIONS.md`](DECISIONS.md) | Architectural decisions and their rationale |

## Implementations (by tool, not platform)

| Repo | Tool | Platforms | Status |
|------|------|-----------|--------|
| `blaine-2050/track-workout-swift` | SwiftUI + Core Data | iOS, macOS | Active — spec v2.1, 10 Maestro flows green |
| `blaine-2050/track-workout-expo` | Expo + React Native | iOS, Android | Extracted — spec v1 parity, v2 gaps documented |
| `blaine-2050/track-workout-api` | Express + Drizzle | Railway (server) | Scaffolded, 9/9 tests passing, not deployed |
| — | Web (framework TBD) | Browser | Planned |

## How platform repos use this repo

Platform repos don't vendor or submodule this repo. They reference it by URL:

- In their `CLAUDE.md`, a "source of truth" section points to files here.
- When a change to a platform impacts the contract (data model, sync, workout scripts, computer-use protocol), update this repo first, then the platforms.

This is **delegation, not inheritance**. Platform repos can be cloned and worked on from anywhere without needing this repo checked out locally.

## Conventions

- Prose specs only — no code, no configs, no executables.
- Markdown only.
- When a spec changes in a way that breaks platforms, bump a version line at the top of the affected file and note the migration in `DECISIONS.md`.
