# Workout Scripts

Plain-English acceptance scenarios each platform must satisfy.

These are **prose**, not executable. Each platform repo translates a script into its own test harness invocation (see `COMPUTER_USE_PROTOCOL.md`).

## Naming

`<nn>-<short-slug>.md` — e.g. `01-single-set-bench-press.md`.

Numeric prefixes keep ordering stable. Scenarios progress from trivial to complex.

## Structure of a script

Each script file contains:

1. **Goal** — what behavior is being verified.
2. **Preconditions** — app state before starting (fresh install? existing workout? offline?).
3. **Steps** — numbered user actions. Each step says what the user does and what should be visible after.
4. **Pass criteria** — what must be true at the end (data, UI, timing).

## Catalog

| Script | Goal |
|--------|------|
| [`01-single-set-bench-press.md`](01-single-set-bench-press.md) | Log one set of bench press from a clean state |
| [`02-multi-set-sticky-inputs.md`](02-multi-set-sticky-inputs.md) | Verify sticky-input behavior across sets |
| [`03-workout-stop-summary.md`](03-workout-stop-summary.md) | Verify Stop triggers a correct workout summary |
| [`04-interval-cardio.md`](04-interval-cardio.md) | Log an interval (treadmill) with HH:MM:SS entry |
| [`05-offline-queue-and-sync.md`](05-offline-queue-and-sync.md) | Log offline, then verify outbox drains on reconnect |

Add new scripts as features stabilize. Scripts should stay small and focused; build complexity through composition, not large scripts.
