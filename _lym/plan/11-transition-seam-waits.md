# Card 11 — transition-seam-waits

**Create:**
- `plan:internal/statuscommit#TestNew_MergeOperationWaits` -> `func TestNew_MergeOperationWaits(t *testing.T)`

**Edit:**
- `internal/statuscommit#Deps`
- `internal/statuscommit#New`
- `internal/statuscommit#failureDisposition`
- `internal/statuscommit#TestNew_PushErrorReturnsNil`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#newCommitStatusSeam`
- `internal/loomcli#statusCommitPathspec`
- `internal/battencli#commitStatusDeps`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#newCommitStatusSeam`
- `internal/loomcli#TestCommitStatusSeam_Real`

**Uses:**
- `internal/loomcli/doc.go`
- `plan:internal/lymcwd#OneRepository`
- `plan:internal/fabricengine#MergeStateOperation`
- `plan:internal/fabricengine#IsPushProtected`
- `plan:internal/fabricengine#IsPrimeBranchRefusal`
- `plan:internal/hubforge#NewOneRepoHub`
- `internal/hubforge#AddPairWith`
- `internal/statuscommit#TestNew_MergeActiveSkips`
- `internal/statuscommit#TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`

**Intent:** D4 and D7 at the transition seam: in a one-repository worktree it waits on a blocking git operation or a prime off its branch in place of its skip, and it completes a transition whose push failed as `protected` only.
A two-repository worktree keeps today's skip.
There `MergeStateOperation` blocks on any weft merge state,
so a run's own parked weft merge-in, a Parent-Align or Finalize leftover that loom start lets through for the landing to abort, would otherwise hold the run's first transition forever.
`Protected` reads true only for a push of a one-repository prime's branch, the one card 4 classifies `protected`; any other protected refusal halts the run as today.

`statuscommit.Deps`:
- New `MergeOperation func() (operation string, err error)` names the operation that blocks the records commit, `""` for none. A nil one reads none.
- `MergeActive` stays, with today's skip; the wiring sets exactly one of the two, `MergeOperation` in a one-repository worktree and `MergeActive` in a two-repository one.
- New `Stalled func(err error) bool` reads a commit failure that waits rather than halts, the prime-branch refusal. A nil one reads none.
- New `Protected func(err error) bool` reads a push failure as protected-only. A nil one reads none.

`New`'s closure:
- Before the commit, an operation `MergeOperation` names, or its probe error, makes it wait: it logs a warning naming the operation (or the probe error) and its way forward, conclude or abort it,
  sleeps through `deps.Sleep` with the owed-push backoff (one second doubling to `owedRetryCeiling`), and probes again, until the probe answers none. Then it commits.
  The wait takes no admission, so a pause holds the run as it holds an owed retry.
- `failureDisposition`: a commit failure the `MergeOperation` re-probe explains (an operation, or a probe error), or one `Stalled` reads true, waits the same way, logging the failure and its way forward, and then runs the commit again;
  a failure the `MergeActive` re-probe explains still takes the skip; any other failure still halts the run.
  So a one-repository run never acts past a transition whose records commit has not landed, and a prime run stalls until the prime is back on its recorded branch.
- A push error `Protected` reads true logs a warning naming the protection and returns nil, so the run acts with that transition's records unpushed.
  It is checked before `Owed`, and only when the push failed.
The package comment, `New`'s doc and `failureDisposition`'s doc state the dispositions with the one-repository wait beside the two-repository skip, and the protected completion.

Wiring: `loomcli`'s and `battencli`'s `commitStatusDeps` gain `MergeOperation`, `Stalled` and `Protected` fields, set in `commitStatusDepsGuardedBy` and `battenCommitStatusDeps`
to `fabricengine.MergeStateOperation(location)`, `fabricengine.IsPrimeBranchRefusal` and `fabricengine.IsPushProtected`.
`newCommitStatusSeam` passes `Stalled` and `Protected` to `statuscommit.Deps` in both layouts,
and passes `MergeOperation` with `MergeActive` nil when `lymcwd.OneRepository` holds for the location's worktree root, and `MergeActive` alone, as today, otherwise.
Their `MergeActive` fields stay for that and for the callers that refuse on them (`refuseWhileMergeActive`, `commitrecords.go`).
`loomcli`'s `commitStatusDeps` doc says a blocking git operation makes the step wait in a one-repository worktree, and a mid-merge state skips it in a two-repository one.
`loomcli`'s `newCommitStatusSeam` doc names the one-repository wait beside its skip-while-mid-merge, and `battencli`'s `newCommitStatusSeam` doc names it beside "It skips while a merge is in progress".
`statusCommitPathspec`'s doc scopes its clause about a transition whose commit was skipped mid-merge to a two-repository worktree.

Tests:
- `TestNew_MergeActiveSkips` and `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip` keep pinning the two-repository skip unchanged.
- `TestNew_MergeOperationWaits` (tier0) pins the one-repository wait, with `MergeActive` nil. Rows:
  `OperationBeforeTheCommitWaits` (probe answers `"merge"` twice, then none: two sleeps of 1s and 2s, then one commit and one push);
  `ProbeErrorWaits`;
  `OperationAfterAFailedCommitWaitsAndCommitsAgain` (first probe none, commit fails, re-probe `"cherry-pick"` then none: the commit runs twice and the second lands);
  `StalledCommitFailureWaitsAndCommitsAgain` (commit fails with an error `Stalled` reads true, then lands: one sleep, two commits, one push).
  An unexplained commit failure still halts, as `TestNew_CommitErrorPropagates` pins.
- `TestNew_PushErrorReturnsNil` (tier0) gains a `protected` column and a want-nil flag, and the row `ProtectedReadsTrue`: `Protected` reads the push error true, and the seam returns nil with no retry and no sleep.
  Its existing rows leave `Protected` unset and still return the error.
- `TestCommitStatusSeam_Real` (tier1) keeps its rows, the two-repository `mid-merge skips without touching the merge` and `merge going live after the probe skips instead of halting` included;
  the ordinary-path row asserts a nil push error with `Protected` wired.
  New rows over a `NewOneRepoHub` pair through `AddPairWith`: `one-repository cherry-pick in progress waits, then commits` and `one-repository merge with a records path unlike HEAD waits, then commits`.
  Each row's `Sleep` concludes or aborts the plain-git operation on its first call, and the row asserts the transition commit lands after it, reaching the wired `MergeOperation`.

**ImpactSummary:** statuscommit.Deps gains three fields, and New waits in a one-repository worktree where it skipped while two-repository worktrees keep the skip; the two shed seams that build it are rewired, and other users of their MergeActive fields are untouched.
