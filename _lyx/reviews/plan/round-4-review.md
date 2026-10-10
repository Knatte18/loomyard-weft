---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/18-pairteardown-loop.md (Intent, the paragraph starting The bound outlives the lock)
    summary: The teardown's pid-file mark bars only the first invocation after the session end, since card 12's deadLoopStop retires it and writes an envelope the next invocation delivers, so the third invocation spawns a loop before RemovePair; the stated bound and no test row cover this
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/12-loop-detach-wait.md (Intent, the waiter's gone rules)
    summary: A waiter whose own spawn finished before the waiter ever read a pid file naming it, while a later loop's records exist, either adopts the later loop through the loser rule or reports loop-exited and writes failed, contradicting the card's never-prints-another-loop's-envelope claim
---

### [MEDIUM] The teardown's mark is consumed by the first dead-loop stop, so a later invocation can spawn a loop before `RemovePair`

**Location:** `_lyx/plan/18-pairteardown-loop.md`, Intent, "The bound outlives the lock: …" and "A waiter that read the held lock with no pid file, or any later invocation, then reads a free lock beside a pid file, which card 12 treats as a dead loop: it spawns nothing …"; with `_lyx/plan/12-loop-detach-wait.md`, Intent, the `deadLoopStop` and `runUntilStop` sentences.

**Issue:** Focus question 1, settled.
The waiter that was waiting on the teardown-held lock does not spawn: under card 12's `runUntilStop`, a held lock with no pid file re-reads until a pid file appears (it then adopts the mark's id) or the lock reads free (it then sees a free lock beside the mark, a dead loop), and both paths reach `deadLoopStop`, which spawns nothing.
The bound fails one step later.
Card 12's `deadLoopStop` "writes the `interrupted` envelope … as the loop's envelope file under the dead loop's recorded id, retires the pid file and prints the stop".
So after that first stop the steps directory holds a free lock, no pid file and a `loop-exited` envelope whose `loop.state` is the `failed` the stop just wrote.
The second invocation finds a free lock with an envelope; `envelopeCurrent` matches it against the status file (still `failed`), so it delivers it, retires it, and retires no pid file because none is left.
The third invocation finds a free lock, no pid file and no envelope, so `runUntilStop` spawns a loop, whose first child's resume write clears `failed` and steps, in the worktree `RemovePair` is about to delete.
The same consumption happens when an invocation runs `deadLoopStop` over the killed loop's own record in the gap between `killLoop` and the seize's `TryAcquireWriteLock`.
Card 18's sentence "or any later invocation … spawns nothing" therefore holds for one invocation, not for every one up to `RemovePair`, and D11's bound "no step starts in a pair once its session-end step begins" rests on it.
No row covers the path either: `teardown_test.go`'s rows stop at reading the mark back, and card 12's `TestUntilStop_DeadLoopStopsLoopExited` has "a record with no child killing nothing and stopping `loop-exited`", one invocation, with no row for a waiter that read the teardown-held lock with no pid file, nor for a second and third invocation over the mark.
The practical window is narrow, since ending the reed session ends the driver, but the card states the bound for exactly the driver that outlives the session end.

**Fix:** In card 12, have `deadLoopStop` keep a pid file whose record names the teardown as its loop id (the mark card 18 writes) instead of retiring it, and write no loop envelope over it, so every invocation until `RemovePair` stops `loop-exited` and spawns nothing; card 18 then leaves the mark (or rewrites the killed loop's record under the teardown's id) and states that it survives every dead-loop stop.
Add rows to `TestUntilStop_DeadLoopStopsLoopExited`: a waiter that read a held lock with no pid file stops `loop-exited` once the lock is released beside the teardown's mark, and a second and a third invocation over the mark still spawn nothing.
If the fixer prefers to keep the retirement, the alternative is to narrow card 18's bound to the first invocation and state that the session end killing the driver is what bars later ones.

### [LOW] A waiter that never read its own loop's pid file misreads a later loop's records

**Location:** `_lyx/plan/12-loop-detach-wait.md`, Intent, "A waiter matches files by its adopted loop id …" through "a waiter whose own spawn exited before any pid file named it, with the lock free, saw a loop that died before stepping …".

**Issue:** Focus question 3, settled.
A waiter does compare the envelope's and the delivered record's `loop_id` with its adopted id before printing, and a waiter that once read the pid file naming its id stops `busy` with the delivered-elsewhere error when a later loop's delivery replaced the record.
The gap is the waiter that spawned loop L1 and never read L1's pid file: L1 took the lock, stepped, wrote its envelope and removed its pid file between two of the waiter's ticks, another invocation delivered L1's envelope, and a later loop L2 was spawned and replaced the delivered record.
On its next tick that waiter sees its own spawn exited and no file carrying its id, and the card's rules then pick by the lock alone.
With L2 holding the lock, "a waiter whose own spawn exited, with the lock held, adopts the loop the pid file names" fires, so the waiter prints L2's envelope, which the card says a waiter never does.
With the lock free, the waiter reports L1 as `interrupted` with cause `loop-exited` ("saw a loop that died before stepping") and calls `WriteFailedStop`, which writes `failed` when the file reads `running`, for instance after a resume, although L1 finished and no loop is dead.
The race needs the waiter to sleep through two loop lifecycles, which `fswatch` events cutting the wait short make unlikely where the watch opens; without it the backoff reaches `waitPollCeiling`.

**Fix:** Let the waiter tell a finished loop from one that died before stepping without depending on having read the pid file, for instance by keying the delivered record by loop id (a `shedrun` constructor taking the loop id, so a later loop's delivery never replaces it), or by having `spawnLoop`'s channel report a distinct exit status for the loop that lost the lock, so only a loser's waiter adopts the holder.
Add a row to `TestUntilStop_SecondWaiterAdoptsTheLiveLoop` where the spawning waiter never reads its loop's pid file before that loop's envelope is delivered elsewhere and a later loop runs, asserting it prints neither the later loop's envelope nor writes `failed`.

## Focus questions settled as not defects

- Focus 2: today `c.location` is assigned only in `wireLightweight` and at the end of `wire` (`internal/loomcli/wiring.go`), after the hub landing-config load that refuses first, so as the code stands a missing-hub-config refusal would leave the loom pre-run without a location.
  Card 11 lists `internal/loomcli#loomCLI.arm` under `Edit:` and has `arm` record the resolved location on the receiver before the reconcile and `armAt`, while `armAt`'s `resolveRunID` records the run-id before `wire`, so the refusal reaches `ReportLoopArmError` and writes `failed`.
  Card 12's `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` drives that subtree in its third row (`lyx loom step --until-stop` over an absent hub landing config with a `running` status file, asserting `bootstrap`, the closed `loop` key set and the `failed` write), and the plan's `## verify:` runs `go test -tags integration ./...`.
- Focus 4: with the loop lock free and a dead loop's pid file present, card 12 states `ReportLoopArmError` kills the recorded step tree through `proc.KillTree` first, then writes `failed`, and leaves the pid file, so the next invocation's `deadLoopStop` reports `loop-exited` and finds the file no longer `running`; this is D2's kill-then-write order, and a row of `TestUntilStop_DeadLoopStopsLoopExited` pins it.
  A refusal in the detached loop's own pre-run is stated in card 12 (which edits `ReportLoopArmError` and `ArmStop`): with `ArmStop.LoopID` filled from `LoopDetachedFlag`, the envelope is written under that `loop_id` as the loop envelope file, the waiter prints it as its loop's envelope, and with the loop lock held nothing is written and the waiter adopts the holder; a row of the same test pins it.
  Card 11's Intent predates the detached process and states only the invocation's own pre-run, which is correct since card 12 carries the change.
