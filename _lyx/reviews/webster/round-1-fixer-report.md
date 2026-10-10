# Fixer report — webster round 1

Every finding was fixed; nothing was disputed or deferred.

## Fixed

- **F1** (MEDIUM, `42718768f`): `BootSignalWakesBackedOffDiscovery` in `internal/reedcli/watchdog_integration_test.go` now records when the prime was entered, sleeps 70 bases so the boot falls after the 63-base unsignalled cycle, and requires the pair to be entered before 120 bases after that entry.
  The next unsignalled cycle falls 127 bases after the entry, so only the discover signal's wake can pass the step.
  The deadline is anchored on the prime's entry rather than on the boot's return, so the boot's own duration cannot widen the window.
  The step's doc sentence now says this.
  The file is `tmux`-tier: `go vet -tags tmux ./internal/reedcli` compiles it, but the step was not run, because tmux-tier runs are refused in this session.
- **F2** (LOW, `3b13ad902`): in `internal/orchengine/watcher.go`, a passing probe now ends the hold.
  It records the hold's length in a new memory-only `heldWaited` and forgets `lastHeld` and `heldSince`.
  `logTyped` reads `heldWaited` instead of computing the wait from `heldSince`.
  `enter` no longer resets `heldSince`, because every `enter` follows a passing probe.
  The package doc defines a hold as a run of probes that are not idle, ended by the next passing probe.
  For the regression, the existing log test (renamed `TestWatcher_HeldInjectionRecordsReasonLogsOncePerChangeWithinAHoldAndTypedLogsWait`) now holds before the `/clear` is typed and again in clearing with the same reason, and asserts two held lines.
  Before the fix the test failed: it logged one line where two were expected.
- **F3** (LOW, `e591d73df`): `State` gains a persisted `StuckByHold`.
  `logHeld` sets it when it writes `Stuck`, and `markStuck`, `enter`, `toIdle` and `ResetForFreshLaunch` clear it.
  A passing probe clears `Stuck` when `StuckByHold` is set, so ownership survives a restart.
  `logHeld` likewise overwrites a hold-written `Stuck` after a restart.
  The probe's reasons are not a closed set (draft text, `%s` state names), so a flag was chosen over a list of known reason strings.
  `TestWatcher_PassingProbeLeavesOtherStuckReasonsAlone` still passes, since a foreign `Stuck` carries no flag.
  For the regression, `TestWatcher_TooShortProbeRecordsStuckInIdlePhaseAndPassingProbeClearsIt` now restarts the watcher before the passing probe.
- **F4** (LOW, `efd281ad7`): in `docs/benchmarks/daemon-idle-cpu.md`, the unmeasured stat-tick sentence is replaced with what the table shows: the daemon's own CPU is unchanged at about 16 s per hour in both rows.
- **F5** (NIT, `78d92d86a`, `abd062f46`): `defaultPollIntervalMS` and its assertion in `TestPollInterval_FloorsNonPositive` are deleted.
  The config tests already pin the template default at 1000, which is the floor.
  The `awaitStartup` comment no longer cites 500ms; it says "probing at every poll", and the reworded sentence now breaks semantically so that `lyx loom lint-comments` passes.
  The same stale `500ms here` comment in `startup_test.go`, whose config had moved to 1000, now reads `1s here`.

## Verification

- `lyx gate test` passes, untagged and with `--tags integration`, for `./internal/orchengine`, `./internal/shuttleengine` and `./internal/reedcli`.
- `./cmd/lyx` passes untagged.
- `lyx loom lint-comments --base a353b4d02` reports ok.

## Disputed

None.

## Deferred

None.
F1's step was not run live, because the tmux tier is refused here; the change compiles under `-tags tmux`.
