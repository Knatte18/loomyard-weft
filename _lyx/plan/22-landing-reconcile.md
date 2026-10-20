# Card 22 — landing-reconcile

**Create:**
- `plan:internal/landingshed#reconcileVerdict` -> `func reconcileVerdict(ctx context.Context, producer string, reconcile func() (fabricengine.OriginReport, error)) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#MergeIn.Call`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/landingshed#TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Uses:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/pairteardown#HubSeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams`

**Intent:** D8's Merge-In and Finalize: each runs the run-scope reconcile before acting on the parent (PATTERN-told-geometry: landingshed receives it as an injected seam).
`Deps` gains `Reconcile func() (fabricengine.OriginReport, error)`, nil-is-absent like `CommitStatus`; `loomcli#landingDeps` fills it with `ReconcileOrigin` over the run scope of the task's slug, seams from `pairteardown.OriginSeams` with `HubSeamOptions`, and its drift guard keeps it filled.
`MergeIn.Call` runs it after the status commit and before taking the parent lock; `Finalize.Call` runs it at entry under the parent lock the run already holds, right after `holdParent`.
`reconcileVerdict` maps the run's own entry outcome: `ended` is a Stuck with Halt naming the landing (`done`) or the removal from the board, with the way forward of ending the run here (`lyx batten cleanup --apply` from the prime after the run is paused); `stopped` is a Stuck naming the branch, its state and its way forward (D4); `active` proceeds; an `Unreachable` report logs a Warn and proceeds, so the row fails on its own remote steps as today; a reconcile error is returned as an infrastructure error.
The run scope judges and never moves the parent; Merge-In's existing fast-forward under the parent lock moves it.
A run whose local history shows Finalize entered is never ended (card 17), so Finalize re-entered after its done mark proceeds.
`doc.go`'s Merge-In and Finalize sections name the step.

`TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify` records the reconcile call after the status commit and before the lock, and gains rows: an ended report (Finalize not entered) is a Stuck with Halt naming the landing with no lock taken; a stopped report is a Stuck naming the branch and its way forward with no lock taken; an unreachable report proceeds to the lock and the merge; a reconcile error is returned as an error with no lock taken; `TestFinalize_StuckReasons` gains rows: an ended report is Stuck with Halt naming the landing, a stopped one is Stuck naming the branch, an unreachable one proceeds, an active one for a run that entered Finalize proceeds, and a reconcile error is returned as an error.
`TestLandingDeps_EveryFieldPopulated` sees the new field filled.

**ImpactSummary:** One nil-is-absent field on `landingshed.Deps`, filled only by loomcli; both rows gain one early step and keep every later step.

**Commit:** 22: Merge-In and Finalize run the run-scope origin reconcile
