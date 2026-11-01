```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2 as the fixer carried them out.

Both findings were fixed as proposed, so there is no list to open with.

**1. MEDIUM, F1: the merge case in `watchfilechanges_test.go` passes with the defect in place.**
- Changed text: `TheChannelBuffersOneValueSoABurstOfChangesMerges` asserts only `cap(changes) == 1`.
- The review asked that "two writes before a read merge into one value".
- Case: replace the non-blocking `select { case changed <- struct{}{}: default: }` in `watchFileChanges` (`landingdeps.go:384-388`) with a blocking send. The channel keeps capacity 1, so the assertion still passes.
- The blocking send would stall the forwarding goroutine and back up `fswatch`'s unbuffered events channel, which is exactly the defect the merge rule exists to prevent.
- The row pins the buffer size, not the merging. A deterministic version needs a seam, for example the fixer feeding a fake events source. If that is too costly, rename the row to say it pins only the buffer size.

**2. LOW, F1: the absence assertions are timing-based and the subtest names claim more than they prove.**
- Changed text: `assertNoChange` with `watchQuiet` (200ms) in `AWriteToASiblingFileDeliversNothing` and `StopEndsTheWatchAndMayBeCalledAgain`.
- A slow fsnotify delivery can outlast 200ms, so a broken base-name filter or a watch that is not really stopped could pass.
- The failure direction is a false pass, never a flake, which is acceptable. It still weakens the claim that the filter and stop are pinned.
- The stop row has a second gap: the watcher closes its own events channel and the test never checks that. Only `changes` stays silent, and `changes` is never closed, so the row cannot tell a stopped goroutine from an idle one.

**3. LOW, F2: the generic sleep-bound loop was relaxed table-wide.**
- Changed text: `(d < time.Second && !tt.restsFloorRemainder) || d <= 0 || d > time.Minute` at `prreview_wait_test.go:~666`.
- The new `restsFloorRemainder` flag exempts the whole row's sleeps from the one-second minimum, not only the rest sleep. A regression that produced a sub-second interval sleep in that row would pass the loop.
- The row's own `check` pins `[1s, 600ms]` exactly, so this is covered. The loop's invariant is weaker for that row, nothing more.

F2's rows are sound. The 400ms row fails for a zero or stale `wokeAt`, because the sleeps would then be `[1s]` instead of `[1s, 600ms]`. The elapsed-1s row sits on the `rest > 0` boundary, so a `>=` mutation would add a `Sleep(0)` and fail the row.

The `pullRequestDeps` row does pin the watched path. It writes `shedrun.PullRequestRecord(location, shedrun.SelfRunID)` and reads the channel, so a wrong path fails it.

Tool calls made: none; the context already held the review, the changed files and the rubric.
