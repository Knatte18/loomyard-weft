# Plan-Review round 3 — fixer report

All five findings were validated against the code and the decision record and fixed.
`lyx loom validate-plan` answers `ok: true` over the edited plan, with informational findings only.

## Fixed

### F1 (MEDIUM) — the session-end bound lapses when the teardown releases the loop lock

Confirmed against card 12's `runUntilStop` rule: a held lock with no pid file is re-read, and a free lock with no pid file and no envelope spawns.
`internal/pairteardown/teardown.go` keeps `EndSession` and `RemovePair` separate, and `internal/battencli/wire.go` runs them as separate rows, so holding the lock through `RemovePair` is not available.
Card 18: before releasing the loop lock, `seizeLoopLock` leaves a pid file beside it, either the killed loop's own record or a mark it writes through the new `shedverbs.WriteLoopPIDRecord`: teardown loop id, no loop process, no child.
A later invocation reads a free lock beside a pid file as a dead loop, spawns nothing and stops `loop-exited`.
`killLoop` never removes the pid file, and a record with no child kills no group.
The bound sentence, the `teardown_test.go` release row, `doc.go`/`PATTERN-pair-teardown.md` and the ImpactSummary state the mark.
Card 12: `WriteLoopPIDRecord` is created and used by the loop's own write.
`deadLoopStop` states that a record with no child kills nothing, so a zero group id never reaches a kill, and reports `status_moved` false.
`TestUntilStop_DeadLoopStopsLoopExited` gains a row for such a record.

### F2 (MEDIUM) — the loom subtree's `--until-stop` arming refusal has no location

Confirmed: `internal/loomcli/arm.go`'s `arm` resolves the location locally, and `c.location` is set only by `wireLightweight` or at the end of `wire` (`internal/loomcli/wiring.go:782`), after the hub landing config load.
Card 11: `internal/loomcli#loomCLI.arm` joins `Edit:`, and `arm` records the resolved location on `c.location` before the reconcile and `armAt`.
That way every arming refusal after the resolve, the missing hub config included, reaches `ReportLoopArmError`.
The ImpactSummary says so.
Card 12: `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` gains a row that runs `lyx loom step --until-stop` over the same missing-config fixture and asserts the same `bootstrap` answer and `failed` write.

### F3 (LOW) — an arming refusal over a dead loop writes `failed` before the tree kill

Card 12 now edits `ArmStop`, `ReportLoopArmError` and both `resolvePersistentPreRun`s.
`ArmStop` gains `PIDPath`, filled from `shedrun.LoopPIDFile`.
With the loop lock free and a pid file present, `ReportLoopArmError` first kills the recorded step tree through `proc.KillTree`, then writes `failed`, and keeps the pid file so the next invocation's `deadLoopStop` still reports `loop-exited`.
`deadLoopStop` states its behavior on a not-`running` write result: no status write, and the stop goes on as written.
A `TestUntilStop_DeadLoopStopsLoopExited` row covers the refusal followed by the next invocation.

### F4 (LOW) — the detached loop's own arming refusal is lost to the log

Took the reviewer's first option.
`loopDetachedFlag` becomes the exported `LoopDetachedFlag`, and both pre-runs fill `ArmStop.LoopID` from it and `ArmStop.EnvelopePath` from `shedrun.LoopEnvelope`.
With a loop id set, `ReportLoopArmError` writes its envelope under that `loop_id` as the loop envelope file instead of printing it into `loop.log`, so the waiter prints the `bootstrap` stop.
With the loop lock held it writes nothing, and the waiter adopts the holder.
A loop that exits before any pid file names it, with no envelope, is reported `loop-exited` with `detail` naming the loop log and the file written `failed`.
Two `TestUntilStop_DeadLoopStopsLoopExited` rows cover both cases.

### F5 (LOW) — a second waiter can pick up another loop's envelope

Card 12's gone rule is rewritten around the adopted loop id.
A waiter prints the envelope file or the delivered record only when its `loop_id` equals that id.
Its loop is live or dead by the pid file while the pid file names it, and starting while its own spawn has not exited.
A waiter that once read the pid file naming its id, and now finds no file carrying it, knows the loop finished and was delivered elsewhere.
It stops `busy` with a delivered-elsewhere error naming the way forward and writes neither `failed` nor a loop envelope.
`refusal-spec.md` gains the `loop delivered elsewhere` row.
`TestUntilStop_SecondWaiterAdoptsTheLiveLoop` gains a row where the first waiter sleeps through two deliveries.

## Disputed

None.

## Deferred

None.
