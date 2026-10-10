# Card 5 — landing-pre-merge-clear

**Create:**
- `plan:internal/landingshed#preMerge` -> `type preMerge struct`
- `plan:internal/landingshed#newPreMerge` -> `func newPreMerge(deps Deps) preMerge`
- `plan:internal/landingshed#preMerge.clear` -> `func (m preMerge) clear(producer, parentBranch string) (string, error)`
- `internal/landingshed/premerge_test.go`
- `plan:internal/loomcli#conflictSessionStopper` -> `func conflictSessionStopper(status func() (reedengine.StatusResult, error), stop func(guid string) error) func() (string, error)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Publish`
- `internal/landingshed#NewPublish`
- `internal/landingshed#Publish.Call`
- `internal/landingshed#Finalize`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.mergeInStep`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error)) landingshed.Deps`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli/landingdeps_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/loomcli/landingdeps_markdone_test.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/loomcli/wiring_publish_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `internal/landingshed/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#MidMerge`
- `internal/fabricengine#MidMergeState`
- `internal/fabricengine#Fabric.MergeAbort`
- `internal/shuttleengine#Runner.StopStrand`
- `internal/mergeresolve#ConflictRole`
- `internal/agentname#Parse`
- `internal/agentname#MatchesRole`

**Intent:** Publish and Finalize redo their own parked merge-in instead of halting on it (D2).
`Deps` gains three told seams, each nil-is-absent like `CommitStatus`: `MergeState func() (fabricengine.MidMergeState, error)` probes the pair's merge state; `AbortMerge func() error` aborts the pair's parked fabric merge; `StopConflictSession func() (guid string, err error)` stops every live conflict-session strand of the run through shuttle's stop verb and returns an empty guid with a nil error when each stopped or none is live, the guid of the stop that failed beside its error, and an empty guid beside an error when the strand table could not be read.
`preMerge` (new file `internal/landingshed/premerge.go`) holds the three seams; `newPreMerge` copies them from `Deps`, and `NewPublish` and `NewFinalize` store one on a new `premerge` field of each producer, so a struct-literal test with a zero field runs no probe.
`preMerge.clear` runs before the pre-merge clean check, in `Publish.Call` ahead of the `before the merge-in` check and in `Finalize.mergeInStep` ahead of its first check, so both of Finalize's passes behave alike; a nil `MergeState` seam returns `("", nil)` at once.
Its decision over the probe's answer: `MidMergeNone` proceeds.
`MidMergeParked` with `Verb` `merge-in` and `Source` equal to `parentBranch` is the row's own leftover: it calls `StopConflictSession`; an error with an empty guid, a strand table that could not be read, leaves the merge untouched and is returned as an error, like a probe error below; an error with a guid leaves the merge untouched and returns the Stuck reason `a conflict session of this run (strand <guid>) is still live and could not be stopped: <err>; way forward: run "lyx reed remove <guid>", then "lyx loom resume"`; otherwise it calls `AbortMerge`, logs the abort at Info with the producer, the record's `Source`, `SourceSHA` and `StartSHA`, and returns `("", nil)` so the clean check and a fresh merge-in follow; an abort error returns the Stuck reason `the parked merge-in of parent branch %q could not be aborted: <err>; way forward: conclude it with "lyx fabric merge --continue" or discard it with "lyx fabric merge --abort", then resume`.
Any other parked record returns the Stuck reason `a fabric merge (<verb> of <source>) is in progress in the task worktree; way forward: conclude it with "lyx fabric merge --continue" or discard it with "lyx fabric merge --abort", then resume`, and `MidMergeForeign` returns `a git merge, cherry-pick or squash that fabric did not start is in progress in the task worktree; way forward: conclude or abort it with git, then resume`; neither lists staged paths, and nothing is stopped or aborted.
What the step can now discard, and its bound: the abort discards any parked `merge-in` of the parent branch into the task pair at Publish or Finalize entry, whoever started it, so a conflict session's resolution and a `lyx fabric merge-in` an operator or the orch started by hand in the pair are both discarded with any partial resolution; the stop ends only conflict-session strands of the run's own pair, whose work the abort discards anyway, and runs only when the row is about to abort; the abort runs only after every stop succeeds, so no session the row failed to stop is left writing into a redone merge; a parked merge with another verb or source, and foreign state, is never stopped over or touched.
A probe error is returned as an error, which both producers map through their existing `gateStop` to a cancellation or an infrastructure fault.
`premerge_test.go` holds one table-driven test over both producers, driven through `Call` with the package's fake resolver, fake gate seams and recording seam closures: own leftover with no live session aborts, logs and merges afresh; own leftover with a live session stops it first, then aborts; a failing stop leaves the merge untouched and names the guid, `lyx reed remove` and `lyx loom resume`; a stop seam that returns an empty guid beside an error is returned as an error, with no `AbortMerge` call and no merge-in; a stop that succeeds followed by a failing `AbortMerge` stops Stuck naming the parent branch, `lyx fabric merge --continue` and `lyx fabric merge --abort`, with no path list and no merge-in (the fake resolver is not called); another verb or source, and foreign state, stop Stuck with the clause above, no path list, no stop and no abort; a nil seam skips the probe; a probe error is returned; for Finalize, the retry pass after `ErrMergeInRequired` probes again.
`conflictSessionStopper` (new, in `internal/loomcli/landingdeps.go` beside `driverWaitMark`) builds the stop seam over two seams, reed's strand table and `Runner.StopStrand`: it reads the table at call time, finds every live, non-retiring strand whose name parses through `agentname.Parse` to a role `agentname.MatchesRole` matches against `mergeresolve.ConflictRole`, so a numbered `conflict-2` session is found as well as `conflict`, and stops each by guid; the first stop that fails ends the pass and returns its guid beside its error, and a pass that stops every match, or finds none, returns `("", nil)`.
`landingDeps` takes it as its last parameter and fills `StopConflictSession` with it, `MergeState` with `fabricengine.MidMerge(l)` and `AbortMerge` with `fabricengine.Open(l)` then `MergeAbort`; `loomPreRun` and `buildLoomShed` pass `conflictSessionStopper(c.reed.Status, c.runner.StopStrand)`, mirroring their `driverWaitMark` argument.
`TestLandingDeps_EveryFieldPopulated` and the other four test files that call `landingDeps` pass the new argument, and `landingdeps_test.go` gains `TestConflictSessionStopper`, a table over the two seams in `TestDriverWaitMark`'s shape: a live `conflict` strand is stopped by guid; a live numbered `conflict-2` strand is stopped by guid; two live conflict strands are both stopped; each returns an empty guid and nil; a dead or retiring conflict strand, and no conflict strand, stop nothing and return an empty guid and nil; a status error is returned beside an empty guid; a stop error is returned beside the failing strand's guid.
The test budget rows for `internal/landingshed` and `internal/loomcli` each rise by one.
The refusal-spec's loom section gains `Publish or Finalize: merge in progress` (a parked fabric merge with another verb or source, a parked merge-in the row's own abort could not discard, or foreign git merge state; correctness halt; the way forward clauses above) and `Publish or Finalize: conflict session not stopped` (the row's own parked merge-in with a live conflict session shuttle could not stop; correctness halt; `lyx reed remove <guid>` then `lyx loom resume`).
The landingshed package doc gains a paragraph on the pre-merge step: what it aborts, what it stops, and that the resolver's own abort stays as crash recovery within a step.

**ImpactSummary:** Three new nil-is-absent seams on `Deps`, one re-signed assembly function with two callers in `internal/loomcli`, and a new pre-merge step in both producers that changes no outcome on a clean pair.
