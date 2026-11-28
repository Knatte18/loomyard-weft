```yaml
lens: fix-code-correctness
outcome: reported
```

Checked the correctness of the code changed in `64ac643b5` and `f2e680570`. One finding.

**MEDIUM: a live dobby can hold a catch-up wait that nothing retries or clears, and the fix now sends it to the prime's summary.**

- **Changed text:** `hubBeamCatchUpWaits` and `liveCatchUpWait` in `internal/beamcli/dobbydeps.go`, and `addCatchUp` in `internal/dobbyengine/pushsummary.go`. The same assumption is in the doc comment ("A beam whose dobby is not live is left out, since nothing retries or clears its record"), in `doc.go` ("every live dobby's wait on origin") and in the matching `_lym/landing/summary.md` bullet.
- **Defect:** being live does not mean the record is retried. `catchUpOwed` exists only in memory.
- **Concrete case:**
  1. Origin is down, and beam `b`'s dobby records `CatchUp{Reason: origin}`.
  2. The `lym` binary changes and the dobby re-execs.
  3. `recordStart` (`internal/dobbyengine/loop.go`) returns early for the same pid and start ticks, so it never calls `oweCatchUp`.
  4. `catchUp` returns at `!d.catchUpOwed`, so `catchUpSucceeded`, the only code that clears `State.CatchUp`, never runs.
- **Result:** after origin is back, the prime reports "b's catch-up from origin for <n>m" with the dobby live. The owed period never ends and no all-clear is posted, until a sleep, a hand-over or a pass exit happens to owe a new catch-up. The prime's own record behaves the same way after its own re-exec. That part predates this round, but the fix extends it to every beam and states the opposite assumption.
- **Related:** a stopped dobby started again keeps its old record. Until its first try, the prime can read that record with its old `Since`. If that try also fails on origin, `catchUpFailed` keeps the old `Since`, so the reported minutes include the time the dobby was down.

I found nothing else wrong:
- `oweCatchUpForStart`, the new `firstPassExited` case, `passExitCode` and the `passExited` list behave as stated on the inputs I traced.
- By reading, each new engine test would fail with its defect in place. The `CatchUpRefused` row is the exception: its defect is in `passExitCode`, which the `owner_test` rows cover.

Tool calls: none.
