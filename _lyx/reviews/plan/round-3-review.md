---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/18-pairteardown-loop.md:38
    summary: The teardown releases the loop lock at session end, so a surviving waiter that read the held lock with no pid file spawns a fresh detached loop before the worktree is removed
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-loop-body.md:64
    summary: The loom pre-run has no resolved location when wire fails, so lyx loom step --until-stop falls back to ReportArmError and its arming refusal leaves the status file running
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/11-loop-body.md:65
    summary: ReportLoopArmError writes failed over a dead loop's pid file without the tree kill D2 puts before that write
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/12-loop-detach-wait.md:51
    summary: An arming refusal in the detached loop process's own pre-run goes to loop.log and reaches the driver as an interrupted loop-exited stop instead of a bootstrap error
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/12-loop-detach-wait.md:57
    summary: A waiter never checks the envelope's loop_id against its adopted id, and the single delivered slot can be overwritten, so a slow-ticking second waiter can print a later loop's envelope or report a false loop-exited stop
---

### [MEDIUM] The session-end bound lapses when the teardown releases the loop lock

**Location:** `_lyx/plan/18-pairteardown-loop.md:38`, with the waiter rule at `_lyx/plan/12-loop-detach-wait.md:50`.

**Issue:** D11 bounds the teardown: "no step starts in a pair once its session-end step begins".
Card 18's `seizeLoopLock` holds the loop lock only "until `EndSession` has ended the reed session, releasing it then", and its stated bound covers only the time the teardown holds the lock.
Card 12's `runUntilStop` treats "a held loop lock with no pid file" as a probe or a starting loop, re-reads on the timer, and once "the lock reads free … takes this decision again", which with no pid file and no envelope ends in `spawnLoop`.
The teardown holds the lock with no pid file beside it whenever `killLoop` found none: no loop was live, the common case for a parked driver, or a loop had not yet written its pid file.
A driver that launches `--until-stop` during the session-end step, for example in answer to a message, therefore leaves a waiter polling the held lock.
If that waiter outlives the reed session end, which nothing in the plan rules out, it spawns a new loop through `proc.DetachBreakaway` once the lock is released.
That loop runs in a new session that survives every later kill, and it steps in a worktree that `RemovePair` is about to delete.
In the case where a pid file did exist, the leftover record of the killed loop blocks a respawn ("an invocation never spawns a loop over a dead loop's pid file"), so the gap is the no-pid-file case only.

**Fix:** Make the teardown's mark outlive the session end.
One option: `seizeLoopLock` writes a pid-file-shaped tombstone (or leaves the killed loop's record) before releasing, so any later invocation reads a dead loop and never spawns.
Another: hold the loop lock through `RemovePair`.
Either way, extend card 18's bound sentence and the `teardown_test.go` call-sequence rows to cover an invocation that reads the lock free after the session ends.

### [MEDIUM] The loom subtree's `--until-stop` arming refusal has no location to build its `ArmStop`

**Location:** `_lyx/plan/11-loop-body.md:64`

**Issue:** Card 11 routes an `--until-stop` arming error through `ReportLoopArmError` "whenever they resolved a location and a run-id", and through `ReportArmError` otherwise.
In `internal/shedcli` that holds, because `resolvePersistentPreRun` calls `lyxcwd.Resolve` itself before `armFromSeed` (`internal/shedcli/cli.go:158-187`).
In `internal/loomcli`, `resolvePersistentPreRun` only calls `c.arm(cwd, …)` (`internal/loomcli/cli.go:196`).
`arm` resolves the location internally, and `c.location` is assigned only at the end of `wire` (`internal/loomcli/wiring.go:782`), after the hub landing config load at line 415.
The missing-hub-config refusal, the record's named example, therefore reaches the loom pre-run with `c.runID` set and no location.
Card 11 edits only `loomCLI.resolvePersistentPreRun` and leaves `loomCLI.arm` and `armAt` untouched, so under the card's own rule `lyx loom step --until-stop` takes the `ReportArmError` path.
On that path there is no `loop` object and no `failed` write, and the status file stays `running`.
That breaks `PATTERN-failure-stop-visible` for the second surface D1 names ("and so `lyx loom step --until-stop`").
No planned test drives the loom subtree's refusal: `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` runs `lyx shed step`, and the `cli_test.go` rows call `ReportLoopArmError` in-process.

**Fix:** State how the loom pre-run obtains the location on an arming failure.
One way: have `arm` return the resolved location alongside its error, adding `internal/loomcli#loomCLI.arm` to card 11's `Edit:` list.
The other: resolve it once in the pre-run and pass it to `armAt`.
Then add a row to `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` that runs `lyx loom step --until-stop` over the same missing-config fixture.

### [LOW] An arming refusal over a dead loop writes `failed` before the dead tree is killed

**Location:** `_lyx/plan/11-loop-body.md:65`

**Issue:** `ReportLoopArmError` calls `WriteFailedStop` whenever the status file exists and the loop lock is free.
It never looks at the pid file.
When a dead loop's pid file is present, D2 orders the stop: "first kills what is left of the dead loop's step tree …, then writes `failed`".
Here the refusal writes `failed` while the dead step's grandchildren may still run in their group, and leaves the tree until a later invocation's `deadLoopStop` reaches it.
That later stop then meets a `NotRunning` file, a result card 12's `deadLoopStop` does not describe.

**Fix:** Give `ArmStop` the pid-file path.
When the lock is free and a pid file is present, `ReportLoopArmError` should either run the `proc.KillTree` over the recorded child before the write, or write nothing and leave the dead-loop stop to the next invocation.
Also state `deadLoopStop`'s behavior on a `NotRunning` write result.

### [LOW] The detached loop's own arming refusal is lost to the log

**Location:** `_lyx/plan/12-loop-detach-wait.md:51`

**Issue:** The detached process `spawnLoop` starts runs the subtree's pre-run again before `runLoop`.
Its argv is "the loop's own plus `loopDetachedFlag`", so it carries `--until-stop`, and card 11's pre-run rule sends its arming error through `ReportLoopArmError`.
At that point the loop lock is free (`runLoop` has not taken it), so the refusal writes `failed` and prints the `bootstrap` envelope to stdout, which `spawnLoop` sends to `loop.log`.
The waiter then sees its spawn exited, the lock free, no envelope and no pid file, and reports `interrupted` with cause `loop-exited`.
The driver therefore gets an interrupted stop whose `stderr_path` and `trace_copy` do not name the log that holds the cause.
The waiter's own pre-run arms the same way just before, so this needs an arming input that changes or fails transiently in between.

**Fix:** In the detached process, write an arming refusal's envelope as the loop envelope file instead of printing it, or exclude `--loop-detached` from `ReportLoopArmError` and let the waiter's gone rule name `loop.log` in `detail`.
Either way, state it in card 12's Intent.

### [LOW] A second waiter can pick up another loop's envelope

**Location:** `_lyx/plan/12-loop-detach-wait.md:57`

**Issue:** The envelope file carries `loop_id`, but card 12 never says a waiter compares it with its adopted id before printing, and the delivered record is one slot "replacing an older delivered record".
The failure needs a waiter W1 adopted on loop L1 that sleeps up to `waitPollCeiling` (30 s) with no `fswatch` event.
In that time W2 delivers L1's envelope, the driver relaunches, and L2 starts, so W1 then waits on L2's lock and prints L2's envelope as its own.
If L2 also finishes and is delivered in that window, W1 meets a delivered record carrying L2's id and reports `interrupted` `loop-exited` for a loop that finished cleanly.
On that path the plan does not say whether W1 writes a loop envelope that `envelopeCurrent` would then deliver to the next invocation.
This is the focus's two-waiter question: the delivered slot settles the common case, where both waiters wake on the envelope's own file events.

**Fix:** State that a waiter prints only an envelope or delivered record whose `loop_id` equals its adopted id.
A waiter whose adopted loop is gone, with neither file carrying that id, should stop with a stop that writes neither `failed` nor a loop envelope (for example `busy`, way forward: re-run).
Add a row to `TestUntilStop_SecondWaiterAdoptsTheLiveLoop`.

## Settled focus questions

- Cards 9, 11, 12, arming refusal under `--until-stop`: settled for `lyx shed step`.
  Card 11's `ReportLoopArmError` turns the pre-run's refusal into a `loop` object with a `failed` write, and card 12's integration row runs the built binary on that surface.
  The loom subtree is not settled, see F2.
- Card 12, held-lock probes: settled.
  A waiter adopts an id only from its own spawn or a pid file, which a loop writes only after taking its lock.
  It reaches the gone rule only with an adopted id, and `loopLockGrace` keeps a probe's instant hold from failing a starting loop.
- Card 12, two waiters: settled for the common case by the delivered record; the residual race is F5.
- Card 18, session-end order: not settled, see F1.
- Card 9, way forward on every `bootstrap` arming error: settled.
  `ReportArmError` appends the generic clause to any text naming none, covering the unknown recipe and the raw seed read.
  `TestArmFromSeed_RefusalEnvelopeKinds` asserts it on the unknown-recipe row and the no-double-clause row, which is one behavior reached by one table, as `PATTERN-refusal-way-forward` allows.
