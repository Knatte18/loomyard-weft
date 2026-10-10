---
verdict: APPROVED
---

No findings.

Review range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `58791f2b0`).
The `## Publish failure` section is `none`.

## Focus verdicts

**1. `BootSignalWakesBackedOffDiscovery` (`internal/reedcli/watchdog_integration_test.go`): not a defect, the step fails without the signal wake.**
The loop starts at a 100 ms base with a 30 s ceiling.
The prime is already booted, so the first cycle sees a changed set and leaves the cadence at the base.
Each later cycle finds the set unchanged and doubles, so unsignalled cycles fall 1, 3, 7, 15, 31, 63 and 127 bases after the first.
The pair boots after `time.Sleep(70 * base)`, past the 63rd, and the wait ends at `primeEnteredAt + 120 * base`, before the 127th.
Remove the `case _, ok := <-events` branch of `runWatchdogLoop` and nothing else reaches the pair's session inside that window, so `waitForCondition` fails.
The margin is about seven bases, which absorbs the poll lag between the cycle and the observed log line; each cycle's `ListSessions` latency pushes the 127th later, never earlier.
The wake itself works on Linux: truncating the existing empty signal file with `os.WriteFile` raised an fsnotify `WRITE` event in a scratch run against `fsnotify v1.9.0`, and `fswatch` forwards `write`.
`touchDiscoverSignal` runs after `hasSession` reports the session up, so the woken cycle sees the pair.
The `tmux` tier was not run here (refused in this environment); this verdict rests on the code and that scratch check.

**2. `probeIdle`, `logHeld`, `logTyped` (`internal/orchengine/watcher.go`): not a defect.**
A passing probe zeroes `heldWaited`, derives it from `heldSince`, then clears `lastHeld` and `heldSince`, whatever the caller types next.
A later hold with the same reason therefore logs `orch: injection held` again, whether the injection typed after the pass was a reload step or not.
`TestWatcher_HeldInjectionRecordsReasonLogsOncePerChangeWithinAHoldAndTypedLogsWait` drives a non-reload handoff injection between two holds of one reason and asserts two held logs.
Every `typeReloadStep` path is entered after a probe on the same tick, so `heldWaited` is fresh when `logTyped` reads it.
`enter` no longer resets `heldSince` as card 9 states.
The round-1 fix `3b13ad902` removed that reset on purpose, with the hold defined as a run of non-idle probes that only a passing probe ends; the code, the field comment and `doc.go` agree on that definition, so I raise no finding.

**3. `State.Stuck` written by a hold across a restart: not a defect.**
`StuckByHold` is a persisted field (`stuck_by_hold`), set only by `logHeld` and cleared by `probeIdle`, `enter`, `toIdle`, `ResetForFreshLaunch` and `markStuck`'s overwrite.
`probeIdle` clears on `st.StuckByHold`, not on `lastHeld`, so a restart that clears the hold memory in `initCursor` still clears the Stuck text on the first passing probe.
`TestWatcher_TooShortProbeRecordsStuckInIdlePhaseAndPassingProbeClearsIt` restarts the watcher with `e.newWatcher()` while a hold's reason sits in `State.Stuck`, then asserts the passing probe clears it.

## Prior round

Read after this review was saved.
Round 1's F1 to F5 each have a fix commit in the range (`42718768f`, `3b13ad902`, `e591d73df`, `efd281ad7`, `78d92d86a`, `abd062f46`), and the three focus verdicts above re-check F1, F2 and F3 against the current code; none regressed.
Round 1 disputed and deferred nothing.

## Other checks

- Per-card mechanical check: card 5 is the only card with a `Delete` group (`watchdogSignalTick`), and no occurrence of `watchdogSignalTick` or `SignalTick` remains outside `_lyx/` and `.git/`.
- Tests: `lyx gate test ./...` is green.
  `cmd/lyx` carries the test-budget check and the new `daemonwakeups_test.go` guard, and both pass.
- The plan's cards match the diff at the sites read: the discovery wake planner and its signal branch (card 6), the readiness reading and `SessionIdle` (card 8), the resume-context engine, verb and hook command (cards 11, 12), `ChangeDir` (card 10), the seed `started_at` and the spawn order (card 3), the name-repair backoff (card 5), the floors (card 2) and the guard (card 13).
