# Card 11 — finalize-push-order

**Create:**
- `plan:internal/landingshed#Finalize.handOverIfOwnedElsewhere` -> `func (f *Finalize) handOverIfOwnedElsewhere() (handedOver bool, err error)`
- `plan:internal/fabricengine#ArchiveParentMerge` -> `func ArchiveParentMerge(parent *lymcwd.Location, slug string) (tag string, err error)`
- `plan:internal/fabricengine_test#TestArchiveParentMerge` -> `func TestArchiveParentMerge(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.markTaskDone` -> `func (fz *Finalize) markTaskDone() error`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestFinalize_MergeInRequiredRetry`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed#TestFinalize_OwnerChecks`
- `internal/landingshed#TestFinalize_TransportPushFailure_ReturnsClassifiedError`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#archiveParentMerge`
- `internal/fabricengine#resetKeepUnderArchive`
- `internal/pairteardown#HandOver`
- `internal/pairteardown#PushRun`
- `internal/pairteardown#ThisMachine`
- `internal/boardengine#PullBolt`
- `internal/boardengine#ReadOwners`
- `plan:internal/boardengine#Board.SetStatusPushed`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomcli/doc.go`

**Intent:**
A Finalize past its parent-side merge pushes the parent branch, then its transition, then the board `done`, and reads the owner from a fetched Bolt before its merge and before each later push (D1, D2).

`Deps` gains four seams, each with a stated nil meaning so a `Deps` a test builds by hand keeps today's behaviour:
- `OwnedElsewhere func() (owner, subject string, elsewhere bool, err error)`: `landingDeps` pulls the Bolt (`boardengine.PullBolt`), reads the run's effective owner with `ReadOwners` and `hubgeom.ParentOf`, and compares it against `pairteardown.ThisMachine`'s names, current and former.
  Nil reads owned here.
- `HandOver func(owner, subject string) error`: `pairteardown.HandOver` over the run with a takeover entry naming owner and subject, `RunPush{InsidePair: true, Parent: <the parent pair>}` (the Finalize runs in the task pair), so the local parent merge is archived and reset (D2). Nil is never called, since a nil `OwnedElsewhere` never answers elsewhere.
- `PushRecords func() error`: `pairteardown.PushRun` over the task pair with `InsidePair: true`, `TeardownWait: StatusPushLockWait` and the parent pair, after `CommitStatus`. Nil pushes nothing.
- `ArchiveParentMerge func() error`: `fabricengine.ArchiveParentMerge` over the parent pair. Nil leaves the rejected parent push Stuck as today.
`landingDeps` fills all four, so `TestLandingDeps_EveryFieldPopulated` and the wiring seam tests pass as they stand;
envkit's `nilLegal` gains the four `Landing.` paths, since `envkit.LandingDeps` builds a test landing that drives no owner, push or archive.
`PushTask` in `landingDeps` moves onto `PushRun` too.

The owner before the merge: `ownerChecks`' branch for a run another machine owns hands over instead of halting.
It runs `HandOver` with the owner and subject the reconcile and `OwnedElsewhere` read, and returns a halt whose reason names the new owner and that the landing finishes there; nothing is merged.
A parent another machine owns keeps today's `ownerHalt`.
After the merge, `handOverIfOwnedElsewhere` asks `OwnedElsewhere` and, when another machine owns the run, runs `HandOver` and reports true; `pushParent` calls it before the parent push and again before the records push and the board write, and a true answer returns the same halt and pushes nothing more.
An unreadable owner, an unreachable origin included, returns an error naming the read, whose way forward is the landing's re-step once origin answers; the driver re-steps it as a transient failure, so the Finalize never goes on unread.

The order after the merge into the local parent branch:
1. `pushParent` pushes the parent branch.
   A push rejected while origin's fetched parent branch already holds the task's work (`OriginParentHoldsWork`) calls `ArchiveParentMerge` and goes on past the merge (D2: the adopter merged first).
   A transient failure returns an error as today, and the re-step's merge reads up to date and pushes again before any transition, which also covers a Finalize cut between its merge and its push by a crash or a stop (D2).
   Any other rejection keeps today's Stuck and `adopterFirstRepair`'s by-hand repair, whose steps an operator can run.
2. `CommitStatus` then `PushRecords` put the transition's records on origin.
3. `markTaskDone` writes the board `done` through the synchronous write (card 10).
   A failed write now returns an error in place of a logged warning, so the driver re-steps; the re-step's merge reads up to date, the parent and records pushes have nothing to send, and the `done` is written again.
Under the push skip, `pushParent` pushes nothing and the `done` follows the local merge.
The `MergeInRequired` retry path takes the same order.

`ArchiveParentMerge` wraps card 6's `archiveParentMerge` over the parent pair's code worktree: it fetches, tags the local tip `archive-owner/<slug>/<sha12>`, pushes the tag and resets the local parent branch to origin's tip with `git reset --keep`.
A local parent branch level with origin archives nothing and returns an empty tag.
The `git reset --keep` runs only through card 6's `resetKeepUnderArchive`, the executor in `internal/fabricengine/destroy.go` that checks containment, ownership, dirtiness and force in that order (`PATTERN-fabric-boundary`) and refuses unless the archive tag is on origin at the tip being left.

The landing doc's order section is rewritten to the new order; the fabric doc gains `ArchiveParentMerge`.

Tests:
- `TestFinalize_MergeMarkPushOrderAndVerdict` (tier0): its order rows read merge, push, commit, push-records, mark; a push-skipped row reads merge, commit, mark; a failing parent push leaves no mark;
  a rejected push over a parent that holds the work archives and marks; a failing mark returns an error.
- `TestFinalize_MergeInRequiredRetry` and `TestFinalize_StuckReasons` (tier0) follow the new order.
- `TestFinalize_OwnerChecks` (tier0) gains rows: a run the reconcile reads owned elsewhere hands over with the parent pair and merges nothing;
  `OwnedElsewhere` answering elsewhere after the merge and before the push hands over and pushes nothing;
  an unreadable owner returns its error with nothing pushed.
- `TestFinalize_TransportPushFailure_ReturnsClassifiedError` (tier0) gains the re-step row: a merge already in the local parent branch with the parent push not yet made pushes the parent before `CommitStatus` and `PushRecords`.
- `TestArchiveParentMerge` (tier1): a rejected push over an origin parent branch that already holds the task's work leaves the local parent branch level with origin, the local merge under a pushed `archive-owner/` tag, and origin's parent branch holding the task's merge once;
  a level branch archives nothing. The hand-over path's reset to the upstream is card 6's `TestHandOverTips` row and is not repeated.

**ImpactSummary:** Reorders the landing's last three steps, turns its owner halt for a run owned elsewhere into a hand-over, and adds four nil-safe landing seams that `landingDeps` fills; the Finalize tests that pinned mark-before-push change, and no other recipe row reads these seams.
