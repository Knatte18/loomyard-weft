```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, one concrete input per function or branch.

**Per-function list**

- `goJSONCounter.observeTest`, output → fail, `t.Errorf` case: events `=== RUN`, then `    a_test.go:12: ...`, then `--- FAIL`, then `fail`. Status lines are skipped, the first real line is kept, and `fail` appends it. Correct.
- `observeTest`, panic case: `panic: boom` arrives as an output event on the test, and `fail` records it as `FirstLine`. Correct, and the panic row in `counters_test.go` pins it.
- `observeTest`, empty or whitespace output: `line == ""` returns without storing. Correct.
- `observeTest`, output after `fail` or `pass` for the same key: the `delete` has already run, so the late line re-inserts `firstLines[key]` and nothing removes it. Nothing reads that entry again, so no result is wrong; it is a small leak, and a re-run of the same name would see a stale first line. Not a defect under go test's usual ordering.
- `observeTest`, a test with no output besides status lines: `FirstLine` is `""`. Correct, and the `TestB` row pins it.
- `observeTest`, `pass` or `skip`: deletes the key. Correct.
- `observeTest` and `handle`, lock: `handle` runs under `lines.mu`, so the `firstLines` and `failures` writes are guarded. Correct.
- `Failures()`, parent and subtest of the same prefix: `TestA` and `TestA/sub` both failed, so `TestA` is dropped and `TestA/sub` kept. Correct.
- `Failures()`, `TestA` and `TestAB` as siblings: the prefix test uses `Test+"/"`, so `TestAB` does not drop `TestA`. Correct.
- `Failures()`, parent failed alone (subtests passed): the parent is kept. Correct.
- `Failures()`, parent that failed with its own error line plus a failed subtest: the parent's own error is lost. This matches D1's "leaves only", so not a defect.
- `Failures()`, no failures: returns nil, so `omitempty` drops the key. Correct.
- `Failures()`, aliasing: it builds a fresh slice under the lock, so a later event cannot mutate what a caller holds. Correct.
- `Failures()`, cost: the parent check is quadratic in failures. It runs once per tier, and only a mass failure makes it noticeable. LOW at most and not worth a finding.
- `vstestCounter.Failures`: returns nil. Correct.
- `tierNotice`, green tier: returns the summary line only, unchanged from before. Correct.
- `tierNotice`, failed tier with no failures (build break, VSTest): the summary plus `log <path>`. Correct.
- `tierNotice`, exactly 10 failures: no "and more" line. Correct.
- `tierNotice`, 11 failures: 10 lines, then `and 1 more`, then the log. Correct, and the 13-failure test pins the tail.
- `tierNotice`, empty `FirstLine`: the line is `pkg test` with no trailing colon. Correct.
- `Ended`: `Failures: ended.Failures` shares the slice with the run record, but nothing mutates it afterwards. Correct.
- `CheckTier` JSON: `failures,omitempty` means old records still read and green tiers write no key. Correct.
- `runTier`: `record.Failures = counter.Failures()` runs after the counter is flushed, assuming `counts` and `Launches` are read at the same point. Correct.
- `advanceUntilReturned`: forwards the result and closes `returned`, then `AdvanceByWaitsWhere` steps until `done` closes. If the verb never returns, `AdvanceByWaitsWhere` gives up after 10 s of real time and `awaitRun(forwarded)` then fails on its own timeout, so it neither hangs nor loops unbounded. Correct.
- `AdvanceByWaitsWhere`, `stepped` starts at 0, so waits recorded earlier by other verbs' workers cause extra advances. Those advances are by a wait duration and fire only timers that were already due. The TTL step needs 61 s of fake time and gets it from repeated probe-wait steps. Correct.
- EXTERNAL removed-receiver step: `SetStrands` is called after `awaitLogged` and before stepping, so the sweep sees the receiver gone on its next step. This follows the earlier ordering. Correct.
- Hook-wait restart subtest, supersede sync: two waiters of the same strand run, whichever registers second supersedes the first, and the first to answer must be `OutcomeSuperseded`. That proves the other is registered before `first.stop()`. Correct.
- Hook-wait restart subtest, `nil` spawn: `connect` returns `connLostError` when `spawn == nil` (`client.go:76`), and a hook-wait resumes by resending (`client.go:172`). After the stop, the survivor therefore waits on the client clock, which `clientClock.AwaitWaits(1)` expects, and no daemon can be started behind the test. Correct.
- Hook-wait restart subtest, superseded waiter: it has already returned, so it records no client-clock wait, and `AwaitWaits(1)` counts only the survivor. Correct.

No findings.

**Tool calls**
1. `grep -n "func (c \*Clock) AdvanceByWaitsWhere"` in `internal`: my context did not hold its body or its 10 s deadline.
2. `grep -n "spawn\|Spawn" internal/creelengine/client.go`: to see how a client with a nil spawn behaves.
3. `sed -n 60,80p` and a grep for hook-wait in `client.go`: to confirm that `connect` returns `connLostError` for a nil spawn and that a hook-wait resumes by resending.
