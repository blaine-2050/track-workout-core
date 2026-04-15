# Computer Use Protocol

Conventions for AI-driven self-testing, operational feedback, and deployment across all platforms.

## Premise

Manual regression testing is slow and error-prone. Each platform must expose a path where an AI (Claude Desktop, Claude Code, or equivalent) can:

1. **Run the app** in a test harness (simulator, emulator, or local dev server).
2. **Drive the app** through predefined workout scripts (see `WORKOUT_SCRIPTS/`).
3. **Capture screenshots** at each significant step.
4. **Compare** against expected state.
5. **Surface operational deficiencies** — crashes, slowness, confusing flows, broken sync.
6. **Propose fixes** or hand the user the repro with screenshots attached.

The user reviews screenshots, suggests improvements, and the AI iterates.

## Roles

### AI
- Builds and launches the app.
- Executes workout scripts step-by-step.
- Takes screenshots after each step; names them `<script>-<step>-<timestamp>.png`.
- Writes a run report with: script name, pass/fail per step, screenshot paths, notable observations (timing, layout, unexpected modals).
- Never claims a run passed without proof. A green log without screenshots is not a pass.

### User
- Reviews screenshot + report bundles.
- References screenshots when suggesting improvements ("in `bench-press-log-step-3.png` the keypad is cut off on the right").
- Does gym-side real-device testing the AI cannot.
- Authorizes deploys.

## Self-Testing Surface per Platform

Each platform repo must document in its own `CLAUDE.md` the concrete commands for these capabilities. This spec names the capabilities; platforms bind them.

| Capability | iOS Swift | iOS Web | Android RN | macOS Swift |
|------------|-----------|---------|------------|-------------|
| Build | `xcodebuild` | `npm run build` | `./gradlew` | `xcodebuild` |
| Launch in test harness | `xcrun simctl launch` | dev server + Playwright | `adb install` + emulator | `open <app>` |
| Screenshot | `xcrun simctl io booted screenshot` | Playwright `page.screenshot()` | `adb exec-out screencap` | `screencapture` |
| Drive UI | simctl + accessibility IDs | Playwright | Maestro or adb input | AppleScript / accessibility |

## Run Report Format

Every self-test run writes a markdown report to the platform repo's `runs/<date>-<script>.md`:

```markdown
# Run: <script-name> — <ISO timestamp>

Platform: <platform repo>
Commit: <git sha>
Harness: <simulator model / browser / emulator>

## Steps
1. <step description> — PASS — `runs/screens/<file>.png`
2. <step description> — FAIL — `runs/screens/<file>.png` — <what went wrong>

## Observations
- <any operational notes — timing, layout, sync issues>

## Next actions
- [ ] <proposed fix>
- [ ] <question for user>
```

Screenshots go alongside in `runs/screens/`. Reports and screenshots are committed so the user can review via GitHub on mobile.

## Deploy Protocol

### Pre-paid-developer phase (current)
- iOS: build + install on physical device via cable and free provisioning. Re-sign weekly (7-day cert expiry). No TestFlight.
- Web: deploy to Railway or a static host.
- Android: sideload APK via adb.

### Post-paid-developer phase (future)
- iOS: TestFlight via `fastlane pilot` or `xcrun altool`.
- Android: Play internal track via `fastlane supply`.

Each platform repo documents the exact commands.

## Feedback Loop

```
┌─────────────────────────────────────────────────────┐
│ 1. AI runs workout script                           │
│ 2. AI commits report + screenshots to platform repo │
│ 3. User reviews on GitHub (mobile OK)               │
│ 4. User comments on issue or drops notes in repo    │
│ 5. AI iterates → back to step 1                     │
└─────────────────────────────────────────────────────┘
```

## Non-goals

- Full end-to-end physical-device automation. Real-gym testing stays human-driven.
- Replacing product judgment. The AI surfaces facts; humans decide what matters.
