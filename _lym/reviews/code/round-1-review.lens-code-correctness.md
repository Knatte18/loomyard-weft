```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the `0fdf98d3` diff. Each item below is the case I ran and whether it is correct.

**List**
- `startStallTimer` default, `gitexec.go:55`: **correct.** `time.AfterFunc(bound, check).Stop` is the old `stallTimer.Stop` as a method value, and `defer stopStallTimer()` discards the same bool the old `defer stallTimer.Stop()` did. Production behavior is unchanged. The doc comment's claim that `tracePath` goes unused holds, because the body never reads it.
- `runAttempt` call site, `gitexec.go:299-308`: **correct.** The check closure is untouched. `sawStall.Store(true)` precedes `cancel()`, and `sawStall` is read only after `runCommand` returns. A late `check()` after the attempt has ended is harmless.
- `SetStallCheckForTest`, one armed attempt where `ready` becomes true: **correct.**
  - The goroutine polls every 5 ms and calls `check()` once, then returns.
  - Stop closes the channel through `sync.Once`, so a stop after the check already ran is safe.
  - Cleanup restores `previous`.
  - `bounds` is guarded by the mutex, and `armed()` returns a copy.
- Same function, `ready` never true: **no goroutine leak and no hang.** `defer stopStallTimer()` closes `stopped` when the attempt returns, so the poll goroutine exits when the attempt ends. The attempt itself runs until the deadline, and then reports a non-stalled timeout `GitError`. The row fails with a clear message instead of hanging.
- Silent row, attempt 1: **correct.**
  - `silent.count` starts empty, because `scriptDir` is per run and `remoteAttempts` is the first user of the `silent` name.
  - `ready(1)` fires once the script has appended `run`.
  - The trace file exists from `runAttempt`.
  - git writes no trace line before the remote's first packet, so `os.Stat` size 0 holds and the check kills the attempt.
- Silent row, retry attempt 2: **correct on the normal path.**
  - `bounds` has two entries, so `attempt` is 2.
  - `ready` needs two `run` lines, so it fires only after the second script start.
  - The first attempt's script was group-killed and cannot add a line.
  - The result is a `Stalled` `GitError` with `Timeout == stall`, as asserted.
- Silent row, deadline: **correct.** The row sets `10*deadline`, and the `t.Cleanup` of `SetRemoteDeadlineForTest` restores the previous 1 s for the next rows. `stallBound` 200 ms is still under the 10 s deadline, so the check stays armed.
- Unarmed row: **correct.** `stallBound == deadline` fails `stallBound < deadline`, so the seam is never called and `armed()` is empty. The `ready` that returns true is never invoked.
- Answering row: **correct for mutants.** `ready` fires only once the trace is non-empty, so the production check evaluates `Size() == 0` as false and does not cancel. If the size-0 condition were dropped, `os.Stat` would succeed, the check would cancel, and the row would report a stall and fail. It does not pass trivially. It does not pin the production check for a trace that is empty at the bound but becomes non-empty later. That case could not be tested deterministically before the change either.

**Findings**

1. **LOW / design** — `internal/gitexec/export_test.go:22-26`, `gitexec_unix_test.go:219-222`. The silent row's readiness `count >= attempt` assumes one `runAttempt` per attempt number.
   - `runSpawning` calls `runAttempt` again when git cannot start for EAGAIN, and each call arms the seam and advances `attempt`. Evidence: `gitexec.go:258-266`.
   - In that case the script has run fewer times than `attempt`, so `ready` stays false until the 10 s deadline and the row fails.
   - EAGAIN is the host-pressure condition this fix targets, so the residual flake is narrow.
   - Fix: key readiness on something other than the call count, or keep the cost and document the assumption.

2. **LOW / design** — `gitexec_unix_test.go:283-286`, `gitexec_unix_test.go:291-296`. The answering row asserts only `armed() == [stall]`, not that the check actually ran.
   - If git never writes the trace, `ready` stays false for the whole 1 s deadline, the kill is a plain deadline timeout, and every assertion still passes. The row then proves nothing about sparing an answering remote.
   - Fix: have `ready` record that it fired and assert that.

**Tool calls:** none. The manifest context covered everything.
