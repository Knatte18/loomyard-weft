# Card 7 — read-run-owed-and-tip

**Create:**
- `internal/fabricengine#CodeTipReading`
- `internal/fabricengine#OriginCodeTip`
- `internal/fabricengine#FailedPushesOwedBy`
- `internal/fabricengine#pushJob.resetUnnamedTip`
- `internal/fabricengine_test#TestOriginCodeTip`

**Edit:**
- `internal/fabricengine#pushJob.repushOverOriginContained`
- `internal/fabricengine#SyncOptions`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/pairteardown#runReader`
- `internal/pairteardown#hubRunReader`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#runFakes`
- `internal/pairteardown#runFakes.reader`
- `internal/pairteardown#TestReadRun`
- `internal/pairteardown/doc.go`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`

**Uses:**
- `internal/fabricengine#lastTransitionCodeTip`
- `internal/fabricengine#TransitionTrailerKey`
- `internal/fabricengine#ListFailedPushes`
- `internal/fabricengine#updateRemoteBranch`
- `internal/fabricengine_test#newPairFixtureAt`

**Intent:**
`ReadRun` fills the two inputs nothing fills today (D1, D2).

The owed push: `FailedPushesOwedBy` returns the failed-push records of class `transport`, `deferred` or `teardown_lock` whose `Runs` holds the slug (card 3), newest `LastSeen` first.
A record whose `Runs` lacks the slug is not this run's: an owed push of the prime's records branch never holds a run whose transition did not fail it,
and one that two runs' transitions failed holds both.
`runReader` gains `owedPush func(slug string) ([]fabricengine.FailedPush, error)`; `read` sets `Inputs.PushOwed` when one stands, `Inputs.PushOwedError` to the newest record's `Error` and `Inputs.PushOwedWorktree` to its `RepoPath`.
Any other class is no owed push: such a push is no transient failure and the run halted on it (card 8).

The code tip: `OriginCodeTip` reads, with no network call, from origin's fetched records branch of the slug, the code tip the run's last transition names (`lastTransitionCodeTip`),
whether origin's fetched code branch holds it, the machine of the child record's last history entry at that tip, and that commit's time.
`CodeTipReading` holds `Tip`, `Missing`, `Machine` and `At`; a slug with no records branch on origin reads not missing.
`runReader` gains `codeTip func(slug string) (fabricengine.CodeTipReading, error)`; `read` sets `CodeTipMissing`, `CodeTip` and `CodeTipMachine` from it.
Until the tip lands the act rule answers `wait` (D2).

The owner's guard (D2, as narrowed by the decision record's `Added after Discussion` entry "The owner's guard resets a code tip off the run's last transition's line"): `repushOverOriginContained`, on finding origin's code branch ahead, calls `resetUnnamedTip` for a task pair's code side pushed with the new `SyncOptions.AfterRecords` set.
`PushPairAnchored` and `coalescePushSides` set it on the code side they send after the run's own records side landed; no other push sets it.
It fetches the records branch and reads the tip the run's last transition on origin names (`lastTransitionCodeTip`), the record's operand.
When origin's code tip is not that tip and not an ancestor of it, it resets origin's code branch with a lease at origin's tip to that named tip and pushes again.
This drops from origin's task code branch every commit past the named tip, whoever pushed it.
It is bounded in three ways: it runs only on a task pair's rejected code-side push sent with `AfterRecords`, after the run's own records side landed, so a machine a takeover moved never reaches it,
and a code-side push with no records side before it, such as Finalize's parent push through `Fabric.PushBranch`, keeps the rejection as today;
it is leased at the origin tip it read; and it never runs for the prime's code branch.
The reset runs through `updateRemoteBranch`, `destroy.go`'s executor of the leased move, which checks containment, ownership, dirtiness and force in that order (`PATTERN-fabric-boundary`).
`resetUnnamedTip` passes it a `remoteBranchUpdateRequest` over the task pair's warp checkout: ownership `ownedPairWarpCheckout`, `leaseSHA` the origin tip it read,
and dirtiness `dirtinessNA`, the reason being that a move of origin's branch touches no checkout file.
A lost lease (`errors.Is(err, gitrepo.ErrLeaseRejected)`, which card 6 makes the executor wrap) keeps the rejection as today; any other failure of the move is returned.
History is not searched: a tip given up through `ReturnToPreviousTransition` (card 12) stays named by an older transition commit's `Warp-SHA`, so only the last transition counts.
So a tip left by an owing machine that died before its re-read goes too; origin's tip equal to the last transition's, or behind it, keeps the rejection as today.

Tests:
- `TestOriginCodeTip` (tier1): rows for a named tip origin's code branch holds, one it lacks (with the machine and time read), and a slug with no records branch;
  and a row where a `lym fabric commit` style commit, carrying `Warp-SHA` and no `Run-Transition`, is the newest on origin's records branch, and the tip read is still the last transition's.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` (tier1) gains the owner's guard rows:
  origin's code branch carries a commit past the tip the last transition names, and the owner's push resets it under the lease and lands its own code side;
  origin's code tip equal to the tip the last transition names keeps the rejection, with origin unchanged;
  origin's code branch past the named tip, pushed by `PushWarpRebaseFreeAt`, which sends no records side first, keeps the rejection, with origin unchanged.
- `TestFailedPushRecord` (tier0) gains rows for `FailedPushesOwedBy`: a pair's transport record, a pair's `deferred` record and a prime records record whose `Runs` holds the slug count; a `not_fast_forward` record does not; a prime records record naming only another run, or no run, does not;
  a prime records record two runs' failures wrote reads owed by each of them.
- `TestReadRun` (tier0) gains rows through `runFakes`' two new fields: an owed push sets `PushOwed` with its error and worktree; a missing tip sets the three code-tip inputs; a read error of either names the read and the slug.

**ImpactSummary:** Fills two act-rule inputs every `ReadRun` caller already judges, so cancel, resume and batten's own verbs now meet the owed-push and code-tip waits their `Aside` does not drop; `lym batten cancel` waits on a `code_tip` verdict until the owing machine pushes or card 12 gives the tip up; the code-side guard acts only on a rejected task-branch push.
