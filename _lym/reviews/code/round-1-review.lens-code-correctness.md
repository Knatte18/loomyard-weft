```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. No defects found, apart from two LOW weaknesses.

**List: function or branch, case run, result**

- `gitexec.armDeadline` (production default): deadline 10 min gives `context.WithTimeout(Background, d)`. Correct and unchanged.
- `gitexec.armDeadline` (`SetDeadlineForTest`): `ready` fires `cancel()`, so `ctx.Err()` is `Canceled`. `runAttempt` tests `ctx.Err() != nil` at `gitexec.go:348`, so it still returns `GitError{Timeout: deadline}`. Correct.
- `SetDeadlineForTest`: the helper goroutine exits on `ctx.Done()`, and `runAttempt` defers `cancel` at `gitexec.go:310`. No leak. The cleanup restores `armDeadline`. Correct.
- `SetDeadlineForTest` on a local command: no deadline is armed, `armed()` returns empty. Correct.
- `RunWithin` row: `os.Remove(pidFile)` fails the test if no earlier row wrote the file. The earlier "hung remote killed" row always writes it. Correct, but the row now depends on that order.
- `termwindow.attachLineWaiting`, polls=30 and interval=1s: it renders `sleep 1` and "waiting up to 30s", the same as before. Correct.
- `termwindow.attachLineWaiting`, 50ms: it renders `sleep 0.05` through `FormatFloat('f', -1)`, with no exponent form. Correct.
- `lymbin.compile`: `MkdirAll` on a path under a regular file returns an `*fs.PathError`, wrapped with `%w`, before any build. `errors.As` in the test matches. An existing directory is a no-op. Correct.
- `verifyrun` `expiringContext`: `Err()` returns `DeadlineExceeded` after `done` closes. `Cause` falls back to `Err()`. The pid-file wait is bounded at 10 s, so a missing pid file fails the test and does not hang it. Correct.
- `verifyrun` `TestRun_Cancellation`: `firstWrite` closes `seen` once and the cancel fires on the child's first output. Correct. Two weaknesses remain, below.
- `impactset.isWatchLine`: `//lym:watch`, `//lym:watch x`, `//lym:watchful` give true, true, false. Correct.
- `impactset.watchedTestsInFile`, no pattern or malformed pattern: `WatchMalformed` is returned with the marker's line. Correct.
- `impactset.watchedTestsInFile`, marker above a non-test func, on a receiver, or followed by prose: `GuardMisplaced`. Correct.
- `impactset.UnwatchedTests`, nil watched or all matched: returns nil. Correct.
- `impactset.UnwatchedTests`, own file changed: the test is kept. Correct.
- `WatchedTest.watches`: a pattern with `/` matches the full path, any other the base name. Correct.
- `impactset.targetsFor`: guards in the set are untouched, and unwatched guards outside the set are dropped. `Unwatched` lists only packages inside the set. Correct.
- `gatecli.computeScope`, card form: targets such as `./...` or a not-yet-created package fail `os.Stat` and are left out of `dirs`. Nothing is skipped, which is the safe direction. Correct.
- `gatecli.cardChanges`: with no parent branch or an unreadable diff it returns false, so `changesRead` stays false and nothing is skipped. Correct.
- `gatecli.scanGuards`: `FindWatchedTests` keys are relative to `moduleDir`, and `moduleRelative` makes the changed paths module-relative too. For `.` it is the identity. Consistent.
- `gatecli.planTierRuns`, skip: the conditions are not whole, skip names present, invocations present, `{args}` in the command, and no filter flag in the call's args. The green entry stores `run.args` including the `-skip`, at `green.go:447`. Such an entry matches a later request only on exact args, so it cannot cover a whole run. An empty-args entry does cover a skip request. Correct.
- `gatecli.holdsTierTest`, skipped names: a package whose only tier test is skipped gives no empty-tier failure. Correct.
- `detachSeams.frontPoll` and `leaseRead`: zero or negative falls back to the production second, so `NewTicker(period)` never gets a non-positive period. Production defaults unchanged. Correct.
- `gatecli.awaitRelease`: an empty path returns at once, so a role with no release env is unchanged. Correct.

**Findings**

1. LOW, `design`, `internal/gatecli/cli.go:240`. The `GuardMisplaced` refusal now appends `%v` of `err`. A watch misplacement error already carries its own "way forward: move the `//lym:watch` line…" text (`watches.go:1461`), so the refusal shows two way-forward clauses. This is cosmetic. The fix is to print the location only, or drop `%v` for the watch case.
2. LOW, `design`, `internal/verifyrun/run_integration_test.go:~172`. `TestRun_Cancellation` now cancels on the command's first output, which may arrive before `sleep 30` has forked. The old 300 ms pause let the child tree exist. The test still pins the error and the return time, but no longer pins that a running child gets killed. Fix: echo `up` after the sleep has started, for example `sleep 30 & echo up; wait`.

**Tool calls**

- Read `gitexec.go:291-360`. The context lacked how `runAttempt` classifies a cancelled context.
- `grep greenEntry{` in `internal/gatecli`. The context lacked which args the green record stores for a skipping run.
- Read `targets.go:52-92`. The context lacked which `dirs` `DeriveTargets` passes to `FindWatchedTests`.
