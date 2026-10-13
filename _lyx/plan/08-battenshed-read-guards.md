# Card 8 — Batten's polled seam failures log once per episode

**Create:**
- `plan:internal/battenshed#TestInnerRun_ReadFailuresLogOncePerEpisode` -> `func TestInnerRun_ReadFailuresLogOncePerEpisode(t *testing.T)`

**Edit:**
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#innerRunProducer.lockStep`
- `internal/battenshed#innerRunProducer.handedLockStep`
- `internal/battenshed#innerRunProducer.loopLockStep`
- `internal/battenshed#innerRunProducer.releaseLock`
- `internal/battenshed#innerRunProducer.parentMovedUnderGate`
- `internal/battenshed#innerRunProducer.reviewWaitNote`
- `internal/battenshed#innerRunProducer.reviveDeadDriver`
- `internal/battenshed#innerRunProducer.driverAlive`
- `internal/battenshed#innerRunProducer.noticeStep`
- `internal/battenshed#innerRunProducer.noticeDriverBusy`
- `internal/battenshed#innerRunProducer.agentActivity`
- `internal/battenshed#innerRunProducer.arbiterHolds`
- `internal/battenshed#innerRunProducer.stopReport`
- `internal/battenshed#innerRunProducer.undecided`
- `internal/battenshed#innerRunProducer.deliver`
- `internal/battenshed#innerRunProducer.publishRetryGoto`
- `internal/battenshed#innerRunProducer.publishRetryResume`
- `internal/battenshed#innerRunProducer.resumeAwaitingChild`
- `internal/battenshed#TestInnerRun_RunningArmReviewWaitNote`
- `internal/battenshed#TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn`
- `internal/battenshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D11 for `internal/battenshed`: every Info-or-above line the audit in `00-overview.md` lists for this package with a verdict of guard or demote gets its change here; the lines it lists as keep stay.

`innerRunProducer` gains a `failures logger.EpisodeSet`, which lives as long as the producer, across its Calls.
Each guarded site keeps its message and fields and swaps `logger.Warn(msg, …)` for `p.failures.Fail(msg, msg, …)`, keyed by its own message, and on the same seam's next success calls `p.failures.Recover(msg, "battenshed: a repeated failure recovered", "failure", msg, "producer", p.name, "slug", p.slug)`:
the parent lock record read (`lockStep`), the decision read for the handed lock (`handedLockStep`), the agent states read (`loopLockStep`), the release failure (`releaseLock`), the decision read and the parent-moved read for the stale-base check (`parentMovedUnderGate`), the reviewer note (`reviewWaitNote`), the driver strand read (`reviveDeadDriver`), the driver liveness read (`driverAlive`), the pair directory (`noticeStep` and `noticeDriverBusy`, which share one key), the agent activity read (`agentActivity`), the arbiter marker read (`arbiterHolds`), the stop report read (`stopReport`), the decision read for the notice step (`undecided`) and the unreadable orch state (`deliver`).
Each of these runs once per probe check or once per check while its seam keeps failing, so before this card it repeated a Warn with no state change.
A site whose read is skipped on a check neither fails nor recovers its episode on that check.
`releaseLock`'s `released the child's parent lock` line moves from Warn to Info: it records a state change once per release and is no failure.
`publishRetryGoto` and `publishRetryResume` log their `sending the child…`, `resuming the child…` and `inner shed run wait complete` lines at Info only when `w.publishRefusal` is empty (the Call's first attempt) and at Debug on each retry after a refusal;
`resumeAwaitingChild` does the same for `resuming awaiting inner shed run` and its `wait complete` line, at Info while neither `w.notParkedLogged` nor `w.busyLogged` is set.
The doc comments that say a site warns once per read, or that every release or seam failure is a Warn (`innerrun_parentlock.go`'s `lockStep` comment and `doc.go`'s parent-lock paragraph), say what the site does now.

`TestInnerRun_ReadFailuresLogOncePerEpisode` is serial (it captures the process-global logger at Debug) and table-driven: each row runs one `NewInnerRun` Call built by `newInnerRunDeps`, with `NoticeProbe` zero so every check probes, a seam that fails on its first three calls and succeeds after, and a `fakeClock.pauseAtSleep` that ends the wait after the success; it asserts one `level=WARN` line and at least one `level=DEBUG` line carrying the row's failure message, and one `level=INFO` `a repeated failure recovered` line naming it.
A row whose seam answers through the `ParentLock` deps builds an `InnerRunDeps.ParentLock` value with `Read`, `Agents`, `Release`, `ParentMoved`, `SelfPID`, `GateRow` and `IdleBound` set, since `newInnerRunDeps` wires none; the agent-activity row sets `DriverAlive` to answer true, since `newInnerRunDeps`' dead driver otherwise ends `noticeCondition` before the agents are read, and every notice row wires `Notify` and `PairDir`.
Guard rows, one per guarded site:
the parent lock record read (`ParentLock.Read`, running child);
the decision read for a handed lock (`ReadDecision`, a record whose owner pid is `SelfPID`, child awaiting at `GateRow`);
the agent states read (`ParentLock.Agents`, a record owned by another pid, running child);
the release failure (`ParentLock.Release`, a handed record, failed child);
the stale-base decision read and the parent-moved read (`ReadDecision`, then `ParentMoved`, child awaiting at `GateRow` with no decision), one row each;
the driver liveness read (`DriverAlive`, done child with a grace longer than the checks);
the pair directory (`PairDir`, a running child whose driver is found dead, which makes a notice due);
the agent activity read (`Activity`, running child with a live driver and notices on);
the arbiter marker read (`ArbiterInFlight`), the stop report read (`StopReport`) and the notice step's decision read (`ReadDecision`), each on a blocked or awaiting child whose parked-stop notice is due, one row each;
the unreadable orch state (`OrchStrandRecorded` erroring, a running child whose dead driver makes a notice due).
Demotion rows: the parent lock release (a handed record, failed child, `Release` answering released) logs one `level=INFO` `released the child's parent lock` line and no WARN one;
the Publish-retry goto refused busy twice (built as `innerrun_publishretry_test.go`'s busy-goto row) logs `sending the child to the failed Publish's target` once at INFO and then at DEBUG;
an awaiting child with a decision whose resume is refused not-parked twice logs `resuming awaiting inner shed run` once at INFO and then at DEBUG.
The two seams an existing test already drives failing are extended there instead of repeated here:
`TestInnerRun_RunningArmReviewWaitNote`'s "failing ReviewWait falls back to plain reason" row asserts one `level=WARN` `reviewer wait note unavailable` line across its three checks, the later ones at DEBUG;
`TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn` lets its `DriverStrand` fail on the first three probes and answer after, and asserts one WARN line, DEBUG lines after it, and one INFO `a repeated failure recovered` line naming the strand read.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/battenshed` row rises from 110 to 111.

**ImpactSummary:** log levels and repetition only, across the inner-run producer's polled seams in `internal/battenshed`; no verdict, notice, marker or reason file changes.

**Commit:** 8: battenshed: log a repeating seam failure once per episode
