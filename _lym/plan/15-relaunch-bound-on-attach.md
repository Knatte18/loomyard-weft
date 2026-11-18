# Card 15 — A relaunch past its bound ends or resumes on the next attach

**Create:**
- `internal/shuttleengine#Run.settleStaleRelaunch`
- `internal/shuttleengine#TestAttach_RelaunchingRecordPastItsBound`

**Edit:**
- `internal/shuttleengine#dispositionCandidate`
- `internal/shuttleengine#Runner.reconstruct`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#TestCompletionSignal_NegativeVerdictReturnSites`
- `internal/shuttleengine#TestCompletionSignal_FileContractCallSites`

**Extends:**
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#RunState.relaunching`
- `internal/shuttleengine#updateRunState`
- `internal/shuttleengine#Run.deliverPromptOnAttach`
- `internal/shuttleengine#Run.DeliverPendingPrompt`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#TestAttach_DeliversTheDeferredPrompt`

**Intent:**
D7's bound, so a `Wait` process that dies anywhere in steps 1 to 4 leaves no run read as live past it.
`dispositionCandidate` reads a relaunching record past its bound as attachable over a tracked strand, so its attach's `Wait` settles it, and as today otherwise.
`reconstruct` keeps the record's `RelaunchingAt` on the handle.
At each tick, `Wait` re-reads a relaunching record.
While it is within its bound, `checkLivenessTick` reads its tracked strand, held and paneless, as a strand still starting and counts no status failure, so an attached `Wait` beside a live relaunch, or after its waiter died, waits rather than failing on `errStrandPaneBindingCleared`.
Once it is past its bound, `Wait` calls `settleStaleRelaunch`:
- over a strand that is not live, held or not, it finalizes the run as a strand that died, through the completion-signal check, and removes the strand;
- over a live strand it clears the mark to running in one `updateRunState` hold and waits on as an attach, the record already naming the new launch: its session id and prompt offset come from the record, so the limited turn end is not read again, and `deliverPromptOnAttach` (`DeliverPendingPrompt`) types a `PromptLine` the dead `Wait` never sent.
A record within its bound is left to the live relaunch; a terminal one is left as it is.

`TestAttach_RelaunchingRecordPastItsBound` is tier0 on `fakeReed` and a fake clock, on the setup of `TestAttach_DeliversTheDeferredPrompt`:
- a record marked before step 4 (old session id, held strand not live) attached past its bound ends died with its strand removed, and within its bound attaches as live;
- a record whose step 4 write and reed relaunch happened (new session id, new prompt offset past a usage-limited turn end, a `PromptLine`, live strand) attached past its bound clears the mark, reads the new session id and offset, holds no hold notice for the old turn end, and types the `PromptLine` once;
- a record whose output files all exist ends done, not died;
- an attached `Wait` over a record within its bound with a held, paneless strand keeps waiting through more than `maxStatusRetries` liveness ticks, then ends died once the fake clock passes the bound;
- the tripwire tests pin `settleStaleRelaunch`'s died end and the new `dispositionCandidate` case as audited sites.

**ImpactSummary:** One settle step at `Wait`'s start and one disposition case; records without a mark take no new path.

**Commit:** 15: shuttle: settle a relaunch whose waiter died once its bound passes
