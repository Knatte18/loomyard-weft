# Card 11 — cancelled-record-removal

**Create:**
- `plan:internal/pairteardown#deleteCancelledBattenRecord` -> `func deleteCancelledBattenRecord(prime *lymcwd.Location, slug string, commit func(relPaths []string, msg string) (bool, error)) (deleted bool, err error)`
- `plan:internal/pairteardown#TestDeleteCancelledBattenRecord` -> `func TestDeleteCancelledBattenRecord(t *testing.T)`
- `plan:internal/battencli#TestClearDoneRun_LeavesACancelledRunAndItsTaskBranch` -> `func TestClearDoneRun_LeavesACancelledRunAndItsTaskBranch(t *testing.T)`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.Run`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/battencli#cleanupDeps`
- `internal/battencli#fabricCleanupDeps`
- `internal/battencli#judgeRunDirectory`
- `internal/battencli#TestCleanup`
- `internal/dobbyengine#TestClearDoneRuns`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `plan:internal/shedengine#StateCancelled`
- `internal/battencli#ClearDoneRun`

**Intent:** Every path that removes a cancelled run's pair deletes its batten record in the same step, and `lym batten cleanup` only repairs what a crash left (D3).

`deleteCancelledBattenRecord` reads the prime's batten record of the slug; when its state is `cancelled` it removes the run directory and commits `shedrun.RunDirRel` through `commit`, and reports it deleted; any other record, or none, is left.
`Teardown` gains the seam that calls it; `New` wires it with `fabricengine.CommitAnchoredPaths` on the prime.
`Teardown.Run`, the removal `lym fabric remove <slug>` runs, calls it right after the pair's removal succeeds and spawns the prime's records push when it committed.
`RemovePair` does not call it: the cancel deletes the record itself (card 9).
The dobby teardown of D11 joins this path in a later part.

`cleanupDeps` gains `PairStands func(slug string) (bool, error)`, filled in `fabricCleanupDeps` from the topology's `PairParts`.
`judgeRunDirectory` judges a `cancelled` record before `judgeStatus`:
with no pair part left it is a removal candidate;
with its pair standing it is skipped with the reason "the run was cancelled with its pair kept; way forward: run `lym fabric remove <slug>`, which deletes the record with the pair".
A done record is judged as today, and `judgeStatus` keeps judging `done` alone, so `ClearDoneRun` never takes a cancelled run.
That refusal is reached by a kept cancel's record while its pair stands; `TestCleanup` reaches it.

Tests, all tier0:
`TestDeleteCancelledBattenRecord` deletes a cancelled record and commits its run directory, and leaves a running and a done record with no commit;
`TestTeardown_CallSequence` asserts the seam runs after the pair's removal and never before it or on a refused removal;
`TestCleanup` gains rows for a cancelled record with its pair gone (removed) and standing (skipped with the way forward);
`TestClearDoneRun_LeavesACancelledRunAndItsTaskBranch` asserts `ClearDoneRun` reports nothing cleared and calls no branch discard for a cancelled record;
`TestClearDoneRuns` gains a cancelled run the dobby's clear passes over.

**ImpactSummary:** One new step in the fabric removal's teardown and one new judgement in batten cleanup; done runs and the dobby's done-run clear are unchanged.

**Commit:** 11: cancelled-record-removal
