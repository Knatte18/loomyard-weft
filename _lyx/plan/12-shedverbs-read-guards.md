# Card 12 — The loop watchdog and waiter log a repeating read failure once

**Create:**
- `plan:internal/shedverbs#TestLoopReadFailuresLogOncePerEpisode` -> `func TestLoopReadFailuresLogOncePerEpisode(t *testing.T)`

**Edit:**
- `internal/shedverbs#newestActivity` -> `func newestActivity(traceFiles []string, agent func() (time.Time, bool, error), failures *logger.Episode) (time.Time, bool)`
- `internal/shedverbs#watchIdle`
- `internal/shedverbs#takeSnapshot` -> `func takeSnapshot(loop LoopSpec, follow loopFollow, failures *logger.Episode) loopSnapshot`
- `internal/shedverbs#waitLoopEnvelope`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D11 for `internal/shedverbs`: the two sites the audit in `00-overview.md` lists for this package as guard.

`newestActivity` runs on every wake of the idle watchdog's loop in `watchIdle`, and `takeSnapshot` on every poll of `waitLoopEnvelope`; each logged a Warn on every iteration while its read failed.
Each now takes a `*logger.Episode` its caller owns for the length of its loop (a local in `watchIdle` and in `waitLoopEnvelope`):
`newestActivity` calls `failures.Fail` with its `shed: the watchdog could not read the run's agent activity` message and fields on an agent read error, and `failures.Recover("shed: the watchdog reads the run's agent activity again")` on an answer without one;
`takeSnapshot` does the same for `shed: the waiter could not read the loop pid file`, recovering with `shed: the waiter reads the loop pid file again` on a read without an error.
`watchIdle`'s pause-record Warn is already once per run and stays.

`TestLoopReadFailuresLogOncePerEpisode`, in `loopwait_test.go`, is serial (it captures the process-global logger at Debug) with two rows sharing one assertion: three failing calls log one `level=WARN` and two `level=DEBUG` lines of the row's message, and one succeeding call after them logs one `level=INFO` recovery line.
The agent-activity row calls `newestActivity` with no trace files and an `agent` func that errors three times and then answers.
The pid-file row calls `takeSnapshot` with a `LoopSpec` whose lock, envelope and other paths name files under a temp directory and whose `PIDPath` is a directory, three times, then removes the directory and calls it once more, when the read finds no file and returns no error.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/shedverbs` row rises from 83 to 84.

**ImpactSummary:** two unexported helpers re-signed, each with its one caller listed here, all in `internal/shedverbs`; the loop's kill and wait decisions are unchanged.

**Commit:** 12: shedverbs: log a repeating loop read failure once per episode
