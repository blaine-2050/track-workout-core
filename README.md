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

## Architecture: delegation, not inheritance

Inspired by [Self](https://en.wikipedia.org/wiki/Self_(programming_language)): every project is an **encapsulated peer object**. No hierarchy, no parent-child nesting. Each project has its own state, build, tests, and deploy. References to other projects are explicit URL-based **delegation slots** documented in each project's `CLAUDE.md`.

This repo is the **prototype object** — it defines shared behavior. Other projects delegate TO it. It does not contain or own them.

```
                    ┌──────────────────────────┐
                    │    track-workout-core     │
                    │   (spec: PRD, data model, │
                    │    scripts, decisions)    │
                    └─────────┬────────────────┘
                              │
               ┌──────────────┼──────────────┐
               │  delegates   │  delegates   │  delegates
               ▼              ▼              ▼
       ┌──────────────┐ ┌───────────┐ ┌────────────┐
       │    -swift     │ │   -expo   │ │    -api    │
       │  SwiftUI      │ │ React    │ │  Express   │
       │  Core Data    │ │ Native   │ │  Drizzle   │
       │  iOS + macOS  │ │ iOS +   │ │  MySQL     │
       │              │ │ Android  │ │  Railway   │
       └──────────────┘ └───────────┘ └────────────┘
               │              │              │
               └──── sync ────┴──── sync ────┘
                                │
       ┌───────────────┐  ┌─────┴──────────┐
       │      ble      │  │  body-metrics  │
       │  Polar H10    │─▶│  FIT→CSV       │
       │  R&D          │  │  HR viz hub    │
       └───────────────┘  └────────────────┘
```

### Project layout on disk

Each repo is a peer under `~/Athenia/projects/`. No nesting.

```
projects/
├── track-workout-core/     ← this repo (prototype: spec only)
├── track-workout-swift/    ← SwiftUI + Core Data (iOS, macOS)
├── track-workout-expo/     ← Expo + React Native (iOS, Android)
├── track-workout-api/      ← sync server (Express + Drizzle)
├── body-metrics/           ← peer: FIT→CSV, HR integration hub
├── ble/                    ← peer: Polar H10 BLE research
└── track-workout/          ← archived legacy monorepo (reference only)
```

### How delegation works in practice

Each project's `CLAUDE.md` has a **"Delegates to"** table listing what it references — analogous to inspecting an object's parent slots in a Self browser:

```markdown
## Delegates to
| What            | Where              | How                     |
|-----------------|--------------------|-------------------------|
| Behavior spec   | track-workout-core | Fetch PRD.md, DATA_MODEL.md by URL |
| Sync server     | track-workout-api  | POST /sync/events, opt-in          |
| HR data (FIT)   | body-metrics       | fit_to_csv.py → CSV → file import  |
```

- **Spec wins.** If a platform's code disagrees with `track-workout-core`, the spec wins unless a decision is recorded in the platform's own `CLAUDE.md`.
- **Update core first.** Contract changes (data model, sync, UX invariants) happen here before propagating to platforms.
- **No vendoring.** Platform repos reference this repo by URL, never by submodule or copy.

## Conventions

- Prose specs only — no code, no configs, no executables.
- Markdown only.
- When a spec changes in a way that breaks platforms, bump a version line at the top of the affected file and note the migration in `DECISIONS.md`.
