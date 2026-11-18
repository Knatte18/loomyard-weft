```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F3, F4, F5, F6, F7–F12), by building the case where each is wrong. I made no further tool calls.

**1. MEDIUM — `internal/fabricengine/bolt.go`, `PullThenCommitWritten` (F1 proposal): the process-wide admission now spans every write callback.**
- `Admit` exempts every network call of the process until its release, "so a call on another goroutine or in another repository passes too" (`hold.go`, `Admit` doc).
- The proposal moved the `Admit` out of `pullFirst` and in front of `acquireWriteLock`, held by a `defer` until the function returns.
- The admission now covers each `write.Write()` callback and the commits, not only the pull.
- Callers such as `boardengine/board.go:254`, `ownerset.go:166` and `registrysetter.go:177` pass arbitrary writes. A network or agent call made in one of them now bypasses a pause for the whole sequence.
- Before the fix the admission covered only `pullFirst`.
- Alternative: admit before the lock but release right after `pullFirst`. That does not satisfy the finding, because the write lock is still held at the gate wait. A fix would need `Admit` to be scoped to the pull part, or the writes to be proven free of gated calls.
- `Bolt.Pull` has no callback, so only `PullThenCommitWritten` is hit.

**2. MEDIUM — `internal/fabricengine/doc.go:1244-1250` and the summary's "A held push never holds a lock a pause could keep": both state more than the code does.**
- F1's fix says the gate is passed "before the outermost lock", and "A pause therefore holds a pusher only while it holds no lock".
- `TakeOverRun` (`takeoverwrite.go:100-117`) takes both pairs' teardown locks before `land`. `land`'s `Admit` (`:173`) therefore waits with those teardown locks held.
- This is the same defect class as F1: it blocks that pair's `PushPairAnchored`, `Remove` and `Add`.
- The new takeover test probes only the push locks, so it cannot show this.
- The docs and the summary should either scope the claim (push locks only) or the takeover should admit before its teardown locks.
- Separately, `lockWeftForReplay` is called from `Fabric.Pull` and its callers' held locks were not checked.

**3. LOW — `internal/fabricengine/pushanchored.go`, the new `Admit` in `PushPairAnchored`: no test reaches it.**
- F1's fix asked for a test of the `Bolt` write-lock case only.
- With the outer `Admit` removed, the inner `Admit` in `coalesceFlagged` still takes the first gated wait. Nothing probes the teardown lock, so the suite stays green.
- The teardown lock is the very lock the finding named as the blocker, and it is the one unprobed site.
- F3 covers four sites but not this one. A probe of `TeardownLockPath` at every wait of a `PassingPauseGateForTest` push would pin it.

**Proposals I tried to break and could not:**
- F3's tests: the probe runs at every wait, and a moved or removed `Admit` leaves later gated git calls waiting under the lock, so they fail.
- F6: a leaked admission makes the final `Network` call skip its wait, so `waits` is 0 and the test fails.
- F7 and F8: `errors.Is` on a plain error fails if the skip is widened. The empty `Skipped` check is trivially true on the error path, but the error assertion carries the test.
- F4: the trailer's "never fires mid-step" is consistent with F5's re-acquire wording. The two refer to between-step gates, not a gate inside a step.
- F5, F9, F10, F11, F12: no breakage found.

Tool calls: none; the review, the diffs and the edited files were already in context.
