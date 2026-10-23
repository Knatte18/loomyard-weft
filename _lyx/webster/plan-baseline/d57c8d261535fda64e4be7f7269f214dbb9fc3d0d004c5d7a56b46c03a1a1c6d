# Card 11 — fixer-run

**Create:**
- `plan:internal/landingshed#fixerSeats` -> `type fixerSeats interface`
- `plan:internal/landingshed#fixerEnd` -> `type fixerEnd string`
- `plan:internal/landingshed#fixerOutcome` -> `type fixerOutcome struct`
- `plan:internal/landingshed#fixerRun` -> `type fixerRun struct`
- `plan:internal/landingshed#newFixerRun` -> `func newFixerRun(deps Deps, row string, gate verifyGate) fixerRun`
- `plan:internal/landingshed#fixerRun.run` -> `func (f fixerRun) run(ctx context.Context, mode FixerMode, rec FixerAttempts, cap int, tips [2]string) (fixerOutcome, FixerAttempts, error)`
- `plan:internal/landingshed#fixerOutcome.haltReason` -> `func (o fixerOutcome) haltReason(row string) string`
- `plan:internal/landingshed#LandingStopPrefix` -> `const LandingStopPrefix = "landing stop: "`
- `plan:internal/landingshed#TestFixerRun_Outcomes` -> `func TestFixerRun_Outcomes(t *testing.T)`
- `plan:internal/landingshed#fakeFixerSeats` -> `type fakeFixerSeats struct`
- `plan:internal/landingshed#fakeFixerSeats.Run` -> `func (f *fakeFixerSeats) Run(table seatengine.Table) (seatengine.Result, error)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/testkit/envkit#LandingDeps`
- `internal/loomcli#intentionallyNil`

**Uses:**
- `plan:internal/landingshed#FixerTable`
- `internal/landingshed#ParseFixerReport`
- `plan:internal/landingshed#WriteFixerAttempts`
- `internal/landingshed#newGateFixture`
- `internal/landingshed#fakeVerifyTree`

**Intent:** Run the merge fixer for one row entry, the one loop Merge-In and Finalize share (D4).
`Deps` gains the told seams the loop needs, each documented in `deps.go`'s style: `FixerSeats`, the seat runner (`fixerSeats`, one `Run(seatengine.Table) (seatengine.Result, error)` method, which `*seatengine.Engine` satisfies);
`DecisionRecordPath` and `PlanDir`, the paths the session reads, `PlanDir` empty for a darn run;
`LineChange`, the card-8 read over the task pair;
`IsTestFile`, the cap's language matcher;
`RowEntry`, which answers the current entry of a named row from the run's status history;
and `TaskRemoteTip`, which reads the task branch's remote-tracking tip, `origin/<task branch>`, without a fetch, "" when there is none, so a push the session makes from the task pair, which updates that ref, shows as a changed tip.
`envkit.LandingDeps` fills each with a no-op, so the drift guard keeps every seam filled, and loomcli's `intentionallyNil` lists each `Landing.*` seam beside the others `wire` leaves to the landing assembly.

`fixerRun.run`'s `tips` are the from tip and the merged tip, and `run` fills `FixerInputs.AlignedTip` and `MergedTip` from them; it builds card 7's table for `mode` with the report path `<scratch>/<row>-fixer-report.md` under `Deps.ScratchDir`, archives a stale report first, and stamps one gate entry, `landing`, whose attempts are the turns the entry has left (`cap` minus `rec.Spent`) and whose closure runs after every turn.
A turn is charged when it starts, so a turn that never reaches the closure, a timed-out or killed one included, is still spent and no resume grants a fresh turn: `run` charges the first turn into `rec` and writes the attempt record before the seat run starts, and the closure charges the next turn and writes the record before it returns a failure that re-prompts.
The closure checks the turn's own duration first, measured through the `verifyGate`'s injectable `now` clock from the turn's start: a turn that ran past `fixer_timeout_min` ends the loop as the timeout end.
It then checks the tree is clean, parses the report, and on a `fixed` verdict runs the `landing` gate through `verifyGate.check`, which writes the landing failure record on a failure; a failing gate re-prompts the same session with the failure and the record's path until the turns are spent.
An `escalate` verdict ends the loop and takes its turn back out of `rec` before writing it, so an `escalate` spends no attempt.
The table's timeout is `fixer_timeout_min` times the turns the entry has left, so a turn that hangs and never returns to the closure is ended by the session bound, at most that many minutes, and is the timeout end; a turn that returns is held to `fixer_timeout_min` itself by the closure's check.
`run` is never called with zero turns left: Merge-In and Finalize take their exhausted-entry path first when `rec.Spent` has reached `cap`.
`run` reads `TaskRemoteTip` before the seat run and again after it, and a tip that changed is the `pushed` end naming the new remote ref, since lyx's own records push never runs while the row's session runs.
`run` returns a `fixerOutcome` naming its end: `fixed` over a passing gate, `escalate` with the report's path and conflict, `spent` with the failure record's log, `timeout`, `no-report` (a missing or unparseable report), `dirty` (a dirty tree after a turn) and `pushed`, each with the session's log and the report path.
A seat run that fails to start returns its error marked transient through `internal/shedengine`'s mark (`PATTERN-transient-stop`), with the first turn's charge taken back out of `rec`, so a failed spawn is an error stop the arbiter or batten's re-step may resume and spends no attempt.

`haltReason(row)` words every halting end, and each opens with `LandingStopPrefix` so the hand-back predicate finds it (D11):
`spent` names the log, with the way forward to fix on the task branch, then `lyx loom resume`;
`escalate` names the report and the conflict, with the way forward `lyx loom decision add` in the task worktree, then `lyx loom resume`;
`timeout`, `no-report` and `dirty` name the session's log and the report path, with the way forward to fix or clean the task branch, then `lyx loom resume`;
`pushed` names the pushed ref, with the way forward to check the pushed commits are the task's, then `lyx loom resume`.
Every way forward is one no agent's settings deny (`PATTERN-no-denied-recovery`).

`fakeFixerSeats` is the shared fake seat runner the fixer tests of this card and of Merge-In and Finalize drive: per turn it runs a scripted callback that may write the report, commit, dirty the tree, advance the fake remote tip, take a scripted turn duration by advancing the gate's `now` clock by it, or release a lock record, then calls the table's gate closure, up to the gate's attempts; it can also fail to start or end on the session timeout without reaching the closure.
The table-driven test runs `run` over it, the gate fixture's fake verify tree and fake seams, one row per end, and asserts the outcome, the attempt record after the run (an `escalate` row leaving `Spent` unchanged, a spent row equal to the cap, a session-timeout row with its turn spent, a turn that returns past `fixer_timeout_min` ending as the timeout end, a resumed record with one turn left running exactly one turn), the reason's prefix and way forward, that a remote tip unchanged across the session is no `pushed` end while a moved one is, and that a failed start leaves the record as it was and returns an error the transient mark classifies.

**ImpactSummary:** Seven additive Deps seams, filled by the test kit; the loop has no caller until Merge-In and Finalize use it.
