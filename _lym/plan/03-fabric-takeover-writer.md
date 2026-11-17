# Card 3 — The takeover entry's production writer

**Create:**
- `plan:internal/fabricengine#TakeOverRequest` -> `type TakeOverRequest struct`
- `plan:internal/fabricengine#TakeOverResult` -> `type TakeOverResult struct`
- `plan:internal/fabricengine#TakeOverRun` -> `func TakeOverRun(prime *lymcwd.Location, req TakeOverRequest) (TakeOverResult, error)`
- `plan:internal/fabricengine#takeoverBeforePushHook` -> `var takeoverBeforePushHook func(attempt int)`
- `plan:internal/fabricengine#DropHandOverMark` -> `func DropHandOverMark(hubPath, slug string) error`
- `plan:internal/fabricengine_test#TestTakeOverRun` -> `func TestTakeOverRun(t *testing.T)`
- `plan:internal/fabricengine#OriginParentRunStatus` -> `func OriginParentRunStatus(prime *lymcwd.Location, slug, parent string) (shedengine.Status, bool, error)`
- `plan:internal/fabricengine_test#TestOriginRunStatus` -> `func TestOriginRunStatus(t *testing.T)`

**Edit:**
- `internal/fabricengine#refuseDropLocalAnchorChange`
- `internal/fabricengine#DropLocal`
- `internal/fabricengine#AdoptScratchDir`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestDropLocal`

**Uses:**
- `internal/fabricengine#ArchiveOwnerTips`
- `internal/fabricengine#ReadHandOverMark`
- `internal/fabricengine#HandOverMarkPath`
- `internal/fabricengine#ReadUpstreamTakeover`
- `internal/fabricengine#MachineNames`
- `internal/fabricengine#OriginBattenStatus`
- `internal/fabricengine_test#recordsFixture`
- `internal/fabricengine_test#recordsFixture.originCommit`

**Intent:**
D6 steps 2 and 3 for one moved run, in fabric (`PATTERN-fabric-boundary`), plus the read adopt needs from origin.

`OriginParentRunStatus` fetches the records branch of `parent`'s pair (`""` the prime) and reads the run record of `slug` at its upstream tip, as `OriginBattenStatus` does for the prime; found false when the branch has no upstream or the tip holds no record. Its `Takeover` field is what the moved-run judgement reads.

`TakeOverRequest` names `Slug`, `Subject` (the adopt subject written into the entry), `Machine` (this machine's current name), `Pair` (the run's pair, already present: the caller adds a missing pair from origin first), `Parent` (the parent pair, nil for the prime), `At` and `Delays` (the retry waits, nil meaning `gitrepo.PushBackoffDelays()`).
`TakeOverResult` lists the owner archive `Tags` pushed and `Retries`.
`TakeOverRun`, under the slug's teardown lock:
1. Fetches the pair's code and records branches and the parent's records branch.
2. Archives the local tips origin lacks through `ArchiveOwnerTips` (owner archive tags pushed to origin, as D2 does).
3. Resets the local branches to origin's through `DropLocal`, whose anchor-change refusal stands and is returned unwrapped.
4. Writes `Status.Takeover{Machine, Subject, At}` into the run's status file on the pair's records branch and on the parent's records branch, commits each and pushes it, records side first, with no code-side push.
   `takeoverBeforePushHook`, nil in production, runs before each push attempt with its number, so a test plants origin's later transition between the fetch and the push.
5. A rejected push fetches again, resets the records branch with `reset --keep` to the new upstream tip (the old owner's later transition), rewrites the entry on that tip, and pushes again, waiting out each of `Delays`.
   An exhausted retry returns the last push's error naming the run and that origin kept moving; the caller words the re-run, since fabric knows no adopt subjects.
6. Fast-forwards the local records branches and the code branch to origin's tips.
7. Drops this machine's hand-over mark of the slug through `DropHandOverMark`, so a mark from an earlier hand-over never stops the adopter's own pushes.
A fetch or fast-forward error is returned as it is, a propagated git error; the takeover entry, once pushed, stays, and the caller's re-run finds the run's origin record naming this machine.

`DropHandOverMark` removes the mark file at `HandOverMarkPath`; an absent mark is no error.

The `refuseDropLocalAnchorChange` refusal's way forward replaces "re-run the same `lym adopt` with `--drop-local`" with "re-run the same `lym adopt`", since `--drop-local` retires; its doc and `DropLocal`'s doc name the takeover as the caller.
`AdoptScratchDir`'s doc and doc.go's line drop the adopt record, its lock and the worker's output: the directory holds the adopt check's files.
doc.go gains the takeover writer beside the reader.

`TestTakeOverRun` (tier 1) rows:
`WritesTheEntryOnBothRecordsBranchesAndLands` (`ReadUpstreamTakeover` at origin reads it on each),
`ATakeoverPushRejectedOverTheOldOwnersLaterTransitionWritesOntoOriginsTipAndLands` (the hook commits the old owner's transition with `recordsFixture.originCommit` on attempt 1; the local records branch ends at that transition plus the entry),
`ARetryOriginKeepsRejectingReturnsThePushError` (the hook commits on every attempt, `Delays` of two zero waits; origin holds no entry),
`ALocalTipOriginLacksIsArchivedAndPushedBeforeTheReset`,
`AnAnchorChangeOnTheWeftRefusesBeforeAnyEntryIsWritten`,
`TheLocalBranchesEndAtOriginsTips`,
`AHandOverMarkOfTheSlugIsDropped`, and `NoHandOverMarkIsNoError`.
`TestOriginRunStatus` (tier 1, over `OriginParentRunStatus`) rows: `ReadsTheRunRecordAtTheParentsUpstreamTipWithItsTakeover`, `ABranchWithNoUpstreamReadsNotFound`, `ATipWithNoRecordReadsNotFound`.
`TestDropLocal` (tier 1) changes its two way-forward assertions to the new wording.

**ImpactSummary:** new fabric functions plus one reworded refusal; DropLocal's behaviour is unchanged and its only test is updated.

**Commit:** 3: Write the takeover entry in fabric, with its rejected-push retry and the hand-over mark's drop
