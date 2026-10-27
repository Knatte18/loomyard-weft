```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness review of the code the diff changes.

**Per-function list (case run → verdict)**
- `AppendLog` / `RotatePastBound`: absent file, absent directory, size at the bound → correct. A missing directory returns the open error and creates nothing.
- `ReadsAsReexec`: missing directory, absent record, record without ticks, own ticks unreadable → correct.
- `Run` / `daemon.run`:
  - State read fails → line only, no post. Correct.
  - Fresh start refused → no post. Correct.
  - Refused re-exec → second state read decides. Correct.
  - `Locked` error → lock released by the deferred release. Correct.
  - Fatal error after a re-exec → line and post. Correct.
- `recordStart`: same pid and ticks, same pid with no ticks, differing ticks, no record → correct.
- `worktreeGone` and the loop: prime never gone, git link missing, a tick that ends with the link gone does not overwrite `keptTidy` → correct. The cancelled-context break skips the second check, which is harmless.
- `exitWorktreeGone`: nil `keptTidy`, nil prime shuttle, failed stop → correct.
- `logRunChanges` and `heartbeat`: see F1; otherwise correct.
- `passExited` episode lines and the tick and owner-read episodes → correct.
- `countLandings`: baseline branch, tidy in flight skips the emptying, flag set once → correct. The landings read happens before the update, so a failed read leaves the flag unset.
- `DobbyLandings` / `machineLandings`: `machinename.Name` never returns an empty name (`internal/machinename`, `FromHostname` errors on empty), so the pre-machine entries are never counted by an empty-equals-empty match. Correct.
- Shed `Machine` stamping: every non-goto history append in `internal/shedengine` is stamped (`entry.go:51`, `run.go:337`, `reroute.go:45,55,66`). The two goto appends are unstamped, as the field's doc comment says. Correct.
- `runDobbyWith`, `traceRefusedStart`, `traceFailedWiring`, `reopenStdio`, `rotateOutput`: ordering, no-directory cases and the notices → correct.
- `SpawnDetached` / `rotateOutBeforeSpawn`: Windows only, skipped when the lock reads live → correct.
- `CheckWith`: callbacks fire only after the two-sighting rule and the failed-identity rule → correct.
- `runCheck`: lock held, reconcile failure, a failing tier, a failure with no tier ended, a green run, a failed post → correct.
- `ReadCheck`, `ReadCheckView`, `checkAlive`, `ensureCheck`: absent record, undecodable record, reused pid → correct.
- `gatecli.RunGate`: `runTest` sets `busyQuit` and `watching` itself (`run.go:68,131`), so the zero fields in the request are not a gap. Correct.
- `SeedAdoptGate` / `ReconcileHubWideAt`: no gates mapping, no finalize, adopt already present, second pass, absent file seeded from prime or legacy → correct. The one exception is F2.

**Findings**

- **F1 — LOW, design.** `internal/dobbyengine/passes.go`, `heartbeat`. `lastHeartbeat` is set before `Runs()` is read, and `first` is derived from it. If the start heartbeat's `Runs()` read fails, `loggedRuns` is never primed. After a re-exec, the next `logRunChanges` then logs every listed run as "first seen", a spurious change line. Fix: set `lastHeartbeat` only after a successful read, or track priming in its own flag.
- **F2 — LOW, design.** `internal/gateslot/config.go`, `SeedAdoptGate`, with the sparse strip in `internal/configsync/configsync.go` after the `MigrateHubBase` call. The strip drops per-name entries equal to the template on every pass. A hub that sets `adopt` to the template value gets it stripped, and since `finalize` is still tuned and `adopt` is absent, the next reconcile reseeds it from `finalize`. The decision's "set it through `lym config gate`" way back to the template's `adopt` therefore does not hold for that value. I did not read the strip code, so this follows from the diff context and the decision text. Fix: skip the seed when the file is already marked as having had `adopt` stripped, or document that the template value cannot be pinned.
- **F3 — LOW, design.** `internal/dobbyengine/loop.go`, `readKeptTidy`. A state record that does not decode returns `nil, nil`, so `rememberKeptTidy` replaces a previously kept tidy with nil. A beam worktree removed while the record is undecodable then stops no tidy agent. The Warn also repeats on every tick rather than once per episode. Fix: on `state.ErrDecode`, return the earlier `keptTidy` and log through the `stateRead` episode.

**Tool calls**
1. `grep` of `watching`/`busyQuit` in `gatecli/cli.go`, plus `HistoryEntry{` in `shedengine` and `machinename.Name`: to check `RunGate`'s request defaults, history stamping and the empty-name case, which my context lacked.
2. `grep` of `watching:`/`busyQuit` assignments in `internal/gatecli`: to find where the watchdog fields are set, which settled that `RunGate` is fine.
