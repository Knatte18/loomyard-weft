```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. Cases run per function or branch:

- `selfexec.ResolveLymBinary`, correct for all four cases I ran:
  - Not under test: returns `(path, err)` unchanged.
  - Under test with a lookup error: the error is returned.
  - Under test with a path equal to `os.Executable()`: `ErrTestBinary`.
  - Under test with a stand-in path: returned as is.
- `selfexec.ResolveLymBinary` with a symlink to the test binary as the stand-in: passes, because the comparison is by path. `os.Executable` resolves `/proc/self/exe`, so a symlinked stand-in differs from `self`. This only matters for a test that injects such a stand-in, so I rate it harmless.
- `selfexec.LymBinary` under test: refuses, correct.
- `tmuxkit.refusesReexec`, correct for all four inputs I ran:
  - `nil` and `-test.run=X`: not refused.
  - `-test.run Foo`: not refused, since only the first word is read.
  - `slack bar`: refused, with both words in the message.
- `proc.PIDsWithEnv` (Linux), correct:
  - Matches the whole `NAME=value` field, so `entry+"0"` does not match.
  - A non-numeric `/proc` entry is skipped.
  - An unreadable or vanished `environ` is skipped.
  - A zombie has an empty `environ` and is skipped.
  - An unreadable `/proc` returns an error.
  - The Windows counterpart returns `errors.ErrUnsupported`, so the two platforms agree.
- `processCap.run`, correct for all five cases I ran:
  - Count at the limit: nothing killed.
  - Count over the limit: the line is set once, every listed pid is killed, and later lists are killed whole.
  - A list error: skipped with a Debug log.
  - `ErrUnsupported`: returns after one Info log.
  - `ctx` cancelled: returns the line.
- `SIGKILL` is not in `crashSignals` (`tierrun.go:194`) and `failedOnShortage` excludes `processCapped`, so a killed run is neither reran as a crash nor as a shortage.
- `runInvocations`, correct. `capLine` is written in the goroutine and read only after `stopWatching(); watchdogDone.Wait()` (`tierrun.go:331-332`), so there is no race. A tripped cap forces `exit` to 1 in `runTier`. `runMarker` returns "" when `env` carries no marker, so the cap is then simply not started.
- Findings below.

Findings:

1. **LOW, `design`.** `tmuxkit.Main` refuses before `lymtemp.Dir()` and before cleanup, but a package's `TestMain` runs `HermeticGitEnv` and its own setup first (`internal/gitkit/hermetic.go:17-27`, `tmuxkit.go` `Main`).
   - The removed `refuseCLIReexec` ran at the top of `HermeticGitEnv`, before the neutral git config and its run directory were made.
   - Now each refused re-exec can create a `lymtemp` run directory and git config, then `os.Exit(2)` skips the removal that `Main` does after the run.
   - In a runaway loop that leaves one leaked temp directory per refused process.
   - Evidence: `internal/gitkit/hermetic.go:17-27` (documents `HermeticGitEnv` as the first line of `TestMain` and `Main` as removing the directory), `internal/testkit/tmuxkit/reexecguard.go:28-36`, and the added `refusesReexec` call at the top of `Main` in `tmuxkit.go`.

2. **LOW, `design`, not verified.** The cap's marker is the gate verb's own pid (`gatecli/run.go:151`), so it is shared by every tier and invocation of one gate run.
   - `processCap.run` counts and kills every process carrying that marker, not only the capped tier's.
   - If tiers run concurrently, one tier's runaway would SIGKILL the other tiers' processes. Those tiers would then report plain failures, with no cap line and `processCapped` unset, so they could be classified as a shortage or crash and rerun.
   - Evidence: `internal/gatecli/processcap.go:48-84` (kills the whole list) and `internal/gatecli/tierrun.go:313-319` (the cap starts per tier command).
   - I did not confirm that tiers run concurrently within one gate run.

3. **LOW, `design`.** `loomCLI.loopExecutable` returns "" with only a Warn when the lookup fails, and `loopSpecFor` still builds a `LoopSpec` with an empty `Executable`. Under `go test` with `lymBinary` unset (the default), every loop spec therefore gets "", where it used to get the test binary's path.
   - This is safe, and it matches the old behaviour on a lookup error. A `LoopSpec` with an empty executable then fails or no-ops at the child spawn instead of at wiring.
   - Evidence: `internal/loomcli/looparm.go:71-82`.

Tool calls (3 of 6 used; the first two are inside the single command):
1. `sed -n 296,345p internal/gatecli/tierrun.go`, plus a `grep` over `internal/gatecli` for `crash` and `samplePeriod`, plus `SignalProcess` in `internal/proc`: my context lacked the `runInvocations` body (the `Wait` and read order) and `SignalProcess`'s semantics.
2. `sed -n 188,262p internal/gatecli/tierrun.go`, plus a `grep` for `Parallel` and `go func` in `internal/gatecli/run.go`: my context lacked `crashSignals`, `runnerCrash` and `retryTier`, and the concurrency of tiers.
