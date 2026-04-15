# Data Model

Conceptual schema every platform must honor. Platform-specific ORMs (Core Data, Drizzle, SQLite) may name types differently but must expose these fields.

## Entities

### Move
Exercise catalog entry.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Stable across platforms |
| `name` | string | Human-readable (e.g. "Bench Press") |
| `sortOrder` | int | For alphabetical or user-defined ordering |
| `measurementType` | enum | `strength` or `interval` |

### Workout
Session container.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `userId` | UUID | |
| `startedAt` | ISO8601 UTC | Set when the first move is selected |
| `endedAt` | ISO8601 UTC? | Set on manual Stop |
| `updatedAt` | ISO8601 UTC | Any local edit bumps this |

### LogEntry
One rep-set or interval.

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `userId` | UUID | |
| `workoutId` | UUID | Foreign key to Workout |
| `moveId` | UUID | Foreign key to Move |
| `measurementType` | enum | `strength` or `interval` (denormalized from Move for query convenience) |
| `weight` | number? | Required if `measurementType == strength` |
| `weightUnit` | enum? | `kg` or `lbs`. Per-entry, not global. |
| `reps` | int? | Required if `measurementType == strength` |
| `durationSeconds` | int? | Required if `measurementType == interval` |
| `startedAt` | ISO8601 UTC | When the entry began (keypress/move-select) |
| `endedAt` | ISO8601 UTC | When the entry was logged |
| `updatedAt` | ISO8601 UTC | |

### HeartRateSample (optional)
Pluggable. Each platform may skip until BLE milestone.

| Field | Type |
|-------|------|
| `id` | UUID |
| `workoutId` | UUID |
| `timestamp` | ISO8601 UTC |
| `bpm` | int |
| `source` | string (e.g. `polar-h10`, `fixture`) |

## Sync Contract

- **Transport:** HTTPS JSON POST to a platform-configurable endpoint.
- **Direction:** client → server push of outbox; server → client pull for updates since last sync cursor.
- **Conflict resolution:** **latest `updatedAt` wins**, compared in UTC.
- **Idempotency:** client may POST the same entry repeatedly; server dedupes by `id`.
- **Offline:** all writes go to a local outbox first. Sync drains the outbox.
- **Auth:** JWT bearer token (details deferred to API server repo).

## Seed Data

All platforms ship with the same seed moves on first launch:

1. Bench Press
2. Single Arm Snatch
3. Incline DB Press
4. Military DB Press
5. Squat
6. Split Squat
7. Deadlift
8. Lat Pull Down
9. Bent Over Row
10. Leg Press
11. MTB (interval)
12. Elliptical (interval)
13. Treadmill (interval)

## Versioning

This document is the contract. Breaking changes require:
1. A version bump noted at the top of the affected section.
2. A migration entry in `DECISIONS.md`.
3. Notification to each platform repo's issue tracker.
