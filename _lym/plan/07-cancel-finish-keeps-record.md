# Card 7 — cancel-finish-keeps-record

**Create:**
- `plan:internal/runfsm#FinishCancel` -> `func FinishCancel(prime *lymcwd.Location, slug string, removed bool) error`
- `plan:internal/battencli#TestCancelIntegration_AKeptCancelLeavesItsFinishOnOrigin` -> `func TestCancelIntegration_AKeptCancelLeavesItsFinishOnOrigin(t *testing.T)`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#cancelReport`
- `internal/battencli#cancelRun`
- `internal/battencli#clearRunPhase`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestCancelIntegration_KeptPairJudgeOnly`
- `internal/battencli#TestCancelIntegration_RemoveTearsDownPairAndBranch`
- `internal/battencli#TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits`
- `internal/runfsm/doc.go`

**Uses:**
- `internal/battencli#removePairAndBranch`
- `internal/battencli#removeRunRecords`
- `internal/battencli#ClearDoneRun`
- `plan:internal/runfsm#UpdateIntents`
- `plan:internal/runfsm#PublishIntents`
- `plan:internal/shedengine#StateCancelled`

**Intent:**
A cancel keeps the batten record and writes its finish into it, in place of today's removal of the run's records (D3).
`FinishCancel` sets the cancel intent's finish, the time now and whether the pair was removed, and the shed state `cancelled`, never `done`, in one `state.UpdateJSON` under `shedrun.StatusLock`; it is a no-op on a finish already written.
`clearRunPhase` with keep no longer calls `RemoveRecords`: it calls a new `cancelDeps.FinishRecord` seam, `FinishCancel` then `PublishIntents`, so the finish is on origin when the cancel returns, and clears the claim and lifts the pauses as today.
With `--remove`, the record goes in the same step as the pair: `removePairAndBranch` then `RemoveRecords`, whose removal commit is pushed at once through the same publish in place of the detached `SpawnPush`; a slug released again then reads `queued` and its dobby starts it.
`cancelReport` gains whether the finish was written and pushed, and the cancel's output says so; an unpushed finish names `lym batten cancel <slug>` once origin answers, which publishes it on re-run.
`cancelRun` releases the cancel lock last, as today.
Since the finish's shed state is `cancelled`, `ClearDoneRun`, which takes only `done` runs, leaves the record and its task branch on origin.
Tests: `TestCancelRun` (tier0) asserts the finish seam in place of the records removal for keep, and removal with the pair for remove;
`TestCancelIntegration_AKeptCancelLeavesItsFinishOnOrigin` (tier1) cancels with keep and reads the finish and state `cancelled` from origin's records branch and `cancelled` with keep through `runfsm.Read` on a second clone;
`TestCancelIntegration_RemoveTearsDownPairAndBranch` asserts the record is gone from origin with the pair;
`TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` gains a cancelled run that the clear leaves with its task branch.

**ImpactSummary:** a kept cancel leaves a pushed batten record in state `cancelled` in place of no record; a removing cancel pushes the record's removal at once.

**Commit:** 7: A cancel writes and pushes its finish, keeping the batten record unless it removes the pair
