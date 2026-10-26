# Card 25 — adopt-subject-ways-forward

**Create:**
- `internal/battencli#ownedElsewhereFor`

**Edit:**
- `internal/fabricengine#OwnerReader`
- `internal/fabricengine#EntryOutcome`
- `internal/fabricengine#readOriginOwners` -> `func readOriginOwners(l *lyxcwd.Location, seams OriginSeams, targets []originTarget, outcomes []BranchOutcome, slugs []string) (owners, subjects map[string]string, err error)`
- `internal/fabricengine#classifyEntries` -> `func classifyEntries(scope ReconcileScope, recorded []string, branches []BranchOutcome, owners, subjects map[string]string) []EntryOutcome`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#OriginReport.Summary`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/fabricengine_test#TestDropLocal`
- `internal/pairteardown#OwnerReader`
- `internal/pairteardown#TestTeardown_OriginSeams`
- `internal/battencli#OwnedElsewhereError`
- `internal/battencli#OwnedElsewhereError.Error`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/battencli#battenCLI.claimBoardEntry`
- `internal/battencli#battenCLI.refuseResumeUnreachable`
- `internal/battencli#resumeRefusal`
- `internal/battencli#ownerVerdictOf`
- `internal/battencli#cancelOwnedElsewhere` -> `func cancelOwnedElsewhere(slug, owner, subject string) error`
- `internal/battencli#judgeOrigin`
- `internal/battencli#cancelDeps`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#cancelFake`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestArmAt_RunOwnedElsewhereWritesNoSeed`
- `internal/battencli#TestPassExitCode`
- `internal/battencli#TestBattenRun_LostWorktreeCreateClaimExitsWithTheDriversCode`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestBattenPreStep_RefusalKinds`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestOwnerVerdictOf`
- `internal/battenshed#OwnerVerdict`
- `internal/battenshed#innerRunProducer.ownerHoldReason`
- `internal/battenshed#innerRunProducer.ownedElsewhere`
- `internal/battenshed#innerRunProducer.noticeOwnedElsewhere`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon.noticeOwnedElsewhere`
- `internal/dobbyengine#TestOwnedElsewhere`
- `internal/beamcli#hubDobbyDeps`
- `internal/hibernateengine#WakeRun`
- `internal/hibernateengine#runStep`
- `internal/hibernateengine#TestWatch`
- `internal/hibernatecli#wakeReconcileFromReport`
- `internal/hibernatecli#TestWakeReconcileFromReport`
- `internal/loomcli#ownerVerdict`
- `internal/loomcli#runOwnerVerdict`
- `internal/loomcli#localOwnerVerdict`
- `internal/loomcli_test#TestRunOwnerVerdict`
- `internal/loomcli#loomCLI.refuseOwnedElsewhere`
- `internal/loomcli#TestStartAndResume_RefuseARunOwnedElsewhere`
- `internal/landingshed#Deps`
- `internal/landingshed#ownerHaltReason` -> `func ownerHaltReason(slug, subject, runOwner string, parent ParentOwnership) string`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#TestFinalize_OwnerChecks`
- `internal/loomcli#landingDeps`
- `internal/fabriccli#anchorDeps`
- `internal/fabriccli#hubAnchorDeps`
- `internal/fabriccli#anchorFake.deps`
- `internal/fabriccli#TestChangeAnchor`
- `contracts/stencils/orch/orch-template-role.md`
- `internal/fabricengine/doc.go`
- `internal/battenshed/doc.go`
- `internal/dobbyengine/doc.go`
- `internal/hibernateengine/doc.go`
- `internal/loomcli/doc.go`

**Delete:**
- `internal/battencli#ownedElsewhereRefusal`

**Intent:**
D5's upgrade bound and the scope's "every refusal and notice that names a task owner": a run in flight whose task is owned elsewhere now reads its project's owner, and every way forward that named `lyx adopt <task>` names `lyx adopt <AdoptSubject>`, the beam it belongs to or `root`, which card 4 made the only adopt arguments; every `lyx adopt --prime` names `lyx adopt root`.

- `fabricengine.OwnerReader` answers the subject beside the owner, `func(hubPath string, boardJSON []byte, slug string) (owner, subject string, elsewhere bool, err error)`; `pairteardown.OwnerReader` fills it through `Owners.AdoptSubject` with its parent resolver; `readOriginOwners` returns the subjects beside the owners, `ReconcileOrigin` passes both to `classifyEntries`, which keeps the subject on `EntryOutcome.AdoptSubject`, and `classifyEntries`' way forward and `OriginReport.Summary`'s owned-elsewhere line name `lyx adopt <subject>`, the slug when a reader leaves it empty.
- `battencli`: `OwnedElsewhereError` carries the subject and its text names `lyx adopt <subject>`; the new `ownedElsewhereFor(slug, owner, subject)` builds it, and `ownedElsewhereRefusal(slug, owner)` goes, since its callers move to `ownedElsewhereFor` or, `resumeOwnerClaim`'s, went with card 3; `ownedElsewhereFor` takes the subject from `refuseStartOwnedElsewhere` and `refuseResumeUnreachable` (through `Owners.AdoptSubject` over the owner seams' `ParentOf`), from `claimBoardEntry`'s lost claim, and from `resumeRefusal`'s report entry; `ownerVerdictOf` copies the entry's subject into `battenshed.OwnerVerdict.AdoptSubject`; `cancelOwnedElsewhere` names it, from `judgeOrigin`'s report entry, or, in its branch for an unreachable origin or an absent task worktree, from `cancelDeps.Owner`, which also returns the slug's adopt subject, `func() (machine, owner, subject string, err error)`, read in `hubCancelDeps` through `Owners.AdoptSubject` with the `hubgeom.ParentOf` resolver.
- `battenshed`: the owner pause's hold reason and its one notice per owner episode name `lyx adopt <subject>` from the verdict, the slug when a spawn refusal brought no subject; so the upgrade's run is owner-paused at its next pass naming `lyx adopt <project>`.
- `dobbyengine`: `Deps` gains `AdoptSubject func(slug string) (string, error)`, which `hubDobbyDeps` fills over the local board, and the owner notice names it, the slug when the seam is nil or fails.
- `hibernateengine.WakeRun` gains `AdoptSubject`, `runStep` names it, and `hibernatecli.wakeReconcileFromReport` copies it from the report entry.
- `loomcli`: the owner verdict carries the subject from the report entry, and `localOwnerVerdict`, which has no entry, fills it through `Owners.AdoptSubject` over the `parentOf` it already holds; `refuseOwnedElsewhere` names it.
  `TestRunOwnerVerdict`'s expected verdicts carry the adopt subject, `root` for its root task, and `runOwnerVerdict`'s doc comment sentence "It writes no owner, so a run-held task with no owner recorded reads as owned here." becomes "It writes no owner.".
- `landingshed`: `Deps` gains `AdoptSubject`, the run's own subject, which `loomcli.landingDeps` fills (the beam's slug for a pawl run, the task's beam or `root` otherwise); `ownerHaltReason` names `lyx adopt root` for the prime and `lyx adopt <subject> --keep-local` for the run, never `--prime`.
- `fabriccli`: `anchorDeps` gains `AdoptSubject func(slug string) (string, error)`, filled over the local board, and the foreign-owner refusal names `lyx adopt <subject>`, `lyx adopt root` for the prime.

Tests, each a row or an assertion of the named test: a task member of a beam owned elsewhere names `lyx adopt <beam>` in `TestReconcileOrigin_Moves`' owned-elsewhere entry and summary, `TestRunCmd_ResumeDispositions`' resume refusal, `TestBattenPreStep_RefusalKinds`, `TestWire_CreateWorktreeClaimsTheBoardEntry`'s lost claim, `TestCancelRun`'s owned-elsewhere row and a new row cancelling, with origin unreachable, a task whose beam another machine owns, `TestRefuseStartOwnedElsewhere`'s three rows that asserted `lyx adopt` plus the slug, now the fixture's beam, `TestInnerRun_OwnerCheck`'s hold reason and notice, `TestOwnedElsewhere`'s notice, `TestWatch`'s wake notice and `TestStartAndResume_RefuseARunOwnedElsewhere`; a root task names `lyx adopt root`; `TestFinalize_OwnerChecks` and `TestChangeAnchor` assert `lyx adopt root` where they asserted `--prime`; the slug fallbacks are rows too: `TestOwnedElsewhere` with a nil and with a failing `AdoptSubject` names the slug, `TestReconcileOrigin_Moves`' summary names the slug for an entry whose reader left the subject empty, and `TestInnerRun_OwnerCheck` names the slug for a spawn refusal that brought no subject; `TestTeardown_OriginSeams`, `TestDropLocal`, `TestOwnerVerdictOf`, `TestWakeReconcileFromReport`, `TestPassExitCode` and `TestBattenRun_LostWorktreeCreateClaimExitsWithTheDriversCode` follow the new signatures, the last two building their refusal with `ownedElsewhereFor`; `cancelFake` gains a subject field, which `cancelFake.deps`' `Owner` returns; `TestArmAt_RunOwnedElsewhereWritesNoSeed`, whose other machine owns the prime, asserts `lyx adopt root` where it asserted `lyx adopt` plus the slug; `TestRunOwnerVerdict`'s two unreachable-origin rows assert the subject `root` for the root task.
The docs name the subject, and the orch role stencil's line "`lyx adopt <slug>` from this prime is the takeover, and the operator decides it" becomes "`lyx adopt <beam>`, or `lyx adopt root` for a root task, from this prime is the takeover, and the operator decides it".

**ImpactSummary:** a reworded way forward threaded through the reconcile's owner seam and five consumers; the re-signs of `cancelOwnedElsewhere`, `ownerHaltReason`, `readOriginOwners` and `classifyEntries` have their callers on this card, `ownedElsewhereRefusal` goes with no caller left, and the reader type's one implementation is pairteardown's.

**Commit:** 25: Owner ways forward name the beam or `root` to adopt, never a task
