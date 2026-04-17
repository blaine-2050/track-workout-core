# Track Workout

A multi-platform workout tracker that fits your actual training — strength sets, bike commutes, cardio segments, yoga, and everything in between. One workout, any mix of modalities, with optional heart-rate overlay and cloud sync.

**This repo is the spec.** No executable code. Every platform implementation lives in its own repo and references documents here by URL.

> **Spec version 2.1** — cardio is first-class, free-form move names and notes on every entry, sync is opt-in, HR data from CSV/FIT imports. See [`DECISIONS.md`](DECISIONS.md) for the full history.

## Getting Started

### Pick your platform and clone one repo

| I want to... | Clone this | Then run |
|--------------|-----------|----------|
| **Build for iPhone** (native Swift) | `track-workout-swift` | Open `TrackWorkout.xcodeproj` in Xcode → Run on Simulator or device |
| **Build for iPhone + Android** (one codebase) | `track-workout-expo` | `npm install && npx expo start` → scan QR with Expo Go |
| **Run the sync server** | `track-workout-api` | `npm install && npm run dev` → runs on `localhost:3000` |
| **Read the spec** | This repo | Just read the Markdown files below |

Each repo's `README.md` has full setup instructions. You don't need all four — pick the platform you care about and go. Sync is off by default; the apps work fully offline.

### What a workout looks like

A real session might be: bike commute → 15 min treadmill → 3 sets bench press → 2 sets lat pull-down → 10 min elliptical → bike home → yoga in the back yard. The app accommodates that without making you fight the categories. You can:

- **Log strength** with a numeric keypad (weight + reps in under 5 seconds)
- **Log cardio** with one tap to start, one tap to stop
- **Log anything** by typing a free-form name ("Yoga in the park")
- **Attach a note** to any entry ("right shoulder twinge — back off next time")
- **Import heart-rate data** from a Polar/Garmin/COROS CSV export
- **Optionally sync** to a central server when you're ready

### Heart-rate data

If you have a Polar, Garmin, or COROS watch:
1. Export the workout as a FIT file from the watch app.
2. Convert to CSV: `python body-metrics/fit_to_csv.py workout.fit` (see [body-metrics](https://github.com/blaine-2050/body-metrics)).
3. In the app: Settings → Import HR CSV from file → pick the CSV.
4. The workout in History gets an HR summary badge (avg/max bpm).

Direct BLE streaming from a Polar H10 chest strap is in progress.

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
| `track-workout-swift` | SwiftUI + Core Data | iOS, macOS | Spec v2.1 complete. 10 Maestro test flows green. |
| `track-workout-expo` | Expo + React Native | iOS, Android | Spec v2.1 complete. All features implemented. |
| `track-workout-api` | Express + Drizzle | Railway (server) | Scaffolded, 9/9 tests passing. Not yet deployed. |

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

### Project layout

Four public repos. Each is self-contained.

```
track-workout-core/     ← this repo (spec only, no code)
track-workout-swift/    ← SwiftUI + Core Data (iOS, macOS)
track-workout-expo/     ← Expo + React Native (iOS, Android)
track-workout-api/      ← sync server (Express + Drizzle + MySQL)
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
