# Data Model

> **Spec version:** 2.1 (2026-04-16) — adds `elevationMeters`, `speedKmh`, `distanceKm` to `HeartRateSample`; documents the CSV Import Schema. v2 (2026-04-15) added `notes`, made `moveId` optional, added `moveName`, expanded `measurementType`, promoted `HeartRateSample` to first-class. See `DECISIONS.md`.

Conceptual schema every platform must honor. Platform-specific ORMs (Core Data, Drizzle, SQLite) may name types differently but must expose these fields.

## Entities

### Move
Catalog **suggestion**, not a constraint. Users can log against any name; the catalog provides autocomplete and grouping.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Stable across platforms |
| `name` | string | Human-readable (e.g. "Bench Press") |
| `sortOrder` | int | For alphabetical or user-defined ordering |
| `measurementType` | enum | `strength`, `duration`, or `note_only` |
| `isCustom` | bool | `true` if added by the user (not a seed) |

### Workout
Container for one or more entries spanning any mix of modalities.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `userId` | UUID | |
| `startedAt` | ISO8601 UTC | Set when the first entry is logged |
| `endedAt` | ISO8601 UTC? | Set on manual Stop |
| `updatedAt` | ISO8601 UTC | Any local edit bumps this |

### LogEntry
One logged event: a strength set, a cardio segment, or a free-form note. Independently persisted; grouped by `workoutId`.

| Field | Type | Required when | Notes |
|-------|------|---------------|-------|
| `id` | UUID | always | |
| `userId` | UUID | always | |
| `workoutId` | UUID | always | The owning Workout |
| `moveId` | UUID? | optional | Set when entry references a Move from the catalog. NULL for ad-hoc/free-form moves. |
| `moveName` | string | always | Always populated. Mirrors `Move.name` if `moveId` is set; otherwise the user's typed string (e.g. "Yoga in the back yard"). |
| `measurementType` | enum | always | `strength` \| `duration` \| `note_only` |
| `weight` | number? | `measurementType == strength` | |
| `weightUnit` | enum? | `measurementType == strength` | `kg` or `lbs`. Per-entry. |
| `reps` | int? | `measurementType == strength` | |
| `durationSeconds` | int? | `measurementType == duration` | Computed from start/end if user used tap-start/tap-stop, or entered directly. |
| `startedAt` | ISO8601 UTC | always | When the entry began (keypress / tap-start / move-select). |
| `endedAt` | ISO8601 UTC? | duration entries; optional for strength | When the entry was completed. |
| `notes` | string? | optional | Free-form text up to ~1000 chars. Optional on every entry. |
| `intervalKind` | enum? | optional | For interval-marked cardio: `work` \| `rest`. |
| `intervalLabel` | string? | optional | Human-readable label for the interval. |
| `intensity` | number? | optional | Subjective effort 1–10 or RPE-style. Reserved. |
| `intensityMetric` | string? | optional | What `intensity` represents (`rpe`, `estimated_effort`, etc.). |
| `updatedAt` | ISO8601 UTC | always | |

### HeartRateSample
Time-series sample centered on heart rate, with optional co-located metrics commonly captured by the same device (elevation, speed, distance). **First-class** as of v2; widened in v2.1 to carry the co-metrics that vendor exports ship alongside HR.

Three ingest paths, all targeting this entity:
- **CSV import** (primary) — Polar Flow / Garmin Connect / COROS / compatible export. User selects a CSV; the importer parses, normalizes, and persists samples. See `CSV Import Schema` below for the required column shape.
- **FIT import** (planned) — Garmin/COROS binary FIT format. Requires a parser library; deferred behind CSV.
- **Direct BLE** (planned) — Polar H10 or compatible over GATT Heart Rate Service `0x180D`. Samples stream in live during a workout. Co-metrics (elevation/speed/distance) remain NULL on this path.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `userId` | UUID | |
| `workoutId` | UUID? | Set when alignment to a workout is confident; NULL until aligned. |
| `timestamp` | ISO8601 UTC | Sample time. |
| `bpm` | int | Heart rate in beats per minute. Core field — required. |
| `rrIntervalMs` | int? | Beat-to-beat interval if available (Polar H10 reports this). |
| `elevationMeters` | number? | Altitude in meters. |
| `speedKmh` | number? | Ground speed in km/h. |
| `distanceKm` | number? | Cumulative distance from session start in km. |
| `source` | string | e.g. `polar-h10-ble`, `polar-flow-csv`, `garmin-connect-csv`, `coros-csv`, `fixture`. |
| `importBatchId` | UUID? | Groups samples imported from the same CSV/FIT file. |

#### CSV Import Schema

The importer accepts any CSV whose header row contains these columns. Names are case-insensitive; extra columns are ignored; metric units are canonical.

| Column | Required | Type | Example | Notes |
|--------|----------|------|---------|-------|
| `timestamp` | **yes** | ISO8601 UTC | `2026-04-16T10:57:33Z` | If the CSV carries local-time without offset, the importer assumes device-local timezone and records the normalized UTC. |
| `heart_rate_bpm` | **yes** | int | `142` | |
| `elevation_m` | no | float | `134.5` | Meters. |
| `speed_kmh` | no | float | `8.3` | Km/h. Convert from m/s by `*3.6`. |
| `distance_km` | no | float | `2.15` | Km. Cumulative from session start. |

- **Sampling interval:** second-by-second preferred but not required. The importer accepts any cadence; no interpolation.
- **Missing values:** blank cells → NULL. Sentinel values (`0` for bpm on vendor exports) are treated as NULL.
- **Alignment:** after import, each sample is tagged with the Workout whose `[startedAt, endedAt]` (or `[startedAt, now]` if open) window contains `timestamp`. Unaligned samples keep `workoutId = NULL` and stay importable later when a matching workout exists.
- **Batch handling:** every import assigns a fresh `importBatchId` so the user can undo a single import without losing unrelated samples.

### MoveAlias (planned)
Optional. Maps user-typed strings to canonical Moves so "yoga", "Yoga", "yoga in the park" can roll up for analytics later. Not required in v2.

## Sync Contract

- **Transport:** HTTPS JSON POST to a platform-configurable endpoint.
- **Direction:** client → server push of outbox. (Server → client pull is reserved for a later spec version.)
- **Opt-in:** sync is disabled by default. The app must function fully offline.
- **Conflict resolution:** **latest `updatedAt` wins**, compared in UTC.
- **Idempotency:** client may POST the same entry repeatedly; server dedupes by `id`.
- **Offline:** all writes go to a local outbox first. Enabling sync drains the outbox.
- **Free-form moves:** entries with `moveId == NULL` sync as-is, with `moveName` carrying the user's string. The server does not auto-create a Move row for free-form names.
- **HR samples:** sync deferred. HR data stays local until a future spec version covers HR sync (volume considerations).
- **Auth:** static API key for single-user mode (current); JWT for multi-user (future).

## Seed Data

All platforms ship with the same seed moves on first launch. These are **suggestions**, not the only allowed names:

| # | Name | measurementType |
|---|------|-----------------|
| 1 | Bench Press | strength |
| 2 | Single Arm Snatch | strength |
| 3 | Incline DB Press | strength |
| 4 | Military DB Press | strength |
| 5 | Squat | strength |
| 6 | Split Squat | strength |
| 7 | Deadlift | strength |
| 8 | Lat Pull Down | strength |
| 9 | Bent Over Row | strength |
| 10 | Leg Press | strength |
| 11 | MTB | duration |
| 12 | Elliptical | duration |
| 13 | Treadmill | duration |

(Suggested cardio additions for v2.1: `Bike commute`, `Walk`, `Yoga`, `Stretching` — but these may equally be added by users on demand.)

## Versioning

This document is the contract. Breaking changes require:
1. A version bump in the header.
2. A migration entry in `DECISIONS.md` (with explicit migration guidance for each platform).
3. Notification to each platform repo's issue tracker.

### v2 → v2.1 migration notes (2026-04-16)
- **`HeartRateSample.elevationMeters`, `speedKmh`, `distanceKm`** are new nullable numeric fields. Additive; no backfill needed.
- CSV Import Schema documented. No entity changes beyond the three new optional fields.
- No other platforms have HR support yet, so migration is iOS-only.

### v1 → v2 migration notes (2026-04-15)
- **`LogEntry.moveId` becomes optional.** Existing rows keep their value; new free-form entries set it to NULL.
- **`LogEntry.moveName` is new and required.** Backfill existing rows by reading the linked `Move.name`.
- **`LogEntry.notes` is new and nullable.** No backfill needed.
- **`LogEntry.measurementType` enum gains `note_only`** and renames `interval` → `duration`. Existing `interval` values are mapped to `duration` on read; writes use `duration`.
- **`Move.isCustom` is new.** Backfill seeded moves to `false`.
- **`HeartRateSample.rrIntervalMs`, `source` widening, `importBatchId`** are additive. Existing samples (none in production yet) need no migration.
