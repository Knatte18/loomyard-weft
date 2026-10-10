---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/reedcli/watchdog_integration_test.go:291
    summary: BootSignalWakesBackedOffDiscovery cannot tell the discover-signal wake from the backed-off cadence, so the event branch of runWatchdogLoop has no discriminating coverage
  - id: F2
    severity: LOW
    class: design
    location: internal/orchengine/watcher.go:416
    summary: a passing probe leaves the hold memory lastHeld and heldSince set, so a later hold with the same reason after a non-reload injection is not logged
  - id: F3
    severity: LOW
    class: design
    location: internal/orchengine/watcher.go:425
    summary: a Stuck written by a hold outlives a watcher restart, because initCursor clears lastHeld while State.Stuck persists, and no passing probe clears it any more
  - id: F4
    severity: LOW
    class: consistency
    location: docs/benchmarks/daemon-idle-cpu.md:47
    summary: the report states the daemon's own stat tick costs a few seconds per hour, a figure no measurement in the report supports
  - id: F5
    severity: NIT
    class: consistency
    location: internal/shuttleengine/wait.go:115
    summary: defaultPollIntervalMS is read only by a test after the floor change, and the awaitStartup comment still reasons about a 500ms probe
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (no rework round exists; `product.parent` is `main`), commits 1 to 14.
Build state: `lyx gate test` over the changed packages passes untagged and with `--tags integration`.
The `tmux`-tier files (`internal/reedcli/watchdog_integration_test.go`, `internal/reedengine/watchdog_integration_test.go`, `naming_integration_test.go`) were read, not compiled or run, since tmux-tier runs are refused in this session.

Focus questions settled with no finding:

- Cards 4, 5, 6: the watchdog loops block on `reedengine.FileWatchOpener`; `reedcli` production files import neither `fswatch` nor fsnotify; `NextWakeCadence` and `DiscoverSignalFileName` each have one declarer (`internal/reedengine/watchdog.go`) and `hubgeom` and `reedcli` join the name onto `fabricengine.HubScratchDir`.
- Card 2: each interval is floored with one Warn at load and silently in the accessor or constructor; batten's floor sits in `NewInnerRun` on the told duration; `newFixture` defaults to a fake clock and no test config carries a sub-second `PollIntervalMS`.
- Card 3: `WriteSeed` stamps `started_at` and keeps a given or earlier one; `runStartTime` never reads the mtime and treats a missing, unstamped or unparseable seed as undated; every seed writer goes through `WriteSeed`.
- Cards 7, 11, 12: the mark is written only after a successful render, the role file renders before the pointer in every phase but `resuming`, the hook command is `shell.Posix()` with `ChangeDir` to the prime anchor, and `hookDelivered` keys the skip on `State.CompactionBaseline` and the pending text, leaving the typed fallback reachable.
- Cards 8, 9: `Runner.SessionIdle` reads the hook-derived reading and falls back to the pane only when it is unknown or the strand has no run record; `Watcher.probeIdle` remains the one door.
- Card 13: the guard catches sub-second duration literals and `MS`/`Ms` constants in the declared files, rejects a bare annotation, and fails on a ticker outside the list; PATTERN, the background file and the docs land in the same commits as the code.
- Per-card mechanical checks: the one `Delete` group (`watchdogSignalTick`) has no remaining reader; the signature changes (`watchLoop`, `repairNames`, `runWatchdogLoop`, `buildSettings`) have every caller updated, tagged files included.

### [MEDIUM] BootSignalWakesBackedOffDiscovery cannot tell the signal wake from the backed-off cadence

**Location:** `internal/reedcli/watchdog_integration_test.go:291` and `:294`; the code it should prove is the event branch of `runWatchdogLoop` in `internal/reedcli/watchdog.go:485` to `:500`.
**Issue:** Card 6 asks `TestWatchdogDaemon` for a step asserting that a session booted through reed is discovered within one base cadence after its boot, through the signal the boot touches.
The step uses a 100 ms base and a 30 s ceiling, waits for the prime to be entered, sleeps two seconds, boots the pair, then allows `watchdogSignalTestBase+3*time.Second` for the pair to be entered.
With that base the cadence reaches 1.6 s after about 1.6 s and the next unsignalled cycle falls at about 3.2 s after loop start, so the pair is entered by the backed-off cadence alone, roughly one second after a boot at about 2.2 s, well inside the 3.1 s window.
The step passes with the signal wake deleted.
The tier-1 tests cover the planner (`TestDiscoveryWake`) and `consumeDiscoverSignal`, but the glue that turns an event into `onSignal` plus `timer.Reset` has only this step, so that behavior is uncovered in effect.
**Fix:** Make the window discriminate: sleep until the cadence exceeds the window (or raise the base so one backed-off wait is longer than the allowance) and assert the pair is entered within a window shorter than the next scheduled cycle, or assert on a log line that only the signal path emits.
Replace the fixed two-second sleep with a condition on the loop's state if one is observable.

### [LOW] A passing probe leaves the hold memory set

**Location:** `internal/orchengine/watcher.go:416` (`probeIdle`) with `logHeld` at `:440` to `:450` and `logTyped` at `:459`.
**Issue:** The hold memory (`lastHeld`, `heldSince`) is forgotten only by `logTyped`, which only `typeReloadStep` calls, and by `enter` for `heldSince`.
A passing probe clears `State.Stuck` but leaves `lastHeld`, and the handoff request, `/clear` and `/compact` injections never call `logTyped`.
After a hold on a reason that then passes and types one of those injections, the next hold with the same reason in a later phase compares equal to the stale `lastHeld` and its `orch: injection held` line is not written, although the doc says it is logged once per change of reason.
**Fix:** Forget `lastHeld` and `heldSince` when a probe passes, in `probeIdle`, so a hold is a single run of consecutive not-idle probes; `logTyped` then reads the wait from the hold the caller just saw end.

### [LOW] A hold-written Stuck survives a watcher restart

**Location:** `internal/orchengine/watcher.go:425`, `:443` and `:331`.
**Issue:** `State.Stuck` is persisted, `lastHeld` is memory only and `initCursor` clears it.
After a restart with a hold's text still in `Stuck`, `logHeld` sees `Stuck != "" && Stuck != lastHeld` and never rewrites it, and a passing probe's clear needs `st.Stuck == w.lastHeld`, so the stale text stays on `lyx orch status` until the next `enter`.
The previous rule cleared the pane-too-short text on any probe that did not report it, so this is a regression for that reason across a restart.
**Fix:** Let a passing probe clear a `Stuck` equal to any text a hold can write (the probe's reasons are a closed set of strings plus the two constants), or persist nothing a restart cannot own; the doc sentence about a `Stuck` written by anything else keeps its meaning.

### [LOW] Unsupported claim in the idle CPU report

**Location:** `docs/benchmarks/daemon-idle-cpu.md:47`.
**Issue:** The paragraph says the stat tick the daemon itself runs costs a few seconds per hour and so is not the cost.
No row or step in the report measures the stat tick; the daemon's own CPU is 16 s per hour in both rows, and the report also says the three sessions' resize loops may not have reached signal mode.
A measurement report may pin numbers, but this one asserts a number it did not measure.
**Fix:** Drop the sentence or replace it with what the table shows: the daemon's own CPU is unchanged at about 16 s per hour in both rows and nearly all of the total is in children the shim does not see.

### [NIT] Leftovers of the old default in shuttle's wait file

**Location:** `internal/shuttleengine/wait.go:115` and `:482`.
**Issue:** After the floor change `pollInterval` returns `pollFloor`, so `defaultPollIntervalMS` has no production reader and only `TestPollInterval_FloorsNonPositive` references it.
The `awaitStartup` comment still motivates its cadence with a hazard of "probing every 500ms", a value the template can no longer produce.
**Fix:** Keep the constant only if the template-default assertion needs it (orch derives the same check at compile time), and reword the comment to the floor.
