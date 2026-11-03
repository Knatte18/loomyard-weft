# Card 13 — remove-follow-up

**Create:**
- `plan:internal/pairteardown#BattenFollowUp` -> `func BattenFollowUp(slug string, record shedengine.Status, found, ownerHeld bool) string`
- `plan:internal/pairteardown#TestBattenFollowUp` -> `func TestBattenFollowUp(t *testing.T)`
- `plan:internal/battencli#TestPreflightRunOwner_RemoveFollowUp` -> `func TestPreflightRunOwner_RemoveFollowUp(t *testing.T)`

**Edit:**
- `internal/pairteardown#Result`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.Run`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/fabriccli#removeFields` -> `func removeFields(r fabricengine.RemoveResult, session pairteardown.SessionResult, warning string) map[string]any`
- `internal/fabriccli#runRemoveWithFlag`
- `internal/fabriccli#TestRemoveFields`
- `internal/battencli#battenCLI.preflightRunOwner`
- `internal/battencli#judgeOrigin`
- `internal/battencli#cancelDeps`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#cancelFake`
- `internal/battencli#TestCancelRun`
- `internal/landingshed#Deps`
- `internal/landingshed#stoppedReason` -> `func stoppedReason(entry fabricengine.EntryOutcome, branches []fabricengine.BranchOutcome, followUp func() string) string`
- `internal/landingshed#reportVerdict` -> `func reportVerdict(ctx context.Context, producer string, report fabricengine.OriginReport, followUp func() string) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `internal/landingshed#reconcileVerdict` -> `func reconcileVerdict(ctx context.Context, producer string, reconcile func() (fabricengine.OriginReport, error), followUp func() string) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#ParentAlign.Call`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`
- `internal/tendercli#EndOwnedElsewhere`
- `internal/tendercli#ownedElsewhereEnder`
- `internal/tendercli#ownedElsewhereEnder.run`
- `internal/tendercli#TestOwnedElsewhereEnder`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`

**Extends:**
- `internal/pairteardown/doc.go`
- `internal/fabriccli/doc.go`
- `internal/tendercli/doc.go`
- `internal/landingshed/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/battenrecipe#NameLand`
- `internal/battenrecipe#PastLand`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Decision record D3: a way forward naming `lym fabric remove <slug>` for a slug whose batten run record stands also names the step that settles that record, and `lym fabric remove` warns after removing such a pair.

`BattenFollowUp` words the follow-up from the prime's batten record of the slug:
- No record, a `done` record, a record at Worktree-Teardown (`battenrecipe.PastLand`), or `ownerHeld` (an owner pause covers the slug): `""`.
- A record before Land: `then run \`lym batten cancel <slug> --remove\`, which clears the run's record and the task branch the next run's add would refuse`.
- A record at Land (`battenrecipe.NameLand`): `the run's work already reached its parent, so never cancel it: run \`lym batten run <slug>\`, whose failed stop names the restore of each branch the removal took, run those steps, then \`lym batten run <slug>\` again` — D2's Land way forward, which that pass computes from the tags the removal left.

Wiring:
- `Teardown` gains `battenRecord func(slug string) (shedengine.Status, bool, error)`, reading the prime's `shedrun.StatusFile(prime, slug)`, and `ownerHeld func(slug string) (bool, error)`, reading `pauserecord.ReadCoverage(hubgeom.PauseGeometry(prime).RecordPath, slug, now).OwnerHeld`; `New` sets both, and a nil seam reads as no record (a hand-built test `Teardown` warns nothing).
- `Result` gains `Warning string` (`warning`, omitempty). `Teardown.Run`, after a successful removal of a request without `LeaveClaim` (a batten cancel's own removal settles the record itself), sets it from `BattenFollowUp`, prefixed `the batten run of <slug> still stands; `; a read error is logged and adds no warning. No batten record is written.
- `fakeSubstrates` gains a batten-record fake and an owner-held fake feeding the two seams.
- `runRemoveWithFlag` passes `res.Warning` to `removeFields`, which adds `warning` when non-empty; `TestRemoveFields` gains that row.
- `battenCLI.preflightRunOwner`, which has already read the run's status, adds `pairteardown.BattenFollowUp(slug, status, found, ownerHeld)` to an `ownerPreflight` error whose text names `lym fabric remove`, with `ownerHeld` read as `Teardown`'s `ownerHeld` seam reads it.
  The follow-up replaces the resume refusal's trailing re-run (`; then re-run lym batten run <slug>`, `resumeRefusal`'s reentry) when the error ends in it, and is appended otherwise: before Land the cancel clears the run's record, so no resume follows it, and the Land follow-up names `lym batten run <slug>` itself, so a re-run named before it would reach the gone pair ahead of the restore.
  So a resume refusal of a stopped entry that quotes fabric's gone-branch way forward (`FailedPushWayForward`'s `PushFailureGone` text, through `stoppedRefusal`) names the removal, then the cancel before Land, or the Land way forward; an error naming no `lym fabric remove` is returned unchanged, and a read error of the owner pause is logged and adds nothing.
- `judgeOrigin`, for act cancel, reaches the same `stoppedRefusal`: when the refusal's text names `lym fabric remove`, it ends with `BattenFollowUp`'s text in place of the trailing `then run <rerun> again`, so before Land it names `lym batten cancel <slug> --remove` and at Land it names D2's Land way forward and never a cancel.
  `cancelDeps` gains `OwnerHeld func() (bool, error)`, which `hubCancelDeps` fills as `Teardown`'s `ownerHeld` seam reads; the record is `deps.ReadStatus`'s.
  An empty follow-up, or a read error of either (logged), leaves the refusal unchanged.
- Landingshed's stopped reason (`stoppedReason`, reached through `reportVerdict` from Parent-Align and Finalize) quotes the same text and ends with `then run "lym loom resume" in the task worktree`, a worktree the removal deletes.
  `Deps` gains `RemoveFollowUp func() string`, nil meaning none; `reconcileVerdict` and `reportVerdict` pass it on, and `stoppedReason`, when the quoted way forward names `lym fabric remove` and the follow-up reads non-empty, appends it in place of that re-entry.
  `Finalize.ownerChecks` passes nil on its path where `ParentHoldsTaskWork` reads true: the work already reached the parent while the batten record still reads before Land, so a follow-up naming the cancel would release a landed entry, and that stop keeps today's text, as the adopter-first stop does.
  `internal/loomcli`'s `landingDeps` fills it with `pairteardown.BattenFollowUp` over the prime's batten record of the run's slug and the owner-pause read, so landingshed learns nothing of batten's files; its every-field drift guard keeps it filled, `internal/testkit/envkit`'s `nilLegal` gains `Landing.RemoveFollowUp` as a documented nil default beside `Landing.Reconcile`, and `internal/loomcli/doc.go` gains a sentence on the follow-up `landingDeps` passes.
- `ownedElsewhereEnder` gains `battenRecord` and `ownerHeld`, which `EndOwnedElsewhere` fills from the prime where it builds the ender; `run`'s failed-end error, which names `lym fabric remove <slug>`, appends `BattenFollowUp`'s text when non-empty.
- Finalize's stop for origin's parent already holding the work (`adopterFirstRepair` in `internal/landingshed/finalize_owner.go`) is the exception: its text stays as today, naming `lym fabric remove <slug>` and the standing owner pause, with no cancel, since a cancel lifts the owner pause (`liftSlugPauses`) and another machine has landed the work.

Extend `internal/pairteardown/doc.go` with the warning and its wording rule, `internal/fabriccli/doc.go`'s remove paragraph with `warning`, `internal/tendercli/doc.go` with the follow-up the owned-elsewhere ender's refusal appends, and `internal/landingshed/doc.go` with the follow-up a stopped reason quoting `lym fabric remove` carries from `Deps.RemoveFollowUp`.

Tests:
- `TestBattenFollowUp`, `tier0`, a table over the wording alone: no record, `done`, Worktree-Teardown and owner-held give `""`; Run-Shed and Worktree-Create name `lym batten cancel <slug> --remove`; Land names `lym batten run <slug>` and not `lym batten cancel`.
- `TestTeardown_CallSequence` gains wiring rows driving `Teardown.Run` over `fakeSubstrates`: a batten record at Run-Shed gives a `Result.Warning` equal to the `the batten run of <slug> still stands; ` prefix plus `BattenFollowUp`'s text for that record; with the owner-held fake true it is empty; with `LeaveClaim` set it is empty.
- `TestOwnedElsewhereEnder` gains a row: a failed end with a batten record at Run-Shed names `lym fabric remove <slug>` and then `lym batten cancel <slug> --remove`.
- `TestFinalize_MergeMarkPushOrderAndVerdict`'s adopter-first assertions gain one that the note does not contain `lym batten cancel`.
- `TestPreflightRunOwner_RemoveFollowUp`, `tier0`, a table over `newFakeReceiver` with a status file at Run-Shed and a fake first reconcile returning a stopped verdict: one whose branch way forward names `lym fabric remove` has its trailing `then re-run lym batten run <slug>` replaced by `BattenFollowUp`'s text; one naming no `lym fabric remove` is returned unchanged.
- `TestCancelRun` gains a row: a status at Land and a stopped branch whose way forward names `lym fabric remove some-slug`: the refusal ends with `BattenFollowUp`'s text for that record in place of `then run "lym batten cancel some-slug" again`.
- `TestFinalize_StuckReasons` gains a row: a stopped entry whose branch way forward names `lym fabric remove` and a `Deps.RemoveFollowUp` returning a fixed text: the reason ends with that text and does not contain `lym loom resume`; the same with `ParentHoldsTaskWork` reading true: the reason carries no follow-up.

**ImpactSummary:** Adds two optional seams and one omitempty result field to the pair-teardown composite, two seams to tender's owned-elsewhere ender, one envelope field to `lym fabric remove` (re-signing `removeFields`, whose only caller `runRemoveWithFlag` is on this card) the follow-up to batten's own resume and cancel refusals that name the removal, and one optional seam on landingshed's `Deps` (re-signing three private verdict helpers whose callers are on this card) that loomcli's `landingDeps` fills; every other teardown route (batten cancel, Worktree-Teardown, hub reconcile) either sets `LeaveClaim` or reads a record past Land, so none gains a warning.

**Commit:** `13: A removal of a batten run's pair names the step that settles its record`
