# Card 6 — resume-start-own-leftover

**Create:**
- `plan:internal/loomcli#ownMergeInLeftover` -> `func ownMergeInLeftover(st fabricengine.MidMergeState, producer, parentBranch string) bool`
- `plan:internal/loomcli#loomCLI.currentProducer` -> `func (c *loomCLI) currentProducer() (string, error)`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#loomCLI.refuseOverUnfinishedMerge` -> `func (c *loomCLI) refuseOverUnfinishedMerge(ctx context.Context, out io.Writer, bootstrapLock *lock.FileLock, retry, producer string) bool`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli/start_midmerge_test.go`
- `internal/loomcli/resume_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/fabricengine#MidMergeState`
- `internal/fabricengine#ReadOrigin`
- `internal/loomshed#NamePublish`
- `internal/loomshed#NameFinalize`

**Intent:** `lyx loom resume` and `lyx loom start` let a run through over the parked merge-in its own Publish or Finalize row left behind, since the row aborts and redoes it (D3), and keep the `merge_in_progress` refusal everywhere else.
`ownMergeInLeftover` is the one decision: true when `st.Kind` is `MidMergeParked`, `st.Verb` is `merge-in`, `st.Source` equals `parentBranch` and `producer` is `loomshed.NamePublish` or `loomshed.NameFinalize`.
`loomCLI` gains a seam `recordedParentBranch func(*lyxcwd.Location) (string, error)`, set in `newLoomCLI` to a reader of the pair's origin record through `fabricengine.ReadOrigin` that returns the recorded parent branch, or an error when the record is absent, the same read `wire`'s `ParentBranch` closure makes; a test substitutes a canned branch.
`refuseOverUnfinishedMerge` takes the run's current producer; after the probe, for a parked state it reads the parent branch through the seam, a read error refusing like a probe error, and returns true without a refusal when `ownMergeInLeftover` holds; every other parked or foreign state keeps today's message, kind and `conflicts` list.
`currentProducer` reads the status file as `runHaltedAtHandBack` does and returns its current producer, or `""` with no status file; `runDriverSpawnAndWait` reads it right before its probe and passes it, so `start` proceeds for a run halted at the row and for one still `running` there whose driver died; `resumeFromRunState` passes `status.CurrentProducer` at both of its probe sites, so a parked live driver is woken over the leftover, and with no live driver the existing no-live-driver refusal and its way forward are reported instead of `merge_in_progress`.
The `start` and `resume` help texts that describe the `merge_in_progress` refusal state the exception.
`TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` gains rows: the leftover at a row other than Publish or Finalize is refused; a parked `merge` verb, and a `merge-in` of another source, at Publish are refused; foreign state at Finalize is refused.
`TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds` gains rows for the own leftover at Publish on a halted run and at Finalize on a `running` run with a dead driver, both spawning as on a clean pair; the status written for these rows names the current producer.
`TestResumeVerb` gains rows: a parked live driver at Publish over the own leftover is woken; the same with no live driver reports the dead-driver refusal, not `merge_in_progress`; the leftover at Webster keeps `merge_in_progress`; a parked merge of another source at Finalize keeps `merge_in_progress`.
The refusal-spec's `resume: merge in progress` row states the exception: a run whose current producer is Publish or Finalize over a parked fabric `merge-in` of its own parent branch is let through, since the row aborts and redoes it.
`docs/overview.md`'s loom lines on `start`'s and `resume`'s `merge_in_progress` refusal state the same exception.

**ImpactSummary:** One unexported guard is re-signed with both callers on this card; the only observable change is that the two verbs proceed over a parent-sourced parked merge-in at Publish or Finalize.
