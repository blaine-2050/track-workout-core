# 05 — Offline Queue and Sync

## Goal
Verify the offline-first write path and the outbox drain on reconnect.

## Preconditions
- App open.
- Sync endpoint configured to a reachable test server.
- Device reports online.

## Steps

1. **Put device offline** (airplane mode, disable network in simulator, or point endpoint at a black hole).
2. **Log three entries:** `Bench Press 135×10`, `135×10`, `135×8`.
3. **Observe:** entries appear immediately in history. Sync indicator shows pending count = 3.
4. **Tap Stop.**
5. **Restore network.**
6. **Trigger sync** (automatic on reconnect, or manual button — platform's choice).
7. **Observe:** pending count drops to 0 within a few seconds. Sync status shows success.

## Pass criteria
- All three entries and the workout record exist on the server, with IDs matching the client.
- No duplicate rows on the server even if sync is retriggered.
- Sync status surface is informative on both success and error paths.
- Local writes during step 2 felt instantaneous (< 100ms perceived).
