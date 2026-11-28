# Card 7 — finalize-queue-entry

**Create:**
- `plan:internal/landingshed#Finalize.joinQueue` -> `func (fz *Finalize) joinQueue(ctx context.Context, summary summaryparser.Summary, credit string) *ending`
- `plan:internal/landingshed#Finalize.actOnOutcome` -> `func (fz *Finalize) actOnOutcome(ctx context.Context) (queueAction, *ending)`
- `plan:internal/landingshed#queueAction` -> `type queueAction int`
- `plan:internal/landingshed#Finalize.landHeld` -> `func (fz *Finalize) landHeld(ctx context.Context, notice string) *ending`
- `plan:internal/landingshed#Finalize.ownLandingCommit` -> `func (fz *Finalize) ownLandingCommit() (string, bool, error)`
- `plan:internal/landingshed#Finalize.mergeNeedsSession` -> `func (fz *Finalize) mergeNeedsSession() (bool, error)`
- `plan:internal/landingshed#lockReadingResolver` -> `type lockReadingResolver struct`
- `plan:internal/landingshed#TestFinalize_QueueEntryAndOutcome` -> `func TestFinalize_QueueEntryAndOutcome(t *testing.T)`
- `internal/landingshed/finalize_queue_integration_test.go`
- `plan:internal/landingshed_test#queueHub` -> `type queueHub struct`
- `plan:internal/landingshed_test#newQueueHub` -> `func newQueueHub(t *testing.T, slugs ...string) *queueHub`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.recordLanding` -> `func (fz *Finalize) recordLanding(commit, base string, headErr error)`
- `internal/landingshed#Finalize.landedDiffEmpty` -> `func (fz *Finalize) landedDiffEmpty(base, landing string) bool`
- `internal/landingshed#Finalize.recordLandedSize` -> `func (fz *Finalize) recordLandedSize(base, landing string)`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Uses:**
- `internal/landingqueue#Store`
- `internal/landingqueue#BackstopSeams`
- `internal/fabricengine#NewestLandingOf`
- `internal/fabricengine#Fabric.MergesClean`
- `internal/landingshed#Finalize.mergeUnlocked`
- `internal/landingshed#Finalize.needsGitHubOnDisk`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#Finalize.handedOverStop`
- `internal/landingshed#Finalize.landingTexts`
- `internal/landingshed#Finalize.mergeAndGate`
- `internal/landingshed#finalizeGateFixture`
- `internal/landingshed#recordingParentMerger`
- `internal/landingshed#newTestDeps`
- `internal/landingshed_test#newFinalizeAt`
- `internal/loomcli/doc.go`
- `internal/loomcli#landingParentTarget`
- `internal/loomcli#writeLedgerLanding`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Finalize joins the landing queue, and under the parent lock it runs the backstop and acts on its entry's outcome (D1, D2, D8, D10's first half).
Every run still lands alone here; card 8 adds the batch a run with no outcome leads.

New `Deps` fields, each nil or empty in a struct-literal test, which then joins no queue and lands as today:
- `Queue *landingqueue.Store`, the parent branch's queue.
- `RecordsHead func() (string, error)`, the head of the run's records branch.
- `NewestLanding func() (string, bool, error)`, the newest commit on the local parent branch's first-parent history whose trailer names the run, through `fabricengine.NewestLandingOf`.
- `MergesClean func(ref string) (bool, error)`, whether merging ref into the task branch needs no conflict session, through `Fabric.MergesClean(TaskBranch, ref)`, which resolves both names.
- `Backstop landingqueue.BackstopSeams`, the seams of D6's backstop over the parent.
`internal/loomcli`'s `landingDeps` fills all of them for the run at `l`: the store as `landingqueue.Store{Dir: landingqueue.Dir(fabriclayout.BoltDir(l.HubPath)), Branch: parentBranch}`; `ParentHolds` as `IsAncestor(commit, "refs/heads/"+parentBranch)` and `UpstreamLacks` as its negation against `refs/remotes/origin/`+parentBranch, both over the task pair; `WriteLanding` as `writeLedgerLanding` over `landingParentTarget(l)` and the parent branch.
The env seam tables learn each field: `envkit.LandingDeps` fills every new func field and the `Backstop` struct's funcs; `battencli`'s `intentionallyNil` gains `Landing.Queue`, `Landing.RecordsHead`, `Landing.NewestLanding`, `Landing.MergesClean` and the three `Landing.Backstop.*` paths as "loom-only seam, batten lands no branch"; every reflective seam test that marks `internal/landingshed#Deps` (`TestLandingDeps_EveryFieldPopulated`, loomcli's and battencli's `TestWire_EverySeamFilled`, `TestWire_BeamRecipe`, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe`, envkit's `TestFullEnv_FillsEverySeam`) gains the marker `//lym:seam internal/landingqueue#BackstopSeams`.
The search that found them is `grep -rln "lym:seam internal/landingshed#Deps" internal cmd`.

Join (D1, D2).
`Call` runs `mergeUnlocked`, then `joinQueue`, then `landUnderLock`.
`joinQueue` returns at once with no `Queue`, and for a run `needsGitHubOnDisk` reads as needing GitHub or fails to read (D10: that run writes no entry and lands alone through today's path).
Otherwise it commits the status file through `CommitStatus` and writes the entry through `Queue.Join`: slug, `WorktreeRoot` as the pair path, the lock owner's run id, pid and start time from `Lock.Owner`, `TaskHead`, `RecordsHead`, the message `landingTexts` composes, the time now, and `TrueMerge` set to the shape the run's own parent-side merge takes, `!Config.Squash` or a `PullRequest.Mode` of `merge-in`, so the entry's landing commit has the parents today's `MergeOptions.Squash` at `finalize.go:564` gives it (an at-once landing goes through GitHub and never joins a batch).
A failed write is an error from `Call`, so the driver re-steps the landing.

Under the lock, right after `guard.ensure` and before the pull-request decision, `landUnderLock` runs `Queue.Backstop(deps.Backstop)` and then `Queue.Prune`, in that order (D2, D6).
A backstop error is an error from `Call`, so the driver re-steps it under the next lock hold.
`actOnOutcome` then reads `parentHoldsWork` and the run's entry, and picks one of:
- held: the parent holds the task's work, whatever the entry says (D8).
- `landed` with its commit on the parent's head (`IsAncestor(commit, "refs/heads/"+ParentBranch)`): held.
  This is reached while `parentHoldsWork` reads false, as after a later parent commit that changed a file the landing changed, so the merge-tree of the parent and the task head is no longer the parent's tree.
  A task branch tip past the entry's task head never reaches here, since `Join` cleared that `landed` outcome (card 1).
- `landed` with a commit the parent's head no longer holds, as after an operator's reset of the parent: the outcome is cleared through `SetOutcome(slug, "", "")` and the run acts as with none.
- `failed`: kept; the run merges and gates alone under the lock, fixer included, and lands alone.
- `conflict`: kept; the run frees the lock, runs `mergeUnlocked` with no lock held, takes the lock again at the front and then merges and gates alone, as `lockFreed` does today.
- none: the run leads; in this card it lands alone through today's merge and gate.
Before any merge-in under the lock in queue mode, `mergeNeedsSession` asks `MergesClean(ParentBranch)`; a merge that would need a conflict session frees the lock and runs `mergeUnlocked` first, then takes the lock at the front (D8's last bullet), so a conflict session never runs under the lock.
Bound: the precheck reads the local parent branch; a parent moved between the precheck and the merge still merges through the resolver as today.

`landHeld` is the held path and never merges into the task worktree, runs no gate and writes no landing commit:
owner checks with held set (the reconcile's ended and stopped verdicts), `handedOverStop`, then the run's remaining landing records with its own landing commit, then the entry's removal and Done.
`ownLandingCommit` picks that commit: the entry's `landed` commit while the parent's head holds it; else `NewestLanding`; with neither, the parent's head with no landed size and the landing read as not empty, as a held run with no merged parent head records it today.
`recordLanding` is re-signed to take `base`, the landed diff's base, and hands it to `landedDiffEmpty` and `recordLandedSize`, re-signed the same way; its one caller in `landUnderLock` passes `parentHeadMerged` for a run that merged in this process, and `landHeld` passes the landing commit's first parent for a held run, or "" for the parent-head fallback, which records no landed size and reads the landing as not empty, as an empty `parentHeadMerged` does today.
`pushParent` hands the same commit to `LandLocally`, which writes the ledger landing when none stands and keeps one that does (D6).
`pushParent`'s Done path removes the run's entry through `Queue.Remove`, so a run that ends Finalize at Done leaves none (D2); a halt or Stuck keeps it; a failed removal is logged at Warn and keeps Done.

Tests.
`TestFinalize_QueueEntryAndOutcome`, tier0 in `package landingshed`, over `finalizeGateFixture`'s fakes and a `landingqueue.Store` under `t.TempDir()` with a fake `Alive`, adds about 1 second to one tier0 run.
`lockReadingResolver` is its resolver fake: at each `Resolve` it reads the fixture's lock store record and keeps whether it named the run, then answers resolved.
Rows:
- a run whose `PullRequest.Mode` reads `trail`, with `RecordPath` and `RecordLockPath` under `t.TempDir()` holding no record, writes no entry;
- a run whose `PullRequest.Mode` read fails writes no entry and lands alone;
- a run with no pull request need writes an entry carrying its slug, task head, records head, message and join time, and `TrueMerge` false under `Config.Squash` true and true under `Config.Squash` false;
- a `Queue` whose directory is a file fails `Call` with an error naming the queue and merges nothing into the parent;
- a run whose entry is `failed` merges and gates under the lock alone, runs its fixer on a failing gate and keeps the outcome while the fixer runs;
- a run whose entry is `conflict` runs its conflict session with no lock held (`lockReadingResolver` saw no record naming the run) and then lands alone;
- a run whose merge-in under the lock would need a session frees the lock first;
- a backstop `WriteLanding` error ends `Call` with that error and runs no merge;
- a `Queue.Remove` failure after the landing, with the entry file made unremovable through `permkit.Chmod`, keeps Done.
`finalize_queue_integration_test.go` is tier1, in `package landingshed_test`, over a `hubforge` hub, with `queueHub` holding the hub, the parent pair, one task pair per slug and a real queue store and parent lock store, and building each run's `Finalize` through `newFinalizeAt`'s deps plus the queue seams over the real pairs.
`TestFinalizeQueue_LandsOneRunAlone` rows: a single run with no other entry merges the parent with no lock, gates once with the lock held (the gate command records the parent lock record's holder), lands as today and leaves no entry.
`TestFinalizeQueue_ActsOnItsOutcome` rows:
- a run woken with `landed` and its commit on the parent runs no merge and no gate, keeps the parent's head and ends Done with its `RecordLanding` and `LandLocally` commit equal to its landing commit;
- a run whose `landed` commit is on the parent, after a later parent commit rewrote a file the run changed, takes the same held path;
- a run whose work the parent holds while its entry has no outcome does the same;
- a run whose work the parent holds by tree, from a squash on the parent with no trailer naming it, records the parent's head with no landed size and the landing read as not empty;
- a run whose `landed` commit the parent no longer holds, after the test resets the parent, merges, gates and lands;
- a run whose task branch gained a commit after it was marked `landed` lands that commit;
- a run that takes the lock after a leader died between the move and the ledger write, with a mark standing and two dead entries, writes one ledger landing per moved commit with that member's records commit before the dead entries are pruned.

**ImpactSummary:** Adds five Deps seams filled by loomcli's one builder and registered in the env seam tables; three recordLanding helpers re-signed with their one caller each; a struct-literal Finalize without a queue behaves as before.
