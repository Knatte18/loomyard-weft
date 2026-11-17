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
- `cmd/lym#testOnlyCodeAllowed`

**Uses:**
- `internal/fabricengine#archiveOwnerSides`
- `internal/fabricengine#TeardownLockPath`
- `internal/fabricengine#ErrPairPushRunning`
- `internal/fabricengine#StatusPushLockWait`
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

`TakeOverRequest` names `Slug`, `Subject` (the adopt subject written into the entry), `Machine` (this machine's current name), `Pair` (the run's pair, already present: the caller adds a missing pair from origin first), `Parent` (the parent pair, nil for the prime), `At`, `Delays` (the retry waits, nil meaning `gitrepo.PushBackoffDelays()`) and `LockWait` (the wait for each teardown lock, zero meaning `StatusPushLockWait`).
`TakeOverResult` lists the owner archive `Tags` pushed and `Retries`.
`TakeOverRun` takes, before step 1, the moved slug's teardown lock and then the parent slug's (`root` for the prime), each exclusive through `lock.AcquireWriteLockWithin` with `LockWait`, and holds both until it returns.
A lock still held elsewhere when its wait runs out refuses with an error wrapping `ErrPairPushRunning` that names the run, before anything is fetched or written, as `HandOverTips` refuses.
Holding the parent's lock from the start means no step takes a lock the call already holds: step 2 archives both pairs through `archiveOwnerSides`, never through `ArchiveOwnerTips`, which takes its pair's lock itself.
A pair's remote steps thus hold its slug's lock (`PATTERN-pair-remote`).
That pattern takes a pair push outside a removal shared, but step 5's retry drops and rebases local commits of the parent's records branch, so no other push of that branch may run meanwhile.
Steps:
1. Fetches the pair's code and records branches and the parent's code and records branches.
2. Archives the local tips origin lacks (owner archive tags pushed to origin, as D2 does) of the moved pair and of the parent's pair, each through `archiveOwnerSides`, the in-package helper for a caller that already holds its slug's teardown lock, as `HandOverTips` uses it, so this machine's commits of other runs on the parent's records branch that origin lacks are tagged before step 4 commits onto that branch.
3. Resets the local branches to origin's through `DropLocal`, whose anchor-change refusal stands and is returned unwrapped.
4. Writes `Status.Takeover{Machine, Subject, At}` into the run's status file on the pair's records branch and on the parent's records branch, commits each and pushes it, records side first, with no code-side push.
   `takeoverBeforePushHook`, nil in production, runs before each push attempt with its number, so a test plants origin's later transition between the fetch and the push.
5. A rejected push fetches again and, on the records branch whose push was rejected only, drops the entry commit, rebases this machine's other commits origin lacks (owed transitions of other runs on the parent's branch) onto the new upstream tip (the old owner's later transition), rewrites the entry on top, and pushes again, waiting out each of `Delays`; a records branch whose push landed is left as it is.
   A rebase that conflicts is aborted and returns its error naming the run.
   The branch then ends at its local commits that origin lacked, without this call's entry commit, which the drop already removed, so a later push from this machine never publishes on that branch a takeover entry for an adopt that did not finish; step 2's archive tags hold those commits meanwhile.
   An exhausted retry returns the last push's error naming the run and that origin kept moving; the caller words the re-run, since fabric knows no adopt subjects.
6. Fast-forwards the local records branches and the code branch to origin's tips.
7. Drops this machine's hand-over mark of the slug through `DropHandOverMark`, so a mark from an earlier hand-over never stops the adopter's own pushes.
A fetch or fast-forward error is returned as it is, a propagated git error; the takeover entry, once pushed, stays, and the caller's re-run finds the run's origin record naming this machine.

`DropHandOverMark` removes the mark file at `HandOverMarkPath`; an absent mark is no error.

The `refuseDropLocalAnchorChange` refusal's way forward replaces "re-run the same `lym adopt` with `--drop-local`" with "re-run the same `lym adopt`", since `--drop-local` retires; its doc and `DropLocal`'s doc name the takeover as the caller.
`AdoptScratchDir`'s doc and doc.go's line drop the adopt record, its lock and the worker's output: the directory holds the adopt check's files.
doc.go gains the takeover writer beside the reader.

`cmd/lym`'s `testOnlyCodeAllowed` gains one entry per function this card leaves with no caller a main reaches, each Why naming card 9, which reaches it, as the `internal/runfsm.Order` entry does:
`internal/fabricengine.TakeOverRun`, `internal/fabricengine.DropHandOverMark` and `internal/fabricengine.OriginParentRunStatus`.
The exact keys are the ones the guard reports at this card's tree.

`TestTakeOverRun` (tier 1) rows:
`WritesTheEntryOnBothRecordsBranchesAndLands` (`ReadUpstreamTakeover` at origin reads it on each),
`ATakeoverPushRejectedOverTheOldOwnersLaterTransitionWritesOntoOriginsTipAndLands` (the hook commits the old owner's transition with `recordsFixture.originCommit` on attempt 1; the local records branch ends at that transition plus the entry),
`ARetryOriginKeepsRejectingReturnsThePushError` (the hook commits on every attempt, `Delays` of two zero waits; origin holds no entry),
`ALocalTipOriginLacksIsArchivedAndPushedBeforeTheReset` (the moved pair's archive runs under the held lock),
`ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun` (the parent's records branch holds a local commit of the moved run's own status file that origin lacks, and the hook commits the old owner's later transition of that same file on origin at attempt 1; the error names the run, the local parent records branch ends at that local commit with no rebase in progress, no local commit of the parent's records branch holds a takeover entry, and origin's parent records branch holds no takeover entry),
`AParentRecordsCommitOriginLacksIsArchivedAndKeptAcrossTheRetry` (the parent's records branch holds a local commit of another run origin lacks, and the hook rejects the parent push on attempt 1; its owner archive tag is on origin, and origin's parent branch ends with that commit and the entry),
`AnAnchorChangeOnTheWeftRefusesBeforeAnyEntryIsWritten`,
`APairOrParentTeardownLockHeldElsewhereRefusesBeforeAnyEntryIsWritten` (the test holds the moved slug's lock through `TeardownLockPath`, then in a second case the parent slug's, with a `LockWait` of a few milliseconds; the error wraps `ErrPairPushRunning` and names the run, and origin and both local records branches are unchanged),
`TheLocalBranchesEndAtOriginsTips`,
`AHandOverMarkOfTheSlugIsDropped`, and `NoHandOverMarkIsNoError`.
`TestOriginRunStatus` (tier 1, over `OriginParentRunStatus`) rows: `ReadsTheRunRecordAtTheParentsUpstreamTipWithItsTakeover`, `ABranchWithNoUpstreamReadsNotFound`, `ATipWithNoRecordReadsNotFound`.
`TestDropLocal` (tier 1) changes its two way-forward assertions to the new wording.

**ImpactSummary:** new fabric functions plus one reworded refusal and three test-only-code allowlist entries; DropLocal's behaviour is unchanged and its only test is updated.

**Commit:** 3: Write the takeover entry in fabric, with its rejected-push retry and the hand-over mark's drop
