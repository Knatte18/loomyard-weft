```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

| Behaviour | Pinning test |
|---|---|
| Git start retried on EAGAIN, up to four tries; a start that keeps failing returns the EAGAIN error unwrapped | `TestRun_StartRefusedForEAGAIN` (`RefusedThenStarted`, `RefusedEveryTry`) |
| A non-EAGAIN start failure, or a git that exits non-zero, is never retried | same test (`OtherStartFailure`, `GitExitsNonzero`) |
| Each refusal noted in proc's record | gitexec's test asserts `LastSpawnAgain` after the rows; the `shedtransient` died row also notes one |
| Doubling backoff (250 ms, 500 ms, 1 s) and the Warn log | none (see finding 1) |
| Retry on the remote-subcommand path of `runCoreWithin` | none (see finding 1) |
| `proc.IsSpawnAgain`, `NoteSpawnAgain`, `LastSpawnAgain` | only indirect, through gitexec and shedtransient; the proc package has no test of its own (see finding 2) |
| Reed's tmux retry uses the shared predicate and notes each refusal | no new test; no existing reed test was found (the grep matched none) |
| `shedtransient`: EAGAIN gives `host-resources` | `TestClassAndMark` row "spawn refused for EAGAIN" |
| `shedtransient`: died agent gives `host-resources` only after a noted refusal | `TestClassAndMark` rows "agent died with no spawn refusal noted" and "agent died after a spawn refusal" |
| `shedtransient`: died agent after a refusal older than two minutes stays a hard failure | none (see finding 3) |
| Existing `shedtransient` rows unchanged | the converted old rows in `TestClassAndMark` |
| Seats adapter wraps `ErrAgentDied` only for a died chair | `TestMultiLLMProducer_MapsTheLeadResult` (`wantDied` true on "died", false on "timeout" and "never came up") |
| Cause carries the command and stderr excerpt | `TestLoop_InterruptedChildFillsTheErrorForm` ("exited: exit status 3; command: lym-fake" and the crash subtest) |
| Go crash's first lines picked over a trailing stack | crash subtest "child crashes with no trace after moving the run on" |
| Tail lines when stderr holds no Go crash; absent file gives empty | `TestStderrExcerpt` |
| Trace file falls back to captured stderr | crash subtest (`env["trace_file"] == stderrPath`) |
| Empty or missing stderr gives no trace file; `StopFiles == nil` | none (see finding 4) |
| Blame the status file's producer read after the child exited | crash subtest (hook gets "B", status blames B) |
| Fallback to the pre-exit producer when the status is unreadable after the child | none (see finding 4) |
| Failed-stop write before the interrupt hook | crash subtest (`hookState == failed`) |
| Interrupted stop over a paused run keeps it paused; loop envelope carries `pause_reason` | subtest "child dies before its resume write over a paused run"; key set in `assertLoopKeys` and the `shedcli` key list |
| `pause_reason` populated in `loopExitStop` (`loopwait.go`) | key presence only; the value is not asserted (see finding 5) |
| Driver takes `pause_reason` only when the reason is empty | `TestParseDriveStop` (both new rows); absent-key tolerance is covered by the existing rows |
| Driver ignores `pause_reason` when the loop state is not paused | none (see finding 6) |
| Hook records the status file's state, links a child-recorded error, and dedupes an error that already contains the cause | `TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection` (three rows) |
| Hook falls back to `failed` when the status file is unreadable or absent | none (see finding 7) |
| `ClassifyHalt` gives `interrupted-stop` for a paused interrupted halt | `TestClassifyHalt` row "PausedInterrupted_InterruptedStop" |

## Findings

1. **LOW, design.** `internal/gitexec/startretry_test.go:41`, `internal/gitexec/export_test.go` (`SetStartFailuresForTest`). The test sets the first delay to 0, so the doubling schedule can't fail, and it only runs a local subcommand (`--version`), so the retry on the remote path in `runCoreWithin` (`gitexec.go`, `runSpawning` called with `remote=true`) is also unpinned. A bug in `delay *= 2`, or in either call site, would pass. Fix: add a row that records the waits, or a remote-subcommand row.
2. **LOW, design.** `internal/proc/spawnagain.go:14-37`. There is no `spawnagain_test.go`. The predicate (a wrapped EAGAIN matches, other errors don't), `NoteSpawnAgain` returning false for nil, and the zero state of `LastSpawnAgain` are exercised only through other packages. A short table test in `proc` would pin them directly.
3. **LOW, design.** `internal/shedtransient/shedtransient.go:44` (`spawnRefusedRecently`, the `<= spawnRefusalWindow` bound). No row has a refusal older than two minutes. The function takes `now`, so the expiry is testable without a clock. A window that was dropped or widened would pass every current row. This is the decision record's "only within the window" bound.
4. **LOW, design.** `internal/shedverbs/loop.go:296-298` and `interruptedTraceFile`.
   - Fallback to the pre-exit producer when `readStatusFile` finds nothing after the child exited: no test.
   - Empty stderr file, or `StopFiles == nil`, giving no trace file: no test.
   
   The acceptance criteria state the blame and trace-file rules only for the positive cases. These are the empty and unreadable paths the focus asked about.
5. **LOW, design.** `internal/shedverbs/loopwait.go:495` (`PauseReason: after.PauseReason` in `loopExitStop`). The closed-key-set assertions only check that `pause_reason` is present. No test checks its value on the loop-exit stop, so deleting that line would pass.
6. **NIT, design.** `internal/sheddriver/driverules.go:87`. The `decoded.Loop.State == paused` guard has no row where the loop state is not paused but `pause_reason` is set and `reason` is empty. The guard is the behaviour "take it only for a paused stop".
7. **LOW, design.** `internal/loomcli/looparm.go` (`loomAfterInterrupt` default `State: failed` when `ReadJSON` errors or the file is not found). All three hook rows seed a readable status, so the fallback path is untested.

Reed has no new test for the shared predicate and the noting. The grep for `retryExecAgain` and `LastSpawnAgain` in test files matched only `startretry_test.go`. I found no existing reed retry test either. This is partly a regression gap in code that moved, so I'm not rating it separately. Add a reed row only if one exists to extend.

Tool calls:
1. `git diff` of the test files (`loop_test.go`, `loop_helper_test.go`, `driverules_test.go`, `halt_test.go`, `anomaly_test.go`, `multillm_test.go`, `cli_test.go`): my context lacked the test-file diffs.
2. `grep` for `pause_reason`/`PauseReason` in the shedverbs tests, and for `retryExecAgain`/`LastSpawnAgain` in all test files: to check coverage of `loopExitStop` and reed's retry.
