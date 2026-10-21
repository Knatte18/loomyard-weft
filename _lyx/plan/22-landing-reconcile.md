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
`reconcileVerdict` maps the run's own entry outcome: `ended` is a Stuck with Halt naming the landing (`done`) or the removal from the board, with the way forward of ending the run here (`lyx cleanup --apply` from the prime after the run is paused, whose hub reconcile tears the ended entry's pair down through D6, card 20); `stopped` is a Stuck naming the branch, its state and its way forward (D4), the branch's `BranchOutcome.WayForward`, then the row re-entry verb, `lyx loom resume` in the task worktree, as `preflightshed`'s `reconcileRefusal` appends it; `active` proceeds; an `Unreachable` report logs a Warn and proceeds, so the row fails on its own remote steps as today; a reconcile error is returned as an infrastructure error.
A row has no envelope, so `reconcileVerdict` logs each entry of the report's mutation record at Info through `internal/logger` before mapping the outcome.
The run scope judges and never moves the parent; Merge-In's existing fast-forward under the parent lock moves it.
A run whose Finalize merged into the local parent here is never ended (`finalizeEntered`, card 4), so Finalize re-entered after its done mark proceeds; a Finalize entering with no such merge, its entry done on origin, reads ended and stops with Halt, since another machine landed the run.
`doc.go`'s Merge-In and Finalize sections name the step.

`TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify` records the reconcile call after the status commit and before the lock, and gains rows: an ended report (Finalize not past its parent-side merge) is a Stuck with Halt naming the landing and `lyx cleanup --apply` from the prime, with no lock taken; a stopped report is a Stuck naming the branch, its reason containing the fake report's `WayForward` verbatim followed by `lyx loom resume`, with no `lyx cleanup --apply` and no lock taken; an unreachable report proceeds to the lock and the merge; a reconcile error is returned as an error with no lock taken; `TestFinalize_StuckReasons` gains rows: an ended report is Stuck with Halt naming the landing and `lyx cleanup --apply` from the prime, a stopped one is Stuck naming the branch, its reason containing the fake report's `WayForward` verbatim followed by `lyx loom resume`, an unreachable one proceeds, an active one for a run whose Finalize merged into the local parent proceeds, and a reconcile error is returned as an error.
`TestLandingDeps_EveryFieldPopulated` sees the new field filled.

**ImpactSummary:** One nil-is-absent field on `landingshed.Deps`, filled only by loomcli; both rows gain one early step and keep every later step.

**Commit:** 22: Merge-In and Finalize run the run-scope origin reconcile
