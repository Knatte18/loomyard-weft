# Card 10 — resume-restores-detached

**Create:**
- `plan:internal/shedengine#Shed.restoreUnderRunLock` -> `func (s *Shed) restoreUnderRunLock() error`
- `plan:internal/shedengine#ErrRestoreRefused` -> `var ErrRestoreRefused = errors.New("shedengine: the resume restore refused the drive")`
- `plan:internal/landingshed#Finalize.detachedStop` -> `func (fz *Finalize) detachedStop(ctx context.Context) *ending`
- `plan:internal/loomcli#resumeRestore` -> `func resumeRestore(l *lymcwd.Location, runID string) func() error`
- `plan:internal/loomcli#TestNewShed_RestoresADetachedTaskWorktree` -> `func TestNewShed_RestoresADetachedTaskWorktree(t *testing.T)`
- `plan:internal/landingshed_test#TestFinalizeQueue_ResumedDetached` -> `func TestFinalizeQueue_ResumedDetached(t *testing.T)`

**Edit:**
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.Run`
- `internal/shedengine#Shed.Step`
- `internal/shedengine#TestLockedCheck_RefusesUnderTheRunLock`
- `internal/loomcli#loomCLI.newShed`
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/loomcli#landingDeps`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`

**Uses:**
- `internal/shedengine/doc.go`
- `internal/shedengine#Shed.refuseUnderRunLock`
- `internal/fabricengine#RestoreDetached`
- `internal/fabricengine#ReadDetachState`
- `internal/landingshed_test#queueHub`
- `internal/landingshed_test#newQueueHub`
- `internal/loomcli/doc.go`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** A run's resume puts a task worktree a dead leader left detached back on its branch before any row runs, and stops a resume it cannot restore (D5).

- `Shed` gains `Restore func() error`, called by `restoreUnderRunLock` in `Run` and in `Step` right after `refuseUnderRunLock`, so every row's resume and `lym loom goto`'s next drive pass through it before any producer runs.
  A non-nil error is returned wrapped by `ErrRestoreRefused`, whose text leads so the hook's own trailing way forward stays last, as `ErrLockedCheckRefused` does; nil means no restore.
- `loomCLI.newShed` sets `shed.Restore` to `resumeRestore(c.location, c.runID)`, which calls `fabricengine.RestoreDetached` and logs a restore at Info.
  A clean detached worktree with the marker is checked out on its branch at whatever commit, its leftovers removed and its marker dropped.
  A dirty detached worktree, and one with no marker, are refused with D5's way forward when the status file's `current_producer` is any row but Finalize: the dirty paths and "clean them in the task worktree, then `lym loom resume`", or the commit and "check the task branch out in the task worktree, then `lym loom resume`".
  At Finalize the closure returns nil for them, so Finalize's own stop below makes the run Stuck as D5's criterion asks.
  Any other restore error, card 4's marker naming a deleted branch included, is returned with its own way forward.
- `Deps` gains `DetachState func() (fabricengine.DetachState, error)`, filled by `landingDeps` with `fabricengine.ReadDetachState(l)`; nil reads as not detached.
  `envkit.LandingDeps` fills it and `battencli`'s `intentionallyNil` gains `Landing.DetachState` as "loom-only seam, batten lands no branch".
- `Finalize.Call` runs `detachedStop` before step 1a2.
  A detached dirty worktree is Stuck naming its dirty paths, with the way forward "clean them in the task worktree, then `lym loom resume`".
  A detached one with no marker is Stuck naming its commit, with the way forward "check the task branch out in the task worktree, then `lym loom resume`".
  Neither moves anything.
  A detached clean one with a marker never reaches it, since the shed restore ran first; `detachedStop` returns a fail-loud wiring error naming `Shed.Restore` for it.

Tests.
`TestLockedCheck_RefusesUnderTheRunLock`, tier0 in `package shedengine`, gains rows for the restore hook for both `Run` and `Step`, adding about 0.3 seconds to one tier0 run: the restore is called once before the producer, with the run lock held; a restore error is returned wrapping `ErrRestoreRefused` with its way forward last, no producer is called and the status bytes are unchanged; a nil restore drives the producer.
`TestNewShed_RestoresADetachedTaskWorktree`, tier1 in `package loomcli`, over a `hubforge` pair whose status file sits at a row other than Finalize, rows: a clean worktree detached with the marker is back on its task branch with its ref unchanged and the marker gone after the built shed's `Restore` runs; a dirty detached one is refused naming the dirty paths and `lym loom resume`; one with no marker is refused naming the commit; at Finalize the dirty one is not refused.
`TestFinalizeQueue_ResumedDetached`, tier1, over `queueHub` with the task worktree detached at a chain commit as a leader that died mid-gate leaves it, rows: a dirty worktree is Stuck naming the dirty paths and the parent's head and the task branch ref are unchanged; one with no marker is Stuck naming the commit.

**ImpactSummary:** One new Shed hook beside LockedCheck with one setter in loomcli, one new Deps seam with one filler and its seam-table entries; Finalize gains one entry check.
