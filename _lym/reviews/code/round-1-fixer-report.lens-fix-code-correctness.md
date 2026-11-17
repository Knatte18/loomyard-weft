Lens fix-code-correctness: I ran each added or changed branch through a concrete input and checked the callers of each changed signature.

**Findings**

1. **MEDIUM** — `internal/beamcli/cli.go` (`startCmd` flag, `start(..., noPrimeClaim)`, `BeamUpOptions{NoPrimeClaim: noPrimeClaim}`).
   - Summary: no test covers the link from the `--no-prime-claim` flag to `BeamUpOptions.NoPrimeClaim`, so F1's end-to-end premise is unpinned.
   - Evidence:
     - The grep shows `NoPrimeClaim` and the flag only in `cli.go`, `orchcli` (tests included) and `machinecli`. No `beamcli` test mentions either.
     - Dropping the flag registration, or the `NoPrimeClaim: noPrimeClaim` argument at `cli.go:532`, leaves every test green.
     - The `machinecli` rows only assert the argument string, using a fake `beamStart`.
     - The `orchcli` row only asserts `BeamUp` honours the option.
   - With the defect in place, the real `lym beam start <slug> --no-prime-claim` child would claim the prime again, which is the original F1 behaviour.
   - The review's fix text asked for a `beamcli` or `orchcli` row. The `orchcli` row is present, but nothing covers the CLI hop.

**Checked, no finding**
- **F11:** in `Add` and `Remove`, `held, err :=` redeclares correctly. Nothing acquires the lock twice (`Shortnames` takes none, and `removeGoneLaunchers` calls `Remove` one hub at a time), so there is no self-deadlock. The lock path `<state>/lym/<hash8>/launchers.lock` has no `_lym` segment, so `refuseTrackedTree` does not refuse it.
- **F10:** the dollar-sign, backquote and backslash rows would have been accepted before the fix, so they fail with the defect in place. Single-quote paths are no longer refused. Inside the double quotes `\'` stays literal, so that path is still correct.
- **F5:** with an `addLauncher` that ignored the owner, the OwnedElsewhere and NoRecordedOwner rows would leave the launcher standing, since the board's prime is owned here. The new beam-under-foreign-prime row fails if the start never adds the launcher.
- **F3, F2:** an entry with outcome `failed` or `refused` becomes `startFailed`, and the exit code is 1 through `hub.failed()`. Its warnings are prefixed with the worktree. The summary lines are asserted through `output.Summary` for both the hold and the attach command. `Worktrees` tagged `json:"-"` stays populated in-process, so `failed()` works.
- **F4 and the F6 rows:** the expected strings match what the code produces. The no-JSON-line case gives the Go error "unexpected end of JSON input" from `json.Unmarshal(nil, ...)`.
- **Changed signatures:** `prepare(report, claimPrime)` has one caller. `freedesktopDesktopDir` has one caller. `start(...)` in `beamcli` has one caller. All compile, and tier0 and tier1 pass.

**Tool calls**
- One call: a repo-wide grep for `no-prime-claim|NoPrimeClaim` outside `machinecli` and `orchcli`, to confirm no `beamcli` test covers the flag wiring.
