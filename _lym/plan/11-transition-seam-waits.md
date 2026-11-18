# Card 11 — transition-seam-waits

**Create:**
- `plan:internal/statuscommit#TestNew_MergeOperationWaits` -> `func TestNew_MergeOperationWaits(t *testing.T)`
- `plan:internal/statuscommit#TestNew_ProtectedPushCompletes` -> `func TestNew_ProtectedPushCompletes(t *testing.T)`

**Edit:**
- `internal/statuscommit#Deps`
- `internal/statuscommit#New`
- `internal/statuscommit#failureDisposition`
- `internal/statuscommit#TestNew_OrdinaryPath`
- `internal/statuscommit#TestNew_CommitErrorPropagates`
- `internal/statuscommit#TestNew_PushErrorReturnsNil`
- `internal/statuscommit#TestNew_OwedPushRetriesBehindTheGate`
- `internal/statuscommit#TestNew_NilPushPanics`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#newCommitStatusSeam`
- `internal/battencli#commitStatusDeps`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#newCommitStatusSeam`
- `internal/loomcli#TestCommitStatusSeam_Real`

**Delete:**
- `internal/statuscommit#TestNew_MergeActiveSkips`
- `internal/statuscommit#TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`

**Uses:**
- `internal/loomcli/doc.go`
- `plan:internal/fabricengine#MergeStateOperation`
- `plan:internal/fabricengine#IsPushProtected`
- `plan:internal/fabricengine#IsPrimeBranchRefusal`

**Intent:** D4 and D7 at the transition seam: it waits on a blocking git operation or a prime off its branch in place of its skip, and completes a transition whose push failed as `protected` only.

`statuscommit.Deps`:
- `MergeActive` becomes `MergeOperation func() (operation string, err error)`, naming the operation that blocks the records commit, `""` for none. A nil one reads none.
- New `Stalled func(err error) bool` reads a commit failure that waits rather than halts, the prime-branch refusal. A nil one reads none.
- New `Protected func(err error) bool` reads a push failure as protected-only. A nil one reads none.

`New`'s closure:
- Before the commit, an operation, or a probe error, makes it wait: it logs a warning naming the operation (or the probe error) and its way forward, conclude or abort it,
  sleeps through `deps.Sleep` with the owed-push backoff (one second doubling to `owedRetryCeiling`), and probes again, until the probe answers none. Then it commits.
  The wait takes no admission, so a pause holds the run as it holds an owed retry.
- `failureDisposition`: a commit failure the re-probe explains (an operation, or a probe error), or one `Stalled` reads true, waits the same way, logging the failure and its way forward, and then runs the commit again; any other failure still halts the run.
  So a run never acts past a transition whose records commit has not landed, and a prime run stalls until the prime is back on its recorded branch.
- A push error `Protected` reads true logs a warning naming the protection and returns nil, so the run acts with that transition's records unpushed.
  It is checked before `Owed`, and only when the push failed.
The package comment and `New`'s doc state the four dispositions with the wait and the protected completion.

Wiring: `loomcli`'s and `battencli`'s `commitStatusDeps` gain `MergeOperation`, `Stalled`, `Protected` and `Sleep` fields, set in `commitStatusDepsGuardedBy` and `battenCommitStatusDeps`
to `fabricengine.MergeStateOperation(location)`, `fabricengine.IsPrimeBranchRefusal`, `fabricengine.IsPushProtected` and `time.Sleep`, and `newCommitStatusSeam` passes them to `statuscommit.Deps`.
Their `MergeActive` fields stay for the callers that refuse on them (`refuseWhileMergeActive`, `commitrecords.go`).
`loomcli`'s `commitStatusDeps` doc says a blocking git operation makes the step wait, where it said a mid-merge state skips it.

Tests:
- Every listed `statuscommit` test sets `MergeOperation` in place of `MergeActive`; their assertions are unchanged.
- `TestNew_MergeOperationWaits` (tier0) replaces the two deleted tests. Rows:
  `OperationBeforeTheCommitWaits` (probe answers `"merge"` twice, then none: two sleeps of 1s and 2s, then one commit and one push);
  `ProbeErrorWaits`;
  `OperationAfterAFailedCommitWaitsAndCommitsAgain` (first probe none, commit fails, re-probe `"cherry-pick"` then none: the commit runs twice and the second lands);
  `StalledCommitFailureWaitsAndCommitsAgain` (commit fails with an error `Stalled` reads true, then lands: one sleep, two commits, one push);
  `UnexplainedCommitFailureHalts` stays in `TestNew_CommitErrorPropagates`.
- `TestNew_ProtectedPushCompletes` (tier0) rows: `ProtectedOnlyCompletes` (push error read protected: nil, no retry, no sleep); `ProtectedNilReadsNone` (the error returns, as today).
- `TestCommitStatusSeam_Real` (tier1) rewrites its rows `mid-merge skips without touching the merge` and `merge going live after the probe skips instead of halting`
  into `mid-merge waits until the merge ends, then commits` and `merge going live after the probe waits, then commits`: each row's `Sleep` concludes or aborts the plain-git merge on its first call,
  and the row asserts the transition commit lands after it. A new row `cherry-pick in progress waits, then commits` does the same over a conflicted cherry-pick.
  These reach the wired `MergeOperation`; the ordinary-path row asserts a nil push error with `Protected` wired.

**ImpactSummary:** statuscommit.Deps changes one field's shape and adds two, and New waits where it skipped in both layouts; the two shed seams that build it are rewired, and other users of their MergeActive fields are untouched.
